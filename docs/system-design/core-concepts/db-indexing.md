# Database Indexing

By maintaining separate data structures optimized for searching, indexes allow databases to quickly locate the exact records we need without examining every row.

---

## B-Tree Indexes

- the most common type of database index
- providing an efficient way to organize data for fast searches and updates
- achieve this by maintaining a balanced tree structure that minimizes the number of disk reads

### The Structure of B-Trees

Every node in a B-tree follows strict rules:

- All leaf nodes must be at the same depth
- Each node can contain between `m/2` and `m` keys(where `m` is the order of the tree)
- A node with k keys must have exactly k+1 children
- Keys within a node are kept in sorted order

### Pros and Cons

- optimized for reads
- Updating the B-tree is relatively expensive as it involves random I/O and might include updating multiple pages on disk

### Real-World Examples

B-trees are everywhere in modern databases. When you create a table like this in PostgreSQL:

``` sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE
);
```

PostgreSQL automatically creates two B-tree indexes: one for the primary key and one for the unique email constraint. **These B-trees maintain sorted order**, which is crucial for both uniqueness checks and range queries.

---

## LSM Trees (Log-Structured Merge Trees)

B-trees are great for balanced workloads, but what happens when you need to **handle tons of writes**? With B-trees, each write means finding the right leaf page, reading it into memory, updating it, and writing it back to disk. These random disk seeks become bottleneck for processing 100,000 writes per second.

Instead of updating data in place like B-trees, LSM trees use an append-only approach that's built for write-heavy workloads.

### How LSM Trees Work

LSM trees batch writes in memory and flushing them to disk sequentially.

1. **Memtable(Memory Component)**: New writes go into an in-memory structure called a memtable, typically implemented as a sorted data structure like a red-black tree or skip list. This is exetremely fast since it's all in RAM.
1. **Write-Ahead Log(WAL)**: Every write is also appended to a write-ahead log on disk. This is a sequential append operation, which is much faster than random writes.
1. **Flush to SSTable**: Once the memtable reaches a certain size, its flushed to dis as an immutable Sorted String Table(SSTable). This is a single sequential write operation that can write megabytes of data at once.
1. **Compaction**: Over time, you accumulate many SSTables on disk. A background process called compaction periodically merges these files, removing duplicates and deleted entries.

### Pros and Cons

- excels at writes, ideal for write-heavy workloads like time-series database, logging system, and analytics platforms.
- slow reads. the query for a specific key needs to check the memtable and all SSTables on disk, starting from the newest. To mitigate the slow read problem, LSM trees typically employ several optimizations:
  - **Bloom Filters**: Each SSTable has an associated bloom filter - a probabilistic data structure that can quickly tell you if a key is definitely NOT in that file.
  - **Sparse Indexes**: Since SSTables are sorted, they maintain sparse indexes that tell you the range of keys in each block. If you're looking for user_id=500 and an SSTable only contains keys 1000-2000, you can skip it entirely.
  - **Compaction Strategies**:
    - **Size-tiered compaction** minimizes write amplication but can lead to more files to check.
    - **Leveled compaction** maintains fewer files but requires more frequent rewrites.

### Real-World Examples

- Cassandra handles Netflix's billions of viewing events. 
- RocksDB (built by Facebook) handles millions of social interactions per second.
- DynamoDB uses an LSM-tree-style storage architecture optimized for high write throughput.

---

## Hash Indexes

- Hash indexes excel at exact-match queries, trading flexibility for super-fast O(1) lookups.
- Redis, for example, uses hash tables as its primary data structure for key-value lookups.

### Pros and Cons

- the fastest possible exact-match lookups
- not supporting range queries or sorting

---

## Geospatial Indexes

Rather than treating latitude and longitude as independent dimensions, spatial indexes let us organize points based on their actual proximity in 2D space.

### Core Approaches

Three main approaches are commonly used: **Geohash**, **Quadtree**, and **R-tree**.

#### Geohash

- **How it works**: Encodes 2D coordinates (latitude, longitude) into a 1D alphanumeric string (or integer) by interleaving their binary bits using a space-filling curve (Z-order curve).
- **Key property**: Shared prefixes indicate spatial proximity. The longer the shared prefix, the smaller and more precise the shared bounding box.
- **Querying**: Proximity searches reduce to prefix matching or 1D range queries. To resolve the "edge effect" (nearby points across a grid boundary having different prefixes), queries search the target cell plus its **8 adjacent neighboring cells**.
- **Pros & Cons**:
  - **Pros**: Compact 1D representation; works seamlessly with standard B-Trees and key-value stores without specialized geospatial engines.
  - **Cons**: Fixed grid resolution; does not adapt to varying data density; limited to points and rectangular grid cells.
- **Real-World Examples**: Redis (`GEOADD`, `GEOSEARCH` store 52-bit integer geohashes in sorted sets), Elasticsearch, DynamoDB.

#### Quadtree

- **How it works**: A hierarchical 2D tree structure where each internal node has exactly four children, recursively dividing a 2D bounding area into four quadrants (`NW`, `NE`, `SW`, `SE`).
- **Key property**: **Adaptive resolution**. A node only subdivides when the number of points within it exceeds a defined threshold (e.g., 100 points). Dense urban areas are partitioned into finer grids, while sparse rural areas remain large quadrants.
- **Querying**: Traverses top-down from the root, pruning branches that don't intersect the search radius or bounding box.
- **Pros & Cons**:
  - **Pros**: Dynamically adapts to uneven data density; highly efficient for in-memory nearest-neighbor ($k$-NN) and range searches.
  - **Cons**: Difficult to persist and rebalance on disk (primarily kept in-memory); high write/update overhead for moving objects due to frequent node splits, merges, and locking.
- **Real-World Examples**: In-memory spatial indexes for Yelp (places search) and Uber (candidate driver matching).

#### R-tree

- **How it works**: A balanced, multi-dimensional tree (analogous to a B-tree for 2D/3D space) that groups neighboring spatial objects into hierarchical **Minimum Bounding Rectangles (MBRs)**.
- **Key property**: Bounding boxes can overlap, allowing the index to represent not just points, but complex shapes, lines, and arbitrary 2D/3D polygons (e.g., delivery zones, geographic borders).
- **Querying**: Traverses the tree by checking intersection with each node's MBR, descending into all overlapping child branches.
- **Pros & Cons**:
  - **Pros**: Disk-friendly and balanced (page-oriented layout, making it the standard for persistent RDBMS); natively supports arbitrary shapes and polygon containment queries.
  - **Cons**: Overlapping bounding boxes can force searches down multiple paths; tree rebalancing and node splitting ($R^*$-tree) are computationally expensive on writes.
- **Real-World Examples**: PostgreSQL / PostGIS (using GiST indexes), SQLite (R*Tree module), MySQL spatial indexes.

### Comparison Summary

| Feature | Geohash | Quadtree | R-tree |
|---|---|---|---|
| **Structure** | 1D string / integer (Z-order curve) | Hierarchical 4-way tree | Balanced tree of bounding boxes (MBR) |
| **Storage Fit** | Disk or RAM (B-Trees, Key-Value) | In-Memory (RAM) | Disk-based (RDBMS / B-Tree variant) |
| **Geometry** | Points only | Points only | Points, Lines, Polygons |
| **Density Handling** | Fixed grid resolution | Dynamically adapts by splitting | Dynamically adapts by grouping MBRs |
| **Best For** | Fast radius lookups in Key-Value stores (Redis) | In-memory proximity search (Yelp, Uber) | Complex GIS geometries and polygons (PostGIS) |

---

## Inverted Index

Consider a simple blogging platform with these posts:

``` bash
doc1: "B-trees are fast and reliable"
doc2: "Hash tables are fast but limited"
doc3: "B-trees handle range queries well"
```

The inverted index creates a mapping:

``` bash
b-trees  -> [doc1, doc3]
fast     -> [doc1, doc2]
reliable -> [doc1]
hash     -> [doc2]
tables   -> [doc2]
limited  -> [doc2]
handle   -> [doc3]
range    -> [doc3]
queries  -> [doc3]
```