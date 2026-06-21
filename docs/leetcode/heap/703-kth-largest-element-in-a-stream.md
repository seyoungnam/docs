# 703. Kth Largest Element in a Stream

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/kth-largest-element-in-a-stream/description/)

## Solution: Min-Heap

To find the $k$-th largest element in a stream efficiently, we maintain a min-heap of size $k$. The min-heap will store the $k$ largest elements seen so far. The minimum element of this heap (the root) is guaranteed to be the $k$-th largest element.

### Thought Process

1.  **Min-Heap of Size $k$**:
    - Any element smaller than the current $k$-th largest element is irrelevant because it can never become the $k$-th largest or greater.
    - By using a min-heap, the smallest of the $k$ largest elements is always at the top (root).
2.  **Constructor**:
    - Initialize the min-heap with the initial `nums` slice.
    - If the number of elements is greater than $k$, repeatedly pop elements from the heap until its size is exactly $k$.
3.  **Add**:
    - Push the new `val` onto the min-heap.
    - If the size of the heap exceeds $k$, pop the smallest element (the root).
    - Return the new root of the heap (`(*this.h)[0]`), which is the current $k$-th largest element.

### Go Code

``` go
import "container/heap"

type minHeap []int

func (h minHeap) Len() int           { return len(h) }
func (h minHeap) Less(i, j int) bool { return h[i] < h[j] }
func (h minHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *minHeap) Push(x interface{}) { *h = append(*h, x.(int)) }
func (h *minHeap) Pop() interface{} {
	last := (*h)[len(*h)-1]
	*h = (*h)[:len(*h)-1]
	return last
}

type KthLargest struct {
	h *minHeap
	k int
}

func Constructor(k int, nums []int) KthLargest {
	h := minHeap(nums)
	heap.Init(&h)
	for h.Len() > k {
		heap.Pop(&h)
	}
	return KthLargest{
		h: &h,
		k: k,
	}
}

func (this *KthLargest) Add(val int) int {
	heap.Push(this.h, val)
	if this.h.Len() > this.k {
		heap.Pop(this.h)
	}
	return (*this.h)[0]
}
```

### Code Efficiency

- **Time Complexity**:
    - **`Constructor`**: $O(n \log n)$ in the worst case. Initializing the heap of size $n$ takes $O(n)$ time, and popping the excess elements down to size $k$ takes $O((n - k) \log n)$ time.
    - **`Add`**: $O(\log k)$ – Pushing a value and popping the smallest element from a heap of size $k$ takes logarithmic time relative to the heap size.
- **Space Complexity**: $O(k)$
    - The heap stores at most $k$ elements. Note that the constructor temporarily takes $O(n)$ space during heap initialization if the initial slice has $n$ elements.