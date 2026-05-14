# 253. Meeting Rooms II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/meeting-rooms-ii/description/)

## Solution 1: Min Heap

### Thought Process

1.  **Ordering**: To process meetings in the order they occur, we first sort the intervals by their **start time**.
2.  **Room Tracking**: We need a way to efficiently track when rooms become available. A **Min-Heap** is ideal for this because it allows us to quickly find the room that will be vacated the soonest (the one with the smallest end time).
3.  **Allocation Logic**:
    - Initialize the heap with the end time of the first meeting.
    - For every subsequent meeting:
        - Check the top of the heap (the earliest end time of all occupied rooms).
        - If the earliest room is free before the current meeting starts (`heap[0] <= current.start`), we can **reuse** that room. We `Pop` the old end time and `Push` the current meeting's end time.
        - If the earliest room is still occupied, we must **allocate a new room**. We simply `Push` the current meeting's end time into the heap.
4.  **Result**: The total number of rooms required is the final size of the heap.

### Go Code

``` go
import (
    "container/heap"
    "sort"
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

func minMeetingRooms(intervals [][]int) int {
    if len(intervals) == 0 {
        return 0
    }

    sort.Slice(intervals, func(i, j int) bool {
        return intervals[i][0] < intervals[j][0]
    })

    h := &minHeap{}
    heap.Init(h)
    heap.Push(h, intervals[0][1])

    for i := 1; i < len(intervals); i++ {
        curr := intervals[i]
        // If the earliest room is free, reuse it
        if (*h)[0] <= curr[0] {
			heap.Pop(h)
		}
        // Always push the current meeting's end time
        heap.Push(h, curr[1])
    }
    return h.Len()
}
```

### Code Efficiency

- **Time Complexity**: $O(N \log N)$
    - Sorting the intervals takes $O(N \log N)$.
    - We iterate through $N$ intervals, performing a `Push` (and potentially a `Pop`) in each iteration. Each heap operation takes $O(\log N)$.
- **Space Complexity**: $O(N)$
    - In the worst case (all meetings overlap), the heap will store $N$ end times.

---

## Solution 2: Chronological Event Sorting (Two Pointers)

The core idea is to treat meeting starts and meeting ends as independent events in time. By sorting these events, we can determine the maximum number of overlapping meetings at any point, which corresponds to the minimum number of rooms needed.

### Thought Process

1.  **Event Separation**: A meeting room is occupied from a `start` time and released at an `end` time. We can collect all `start` times and all `end` times into two separate arrays.
2.  **Sorting**: Sort both the `starts` and `ends` arrays independently. This allows us to process events in chronological order.
3.  **Two-Pointer Strategy**:
    - Use two pointers: `startIdx` for the next meeting starting and `endIdx` for the earliest meeting ending.
    - Iterate through all meetings using `startIdx`.
    - **Room Reuse**: If the current meeting starts *after or at the same time* as the meeting at `endIdx` finishes (`starts[startIdx] >= ends[endIdx]`), we can reuse that room. We increment `endIdx` to track the next earliest finish time.
    - **Room Allocation**: If the current meeting starts *before* the earliest finish time, it means all currently active rooms are occupied. We increment the `rooms` count.
4.  **Final Count**: The `rooms` variable tracks the peak number of concurrent meetings.

### Go Code

``` go
import "sort"

func minMeetingRooms(intervals [][]int) int {
    n := len(intervals)
    if n == 0 {
        return 0
    }
    
    starts := make([]int, n)
    ends := make([]int, n)
    for i := 0; i < n; i++ {
        starts[i] = intervals[i][0]
        ends[i] = intervals[i][1]
    }

    sort.Ints(starts)
    sort.Ints(ends)

    endIdx, rooms := 0, 0
    for startIdx := 0; startIdx < n; startIdx++ {
        if starts[startIdx] < ends[endIdx] {
            // No room is free, need a new one
            rooms++
        } else {
            // A room became free, reuse it and move endIdx
            endIdx++
        }
    }
    return rooms
}
```

### Code Efficiency

- **Time Complexity**: $O(N \log N)$
    - Sorting the `starts` and `ends` arrays takes $O(N \log N)$ time.
    - The two-pointer traversal takes $O(N)$ time.
- **Space Complexity**: $O(N)$
    - We use two auxiliary arrays (`starts` and `ends`) each of size $N$ to store the split intervals.
