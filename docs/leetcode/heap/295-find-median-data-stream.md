# 295. Find Median from Data Stream

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/find-median-from-data-stream/description/)

## Solution: Dual Heaps (Min/Max Heap)

To find the median in $O(1)$ time, we divide the data stream into two equal halves: the smaller half and the larger half. We store the smaller half in a **Max-Heap** (left) and the larger half in a **Min-Heap** (right).

### Thought Process

1.  **Heap Invariants**:
    *   **Value Invariant**: All elements in `left` (Max-Heap) must be less than or equal to all elements in `right` (Min-Heap).
    *   **Size Invariant**: `left` can have at most one more element than `right`: `left.Len() >= right.Len() && left.Len() - right.Len() <= 1`.
2.  **Adding a Number (`AddNum`)**:
    *   Push `num` to `left` (Max-Heap).
    *   To maintain the value invariant, pop the largest element from `left` and push it to `right` (Min-Heap).
    *   If `right` ends up with more elements than `left` (violating the size invariant), pop the smallest element from `right` and push it back to `left`.
3.  **Finding the Median (`FindMedian`)**:
    *   If `left` has more elements than `right`, the median is the top of `left`.
    *   Otherwise, the median is the average of the tops of both heaps: `(left.Top + right.Top) / 2.0`.

### Go Code

``` go
import (
    "container/heap"
)

type minHeap []int

func (h minHeap) Len() int { return len(h) }
func (h minHeap) Less(i, j int) bool { return h[i] < h[j] }
func (h minHeap) Swap(i, j int) { h[i], h[j] = h[j], h[i] }
func (h *minHeap) Push(x interface{}) { *h = append(*h, x.(int)) }
func (h *minHeap) Pop() interface{} {
    last := (*h)[len(*h)-1]
    *h = (*h)[:len(*h)-1]
    return last
}

type maxHeap []int

func (h maxHeap) Len() int { return len(h) }
func (h maxHeap) Less(i, j int) bool { return h[i] > h[j] }
func (h maxHeap) Swap(i, j int) { h[i], h[j] = h[j], h[i] }
func (h *maxHeap) Push(x interface{}) { *h = append(*h, x.(int)) }
func (h *maxHeap) Pop() interface{} {
    last := (*h)[len(*h)-1]
    *h = (*h)[:len(*h)-1]
    return last
}


type MedianFinder struct {
    left *maxHeap
    right *minHeap
}


func Constructor() MedianFinder {
    l, r := &maxHeap{}, &minHeap{}
    heap.Init(l)
    heap.Init(r)
    return MedianFinder{
        left: l,
        right: r,
    }
}


func (this *MedianFinder) AddNum(num int)  {
    heap.Push(this.left, num)
    heap.Push(this.right, heap.Pop(this.left))
    if this.left.Len() < this.right.Len() {
        heap.Push(this.left, heap.Pop(this.right))
    }
}


func (this *MedianFinder) FindMedian() float64 {
    if this.left.Len() > this.right.Len() {
        return float64((*this.left)[0])
    }
    return float64((*this.left)[0] + (*this.right)[0]) / 2.0
}
```

### Code Efficiency

- **Time Complexity**:
    - **`AddNum`**: $O(\log n)$ – Each insertion requires a constant number of push and pop operations on heaps of size up to $n/2$.
    - **`FindMedian`**: $O(1)$ – The roots of both heaps are accessed in constant time.
- **Space Complexity**: $O(n)$
    - We store all $n$ elements from the data stream in the two heaps.
