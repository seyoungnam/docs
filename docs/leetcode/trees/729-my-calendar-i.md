# 729. My Calendar I

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/my-calendar-i/description/)

## Solution: Binary Search on Sorted Intervals

We can implement the calendar booking system by maintaining a sorted list of booked events and using **binary search** to locate the insertion index and check for overlaps.

### Thought Process

1.  **Define Overlaps**:
    *   Two events represented as half-open intervals $[s_1, e_1)$ and $[s_2, e_2)$ overlap if and only if:
        $$s_1 < e_2 \quad \text{and} \quad s_2 < e_1$$
2.  **Maintain Sorted Order**:
    *   Keep the booked events in a slice `events` sorted by their start times.
3.  **Binary Search Lookup**:
    *   When booking a new event $[startTime, endTime)$, use binary search (`sort.Search`) to find the first event at index `idx` whose start time is greater than or equal to `startTime`:
        $$\text{events}[idx][0] \ge \text{startTime}$$
4.  **Overlap Validation**:
    *   **Preceding Event**: Check the event just before index `idx` (i.e. `idx-1` if it exists). It overlaps if its end time extends past the new event's start time:
        $$\text{events}[idx-1][1] > \text{startTime}$$
    *   **Succeeding Event**: Check the event at index `idx` (if it exists). It overlaps if its start time begins before the new event's end time:
        $$\text{events}[idx][0] < \text{endTime}$$
5.  **Insertion**:
    *   If no overlap is detected, insert the new interval at index `idx` to maintain the sorted order, and return `true`. Otherwise, return `false`.

### Go Code

``` go
type MyCalendar struct {
    events [][2]int
}

func Constructor() MyCalendar {
    return MyCalendar{events: [][2]int{}}
}

func (this *MyCalendar) Book(startTime int, endTime int) bool {
    idx := sort.Search(len(this.events), func(i int) bool {
        return this.events[i][0] >= startTime
    })
    if idx > 0 && this.events[idx-1][1] > startTime {
        return false
    }
    if idx < len(this.events) && this.events[idx][0] < endTime {
        return false
    }
    this.events = append(this.events[:idx], append([][2]int{{startTime, endTime}}, this.events[idx:]...)...)
    return true
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$ per `Book` call
    - Searching for the insertion index via binary search takes $O(\log n)$ time.
    - Shifting elements to insert the new event into the slice takes $O(n)$ time in the worst case.
    - For $n$ bookings, the total time complexity is $O(n^2)$.
- **Space Complexity**: $O(n)$
    - We store up to $n$ booked events in the `events` slice.