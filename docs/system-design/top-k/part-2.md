# YouTube Top K

## Deep Dives

### 1. How to cut down on the number of queries to the DB?

We make a query to the database for every request that comes in for top K, which is resource intensive. But our non-functional requirements grant us a 1 minute grace period from when a video view event happens and when it needs to be tabulated in our results. 


??? warning "Good Solution: Cache the Top K for each time window"

    Use distributed cache like Redis or Memcached. If the request is in the cache, return it immediately. If not, we can compute it, store it in the cache with a TTL of a couple of hours, and return it.

    Store cache entries with a key of `top-k:{window}:{truncated_timestamp}`. For cache hits (the overwhelming majority of requests), we can return our results well inside of the 10's of milliseconds requirement we set.

    The biggest problem with this approach is when the cache expires we

        - suddenly have a flood of requests to the database
        - they're all going to break our SLA since our top K request takes longer than 10's of milliseconds.

??? success "Great Solution: Precompute the Top K for each time window"

    Add a cron to our system which, on fixed intervals, will precompute the top K for each time window and warms our cache in the same way.

    - adds some operational complexity
    - When our precomputation job is late, the top-k service can serve stale data for a short period of time. But it's better than nothing.


### 2. How to handle the massive number of writes to the DB?

The calculation for our throughput is:

``` bash
70B views/day / (100k seconds/day) = 700k tps
```

700k tps is far beyond the what modern RDMSs can handle(**10k+ writes per second per node**). 

Let's figure out how much storage we are going to need:

``` bash
Total Videos = 1M videos/day * 365 days/year * 10 years = 3.6B videos ~= 4B videos
Naive Storage = 4B videos * (8 bytes/ID + 8 bytes/count) = 64 GB
```

#### Sharding Ingestion

Scale our View Consumers horizontally by spinning up multiple instances to read from each partition of the `ViewEvent` topic. 

1. The producer hashes `videoId` to choose a Kafka partition, and Kafka stores that partition on some broker.
1. A View Consumer assigned to that partition then reads the event from the Kafka topic and writes to the matching DB for that shard.
    - DB should also be sharded by `videoId` to 70 instances. That way we can bring the throughput for the DB down to around 10k TPS from the expected 700k TPS. 
1. For `video_123`, the flow might be `hash(video_123) -> Kafka partition 7 -> consumer 7 -> database shard 7`.


#### Batching Ingestion

Even with sharding, having 70 database instances is a bit wasteful for this simple functionality. Instead of making a write to the database for each view, we can batch up the writes for each video and flush these batches periodically to the database.

A great option for doing this is **Flink**.

1. Flink is reading from Kafka. If a given host goes down, Flink can rewind to the checkpoint offset in the Kafka topic and resume processing from there.
1. Our Flink job is accepting individual view events and outputting sums of views per video on a 1 hour interval.

By batching, we can probably bring the number of shards down in the 5-10 range. Still not great, but manageable.


### 3. How to optimize the Top K queries?

When we don't have a cache, we are still making calls to the DB. To get a top-K result for the last 24 hours, we need to sum up all the views for each video in the last 24 entries. And then take the aggregated view count and pull the top K out of it. This process is going to take minutes or hours, not seconds.


??? success "Great Solution: Maintain Aggregates for Each Window in DB"

    If we have a `VideoViewsLastHour` table with an index on the views column, our queries are very fast! In this case, all we need to do is have our top K cron read the top K videos directly from our index.

    **VideoViewsLast{Hour,Day,Month,AllTime}**

    - `videoId`: shared by videoId
    - `views`: Index on it.
    - `timestamp`


    - We now have to write to 4 tables instead of just 1.


??? success "Great Solution: (With Caveats) Do Aggregation in Memory with Flink"

    A final approach would be to keep the aggregates in Flink, using Flink's native distributed state management instead of reading and writing to the Views DB. The top-k aggregator is keeping, in memory (likely using a heap), the top K videos for a given window.


### 4. What if we need to support sliding windows?

To keep track of the last hour of views for a video, when we have a new batch of views for the most recent minute 
    - we can increase by the views that have come in during that minute and 
    - decrease by the views that happened 60 minutes ago.

For the video views in the last hour:

1. Our Flink jobs aggregate, at a minute grain, the views for each video.
1. Whenever a new minute has passed and we're ready to write to our database, we'll...
    1. Read all of the views that happened in the minute exactly T-60 minutes ago. Call this our `decrement`.
    1. Write all of the views that happened in the last minute (our `increment`) to `VideoViews`.
    1. Update the `VideoViewsLastHour` table with difference between the `increment` and `decrement`.

We'll do this for each of our windows (last hour, last day, last month). For the all-time window, the decrement is always 0!

We need to keep minute-grained data around for a whole month so we can decrement it when it expires out of the "last month" window. From a product perspective, this is kind of wasteful! One mature path is to propose an alternative to our interviewer: **how about we support sliding windows for the "last hour" and tumbling windows for the "last day" and "last month"?**

### 5. Room for a specialized database?

Specialized databases:

- time-series (e.g., TimescaleDB, InfluxDB)
- real-time OLAP (e.g., Druid, Pinot, ClickHouse)

??? failure "Bad Solution: InfluxDB or Prometheus"

    Ingest view counts via Kafka and write per-minute points keyed by time with `videoId` as a series identifier or label into InfluxDB, and use tasks/continuous queries for downsampling (e.g., minute -> hour -> day). We can then query this with `top()` on aggregated counts to get K.

    - InfluxDB and Prometheus generally can't support this level of cardinality(billions of `videoId`).

??? warning "Good Solution: TimesacleDB"



??? success "Great Solution: OLAP Druid, Pinot, ClickHouse"