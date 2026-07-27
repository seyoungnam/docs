# 1343. Number of Sub-arrays of Size K and Average Greater than or Equal to Threshold

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/number-of-sub-arrays-of-size-k-and-average-greater-than-or-equal-to-threshold/description/)

## Solution: Fixed-Size Sliding Window

To count the number of subarrays of size $k$ with an average greater than or equal to a given `threshold`, we can maintain a sliding window of size $k$ across the array. 

### Thought Process

1.  **Mathematical Optimization**:
    *   The condition for the average of a subarray of size $k$ to meet the threshold is:
        $$\frac{\text{sum}}{k} \ge \text{threshold} \iff \text{sum} \ge k \times \text{threshold}$$
    *   Evaluating $\text{sum} \ge k \times \text{threshold}$ allows us to perform integer operations only, avoiding floating-point division.
2.  **Sliding Window Mechanics**:
    *   We maintain a running sum of the active window of size $k$.
    *   As we iterate through the array at index `i`:
        *   **Subtract Outgoing Element**: If the index `i - k >= 0`, the element at `i - k` has fallen outside the window of size $k$. We subtract it from our running `sum`.
        *   **Add Incoming Element**: Add the current element `num` at index `i` to `sum`.
        *   **Threshold Check**: Once the index `i` is at least `k - 1` (meaning the window has reached size $k$), we check if `sum >= k * threshold`. If so, increment the answer counter `cnt`.
3.  **Completion**:
    *   Return `cnt` once the entire array has been scanned.

### Go Code

``` go
func numOfSubarrays(arr []int, k int, threshold int) int {
    sum, cnt := 0, 0
    for i, num := range arr {
        if i - k >= 0 {
            sum -= arr[i-k]
        }
        sum += num
        if i >= k-1 && sum >= k*threshold {
            cnt++
        }
    }
    return cnt
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We make a single pass through the array.
- **Space Complexity**: $O(1)$
    - The algorithm runs in-place, using only constant auxiliary variables.