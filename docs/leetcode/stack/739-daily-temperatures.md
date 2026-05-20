# 739. Daily Temperatures

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/daily-temperatures/description/)

## Solution: Monotonic Index Stack

To find the number of days until a warmer temperature occurs, we use a monotonic decreasing stack to store indices of days whose next warmer day has not yet been found.

### Thought Process

1.  **Requirement**: For each day, find how many days you have to wait for a warmer temperature.
2.  **Monotonic Stack**: Maintain a stack of indices such that the temperatures at these indices are in non-increasing order.
3.  **Finding Warmer Days**: Iterate through the `temperatures` array:
    - While the current temperature is higher than the temperature at the stack's top index:
        - We have found the "next warmer day" for that top index.
        - Pop the index and calculate the distance: `current_index - popped_index`.
        - Store the distance in the result array.
4.  **Push Current**: Push the current index onto the stack.
5.  **Initialization**: The result array is pre-filled with `0`, which correctly handles cases where no future warmer day exists.

### Go Code

``` go
func dailyTemperatures(temperatures []int) []int {
    n := len(temperatures)
    res := make([]int, n)
    stack := make([]int, 0, n)
    
    for i := 0; i < n; i++ {
        for len(stack) > 0 && temperatures[stack[len(stack)-1]] < temperatures[i] {
            topIdx := stack[len(stack)-1]
            stack = stack[:len(stack)-1]
            res[topIdx] = i - topIdx
        }
        stack = append(stack, i)
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - Although there is a nested loop, each index is pushed onto the stack exactly once and popped at most once. The total number of operations is linear with respect to $n$.
- **Space Complexity**: $O(n)$
    - In the worst case (a strictly decreasing temperature sequence), the stack will store all $n$ indices.


