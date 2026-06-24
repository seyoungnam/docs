# 622. Design Circular Queue

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/design-circular-queue/description/)

## Solution: Array-Based Ring Buffer Simulation

A circular queue (or ring buffer) is a linear data structure that connects the end of a fixed-size buffer back to the beginning, allowing for memory reuse. We can implement it efficiently using a fixed-size slice, a `head` index, a `capacity` limit, and a running `count` of active elements. Keeping a `count` makes it simple to distinguish between empty and full states and dynamically compute the tail index.

### Thought Process

1.  **State Fields**:
    - `data`: A slice of size `k` to hold the queue elements.
    - `head`: The index of the front element in the queue.
    - `count`: The number of elements currently stored in the queue.
    - `capacity`: The maximum number of elements the queue can store (`k`).
2.  **EnQueue**:
    - Check if the queue is full (`count == capacity`). If so, return `false`.
    - Compute the next insertion index: `tailIdx = (head + count) % capacity`.
    - Put the new value at `data[tailIdx]` and increment `count`.
3.  **DeQueue**:
    - Check if the queue is empty (`count == 0`). If so, return `false`.
    - Move the `head` index forward circularly: `head = (head + 1) % capacity`.
    - Decrement `count`.
4.  **Front / Rear Lookups**:
    - `Front()` returns `data[head]` if the queue is not empty.
    - `Rear()` returns the last inserted element located at `data[(head + count - 1) % capacity]` if the queue is not empty.
5.  **Status Checks**:
    - `IsEmpty()` returns whether `count == 0`.
    - `IsFull()` returns whether `count == capacity`.

### Go Code

``` go
type MyCircularQueue struct {
    data        []int
    head        int
    count       int
    capacity    int
}


func Constructor(k int) MyCircularQueue {
    return MyCircularQueue{
        data:       make([]int, k),
        head:       0,
        count:      0,
        capacity:   k,
    }
}


func (this *MyCircularQueue) EnQueue(value int) bool {
    if this.IsFull() {
        return false
    }
    tailIdx := (this.head + this.count) % this.capacity
    this.data[tailIdx] = value
    this.count++
    return true
}


func (this *MyCircularQueue) DeQueue() bool {
    if this.IsEmpty() {
        return false
    }
    this.head = (this.head + 1) % this.capacity
    this.count--
    return true
}


func (this *MyCircularQueue) Front() int {
    if this.IsEmpty() {
        return -1
    }
    return this.data[this.head]
}


func (this *MyCircularQueue) Rear() int {
    if this.IsEmpty() {
        return -1
    }
    tailIdx := (this.head + this.count - 1) % this.capacity
    return this.data[tailIdx]
}


func (this *MyCircularQueue) IsEmpty() bool {
    return this.count == 0
}


func (this *MyCircularQueue) IsFull() bool {
    return this.count == this.capacity
}
```

### Code Efficiency

- **Time Complexity**:
    - **`Constructor`**: $O(k)$ to allocate the slice of size $k$.
    - **`EnQueue` / `DeQueue` / `Front` / `Rear` / `IsEmpty` / `IsFull`**: $O(1)$ constant time operations.
- **Space Complexity**: $O(k)$
    - We allocate a slice of size $k$ to store the elements of the queue.