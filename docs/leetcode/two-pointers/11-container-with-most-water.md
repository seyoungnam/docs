# 11. Container With Most Water

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/container-with-most-water/)

## Solution: Two Pointers

### Thought Process

1.  **Objective**: Find two lines that, together with the x-axis, form a container that holds the most water.
2.  **Initial State**: Start with the widest possible container by placing pointers at the beginning (`l`) and end (`r`) of the array.
3.  **Area Calculation**: The area is determined by the shorter of the two lines (the bottleneck) multiplied by the distance between them: `Area = min(height[l], height[r]) * (r - l)`.
4.  **Moving Pointers**: To potentially find a larger area, we must move one of the pointers.
    -   Moving the pointer pointing to the **taller** line will only decrease the width without any guarantee of increasing the height (the bottleneck remains the shorter line).
    -   Therefore, we move the pointer pointing to the **shorter** line in hopes of finding a taller line that compensates for the loss in width.
5.  **Iteration**: Repeat the process until the pointers meet, keeping track of the maximum area found.

### Go Code

``` go
func maxArea(height []int) int {
    res := 0
    l, r := 0, len(height)-1
    for l < r {
        h := min(height[l], height[r])
        res = max(res, (r-l)*h)
        if h == height[l] {
            l++
        } else {
            r--
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We perform a single pass through the array, moving the pointers toward each other.
- **Space Complexity**: $O(1)$
    - We only use a few variables (`l`, `r`, `res`, `h`) to store state, regardless of the input size.

---

## Optimized Solution: Fast Forwarding

### Thought Process

1.  **The Bottleneck Principle**: In the standard two-pointer approach, we always move the pointer pointing to the shorter line. We know that any subsequent line that is shorter than or equal to the current bottleneck (`h`) cannot possibly yield a larger area because the width (`r - l`) is already shrinking.
2.  **Skipping Irrelevant Lines**: After calculating the current area, instead of moving the pointer by just one step, we can "fast forward" it.
3.  **Implementation**:
    -   While moving the left pointer `l`, skip all lines where `height[l] <= h`.
    -   While moving the right pointer `r`, skip all lines where `height[r] <= h`.
4.  **Efficiency Gain**: This reduces the number of `max` and `area` calculations, which can lead to significant constant-time performance improvements on arrays with many small height fluctuations.

### Go Code

``` go
func maxArea(height []int) int {
    res := 0
    l, r := 0, len(height)-1
    for l < r {
        h := min(height[l], height[r])
        
        area := (r - l) * h
        res = max(res, area)
        
        // fast forward
        for l < r && height[l] <= h {
            l++
        }
        for l < r && height[r] <= h {
            r--
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - Although there are nested loops, each pointer (`l` and `r`) still only moves from one end to the other once. Every element is visited at most once.
- **Space Complexity**: $O(1)$
    - No additional data structures are used; space usage is constant.
