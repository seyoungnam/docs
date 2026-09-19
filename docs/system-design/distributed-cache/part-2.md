# Distributed Cache

## Deep Dives

### 1. Higly avaliable && fault tolerant

we need multiple copies of our data spread across different nodes.

??? failure "Bad Solution: Synchronous Replication"

    When a write comes in, we update all replicas synchronously and only respond once all replicas have acknowledged the write.

    - high latency for write
    - impact availability if any replica is slow or down
    - use this for caches that need **strong consistency** guarantees

??? warning "Good Solution: Asynchronous Replication"

    Update one primary copy immediately and then propagate changes to replicas asynchronously(**eventual consistency**)

    - better write performance and higher availablity
    - easy to scale system with additional replicas since they don't impact write latency
    - {==Replicas may temporarily have stale data until changes fully propagate through the system==}
    - failure recovery becomes more complex since we need to track which updates may have been missed while replicas were down and ensure they get properly synchronized when they come back online.
    - failover process introduces complexity and potential downtime.


??? warning "Good Solution: Peer-to-Peer Replication"

    Each node is equal and can accept both reads and writes. Changes are propagated to other nodes using **gossip protocols**, where nodes periodically exchange information about their state with randomly selected peers.

    - This provides scalability and availability since there's no single point of failure.
    - The implementation is more complex to maintanin connections with multiple peers and handle conflicts.

**Conclusions:**

1. Asynchronous replication for a good balance of availability and simplicity
1. Peer-to-peer for maximum scalability


### 2. Support 1TB of data and 100k RPS(scalability)

Estimate the number of nodes needed by considering both our **throughput** and **storage requirements**. 

- **throughput**: a single host to be able to perform 20,000 requests per second. we would need at least `100,000 / 20,000 = 5` nodes. Adding some buffer for traffic spikes and potential node failures, we should plan for around **8 nodes**.

- **storage requirements**: 
    - a typical AWS instance with 32GB RAM can reliably use about 24GB for cache data after accounting for system overhead.
    - Since we need to store 1TB (1024GB) of data, we would need approximately 1024GB / 24GB = 43 nodes just for storage. 
    - Adding some buffer for data growth and operational overhead, we should plan for about **50 nodes**.

Since our storage-based calculation (50 nodes) exceeds our throughput-based calculation (8 nodes), we should provision based on the storage needs.


### 3. How to ensure an even distribution of keys

The answer is **Consistent hasing** like MurmurHash to get a position on the circle. 

### 4. How to handle hot key issue

They occur when certain keys receive disproportionately high traffic compared to others. It creates a hotspot that can degrade performance for that entire shard.

**Two types of hot key problems:**

1. **Hot reads**: Keys that receive an extremely high volume of read requests, like a viral tweet's data that millions of users are trying to view simultaneously
1. **Hot writes**: Keys that receive many concurrent write requests, like a counter tracking real-time votes.


??? failure "Bad Solution: Vertical Scaling All Nodes"

    The simplest approach is to scale up the hardware of all nodes in the cluster to handle hot keys.

    - an expensive and inefficient solution since we're upgrading resources for all nodes when only some are experiencing high load.
    - still lead to bottlenecks if keys become extremely hot

??? warning "Good Solution: Dedicated Hot Key Cache"

    creating a separate caching layer specifically for handling hot keys. When a key is identified as "hot" through monitoring, it gets promoted to this specialized tier that uses more powerful hardware and is optimized for high throughput.

    - Managing consistency between the main cache and hot key cache adds complexity.
    - need sophisticated monitoring to identify hot keys and mechanisms to promote/demote keys between tiers.


??? warning "Good Solution: Read Replicas"

    create multiple copies of the same data across different nodes to distribute read requests across the replicas.

    - come with significant overhead in terms of storage and network bandwidth since entire nodes need to be replicated. 
    - replication lag and consistency issues
    - may be overkill if only a small subset of keys are actually experiencing high load.

??? success "Great Solution: Copies of Hot keys"

    Unlike read replicas which copy entire nodes, this approach selectively copies only the specific keys that are experiencing high read traffic.

    Here's how it works:

    1. the system monitors key access patterns to detect hot keys
    1. When a key becomes "hot",  the system creates multiple copies with different suffixes:
        - `user:123#1` -> Node A stores a copy
        - `user:123#2` -> Node B stores a copy
        - `user:123#3` -> Node C stores a copy
    1. For reads, clients randomly choose one of the suffixed keys, spreading read load across multiple nodes
    1. For writes, the system must update all copies to maintain consistency

    - When updating a hot key, we need to update all copies of that key across different nodes.
    - There's also overhead in monitoring to detect hot keys and managing the lifecycle of copies.
    - The approach works best when hot keys are primarily read-heavy with minimal writes.

### 5. Hot key for heavy writes

??? success "Great Solution: Write Batching"

    Write batching addresses hot writes by collecting multiple write operations over a short time window and applying them as a single atomic update. This approach is particularly **effective for counters, metrics**, and other scenarios where **the final state matters more than tracking each individual update**.

    - Longer batching windows reduce system load but increase the time until writes are visible.
    - If the batch processor fails, you need mechanisms to recover or replay the buffered writes.
    - Batching introduces slight inconsistency in read operations, as there's always some amount of pending writes in the buffer.


??? success "Great Solution: Sharding Hot Key With Suffixes"

    a hot counter key `views:video123` might be split into 10 shards: `views:video123:1` through `views:video123:10`. When a write arrives, the system randomly selects one of these shards to update.

    - the increased complexity of read operations, which now need to aggregate data from multiple shards.
    - This can increase read latency and resource usage, effectively trading write performance for read performance.


### Low latency

- **Request batching**, which helped us with our hot writes, is also an effective general technique for reducing latency since it reduces the number of round trips between the client and server.
- **Consistent hasing** for reducing latency since it means we don't need to query a central routing service to find out which node has our data -- saving us a round trip.
- **Connection pooling** to ensure there is always a ready-to-use channel for requests, removing expensive round-trip handshakes

