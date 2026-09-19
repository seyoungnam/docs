# 146. LRU Cache

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/lru-cache/description/)

## Solution : Doubly Linked List + Hash Map

### Thought Process

1.  **Requirement**: Achieve $O(1)$ time complexity for both `Get` and `Put` operations.
2.  **Data Structure Choice**:
    -   **Hash Map**: Provides $O(1)$ lookup time for keys. It will store mappings from keys to nodes in a linked list.
    -   **Doubly Linked List**: Allows us to add and remove nodes in $O(1)$ time. By maintaining the nodes in order of their usage, we can easily identify the Least Recently Used (LRU) item at the tail and the Most Recently Used (MRU) item at the head.
3.  **Dummy Nodes**: Use dummy `head` and `tail` nodes to simplify the implementation of adding and removing nodes, avoiding null pointer checks for edge cases (like an empty list).
4.  **Operations**:
    -   `Get(key)`: 
        -   If the key exists in the map, move the corresponding node to the head of the list (marking it as MRU) and return its value.
        -   Otherwise, return -1.
    -   `Put(key, value)`:
        -   If the key exists, update the node's value and move it to the head.
        -   If the key is new:
            -   Create a new node and add it to the head and the map.
            -   If the cache exceeds its `capacity`, remove the node at the tail (LRU) and delete its entry from the map.

### Go Code

``` go
type Node struct {
    key int
    value int
    prev *Node
    next *Node
}

type LRUCache struct {
    capacity int
    cache   map[int]*Node
    head    *Node
    tail    *Node    
}


func Constructor(capacity int) LRUCache {
    lru := LRUCache{
        capacity:   capacity,
        cache:      make(map[int]*Node),
        head:       &Node{}, // dummy head
        tail:       &Node{}, // dummy tail
    }
    lru.head.next = lru.tail
    lru.tail.prev = lru.head
    return lru
}


func (this *LRUCache) Get(key int) int {
    if node, exists := this.cache[key]; exists {
        this.moveToHead(node)
        return node.value
    }
    return -1
}


func (this *LRUCache) Put(key int, value int)  {
    if node, exists := this.cache[key]; exists {
        node.value = value
        this.moveToHead(node)
    } else {
        newNode := &Node{key: key, value: value}
        this.cache[key] = newNode
        this.addNode(newNode)

        if len(this.cache) > this.capacity {
            tail := this.popTail()
            delete(this.cache, tail.key)
        }
    }
}

func (this *LRUCache) addNode(node *Node) {
    temp := this.head.next
    
    this.head.next = node
    node.prev = this.head

    node.next = temp
    temp.prev = node
}

func (this *LRUCache) removeNode(node *Node) {
    prev := node.prev
    next := node.next

    prev.next = next
    next.prev = prev
}

func (this *LRUCache) moveToHead(node *Node) {
    this.removeNode(node)
    this.addNode(node)
}

func (this *LRUCache) popTail() *Node {
    res := this.tail.prev
    this.removeNode(res)
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(1)$ for both `Get` and `Put` operations.
    - Hash map operations (`Get`, `Put`, `Delete`) are average $O(1)$.
    - Doubly linked list operations (`Add`, `Remove`, `Move`) are $O(1)$ given the node pointers.
- **Space Complexity**: $O(C)$
    - $C$ is the capacity of the cache. We store at most $C + 1$ nodes in the linked list and $C$ entries in the hash map.

---

## Extension: LRU Cache with TTL Expiration & Eviction Callbacks

In production systems (e.g., in-memory caches like Redis, Memcached, or Go libraries like `hashicorp/golang-lru`), a standard LRU cache is frequently extended with two essential features:
1. **Time-To-Live (TTL) Expiration**: Each cache entry automatically expires after a designated duration.
2. **Eviction Callbacks**: A hook triggered whenever an entry is evicted, allowing callers to release underlying resources (e.g., closing file descriptors or database connections), flush dirty buffers, or emit monitoring metrics.

### Thought Process & Key Enhancements

1. **Eviction Reasons & Callbacks**:
   - Define an `EvictionReason` enum (`capacity`, `expired`, `manual`) to inform the caller why an item was evicted.
   - Define an `EvictionCallback` type: `func(key int, value int, reason EvictionReason)`.
   - Invoke the callback whenever a node is removed due to LRU capacity overflow, TTL expiration, or manual deletion.
   - *Design Note*: Callbacks run synchronously while holding the lock. To prevent deadlocks, the callback must not call methods on the cache that acquire the lock.

2. **TTL Expiration Management**:
   - **Data Structure**: Extend `Node` with an `expiresAt time.Time` field. If `expiresAt` is zero, the item has no expiration.
   - **Passive (Lazy) Expiration**: On `Get` or `Put`, inspect `node.isExpired()`. If the entry has expired:
     - Remove the node from the doubly linked list and hash map.
     - Fire the eviction callback with `EvictionReasonExpired`.
     - Return `-1, false` (not found).
   - **Active (Periodic) Purge**: Lazy expiration alone can leak memory if expired keys are never queried again. We provide:
     - `DeleteExpired()`: Scans through the doubly linked list and purges all expired nodes.
     - `StartCleanupTimer(interval)` / `Stop()`: Runs a background goroutine ticker to periodically call `DeleteExpired()`.

3. **Concurrency Safety**:
   - Wrap internal state mutations with `sync.Mutex` so the cache can be safely accessed across multiple goroutines, especially when background expiration cleanup is running concurrently with client requests.

4. **Idiomatic Go Interface**:
   - Return `(int, bool)` for `Get(key)` following Go's comma-ok idiom (`val, ok := cache.Get(key)`), avoiding ambiguity between an expired/missing key and an actual stored value of `-1`.
   - Provide `Put(key, value)` (using a default TTL) alongside `PutWithTTL(key, value, ttl)` for custom per-key TTLs.

### Go Code

```go
package main

import (
	"sync"
	"time"
)

// EvictionReason indicates why a key-value pair was removed from the cache.
type EvictionReason string

const (
	EvictionReasonCapacity EvictionReason = "capacity" // Evicted because cache exceeded capacity
	EvictionReasonExpired  EvictionReason = "expired"  // Evicted because TTL expired
	EvictionReasonManual   EvictionReason = "manual"   // Manually removed via Delete
)

// EvictionCallback is invoked when an entry is evicted.
// NOTE: Executed synchronously under the cache lock; do not invoke cache methods inside.
type EvictionCallback func(key int, value int, reason EvictionReason)

// TTLNode extends the doubly linked list node with an expiration timestamp.
type TTLNode struct {
	key       int
	value     int
	expiresAt time.Time // Zero value means no expiration
	prev      *TTLNode
	next      *TTLNode
}

func (n *TTLNode) isExpired() bool {
	if n.expiresAt.IsZero() {
		return false
	}
	return time.Now().After(n.expiresAt)
}

// TTLLRUCache is a thread-safe LRU Cache supporting per-entry TTL and eviction callbacks.
type TTLLRUCache struct {
	mu         sync.Mutex
	capacity   int
	cache      map[int]*TTLNode
	head       *TTLNode
	tail       *TTLNode
	defaultTTL time.Duration
	onEvict    EvictionCallback
	stopChan   chan struct{}
}

// NewTTLLRUCache creates an extended LRU cache.
// If defaultTTL <= 0, items inserted via Put do not expire unless PutWithTTL is used.
// onEvict can be nil if no callback is needed.
func NewTTLLRUCache(capacity int, defaultTTL time.Duration, onEvict EvictionCallback) *TTLLRUCache {
	c := &TTLLRUCache{
		capacity:   capacity,
		cache:      make(map[int]*TTLNode),
		head:       &TTLNode{}, // dummy head
		tail:       &TTLNode{}, // dummy tail
		defaultTTL: defaultTTL,
		onEvict:    onEvict,
	}
	c.head.next = c.tail
	c.tail.prev = c.head
	return c
}

// Get retrieves the value for the key.
// Returns (value, true) if found and not expired; otherwise (-1, false).
func (c *TTLLRUCache) Get(key int) (int, bool) {
	c.mu.Lock()
	defer c.mu.Unlock()

	node, exists := c.cache[key]
	if !exists {
		return -1, false
	}

	// Lazy expiration check
	if node.isExpired() {
		c.removeElement(node, EvictionReasonExpired)
		return -1, false
	}

	c.moveToHead(node)
	return node.value, true
}

// Put inserts or updates a key-value pair using the cache's default TTL.
func (c *TTLLRUCache) Put(key int, value int) {
	c.PutWithTTL(key, value, c.defaultTTL)
}

// PutWithTTL inserts or updates a key-value pair with a custom TTL.
// A ttl <= 0 means the item will never expire.
func (c *TTLLRUCache) PutWithTTL(key int, value int, ttl time.Duration) {
	c.mu.Lock()
	defer c.mu.Unlock()

	var expiresAt time.Time
	if ttl > 0 {
		expiresAt = time.Now().Add(ttl)
	}

	if node, exists := c.cache[key]; exists {
		node.value = value
		node.expiresAt = expiresAt
		c.moveToHead(node)
		return
	}

	newNode := &TTLNode{
		key:       key,
		value:     value,
		expiresAt: expiresAt,
	}
	c.cache[key] = newNode
	c.addNode(newNode)

	// If capacity exceeded, evict the least recently used element (at tail)
	if len(c.cache) > c.capacity {
		tail := c.tail.prev
		reason := EvictionReasonCapacity
		if tail.isExpired() {
			reason = EvictionReasonExpired
		}
		c.removeElement(tail, reason)
	}
}

// Delete manually removes a key from the cache.
func (c *TTLLRUCache) Delete(key int) bool {
	c.mu.Lock()
	defer c.mu.Unlock()

	if node, exists := c.cache[key]; exists {
		c.removeElement(node, EvictionReasonManual)
		return true
	}
	return false
}

// DeleteExpired purges all expired entries from the cache.
// Returns the count of deleted entries.
func (c *TTLLRUCache) DeleteExpired() int {
	c.mu.Lock()
	defer c.mu.Unlock()

	count := 0
	now := time.Now()
	curr := c.head.next
	for curr != c.tail {
		next := curr.next
		if !curr.expiresAt.IsZero() && now.After(curr.expiresAt) {
			c.removeElement(curr, EvictionReasonExpired)
			count++
		}
		curr = next
	}
	return count
}

// StartCleanupTimer runs a background goroutine to periodically clean up expired items.
func (c *TTLLRUCache) StartCleanupTimer(interval time.Duration) {
	c.mu.Lock()
	if c.stopChan != nil {
		c.mu.Unlock()
		return
	}
	c.stopChan = make(chan struct{})
	c.mu.Unlock()

	go func() {
		ticker := time.NewTicker(interval)
		defer ticker.Stop()

		for {
			select {
			case <-ticker.C:
				c.DeleteExpired()
			case <-c.stopChan:
				return
			}
		}
	}()
}

// Stop terminates the periodic background cleanup goroutine.
func (c *TTLLRUCache) Stop() {
	c.mu.Lock()
	defer c.mu.Unlock()

	if c.stopChan != nil {
		close(c.stopChan)
		c.stopChan = nil
	}
}

// Internal linked list helper methods

func (c *TTLLRUCache) addNode(node *TTLNode) {
	temp := c.head.next

	c.head.next = node
	node.prev = c.head

	node.next = temp
	temp.prev = node
}

func (c *TTLLRUCache) removeNode(node *TTLNode) {
	prev := node.prev
	next := node.next

	prev.next = next
	next.prev = prev
}

func (c *TTLLRUCache) moveToHead(node *TTLNode) {
	c.removeNode(node)
	c.addNode(node)
}

func (c *TTLLRUCache) removeElement(node *TTLNode, reason EvictionReason) {
	c.removeNode(node)
	delete(c.cache, node.key)
	if c.onEvict != nil {
		c.onEvict(node.key, node.value, reason)
	}
}
```

### Usage Example

```go
func main() {
	onEvict := func(key, value int, reason EvictionReason) {
		fmt.Printf("[Callback] Evicted key=%d, value=%d, reason=%s\n", key, value, reason)
	}

	// Cache with capacity=2, default TTL=100ms
	cache := NewTTLLRUCache(2, 100*time.Millisecond, onEvict)
	defer cache.Stop()

	// 1. Capacity eviction (LRU)
	cache.Put(1, 10)
	cache.Put(2, 20)
	cache.Put(3, 30) // Exceeds capacity -> key 1 is evicted (reason: capacity)

	// 2. Passive (Lazy) expiration
	time.Sleep(120 * time.Millisecond)
	val, ok := cache.Get(2) // Expired -> key 2 is evicted (reason: expired), returns (-1, false)
	fmt.Printf("Get(2): val=%d, ok=%v\n", val, ok)

	// 3. Active (Periodic) cleanup
	cache.PutWithTTL(4, 40, 50*time.Millisecond)
	cache.StartCleanupTimer(30 * time.Millisecond)
	time.Sleep(100 * time.Millisecond) // Purged by background timer (reason: expired)
}
```

### Code Efficiency

- **Time Complexity**:
    - `Get`, `Put`, `PutWithTTL`, `Delete`: $O(1)$ amortized time.
    - `DeleteExpired`: $O(N)$ where $N$ is the number of entries currently stored in the cache (scans the doubly linked list).
- **Space Complexity**: $O(C)$ where $C$ is the maximum capacity of the cache.

