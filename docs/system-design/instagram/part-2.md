# Instagram

## Deep Dives

### 1. Low latency feed loading (<500ms)

Our current "fan-out on read" approach could not scale to 500M DAU.

- Each feed refresh might need to process 10,000 posts (1,000 followed accounts x 10 posts/day)
- With 500M DAU, if each user refreshes their feed 5 times daily, that's 2.5 billion feed generations per day
- During peak usage (e.g., evenings, major events), we might see 150,000+ feed requests per second

The core issue is that we're postponing the computational work until read time, which is precisely when users expect immediate results.



??? failure "Bad Solution: Simple Caching"

    Adding a cache in front of the `Posts` table to cache each users recent posts. Use Redis. 

    1. Before querying the database for a user's followed users' posts, we check the cache. 
    1. If the posts are in the cache, we return them. 
    1. If not, we query the database and then store the results in the cache for future requests.

    An example key would be: `feed:{user_id}:{cursor}`. The value would be a list of `postId`s.

    - The cache needs to be massive to achieve a meaningful hit rate at Instagram's scale, and we still perform expensive fan-out reads to aggregate posts from all followed users for every feed request.

??? warning "Good Solution: Precompute Feeds (Fan-out on Write)"

Instead of generating the feed when the user requests it (fan-out on read), we generate it when a user posts (fan-out on write).

1. When a user creates a new post, we query the `Follows` table to get all users who follow the posting user.
1. For each follower, we prepend the new postId to their precomputed feed.
1. This precomputed feed can be stored in a dedicated `Feeds` table (in DynamoDB, for example) or in a cache like Redis.

**Data Model(Redis):**

- Key: `feed:{user_id}`
- Type: Sorted Set(ZSET)
- Members: `postId`
- Scores: `timestamp` (or `postId`)


??? success "Great Solution: Hybrid Approach (Precompute + Real-time)"

### 2. Low latency feed loading (<500ms)

1. low latency feed loading (<500ms)
1. low latency media delivery (<500ms)
1. scalable to 500m DAU