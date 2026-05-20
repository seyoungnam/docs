# 239. Sliding Window Maximum

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/sliding-window-maximum/description/)

## Solution: Max-Heap (Priority Queue)

To find the maximum in a sliding window, we can use a Max-Heap to store elements along with their indices. The heap allows us to efficiently retrieve the largest value, while the indices help us identify and remove elements that have "slid" out of the current window.

### Thought Process

1.  **Heap Storage**: Store pairs of `[value, index]` in a Max-Heap. The value determines priority, while the index tracks position.
2.  **Sliding Window**: Iterate through the array. In each step, push the current element into the heap.
3.  **Stale Element Removal**: Before extracting the maximum, check the top of the heap. If the index of the maximum element is no longer within the current window boundary (`index < current_index - k + 1`), pop it.
4.  **Recording Maximums**: Once the first window is full (index $\ge k-1$), the top of the heap is the maximum for that window.

### Go Code

``` go
import (
    "container/heap"
)

type maxHeap [][2]int

func (h maxHeap) Len() int { return len(h) }
func (h maxHeap) Less(i, j int) bool { return h[i][0] > h[j][0] }
func (h maxHeap) Swap(i, j int) { h[i], h[j] = h[j], h[i] }
func (h *maxHeap) Push(x interface{}) { *h = append(*h, x.([2]int)) }
func (h *maxHeap) Pop() interface{} {
    last := (*h)[len(*h)-1]
    *h = (*h)[:len(*h)-1]
    return last
}

func maxSlidingWindow(nums []int, k int) []int {
    n := len(nums)
    res := make([]int, 0)
    h := &maxHeap{}
    heap.Init(h)
    for i := 0; i < n; i++ {
        ele := [2]int{nums[i], i}
        heap.Push(h, ele)
        
        if i < k-1 {
            continue
        }
        
        // Remove the maximum if it is outside the current window
        for (*h)[0][1] < i-k+1 {
            heap.Pop(h)
        }
        
        res = append(res, (*h)[0][0])
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n \log n)$
    - Each of the $n$ elements is pushed into the heap once ($O(\log n)$ per push). In the worst case, the heap size can grow up to $n$. Popping outdated elements also takes $O(\log n)$.
- **Space Complexity**: $O(n)$
    - The heap stores at most $n$ elements in the worst case (e.g., a strictly increasing input array).

---

## Optimized Solution: Monotonic Deque

### Thought Process

1.  **Decreasing Deque**: Maintain a deque of indices such that the corresponding values in `nums` are in strictly decreasing order.
2.  **Remove Stale**: If the index at the front of the deque is outside the current window (`deque[0] < i-k+1`), remove it.
3.  **Maintain Monotonicity**: Before adding current index `i`, remove all indices from the back whose values are $\le$ `nums[i]`, as they can no longer be the maximum.
4.  **Extract Maximum**: Once the window is full, the front of the deque (`deque[0]`) always contains the index of the largest element.

### Go Code

``` go
func maxSlidingWindow(nums []int, k int) []int {
    if len(nums) == 0 {
        return nil
    }
    var res, deque []int
    for i := 0; i < len(nums); i++ {
        // Remove indices outside the window
        if len(deque) > 0 && deque[0] < i-k+1 {
            deque = deque[1:]
        }
        // Maintain decreasing order: remove smaller elements from back
        for len(deque) > 0 && nums[deque[len(deque)-1]] <= nums[i] {
            deque = deque[:len(deque)-1]
        }
        deque = append(deque, i)
        
        if i >= k-1 {
            res = append(res, nums[deque[0]])
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - Each element is pushed into and popped from the deque at most once.
- **Space Complexity**: $O(k)$
    - The deque stores at most $k$ indices (the size of the window).
