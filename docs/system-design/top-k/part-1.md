# YouTube Top K

**What is YouTube Top K?**

At any given moment we'd like to be able to query, precisely, the top K most viewed videos for the last 1 hour, 1 day, 1 month, and all time.

---

## Functional Requirements

1. query the top K videos for all-time (up to a max of 1k results)
2. query *tumbling(not sliding) windows* of 1 {hour, day, month} and all-time (up to a max of 1k results)

**Out of Scope:**

- Arbitrary time periods
- Arbitrary starting/ending points

---

## Non-Functional Requirements

- latency: 
    - write side: 1m delay to writes(tolerate at most 1 min delay between when a view occurs and when it should be tabulated.)
    - read side: We should return results in 10-100ms.
- Cardinality: Massive number of videos (TBD)
- Volume: Massive number of views (TBD)
- No approximations(Our results must be precise)




---

## Core Entities

- Video
- View
- Time Window: last hour, last day, last month, all time

---

## API Design

To retrieve the top K videos:

``` bash
GET /views/top-k?window={WINDOW}&k={K} -> { videoId: string, views: number }[]
```

---

## High Level Design

### 1. Query the top K videos for all-time (up to a 1k)

#### Route to write counts

Use Kafka:

1. A user makes a "view" API call(by watching a video), triggering the producer to create a `ViewEvent` topic.
        ``` json
        {
        "eventId": "view_9f3a",
        "videoId": "video_123",
        "viewerId": "user_456",
        "eventTime": "2026-09-13T10:05:23Z"
        }
        ```
1. The `ViewEvent` topic is partitioned by video ID.
1. The consumer retrieves a view event from the Kafka stream and aggregate that video's count.
1. Once the current one hour window passes, the consumer persists the aggregated counter for the video ID in the Postgres DB.

#### Route to query the top K videos

1. A user makes a `GET /views/top-k` call.
1. This request is routed to Top-K Service and it retrieves the data by making a SQL query to the DB.

``` sql
SELECT "videoId", "views", 
FROM VideoViews 
ORDER BY "views" DESC LIMIT k;
```

- Because we can create an index on the `views` column, the query can be very efficient(`O(k)` operation).
- The cost here is that on every write, updating the `views` index takes `O(log n)`.

### 2. query tumbling windows of 1 {hour, day, month} and all-time

Adjust our table schema to include a timestamp column.

**VideoViews table:**

- `videoId`
- `views`
- `timestamp`: Index on it. representing the start hour.

Note that we use the composite key for the VideoViews table with the combination of `videoId` and `timestamp`.

And adjust the query like below:

``` sql
SELECT "videoId", SUM("views") as "views", 
FROM VideoViews 
WHERE "timestamp" >= {windowStart} AND "timestamp" <= {windowEnd}
GROUP BY "videoId"
ORDER BY SUM("views") DESC LIMIT {k};
```

Unfortunately, the execution of this query is going to be a lot less efficient.