# 57. Insert Interval

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/insert-interval/description/)

You are given an array of non-overlapping intervals `intervals` where `intervals[i] = [start_i, end_i]` represent the start and the end of the $i$-th interval and `intervals` is sorted in ascending order by `start_i`. You are also given an interval `newInterval = [start, end]` that represents the start and end of another interval.

Insert `newInterval` into `intervals` such that `intervals` is still sorted in ascending order by `start_i` and `intervals` still does not have any overlapping intervals (merge overlapping intervals if necessary).

Return `intervals` after the insertion.

---

## Solution: Linear Scan (Three-Stage Merge)

Since the input `intervals` is already sorted by start times, we can insert and merge the new interval in a single linear pass ($O(N)$ time). 

### Thought Process

We can divide the insertion process into three distinct stages:

1.  **Stage 1: Pre-Overlapping Intervals**:
    *   Any interval that ends before `newInterval` starts (i.e., `intervals[i][1] < newInterval[0]`) is completely to the left of the new interval and has no overlap.
    *   Append these intervals directly to the result slice `res`.
2.  **Stage 2: Overlapping & Merging**:
    *   An interval overlaps with `newInterval` if its start time is less than or equal to the end time of `newInterval` (i.e., `intervals[i][0] <= newInterval[1]`).
    *   Merge the overlapping interval into `newInterval` by updating the boundaries:
        $$\text{newInterval[0]} = \min(\text{newInterval[0]}, \text{intervals[i][0]})$$
        $$\text{newInterval[1]} = \max(\text{newInterval[1]}, \text{intervals[i][1]})$$
    *   Continue this loop until no more overlapping intervals are found, then append the merged `newInterval` to `res`.
3.  **Stage 3: Post-Overlapping Intervals**:
    *   Any remaining intervals start after `newInterval` ends. They are completely to the right of the merged interval and have no overlap.
    *   Append the remaining intervals directly to `res`.

### Go Code

``` go
func insert(intervals [][]int, newInterval []int) [][]int {
    n := len(intervals)
    i := 0
    var res [][]int

    // Stage 1: Add all intervals ending before newInterval starts
    for i < n && intervals[i][1] < newInterval[0] {
        res = append(res, intervals[i])
        i++
    }

    // Stage 2: Merge all overlapping intervals
    for i < n && intervals[i][0] <= newInterval[1] {
        newInterval[0] = min(newInterval[0], intervals[i][0])
        newInterval[1] = max(newInterval[1], intervals[i][1])
        i++
    }
    res = append(res, newInterval)

    // Stage 3: Add all remaining intervals
    for i < n {
        res = append(res, intervals[i])
        i++
    }
    
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Where $N$ is the number of intervals in the input list. We iterate through the list exactly once.
- **Space Complexity**: $O(N)$
    - We allocate an output slice `res` to store the result. If we do not count the space used for the output, the auxiliary space complexity is $O(1)$.