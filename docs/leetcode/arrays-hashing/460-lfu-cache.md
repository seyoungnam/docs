# 460. LFU Cache

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/lfu-cache/description/)

## Solution : Doubly Linked List + Hash Map

### Thought Process

Implementing a Least Frequently Used (LFU) cache with $O(1)$ time complexity for both `Get` and `Put` operations requires tracking two dimensions of information:
1. **Frequency of access**: We must know which items have been accessed the fewest times.
2. **Recency of access**: When there is a tie in frequency, we must evict the Least Recently Used (LRU) item.

#### 1. Data Structure Design

To achieve $O(1)$ operations, we use a combination of Hash Maps and Doubly Linked Lists:

- **`Node`**: Represents a cache entry. It stores the `key`, `val`, `frequency`, and pointers (`prev` and `next`) for list operations.
- **`DoublyLinkedList`**: A list that maintains the recency of nodes for a **specific frequency**.
  - Pushing a node to the head of the list represents the most recently accessed node.
  - The tail of the list represents the least recently accessed node for that frequency bucket.
  - Adding, removing, or getting the tail node of a doubly linked list takes $O(1)$ time.
- **`cache` Map (`map[int]*Node`)**: Maps a key directly to its node. This enables $O(1)$ lookups and updates.
- **`freqMap` Map (`map[int]*DoublyLinkedList`)**: Maps a frequency value to a `DoublyLinkedList` containing all nodes with that frequency.
- **`minFrequency`**: An integer tracking the minimum frequency currently present in the cache. This allows us to locate the list to evict from in $O(1)$ time when the cache reaches capacity.

#### 2. Key Algorithms

##### Frequency Update (`updateFrequency`)
Whenever a node is accessed (via `Get` or updated via `Put`), its frequency increases:
1. Remove the node from the `DoublyLinkedList` corresponding to its current frequency.
2. If this list becomes empty and its frequency was the `minFrequency`, increment `minFrequency` by 1.
3. Increment the node's frequency.
4. Move the node to the head of the `DoublyLinkedList` for its new frequency (creating the list if it doesn't exist).

##### Get Operation
1. If the key is not in the `cache` map, return `-1`.
2. If it exists, retrieve the node, call `updateFrequency(node)`, and return its value.

##### Put Operation
1. If capacity is `0`, return immediately.
2. If the key already exists, update its value, call `updateFrequency(node)`, and return.
3. If the cache is at capacity:
   - Locate the list for `minFrequency`.
   - Remove the tail node (the least recently used node in the lowest frequency bucket) from the list.
   - Delete this evicted node's key from the `cache` map.
4. Create a new node with `frequency = 1`.
5. Add the new node to the `cache` map.
6. Reset `minFrequency` to `1` (since the new node has a frequency of 1).
7. Insert the new node into the `DoublyLinkedList` for frequency 1.


### Go Code

``` go
type Node struct {
    key, val    int
    frequency   int
    prev, next  *Node
}

type DoublyLinkedList struct {
    dummyHead   *Node
    dummyTail   *Node
    size        int
}

func NewDoublyLinkedList() *DoublyLinkedList {
    list := &DoublyLinkedList{
        dummyHead: &Node{},
        dummyTail: &Node{},
        size:       0,
    }
    list.dummyHead.next = list.dummyTail
    list.dummyTail.prev = list.dummyHead
    return list
}

func (list *DoublyLinkedList) AddToHead(node *Node) {
    temp := list.dummyHead.next

    list.dummyHead.next = node
    node.prev = list.dummyHead

    node.next = temp
    temp.prev = node
    
    list.size++
}

func (list *DoublyLinkedList) RemoveNode(node *Node) {
    prev := node.prev
    next := node.next

    prev.next = next
    next.prev = prev
    
    list.size--
}

func (list *DoublyLinkedList) RemoveTail() *Node {
    if list.size == 0 {
        return nil
    }
    tail := list.dummyTail.prev
    list.RemoveNode(tail)
    return tail
}


type LFUCache struct {
    cache           map[int]*Node
    freqMap         map[int]*DoublyLinkedList
    capacity        int
    minFrequency    int
}


func Constructor(capacity int) LFUCache {
    return LFUCache{
        cache:          make(map[int]*Node),
        freqMap:        make(map[int]*DoublyLinkedList),
        capacity:       capacity,
        minFrequency:   0,
    }
}

func (this *LFUCache) updateFrequency(node *Node) {
    oldFreq := node.frequency
    oldList := this.freqMap[oldFreq]
    oldList.RemoveNode(node)

    if oldFreq == this.minFrequency && oldList.size == 0 {
        this.minFrequency++
    }

    node.frequency++
    newFreq := node.frequency
    if _, ok := this.freqMap[newFreq]; !ok {
        this.freqMap[newFreq] = NewDoublyLinkedList()
    }
    this.freqMap[newFreq].AddToHead(node)
}


func (this *LFUCache) Get(key int) int {
    if node, ok := this.cache[key]; ok {
        this.updateFrequency(node)
        return node.val
    }
    return -1
}


func (this *LFUCache) Put(key int, value int)  {
    if this.capacity == 0 {
        return
    }
    if node, ok := this.cache[key]; ok {
        node.val = value
        this.updateFrequency(node)
        return
    }

    if len(this.cache) >= this.capacity {
        evictionList := this.freqMap[this.minFrequency]
        deadNode := evictionList.RemoveTail()
        delete(this.cache, deadNode.key)
    }

    newNode := &Node{key: key, val: value, frequency: 1}
    this.cache[key] = newNode
    this.minFrequency = 1

    if _, ok := this.freqMap[1]; !ok {
        this.freqMap[1] = NewDoublyLinkedList()
    }
    this.freqMap[1].AddToHead(newNode)
}
```

### Code Efficiency

- **Time Complexity**: $O(1)$ for both `Get` and `Put` operations.
    - Hash map operations (`Get`, `Put`, `Delete`) are average $O(1)$.
    - Doubly linked list operations (`Add`, `Remove`, `Move`) are $O(1)$ given the node pointers.
- **Space Complexity**: $O(C)$
    - $C$ is the capacity of the cache. We store at most $C + 1$ nodes in the linked list and $C$ entries in the hash map.
