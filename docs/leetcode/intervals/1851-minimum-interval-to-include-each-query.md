# 1851. Minimum Interval to Include Each Query

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/minimum-interval-to-include-each-query/description/)

You are given a 2D integer array `intervals`, where `intervals[i] = [left_i, right_i]` describes the $i$-th interval starting at `left_i` and ending at `right_i` (inclusive). The **size** of an interval is defined as the number of integers it contains, or more formally `right_i - left_i + 1`.

You are also given an integer array `queries`. The answer to the $j$-th query is the **size of the smallest interval** $i$ such that $left_i \le queries[j] \le right_i$. If no such interval exists, the answer is `-1`.

Return *an array containing the answers to the queries*.

---

## Solution: Offline Queries with Min-Heap

A naive search for each query would take $O(N \cdot Q)$ time, which is too slow. Instead, we can process the queries **offline** by sorting them. This allows us to dynamically track active intervals using a min-heap in a single, monotonic sweep.

### Thought Process

1.  **Offline Processing**:
    *   Sort the `intervals` by their start times: `intervals[i][0]`.
    *   Sort the `queries` while preserving their original indices. This allows us to reconstruct the result in the correct order.
2.  **Min-Heap Structure**:
    *   Use a min-heap to keep track of active intervals.
    *   Each element in the heap is a pair `[size, rightBoundary]`.
    *   The heap is sorted by `size` (the first element of the pair) to ensure we can retrieve the smallest interval in $O(1)$ time.
3.  **Monotonic Sweep**:
    *   Iterate through the sorted queries. For each query `q` (and its `originalIdx`):
        *   **Add Candidates**: Push all intervals that start before or at `q` (`intervals[i][0] <= q`) onto the min-heap.
        *   **Remove Stale Intervals**: Pop any intervals from the top of the heap that end before `q` (`rightBoundary < q`). Because queries are processed in ascending order, any interval that ends before `q` will also end before all future queries and can be permanently discarded.
        *   **Answer Query**: If the heap is not empty, the top of the heap is the smallest interval covering `q`. Record its size: `res[originalIdx] = heap[0].size`. If empty, record `-1`.

### Go Code

``` go
import (
    "container/heap"
    "sort"
)

type minHeap [][2]int

func (h minHeap) Len() int           { return len(h) }
func (h minHeap) Less(i, j int) bool { return h[i][0] < h[j][0] }
func (h minHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *minHeap) Push(x interface{}) { *h = append(*h, x.([2]int)) }
func (h *minHeap) Pop() interface{} {
    last := (*h)[len(*h)-1]
    *h = (*h)[:len(*h)-1]
    return last
}

func minInterval(intervals [][]int, queries []int) []int {
    // Sort intervals by start time
    sort.Slice(intervals, func(i, j int) bool {
        return intervals[i][0] < intervals[j][0]
    })

    // Store queries with their original indices and sort them
    queriesWithIdx := make([][2]int, len(queries))
    for i, q := range queries {
        queriesWithIdx[i] = [2]int{q, i}
    } 
    sort.Slice(queriesWithIdx, func(i, j int) bool {
        return queriesWithIdx[i][0] < queriesWithIdx[j][0]
    })

    h := &minHeap{}
    heap.Init(h)

    res := make([]int, len(queries))
    i := 0

    // Monotonic sweep
    for _, qPair := range queriesWithIdx {
        q, originalIdx := qPair[0], qPair[1]

        // Push all intervals starting <= q to the heap
        for i < len(intervals) && intervals[i][0] <= q {
            size := intervals[i][1] - intervals[i][0] + 1
            heap.Push(h, [2]int{size, intervals[i][1]})
            i++
        }

        // Pop all stale intervals ending < q from the top of the heap
        for h.Len() > 0 && (*h)[0][1] < q {
            heap.Pop(h)
        }

        // Record the smallest active interval size
        if h.Len() > 0 {
            res[originalIdx] = (*h)[0][0]
        } else {
            res[originalIdx] = -1
        }
    }
    
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N \log N + Q \log Q)$
    - Where $N$ is the number of intervals and $Q$ is the number of queries. Sorting the intervals takes $O(N \log N)$ and sorting queries takes $O(Q \log Q)$. Since each interval is pushed and popped from the heap at most once, the total heap operation cost is bounded by $O(N \log N)$.
- **Space Complexity**: $O(N + Q)$
    - We use $O(Q)$ space to store the indexed queries, and the min-heap can grow to store at most $O(N)$ active intervals.