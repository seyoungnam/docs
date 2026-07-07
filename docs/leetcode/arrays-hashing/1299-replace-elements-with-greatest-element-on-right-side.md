# 1299. Replace Elements with Greatest Element on Right Side

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/replace-elements-with-greatest-element-on-right-side/description/)

## Solution: Right-to-Left Linear Scan (Single Pass)

To replace each element in the array with the greatest element among the elements to its right, we can scan the array in reverse order. This allows us to maintain a running maximum of all processed elements, solving the problem in a single linear pass.

### Thought Process

1.  **Iterative Direction**:
    *   Scanning from left to right would require looking ahead at all elements to the right for each position, resulting in an inefficient $O(n^2)$ time complexity.
    *   By scanning from right to left (from index $n-1$ down to $0$), we can track the maximum element seen so far in $O(1)$ auxiliary space.
2.  **State Tracking**:
    *   Maintain a variable `rightMax` initialized to `-1` (since the last element must be replaced by `-1`).
3.  **Iteration Steps**:
    *   At each index `i` from $n-1$ down to $0$:
        1. Set the result at `i` to the current `rightMax`: `res[i] = rightMax`.
        2. Update `rightMax` with the value of the current element: `rightMax = max(rightMax, arr[i])`.
4.  **Result**:
    *   Return the populated `res` array.

### Go Code

``` go
func replaceElements(arr []int) []int {
    n := len(arr)
    res := make([]int, n)
    rightMax := -1
    for i := n-1; i >= 0; i-- {
        res[i] = rightMax
        rightMax = max(rightMax, arr[i])
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We scan the array of length $n$ exactly once, performing constant-time assignments and comparisons at each step.
- **Space Complexity**: $O(1)$ auxiliary space
    - The output slice `res` of size $n$ is required for the return value. No additional dynamic memory is allocated.