# 56. Merge Intervals

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/merge-intervals/description/)

## Solution: Sort by Start Time

We can merge overlapping intervals by sorting them based on their starting times. Once sorted, we can iterate through the intervals and merge any interval that overlaps with the previously processed one.

### Thought Process

1.  **Sort**: Sort the intervals in ascending order based on their starting values.
2.  **Initialize**: Push the first interval onto our result slice `res`.
3.  **Traverse and Merge**: Iterate through the remaining intervals:
    *   Compare the current interval's start (`currStart`) with the end of the last interval in `res` (`lastEnd`).
    *   If `currStart <= lastEnd`, there is an overlap. Merge them by updating the end of the last interval in `res` to `max(lastEnd, currEnd)`.
    *   Otherwise, there is no overlap. Append the current interval to `res` as a new entry.

### Go Code

``` go
import "sort"

func merge(intervals [][]int) [][]int {
    if len(intervals) <= 1 {
        return intervals
    }
    
    // Sort intervals by starting value
    sort.Slice(intervals, func(i, j int) bool {
        return intervals[i][0] < intervals[j][0]
    })
    
    var res [][]int
    res = append(res, intervals[0])
    
    for i := 1; i < len(intervals); i++ {
        last := res[len(res)-1]
        lastEnd := last[1]
        
        curr := intervals[i]
        currStart, currEnd := curr[0], curr[1]
        
        // If overlap, merge by updating the end time
        if currStart <= lastEnd {
            res[len(res)-1][1] = max(lastEnd, currEnd)
        } else {
            res = append(res, curr)
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n \log n)$
    - Sorting the intervals of size $n$ takes $O(n \log n)$ time. The subsequent linear scan takes $O(n)$ time.
- **Space Complexity**: $O(n)$ or $O(\log n)$
    - Auxiliary space is $O(\log n)$ for Go's sort implementation if sorting in-place. The space of the output slice is $O(n)$.

---

## Alternative Solution: Independent Starts and Ends Sorting

Instead of sorting the intervals together as 2D slices, we can extract and sort the starting times and ending times independently. This is extremely efficient because it operates on flat, cache-friendly 1D integer slices.

### Thought Process

1.  **Separate Coordinates**: Extract all start coordinates into a `starts` array and all end coordinates into an `ends` array.
2.  **Sort Separately**: Sort both 1D arrays independently in ascending order.
3.  **Two-Pointer Scan**: Iterate through the sorted coordinates. 
    *   A boundary between merged intervals is detected whenever the start of the next interval is strictly greater than the end of the current one: `starts[i+1] > ends[i]`.
    *   When a boundary is found, append the merged interval `[starts[j], ends[i]]` to `res` and update `j = i+1` to mark the start of the next component.

### Go Code

``` go
import "sort"

func merge(intervals [][]int) [][]int {
    n := len(intervals)
    if n <= 1 {
        return intervals
    }
    
    starts := make([]int, n)
    ends := make([]int, n)
    for i, interval := range intervals {
        starts[i] = interval[0]
        ends[i] = interval[1]
    }
    
    // Sort flat 1D arrays (improves cache locality)
    sort.Ints(starts)
    sort.Ints(ends)
    
    var res [][]int
    j := 0 // Tracks the start index of the current merged component
    
    for i := 0; i < n; i++ {
        // If we reach the last interval or find a gap
        if i == n-1 || starts[i+1] > ends[i] {
            res = append(res, []int{starts[j], ends[i]})
            j = i + 1 // Advance start index to the next component
        }
    }
    
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n \log n)$
    - Sorting the two flat slices takes $O(n \log n)$ time. The scan takes a single $O(n)$ pass.
- **Space Complexity**: $O(n)$
    - We allocate two 1D slices (`starts` and `ends`) of size $n$.
