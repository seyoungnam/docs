# 435. Non-overlapping Intervals

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/non-overlapping-intervals/description/)

Given an array of intervals `intervals` where `intervals[i] = [start_i, end_i]`, return *the minimum number of intervals you need to remove to make the rest of the intervals non-overlapping*.

---

## Solution: Greedy (Sorted by Start Time)

We want to minimize the number of deleted intervals. This is equivalent to maximizing the number of non-overlapping intervals we keep. We can solve this greedily by sorting the intervals by their start times and resolving conflicts by keeping the interval that ends earlier.

### Thought Process

1.  **Sorting**:
    *   Sort the intervals in ascending order by their start times.
2.  **Greedy Decisions**:
    *   Initialize `prevEnd := intervals[0][1]` to track the end time of the last scheduled interval.
    *   Initialize `count := 0` to count the number of intervals we remove.
    *   Iterate through the remaining intervals from index 1 to the end:
        *   **Case 1: Overlap (`start < prevEnd`)**:
            *   A conflict occurs. We must remove one of the overlapping intervals.
            *   To maximize the space available for future intervals, we should greedily keep the interval with the **smaller end time** (the one that ends earlier) and delete the other.
            *   Increment `count` (since we delete one interval).
            *   Update `prevEnd` to the minimum of the two end times:
                $$\text{prevEnd} = \min(\text{prevEnd}, \text{end})$$
        *   **Case 2: No Overlap (`start >= prevEnd`)**:
            *   No conflict occurs. We can safely keep the current interval.
            *   Update the end time marker: `prevEnd = end`.
3.  **Result**:
    *   Return `count`.

### Go Code

``` go
import "sort"

func eraseOverlapIntervals(intervals [][]int) int {
    if len(intervals) == 0 {
        return 0
    }
    
    // Sort intervals by start time
    sort.Slice(intervals, func(i, j int) bool {
        return intervals[i][0] < intervals[j][0]
    })
    
    prevEnd := intervals[0][1]
    var count int
    
    for i := 1; i < len(intervals); i++ {
        start, end := intervals[i][0], intervals[i][1]
        if start < prevEnd {
            // Overlap detected: remove the interval that ends later
            count++
            prevEnd = min(prevEnd, end)
        } else {
            // No overlap: accept the interval and update prevEnd
            prevEnd = end
        }
    }
    
    return count
}
```

### Code Efficiency

- **Time Complexity**: $O(N \log N)$
    - Where $N$ is the number of intervals. Sorting the array takes $O(N \log N)$ time, and the subsequent linear scan takes $O(N)$ time.
- **Space Complexity**: $O(1)$
    - We sort in-place and use constant auxiliary variables. (Note: Go's `sort.Slice` uses PDQSort, which requires $O(\log N)$ stack space for recursion).