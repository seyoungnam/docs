# 84. Largest Rectangle in Histogram

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/largest-rectangle-in-histogram/description/)

## Solution: Monotonic Stack

We can use a monotonic increasing stack to store the indices of the heights. When we encounter a height smaller than the bar at the stack's top, we calculate the area of rectangles with the popped bar as the shortest bar (height).

### Thought Process

1.  **Initialize Stack**: Start with a stack containing `-1` to handle the width calculation for the first element.
2.  **Append Sentinel**: Append a height of `0` to the end of the `heights` array to ensure all remaining bars are popped from the stack at the end.
3.  **Monotonic Stack Loop**: Iterate through the heights. If the current height is smaller than the height at the top of the stack:
    *   Pop the top index. This index represents the height of the rectangle.
    *   The width is the distance between the current index `i` and the new top of the stack minus one: `i - stack[top] - 1`.
    *   Calculate the area and update the maximum.
4.  **Push Index**: Push the current index onto the stack.

### Go Code

``` go
func largestRectangleArea(heights []int) int {
    res := 0
    stack := []int{-1}
    heights = append(heights, 0)
    for i, h := range heights {
        for len(stack) > 1 && h < heights[stack[len(stack)-1]] {
            idx := stack[len(stack)-1]
            stack = stack[:len(stack)-1]

            height := heights[idx]
            width := i - stack[len(stack)-1] - 1
            res = max(res, height*width)
        }
        stack = append(stack, i)
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - Each index is pushed and popped from the stack at most once.
- **Space Complexity**: $O(n)$
    - We use a stack that can store up to $n$ elements.

---

## Alternative Solution: Divide and Conquer

We can find the minimum height in the current range, calculate the area using this minimum height as the bottleneck, and recursively calculate the maximum areas in the left and right sub-arrays.

### Thought Process

1.  **Find Minimum**: Find the index of the minimum height in the current range `[start, end]`.
2.  **Calculate Area**: The potential area using this minimum height is `heights[minIdx] * (end - start + 1)`.
3.  **Divide and Conquer**: Recursively calculate the maximum areas of the left subarray `[start, minIdx - 1]` and the right subarray `[minIdx + 1, end]`.
4.  **Result**: Return the maximum of the three calculated areas.

### Go Code

``` go
func largestRectangleArea(heights []int) int {
    return calculateArea(heights, 0, len(heights)-1)
}

func calculateArea(heights []int, start, end int) int {
    if start > end {
        return 0
    }
    minIdx := start
    for i := start; i <= end; i++ {
        if heights[i] < heights[minIdx] {
            minIdx = i
        }
    }
    leftMax := calculateArea(heights, start, minIdx-1)
    rightMax := calculateArea(heights, minIdx+1, end)
    
    return max(heights[minIdx]*(end-start+1), max(leftMax, rightMax))
}
```

### Code Efficiency

- **Time Complexity**: $O(n \log n)$
    - On average, the recursion tree height is $O(\log n)$, leading to $O(n \log n)$ time. In the worst case (e.g., sorted array), it is $O(n^2)$ unless optimized with a Segment Tree.
- **Space Complexity**: $O(n)$
    - The recursion stack can take up to $O(n)$ space in the worst case.
