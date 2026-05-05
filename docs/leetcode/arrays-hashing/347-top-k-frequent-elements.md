# 347. Top K Frequent Elements

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/top-k-frequent-elements/description/)

## Solution 1: Bucket Sort

### Thought Process

1.  **Frequency Count**: Use a hash map (`counter`) to count the frequency of each number in the input array.
2.  **Bucket by Frequency**: Create a list of buckets where the index represents the frequency of elements. The maximum possible frequency is `len(nums)`, so we initialize `buckets` with size `len(nums) + 1`.
3.  **Linear Time Sorting**: Instead of sorting based on frequency (which would take $O(N \log N)$), we iterate through the `counter` map and place each number into the bucket corresponding to its frequency.
4.  **Result Extraction**: Iterate through the `buckets` array in reverse order (from highest frequency to lowest). Append the elements of each non-empty bucket to the result list until we have collected `k` elements.

### Go Code

``` go
func topKFrequent(nums []int, k int) []int {
    counter := map[int]int{}
    for _, num := range nums {
        counter[num]++
    }

    buckets := make([][]int, len(nums)+1)
    for num, freq := range counter {
        buckets[freq] = append(buckets[freq], num)
    }

    res := make([]int, 0, k)
    for i := len(buckets)-1; i >= 0 && len(res) < k; i-- {
        if len(buckets[i]) > 0 {
            res = append(res, buckets[i]...)
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Building the frequency map takes $O(N)$.
    - Distributing elements into buckets takes $O(N)$ (iterating over the map).
    - Collecting the top $k$ elements takes $O(N)$ in the worst case (iterating over the buckets array).
- **Space Complexity**: $O(N)$
    - The frequency map and the buckets array both store at most $N$ distinct elements.

---

## Solution 2: Heap

### Thought Process

1.  **Frequency Count**: Use a hash map (`counter`) to count the frequency of each number.
2.  **Min-Heap of Size k**:
    -   Maintain a min-heap that stores elements based on their frequency.
    -   As we iterate through the `counter` map, we push each element into the heap.
    -   If the heap size exceeds `k`, we pop the element with the minimum frequency.
    -   By the end of the iteration, the heap will contain the `k` elements with the highest frequencies.
3.  **Heap Implementation in Go**:
    -   Implement the `heap.Interface` for a custom struct (e.g., a slice of `[2]int` where the second element is the frequency).
    -   Use `container/heap` to manage the heap operations (`Push`, `Pop`, `Init`).
4.  **Result Construction**: Pop all elements from the heap and add their keys (the numbers) to the result slice.

### Go Code

``` go
import "container/heap"

type minHeap [][2]int

func (h minHeap) Len() int { return len(h) }
func (h minHeap) Less(i, j int) bool { return h[i][1] < h[j][1] }
func (h minHeap) Swap(i, j int) { h[i], h[j] = h[j], h[i] }
func (h *minHeap) Push(x interface{}) { *h = append(*h, x.([2]int)) }
func (h *minHeap) Pop() interface{} {
    last := (*h)[len(*h)-1]
    *h = (*h)[:len(*h)-1]
    return last
}

func topKFrequent(nums []int, k int) []int {
    counter := map[int]int{}
    for _, v := range nums {
        counter[v]++
    }
    h := &minHeap{}
    heap.Init(h)
    for key, val := range counter {
        entry := [2]int{key, val}
        if h.Len() < k || (*h)[0][1] < val {
            heap.Push(h, entry)
        }
        if h.Len() > k {
            heap.Pop(h)
        }
    }
    res := []int{}
    for h.Len() > 0 {
        ele := heap.Pop(h).([2]int)[0]
        res = append(res, ele)
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N \log k)$
    - Counting frequencies takes $O(N)$.
    - Iterating through the map and performing heap operations takes $O(D \log k)$, where $D$ is the number of distinct elements ($D \le N$).
- **Space Complexity**: $O(N + k)$
    - The hash map stores $D$ elements ($O(N)$).
    - The heap stores at most $k$ elements ($O(k)$).
