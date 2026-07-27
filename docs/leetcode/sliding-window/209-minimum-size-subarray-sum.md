# 209. Minimum Size Subarray Sum

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/minimum-size-subarray-sum/description/)

## Solution: Dynamic-Size Sliding Window (Two Pointers)

Since all numbers in the input array `nums` are positive, the running sum of a subarray increases monotonically when we add elements to the right and decreases monotonically when we remove elements from the left. This behavior makes the problem ideal for a **dynamic-size sliding window** (or two pointers) approach.

### Thought Process

1.  **Window Setup**:
    *   Initialize the left pointer `l = 0` and the minimum subarray length `res = math.MaxInt32`.
    *   Maintain a running `sum` representing the sum of elements within the window $[l, r]$.
2.  **Expanding the Window**:
    *   Iterate through `nums` using a right pointer `r` (with value `num`).
    *   Add `num` to the running `sum`.
3.  **Shrinking the Window**:
    *   While the window sum satisfies the condition `sum >= target`:
        *   Update the minimum size of a valid subarray: `res = min(res, r - l + 1)`.
        *   Attempt to shrink the window from the left by subtracting `nums[l]` from `sum` and incrementing `l`.
4.  **Completion**:
    *   If `res` remains `math.MaxInt32` after processing the array, no valid subarray was found; return `0`.
    *   Otherwise, return `res`.

### Go Code

``` go
import "math"

func minSubArrayLen(target int, nums []int) int {
    l := 0
    res := math.MaxInt32
    sum := 0
    for r, num := range nums {
        sum += num
        for l <= r && sum >= target {
            res = min(res, r-l+1)
            sum -= nums[l]
            l++
        }
    }
    if res == math.MaxInt32 {
        return 0
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - Although there is a nested loop, both the left pointer `l` and the right pointer `r` traverse the array at most once. Each element is added to and subtracted from the `sum` at most once, leading to an overall linear time complexity.
- **Space Complexity**: $O(1)$
    - The algorithm runs in-place, using only constant auxiliary variables.