# 252. Meeting Rooms

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/meeting-rooms/description/)

Given an array of meeting time `intervals` where `intervals[i] = [start_i, end_i]`, determine if a person could attend all meetings.

---

## Solution: Sorting by Start Time

To attend all meetings, a person cannot have any overlapping meetings in their schedule. By sorting the meetings by their start times, we can detect overlaps by checking if any meeting starts before the previous one ends.

### Thought Process

1.  **Sorting**:
    *   Sort the meetings in ascending order by their start times.
2.  **Overlap Check**:
    *   Iterate through the sorted meetings from index 1 to the end.
    *   Compare the current meeting's start time against the previous meeting's end time:
        *   If the previous meeting ends *after* the current meeting starts:
            $$\text{intervals[i-1][1]} > \text{intervals[i][0]}$$
            an overlap exists. Return `false` immediately.
3.  **Result**:
    *   If no overlaps are detected after checking all adjacent pairs, return `true`.

### Go Code

``` go
import "sort"

func canAttendMeetings(intervals [][]int) bool {
    // Sort meetings by start time
    sort.Slice(intervals, func(i, j int) bool {
        return intervals[i][0] < intervals[j][0]
    })
    
    // Check for overlapping meetings
    for i := 1; i < len(intervals); i++ {
        if intervals[i-1][1] > intervals[i][0] {
            return false
        }
    }
    
    return true
}
```

### Code Efficiency

- **Time Complexity**: $O(N \log N)$
    - Where $N$ is the number of meetings. Sorting the array takes $O(N \log N)$ time, and checking adjacent intervals for overlaps takes $O(N)$ time.
- **Space Complexity**: $O(1)$
    - We sort in-place and use constant auxiliary space. (Note: Go's `sort.Slice` uses PDQSort, which requires $O(\log N)$ stack space for recursion).