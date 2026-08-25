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
