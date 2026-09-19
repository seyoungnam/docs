# 1797. Design Authentication Manager

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/design-authentication-manager/description/)

## Solution 1: Hash Map (Direct Simulation)

A straightforward way to solve this problem is to use a Hash Map storing each `tokenId` alongside its `expireTime`.

### Thought Process

1. **Storage**: Use a map `map[string]int` mapping `tokenId -> expireTime`.
2. **Operations**:
    - `Generate(tokenId, currentTime)`: Store `currentTime + timeToLive` in the map for the given `tokenId`.
    - `Renew(tokenId, currentTime)`: Check if `tokenId` exists in the map and its `expireTime > currentTime` (unexpired). If valid, update its `expireTime` to `currentTime + timeToLive`.
    - `CountUnexpiredTokens(currentTime)`: Iterate over all entries in the map and count how many tokens satisfy `expireTime > currentTime`.

### Go Code

```go
type AuthenticationManager struct {
    tokens map[string]int
    ttl    int
}

func Constructor(timeToLive int) AuthenticationManager {
    return AuthenticationManager{
        tokens: make(map[string]int),
        ttl:    timeToLive,
    }
}

func (this *AuthenticationManager) Generate(tokenId string, currentTime int) {
    this.tokens[tokenId] = currentTime + this.ttl
}

func (this *AuthenticationManager) Renew(tokenId string, currentTime int) {
    if expTime, exists := this.tokens[tokenId]; exists && expTime > currentTime {
        this.tokens[tokenId] = currentTime + this.ttl
    }
}

func (this *AuthenticationManager) CountUnexpiredTokens(currentTime int) int {
    count := 0
    for _, expTime := range this.tokens {
        if expTime > currentTime {
            count++
        }
    }
    return count
}
```

### Code Efficiency

- **Time Complexity**:
    - `Constructor`: $O(1)$
    - `Generate`: $O(1)$
    - `Renew`: $O(1)$
    - `CountUnexpiredTokens`: $O(n)$, where $n$ is the total number of tokens generated, because we iterate through all entries in the map.
- **Space Complexity**: $O(n)$ to store all tokens.

---

## Solution 2: Doubly Linked List + Hash Map (Optimized $O(1)$ Amortized)

When `CountUnexpiredTokens` is called frequently, iterating through all tokens in $O(n)$ becomes slow. We can achieve **$O(1)$ amortized time** across all operations by combining a **Doubly Linked List** with a **Hash Map** (similar to an LRU cache).

### Thought Process

1. **Chronological Property**:
    - The problem guarantees that `currentTime` is strictly monotonically increasing across all calls.
    - When a token is generated or renewed at `currentTime`, its new expiration time is `currentTime + ttl`.
    - Because `currentTime` always increases, newly added or renewed tokens will always have a larger `expireTime` than previously added tokens.
2. **Doubly Linked List as an Expiry Queue**:
    - Maintain nodes in a doubly linked list sorted by their expiration time in ascending order.
    - **Head**: Contains tokens that expire earliest.
    - **Tail**: Contains tokens that expire latest.
    - Adding a new token or renewing an existing token simply moves it to the **tail**.
3. **Lazy Eviction (`removeExpiredNodes`)**:
    - Before handling operations, start traversing from `head.next`.
    - Delete nodes from the map and list as long as `node.expireTime <= currentTime`.
    - Stop immediately when encountering a node where `node.expireTime > currentTime` because all subsequent nodes are guaranteed to still be valid.
    - Each node is added once and deleted at most once, resulting in **$O(1)$ amortized** cleanup time.
4. **$O(1)$ Counting**:
    - After evicting expired nodes from the head, every remaining token in `cache` is valid.
    - `CountUnexpiredTokens` simply returns `len(this.cache)` in $O(1)$ time.

### Go Code

```go
type Node struct {
    key        string
    expireTime int
    prev       *Node
    next       *Node
}

type AuthenticationManager struct {
    cache map[string]*Node
    ttl   int
    head  *Node
    tail  *Node
}

func Constructor(timeToLive int) AuthenticationManager {
    head, tail := &Node{}, &Node{}
    head.next = tail
    tail.prev = head
    return AuthenticationManager{
        cache: make(map[string]*Node),
        ttl:   timeToLive,
        head:  head,
        tail:  tail,
    }
}

func (this *AuthenticationManager) Generate(tokenId string, currentTime int) {
    this.removeExpiredNodes(currentTime)
    if _, exists := this.cache[tokenId]; exists {
        return
    }
    newNode := &Node{
        key:        tokenId,
        expireTime: currentTime + this.ttl,
    }
    this.addNode(newNode)
    this.cache[tokenId] = newNode
}

func (this *AuthenticationManager) Renew(tokenId string, currentTime int) {
    this.removeExpiredNodes(currentTime)
    if node, exists := this.cache[tokenId]; exists && currentTime < node.expireTime {
        node.expireTime = currentTime + this.ttl
        this.removeNode(node)
        this.addNode(node)
    }
}

func (this *AuthenticationManager) CountUnexpiredTokens(currentTime int) int {
    this.removeExpiredNodes(currentTime)
    return len(this.cache)
}

func (this *AuthenticationManager) addNode(node *Node) {
    prev, next := this.tail.prev, this.tail

    prev.next = node
    node.prev = prev

    node.next = next
    next.prev = node
}

func (this *AuthenticationManager) removeNode(node *Node) {
    prev, next := node.prev, node.next

    prev.next = next
    next.prev = prev
}

func (this *AuthenticationManager) removeExpiredNodes(currentTime int) {
    curr := this.head.next
    for curr != this.tail && curr.expireTime <= currentTime {
        delete(this.cache, curr.key)
        this.removeNode(curr)
        curr = curr.next
    }
}

/**
 * Your AuthenticationManager object will be instantiated and called as such:
 * obj := Constructor(timeToLive);
 * obj.Generate(tokenId,currentTime);
 * obj.Renew(tokenId,currentTime);
 * param_3 := obj.CountUnexpiredTokens(currentTime);
 */
```

### Code Efficiency

- **Time Complexity**:
    - `Constructor`: $O(1)$
    - `Generate`: $O(1)$ amortized (adding to tail is $O(1)$, expired node eviction touches each node at most once).
    - `Renew`: $O(1)$ amortized (map lookup and moving node to tail are $O(1)$).
    - `CountUnexpiredTokens`: $O(1)$ amortized (returns `len(this.cache)` after evicting expired tokens from the head).
- **Space Complexity**: $O(n)$, where $n$ is the number of active unexpired tokens stored in the hash map and doubly linked list.