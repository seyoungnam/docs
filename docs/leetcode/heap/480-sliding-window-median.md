# 480. Sliding Window Median

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/sliding-window-median/description/)

## Solution: Dual Heaps (with Lazy Deletion)

To find the median of a sliding window of size $k$ efficiently, we can extend the dual-heap strategy (using a max-heap for the smaller half of numbers and a min-heap for the larger half of numbers). 

However, since the window slides, we also need to **remove** elements that fall out of the sliding window. Removing arbitrary elements from a binary heap is normally an $\mathcal{O}(N)$ operation. To maintain $\mathcal{O}(\log k)$ performance, we use **lazy deletion**:

- When an element falls out of the sliding window, we do not remove it from the heap immediately.
- Instead, we record its removal in a hash map `delayed` tracking how many times each number has been lazily deleted.
- We only remove (or "prune") elements from the top of the heaps when they are no longer valid. This ensures that the heap roots always represent active elements.

### Thought Process

1.  **Heap Invariants**:
    *   `small`: A max-heap storing the smaller half of the active elements.
    *   `large`: A min-heap storing the larger half of the active elements.
    *   `smallSize` and `largeSize`: Keep track of the number of *active* elements in each heap (excluding lazily deleted ones). We maintain:
        *   `smallSize >= largeSize`
        *   `smallSize - largeSize <= 1`
2.  **Pruning (`pruneSmall` / `pruneLarge`)**:
    *   If the top element of the heap has a positive count in the `delayed` map, it has been lazily deleted.
    *   Decrement its count in `delayed` and pop it from the heap.
    *   Repeat until the top of the heap is an active element.
3.  **Inserting a Number (`Insert`)**:
    *   If `small` is empty or the number is less than or equal to the top of `small`, push it to `small` and increment `smallSize`.
    *   Otherwise, push it to `large` and increment `largeSize`.
    *   Call `makeBalance()` to restore size invariants.
4.  **Erasing a Number (`Erase`)**:
    *   Add the number to the `delayed` map to mark it for lazy deletion.
    *   Decrement `smallSize` if the number belongs to `small` (i.e. <= `small.top`), otherwise decrement `largeSize`.
    *   If the erased number is at the top of either heap, immediately call the corresponding prune function.
    *   Call `makeBalance()` to restore size invariants.
5.  **Balancing (`makeBalance`)**:
    *   If `smallSize > largeSize + 1`, pop the top of `small` and push it to `large`, adjust size counters, and prune `small`.
    *   If `smallSize < largeSize`, pop the top of `large` and push it to `small`, adjust size counters, and prune `large`.

### Go Code

```go
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

type maxHeap []int

func (h maxHeap) Len() int           { return len(h) }
func (h maxHeap) Less(i, j int) bool { return h[i] > h[j] }
func (h maxHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *maxHeap) Push(x interface{}) { *h = append(*h, x.(int)) }
func (h *maxHeap) Pop() interface{} {
	last := (*h)[len(*h)-1]
	*h = (*h)[:len(*h)-1]
	return last
}

type DualHeap struct {
	small     maxHeap
	large     minHeap
	delayed   map[int]int
	smallSize int
	largeSize int
	k         int
}

func NewDualHeap(k int) *DualHeap {
	return &DualHeap{
		small:   maxHeap{},
		large:   minHeap{},
		delayed: make(map[int]int),
		k:       k,
	}
}

func (dh *DualHeap) pruneSmall() {
	for dh.small.Len() > 0 {
		top := dh.small[0]
		if count, ok := dh.delayed[top]; ok && count > 0 {
			dh.delayed[top]--
			if dh.delayed[top] == 0 {
				delete(dh.delayed, top)
			}
			heap.Pop(&dh.small)
		} else {
			break
		}
	}
}

func (dh *DualHeap) pruneLarge() {
	for dh.large.Len() > 0 {
		top := dh.large[0]
		if count, ok := dh.delayed[top]; ok && count > 0 {
			dh.delayed[top]--
			if dh.delayed[top] == 0 {
				delete(dh.delayed, top)
			}
			heap.Pop(&dh.large)
		} else {
			break
		}
	}
}

func (dh *DualHeap) makeBalance() {
	if dh.smallSize > dh.largeSize+1 {
		top := heap.Pop(&dh.small).(int)
		heap.Push(&dh.large, top)
		dh.smallSize--
		dh.largeSize++
		dh.pruneSmall()
	} else if dh.smallSize < dh.largeSize {
		top := heap.Pop(&dh.large).(int)
		heap.Push(&dh.small, top)
		dh.largeSize--
		dh.smallSize++
		dh.pruneLarge()
	}
}

func (dh *DualHeap) Insert(num int) {
	if dh.small.Len() == 0 || num <= dh.small[0] {
		heap.Push(&dh.small, num)
		dh.smallSize++
	} else {
		heap.Push(&dh.large, num)
		dh.largeSize++
	}
	dh.makeBalance()
}

func (dh *DualHeap) Erase(num int) {
	dh.delayed[num]++
	if num <= dh.small[0] {
		dh.smallSize--
		if num == dh.small[0] {
			dh.pruneSmall()
		}
	} else {
		dh.largeSize--
		if num == dh.large[0] {
			dh.pruneLarge()
		}
	}
	dh.makeBalance()
}

func (dh *DualHeap) GetMedian() float64 {
	if dh.k%2 == 1 {
		return float64(dh.small[0])
	}
	return (float64(dh.small[0]) + float64(dh.large[0])) / 2.0
}

func medianSlidingWindow(nums []int, k int) []float64 {
	if len(nums) == 0 || k == 0 {
		return []float64{}
	}

	dh := NewDualHeap(k)
	for i := 0; i < k; i++ {
		dh.Insert(nums[i])
	}

	res := make([]float64, 0, len(nums)-k+1)
	res = append(res, dh.GetMedian())

	for i := k; i < len(nums); i++ {
		dh.Insert(nums[i])
		dh.Erase(nums[i-k])
		res = append(res, dh.GetMedian())
	}
	return res
}
```

### Code Efficiency

- **Time Complexity**: $\mathcal{O}(N \log k)$, where $N$ is the number of elements in `nums`.
    - Iterating through the array takes $\mathcal{O}(N)$ time.
    - Each `Insert` and `Erase` takes $\mathcal{O}(\log k)$ time.
    - Pruning operations amortize to $\mathcal{O}(\log k)$ per element since each element is pushed and popped at most once.
- **Space Complexity**: $\mathcal{O}(N)$
    - In the worst case (e.g., sorted inputs where deletions are always at the bottom of the heaps), lazily deleted elements remain in the heaps, so the heaps can grow up to $\mathcal{O}(N)$ size.
    - The `delayed` hash map stores at most $N - k$ elements.