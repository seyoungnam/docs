# 853. Car Fleet

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/car-fleet/description/)

## Solution: Monotonic Stack

By sorting cars by their starting position in descending order, we can calculate each car's arrival time. A car behind catches up to the car in front if it takes less or equal time, forming a fleet. A stack helps track these fleet times.

### Thought Process

1.  **Pair and Sort**: Group positions with speeds and sort them by position in descending order (closest to target first).
2.  **Calculate Time**: Compute the time to reach the target for each car: `(target - position) / speed`.
3.  **Compare Times**: If the current car takes longer than the fleet in front, it becomes a new fleet leader. Push its time onto the stack.
4.  **Result**: The size of the stack is the total number of fleets.

### Go Code

``` go
func carFleet(target int, position []int, speed []int) int {
    n := len(position)
    if n == 0 {
        return 0
    }
    
    pair := make([][2]int, n)
    for i := 0; i < n; i++ {
        pair[i] = [2]int{position[i], speed[i]}
    }
    
    sort.Slice(pair, func(i, j int) bool {
        return pair[i][0] > pair[j][0]
    })
    
    stack := make([]float64, 0)
    for i := 0; i < n; i++ {
        seconds := float64(target-pair[i][0]) / float64(pair[i][1])
        if len(stack) == 0 || stack[len(stack)-1] < seconds {
            stack = append(stack, seconds)
        }
    }
    return len(stack)
}
```

### Code Efficiency

- **Time Complexity**: $O(n \log n)$
    - Sorting the cars takes $O(n \log n)$ time.
- **Space Complexity**: $O(n)$
    - We store the car pairs and the monotonic stack.

---

## Optimized Solution: Space-Optimized Traversal

Instead of using a stack, we only need to compare the current car's time with the slowest time of the fleet leader directly in front of it.

### Thought Process

1.  **Pair and Sort**: Group positions with speeds and sort in descending order of starting positions.
2.  **Track Bottleneck**: Keep a `maxTime` variable for the slowest fleet arrival time seen so far, and a `fleets` counter.
3.  **Count Fleets**: If a car takes longer than `maxTime`, it forms a new fleet. Increment `fleets` and update `maxTime`.

### Go Code

``` go
func carFleet(target int, position []int, speed []int) int {
    n := len(position)
    if n == 0 {
        return 0
    }
    
    pair := make([][2]int, n)
    for i := 0; i < n; i++ {
        pair[i] = [2]int{position[i], speed[i]}
    }
    
    sort.Slice(pair, func(i, j int) bool {
        return pair[i][0] > pair[j][0]
    })
    
    fleets := 0
    var maxTime float64
    for i := 0; i < n; i++ {
        seconds := float64(target-pair[i][0]) / float64(pair[i][1])
        if seconds > maxTime {
            maxTime = seconds
            fleets++
        }
    }
    return fleets
}
```

### Code Efficiency

- **Time Complexity**: $O(n \log n)$
    - Sorting the cars takes $O(n \log n)$ time.
- **Space Complexity**: $O(n)$
    - We use $O(n)$ space for the pairs. Auxiliary space for the traversal is $O(1)$.
