# 2502. Design Memory Allocator

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/design-memory-allocator/description/)

## Solution: Array Simulation (Brute Force)

We can represent the memory space using an array/slice of size $n$, where each element at index `i` stores the memory ID (`mID`) allocated to it (or `0` if the block is currently free). Because the limits on the memory size and queries are relatively small ($n \le 1000$), this simple brute force array traversal is highly performant and straightforward to implement.

### Thought Process

1.  **Memory Initialization**:
    - Create a slice `memory` of size `n` filled with `0`s.
2.  **Memory Allocation (`Allocate`)**:
    - Keep a running counter `freeCount` to track consecutive free slots.
    - Iterate through the array:
        - If `memory[i] == 0`, increment `freeCount`.
        - If `freeCount` reaches the requested `size`, we have found a contiguous free block. Compute the starting index: `startIdx = i - size + 1`. Overwrite all slots from `startIdx` to `i` with `mID` and return `startIdx`.
        - If `memory[i] != 0`, reset `freeCount = 0` and continue searching.
    - If the loop finishes without allocating memory, return `-1`.
3.  **Memory Freeing (`FreeMemory`)**:
    - Iterate through the entire array.
    - Whenever `memory[i] == mID`, reset it to `0` and increment a counter of `freeBlocks`.
    - Return the total count of freed blocks.

### Go Code

``` go
type Allocator struct {
    memory  []int
    size    int
}


func Constructor(n int) Allocator {
    return Allocator{
        memory: make([]int, n),
        size:   n,
    }
}


func (this *Allocator) Allocate(size int, mID int) int {
    freeCount := 0
    for i := 0; i < this.size; i++ {
        if this.memory[i] == 0 {
            freeCount++
            if freeCount == size {
                startIdx := i - size + 1
                for j := startIdx; j <= i; j++ {
                    this.memory[j] = mID
                }
                return startIdx
            }
        } else {
            freeCount = 0
        }
    }
    return -1
}


func (this *Allocator) FreeMemory(mID int) int {
    freeBlocks := 0
    for i := 0; i < this.size; i++ {
        if this.memory[i] == mID {
            this.memory[i] = 0
            freeBlocks++
        }
    }
    return freeBlocks
}
```

### Code Efficiency

- **Time Complexity**:
    - **`Constructor`**: $O(n)$ to initialize the slice of size $n$.
    - **`Allocate`**: $O(n)$ to scan the memory array.
    - **`FreeMemory`**: $O(n)$ to scan and clear memory indices matching `mID`.
- **Space Complexity**: $O(n)$
    - We allocate a slice of size $n$ to represent the memory units.

---

## Alternative Solution: Doubly Linked List (Block Coalescing)

Instead of using a static array, we can represent memory dynamically using a doubly linked list of memory blocks. Each node in the list represents a contiguous block of memory with a start index, end index, and an associated `mID` (where `mID = 0` represents free memory). This allows us to merge (coalesce) adjacent free blocks during deallocation, reducing memory fragmentation.

### Thought Process

1.  **Block Structure**:
    - Define a `MemNode` containing: `start` (inclusive index), `end` (exclusive index), `mID` (allocation ID), and pointers `prev` and `next` to enable traversal in both directions.
2.  **Allocation (`Allocate`)**:
    - Traverse the list starting from the `head` to find the first free block (`mID == 0`) with capacity `currSize >= size`.
    - **Exact Match**: If the free block's size is exactly `size`, update `curr.mID = mID`.
    - **Split Block**: If the free block is larger than `size`, create a new node of size `size` with `mID`, insert it before `curr` in the list, and shrink the free block by updating its start pointer: `curr.start += size`.
    - Return the starting index. If no block satisfies the request, return `-1`.
3.  **Deallocation (`FreeMemory`)**:
    - Traverse the list. For each node `curr` where `curr.mID == mID`:
        - Add its block size (`curr.end - curr.start`) to `freedBlocks`.
        - Set `curr.mID = 0` to mark it as free.
        - **Coalesce Right**: If the next node `curr.next` is also free, merge it into `curr` by extending `curr.end = curr.next.end` and removing `curr.next` from the list.
        - **Coalesce Left**: If the previous node `curr.prev` is also free, merge `curr` into `curr.prev` similarly, updating the pointer references.

### Go Code

``` go
type MemNode struct {
    start   int
    end     int
    mID     int
    prev    *MemNode
    next    *MemNode
}

type Allocator struct {
    head *MemNode
}


func Constructor(n int) Allocator {
    initialBlock := &MemNode{
        start: 0,
        end:   n,
        mID:   0,
    }
    return Allocator{head: initialBlock}
}


func (this *Allocator) Allocate(size int, mID int) int {
    curr := this.head
    for curr != nil {
        currSize := curr.end - curr.start
        if curr.mID == 0 && currSize >= size {
            allocStart := curr.start
            if currSize == size {
                curr.mID = mID
            } else {
                newNode := &MemNode{
                    start:  curr.start,
                    end:    curr.start + size,
                    mID:    mID,
                    prev:   curr.prev,
                    next:   curr,
                }

                if curr.prev != nil {
                    curr.prev.next = newNode
                } else {
                    this.head = newNode
                }
                curr.prev = newNode
                curr.start = curr.start + size
            }
            return allocStart
        }
        curr = curr.next
    }
    return -1
}


func (this *Allocator) FreeMemory(mID int) int {
    freedBlocks := 0
    curr := this.head

    for curr != nil {
        if curr.mID == mID {
            freedBlocks += (curr.end - curr.start)
            curr.mID = 0

            if curr.next != nil && curr.next.mID == 0 {
                deadNode := curr.next
                curr.end = deadNode.end
                curr.next = deadNode.next
                if deadNode.next != nil {
                    deadNode.next.prev = curr
                }
            }

            if curr.prev != nil && curr.prev.mID == 0 {
                prevNode := curr.prev
                prevNode.end = curr.end
                prevNode.next= curr.next
                if curr.next != nil {
                    curr.next.prev = prevNode
                }
                curr = prevNode
            }
        }
        curr = curr.next
    }
    return freedBlocks
}
```

### Code Efficiency

- **Time Complexity**:
    - **`Constructor`**: $O(1)$ to create a single memory node.
    - **`Allocate`**: $O(m)$ where $m$ is the number of blocks in the linked list. In the worst case (maximum fragmentation), $m$ can approach $n$, but it is typically much smaller.
    - **`FreeMemory`**: $O(m)$ to scan the linked list and perform $O(1)$ block merges.
- **Space Complexity**: $O(m)$
    - We store $m$ nodes in the doubly linked list.