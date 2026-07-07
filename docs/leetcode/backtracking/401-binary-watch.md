# 401. Binary Watch

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/binary-watch/description/)

## Solution: Backtracking (DFS Closure)

A binary watch has 4 LEDs for hours and 6 LEDs for minutes. Given the number of active LEDs `turnedOn`, we can search all possible combinations using recursive depth-first backtracking, pruning invalid time representations early.

### Thought Process

1.  **Grid Layout Representation**:
    *   Map the 10 LEDs to a single slice `leds` of size 10:
        *   **Hours LEDs** (indices `0` to `3`): `1`, `2`, `4`, `8`.
        *   **Minutes LEDs** (indices `4` to `9`): `1`, `2`, `4`, `8`, `16`, `32`.
2.  **Backtracking DFS Closure**:
    *   Define a recursive function `dfs(i, count, h, m)`:
        *   `i`: Current LED index.
        *   `count`: Running total of active LEDs.
        *   `h`: Current hour sum.
        *   `m`: Current minute sum.
3.  **Validation & Pruning**:
    *   **Invalid Time bounds**: Android hours cannot exceed $11$ (`h > 11`), and minutes cannot exceed $59$ (`m > 59`). If these bounds are violated, return immediately.
4.  **Recursive Decisions**:
    *   At index `i`, we have two paths:
        *   **Branch 1 (LED On)**:
            *   If `i < 4` (hours LED), add `leds[i]` to `h`: `dfs(i+1, count+1, h+leds[i], m)`.
            *   If `i >= 4` (minutes LED), add `leds[i]` to `m`: `dfs(i+1, count+1, h, m+leds[i])`.
        *   **Branch 2 (LED Off)**: Keep the LED off and move to the next LED index: `dfs(i+1, count, h, m)`.
5.  **Base Cases**:
    *   If `count == turnedOn`, we have matched the target count: format the time using `fmt.Sprintf("%d:%02d", h, m)`, append it to the results list `res`, and return.
    *   If `i >= len(leds)` (exceeded LED capacity), return.

### Go Code

``` go
import "fmt"

func readBinaryWatch(turnedOn int) []string {
    res := make([]string, 0)
    leds := []int{1, 2, 4, 8, 1, 2, 4, 8, 16, 32}

    var dfs func(i int, count int, h int, m int)
    dfs = func(i int, count int, h int, m int) {
        if h > 11 || m > 59 {
            return
        }
        if count == turnedOn {
            res = append(res, fmt.Sprintf("%d:%02d", h, m))
            return
        }

        if i >= len(leds) {
            return
        }

        // Branch 1: Turn LED i on
        if i < 4 {
            dfs(i+1, count+1, h+leds[i], m)
        } else {
            dfs(i+1, count+1, h, m+leds[i])
        }
        
        // Branch 2: Keep LED i off
        dfs(i+1, count, h, m)
    }
    dfs(0, 0, 0, 0)
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(1)$
    - Since the total number of LEDs is constant (10), the recursion tree has at most $2^{10} = 1024$ nodes. The total combinations are fixed, meaning the algorithm executes in constant time.
- **Space Complexity**: $O(1)$ auxiliary space. The recursion call stack depth goes up to a maximum of 10. (The output slice `res` holds at most 1024 formatted strings).