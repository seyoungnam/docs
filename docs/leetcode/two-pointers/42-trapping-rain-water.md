# 42. Trapping Rain Water

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/trapping-rain-water/description/)

## Solution: Pre-calculating Prefix/Suffix Maximums

### Thought Process

1.  **Core Principle**: The amount of water trapped above any bar at index `i` is determined by the minimum of the maximum heights to its left and right, minus the height of the bar itself: `water[i] = max(0, min(max_left, max_right) - height[i])`.
2.  **Efficiency Goal**: Instead of searching for the left and right maximums for every bar (which would be $O(n^2)$), we can pre-calculate these values in linear time.
3.  **Prefix Max (`leftMax`)**: Create an array where `leftMax[i]` stores the maximum height encountered from index `0` to `i`.
4.  **Suffix Max (`rightMax`)**: Create an array where `rightMax[i]` stores the maximum height encountered from index `n-1` down to `i`.
5.  **Calculation**: Iterate through the bars (typically from index 1 to n-2) and calculate the trapped water using the pre-calculated maximums of the neighbors.
6.  **Summation**: Accumulate the trapped water at each position to get the final result.

### Go Code

``` go
func trap(height []int) int {
    res := 0
    n := len(height)
    if n == 0 {
        return 0
    }
    
    leftMax, rightMax := make([]int, n), make([]int, n)
    leftMax[0], rightMax[n-1] = height[0], height[n-1]

    for i := 1; i < n; i++ {
        leftMax[i] = max(leftMax[i-1], height[i])
    }
    for i := n-2; i >= 0; i-- {
        rightMax[i] = max(rightMax[i+1], height[i])
    }

    for i := 1; i < n-1; i++ {
        h := max(0, min(leftMax[i-1], rightMax[i+1]) - height[i])
        res += h
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We iterate through the array three times: once for `leftMax`, once for `rightMax`, and once for the final sum.
- **Space Complexity**: $O(n)$
    - We use two auxiliary arrays of size $n$ to store the prefix and suffix maximums.

---
## Optimized Solution: Two Pointers (On-the-fly Max)

### Thought Process

1.  **Eliminating Space**: The previous approach required $O(n)$ extra space to store all prefix and suffix maximums. We can optimize this to $O(1)$ by using two pointers moving toward each other.
2.  **Tracking Dynamic Maximums**: Use two variables, `leftMax` and `rightMax`, to track the highest bars seen so far from the left and right ends.
3.  **The Bottleneck Principle**: At any step, the amount of water that can be trapped depends on the **shorter** of the two dynamic maximums.
    -   If `leftMax < rightMax`, we know that the bottleneck is on the left. We move the `l` pointer forward, update `leftMax`, and add the difference (`leftMax - height[l]`) to our result.
    -   If `rightMax <= leftMax`, the bottleneck is on the right. We move the `r` pointer backward, update `rightMax`, and add the difference (`rightMax - height[r]`) to our result.
4.  **Single Pass**: By moving the pointer with the smaller maximum, we ensure that we always have a tall enough bar on the opposite side to trap the water we calculate.

### Go Code

``` go
func trap(height []int) int {
    if len(height) == 0 {
        return 0
    }

    res := 0
    l, r := 0, len(height)-1
    leftMax, rightMax := height[l], height[r]

    for l < r {
        if leftMax < rightMax {
            l++
            leftMax = max(leftMax, height[l])
            if height[l] < leftMax {
                res += leftMax - height[l]
            }
        } else {
            r--
            rightMax = max(rightMax, height[r])
            if height[r] < rightMax {
                res += rightMax - height[r]
            }
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We perform a single pass through the array, processing each element exactly once.
- **Space Complexity**: $O(1)$
    - We only use a constant amount of extra space for pointers and maximum height variables.
