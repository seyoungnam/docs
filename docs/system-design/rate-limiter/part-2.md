# Rate Limiter

## Potential Deep Dives

### 1. How to scale to 1M RPS?

- A typical Redis instance can handle around 100-200k operations per second depending on the operation complexity.
- At 1M requests/second, we need to distribute the Redis load across multiple instances.
- We need a distribution algorithm like **consistent hashing** to solve this. Consistent hashing against user ID, IP address, or API key ensures {++each client's rate limiting state lives on exactly one shard++}, while distributing the load evenly across all shards.
- With 10 Redis shards, each handling ~100k operations/second, we should be able to handle our 1M request/second target.


### 2. How to ensure high availablity and fault tolerance?

If any Redis shard goes down, all users whose rate limiting data lives on that shard lose their rate limiting functionality, creating availability issues. We have two options:

??? warning "Good Solution: Fail-Closed"

    When the rate limiter can't reach Redis, reject all requests with `HTTP 503 "Service Unavailable"` or `HTTP 429 responses`.

    - Users may retry aggressively when they see 503 errors, creating even more load on your systems.
    - legitimate use cases: Financial systems processing payments might prefer to reject transactions rather than risk processing them without rate limits. 

??? warning "Good Solution: Fail-Open"

    - Failing open sends ALL that traffic downstream, **potentially causing total system collapse**.
    - For social media platforms, this is especially dangerous during viral events when traffic spikes are already stressing the system.

- The better approach is preventing Redis failures in the first place through high availability design. 
- The standard solution for Redis availability is **master-replica replication**.
- When the master fails, one of the replicas is automatically promoted to become the new master.

### 3. How to minimize latency overhead?

- Every rate limit check requires a network round trip to Redis, which adds latency to user requests.
- The most important optimization is **connection pooling**. Instead of establishing a new TCP connection to Redis for each rate limit check, your API gateways maintain a pool of persistent connections.
- Geographic distribution provides the biggest latency wins. The trade-off is complexity around data consistency across regions, but for rate limiting, you can often accept eventual consistency between regions in exchange for lower latency.

### 4. How to handle hot keys?

**For Legitimate High-Volume Clients:**

- **Client-side rate limiting**: Encourage well-behaved clients to implement their own rate limiting to smooth traffic patterns. This prevents legitimate users from accidentally creating hot shards while reducing server load.
- **Request queuing/batching**: Allow clients to batch multiple operations into single requests, reducing the total number of rate limit checks needed.

**For Abusive Traffic:**

- **Automatic blocking**: When a client hits rate limits consistently (say, 10 times in a minute), temporarily block their IP/API key entirely by adding them to a blocklist.
- **DDoS protection**: Use services like Cloudflare or AWS Shield that can detect and block malicious traffic before it reaches your rate limiter.

### 5. How to handle dynamic rule configuration?

Two main approaches for handling dynamic configuration updates:

??? warning "Good Solution: Poll-Based Configuration"

    Store rule configuration in a database or dedicated configuration service. Your API gateways periodically poll for configuration changes (say, every 30 seconds) and update their rate limiting logic accordingly.

    The main downside is **update delay**. 

??? success "Great Solution: Push-Based Configuration"

    **ZooKeeper** maintains configuration data and notifies all connected clients (your API gateways) immediately when any configuration changes.

    The operational complexity: need to handle connection failures, ensure all gateways receive updates, and deal with partial failures.