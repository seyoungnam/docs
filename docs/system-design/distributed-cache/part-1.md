# Distributed Cache

**What is a Distributed Cache?**

A distributed cache is a system that stores data as key-value pairs in memory across multiple machines in a network. The cache cluster works together to partition and replicate data, ensuring high availability and fault tolerance when individual nodes fail.

---

## Functional Requirements

1. be able to `set`, `get`, and `delete` key-value pairs.
2. be able to configure the expiration time for key-value pairs.
3. Data should be evicted according to Least Recently Used (LRU) policy.

**Out of Scope:**

- Users should be able to configure the cache size.

---

## Non-Functional Requirements

We need to store up to 1TB of data and expect to handle a peak of up to 100k RPS.

- availability >> consistency(eventual consistency)
- low latency operations (< 10 ms for `get` and `set`)
- support the expected 1TB of data and 100k requests per second

**Out of Scope:**

- Durability (data persistence across restarts)
- Strong consistency guarantees
- Complex querying capabilities
- Transaction support

---

## Core Entities

Our entities are `keys` and associated `values`.

---

## API Design

### SET: Setting a key-value pair

``` json
POST /:key
{
  "value": "..."
}
```

### GET: Getting a key-value pair

``` json
GET /:key -> {value: "..."}
```

### DELETE: Deleting a key-value pair

``` json
DELETE /:key
```

---

## High Level Design

### 1. Able to `set`, `get`, and `delete` key-value pairs

At its core, a cache is just a hash table. 

``` py
class Cache:
    data = {}  # Simple hash table

    get(key):
        return self.data[key]

    set(key, value):
        self.data[key] = value

    delete(key):
        delete self.data[key]
```

### 2. Able to configure the expiration time for key-value pairs

The key changes are:

1. Instead of storing just values, we now store tuples of `(value, expiry_timestamp)`
1. The `get()` method checks if the entry has expired before returning it
1. The `set()` method takes an optional TTL parameter and calculates the expiry timestamp

``` py
# Check the expiry time of the key on get
get(key):
    (value, expiry) = data[key]
    if expiry and currentTime() > expiry:
        # Key has expired, remove it
        delete data[key]
        return null
        
    return value

# Set the expiry time of the key on set
set(key, value, ttl):
    expiry = currentTime() + ttl if ttl else null
    data[key] = (value, expiry)
```

To remove expired entries, we need a background process (often called a "janitor") that periodically scans for and removes expired entries:

``` py
cleanup():    
    # Find all expired keys and delete
    for key, value in data:
        if value.expiry and current_time > value.expiry:
            delete data[key]
```

This cleanup process can **run on a schedule (say every minute)** or **when memory pressure hits certain thresholds**.


### 3. Data should be evicted according to LRU

To allow `get()`, `set()`, and `cleanUp()` methods to take `O(1)` time, we combine two data structures:

1. A **hash map** that maps a key to the corresponding doubly linked list node
1. A **doubly linked list** to track access order


``` go
import (
    "time"
)

type Node struct {
    key     int
    val     int
    expiry  int
    prev    *Node
    next    *Node
}


type LRUCache struct {
    capacity    int
    cache       map[int]*Node
    ttl         int
    head        *Node
    tail        *Node
}


func Constructor(capacity int, ttl int) LRUCache {
    head, tail := &Node{}, &Node{}
    head.next = tail
    tail.prev = head
    return LRUCache{
        capacity: capacity,
        cache:    map[int]*Node{},
        ttl:      ttl,
        head:     head,
        tail:     tail,
    }
}


func (this *LRUCache) Get(key int) int {
    node, ok := this.cache[key]
    if !ok {
        return -1
    }
    currTime := time.Now()
    if node.expiry < currTime {
        this.removeNode(node)
        delete(this.cache, key)
        return -1
    }
    this.moveToHead(node)
    return node.val
}


func (this *LRUCache) Set(key int, value int)  {
    currTime := time.Now()
    if node, ok := this.cache[key]; ok {
        node.val = value
        node.expiry = currTime + this.ttl
        this.moveToHead(node)
        return
    }
    newNode := &Node{key: key, val: value, expiry: currTime + this.ttl}
    this.cache[key] = newNode
    this.addNode(newNode)
    if len(this.cache) > this.capacity {
        deadNode := this.tail.prev
        this.removeNode(deadNode)
        delete(this.cache, deadNode.key)
    }
}

func (this *LRUCache) CleanUp() {
    expiredKeys := []int{}
    
    currTime := time.Now()
    curr := this.tail.prev
    for curr != this.head {
        if curr.expiry < currTime {
            expiredKeys = append(expiredKeys, curr.key)
        }
        curr = curr.prev
    }

    for _, key := range expiredKeys {
        node := this.cache[key]
        this.removeNode(node)
        delete(this.cache, key)
    }
}

func (this *LRUCache) moveToHead(node *Node) {
    this.removeNode(node)
    this.addNode(node)
}

func (this *LRUCache) removeNode(node *Node) {
    prev, next := node.prev, node.next

    prev.next = next
    next.prev = prev
}

func (this *LRUCache) addNode(node *Node) {
    prev, next := this.head, this.head.next

    prev.next = node
    node.prev = prev

    node.next = next
    next.prev = node
}
```

