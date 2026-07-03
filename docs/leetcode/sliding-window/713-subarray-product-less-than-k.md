# 713. Subarray Product Less Than K

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/subarray-product-less-than-k/description/)

## Solution: Sliding Window (Two Pointers)

This problem can be solved efficiently in linear time using a **Sliding Window** (Two Pointers) approach. Since all elements in `nums` are strictly positive integers, the product of elements in a window is monotonically increasing as the window expands, and decreasing as the window shrinks.

### Thought Process

1.  **Base Case**:
    *   If $k \le 1$, it is impossible for any subarray to have a product strictly less than $k$ because the elements are positive integers ($\ge 1$). The smallest possible product of a subarray is $1$, which is not strictly less than $1$ or any smaller value. In this case, we return `0` immediately.
2.  **Initialize Window Variables**:
    *   `l = 0`: The left boundary of our sliding window.
    *   `p = 1`: The running product of elements inside the current sliding window.
    *   `res = 0`: The accumulator counting the number of valid subarrays.
3.  **Expand the Window**:
    *   Iterate the right boundary `r` from `0` to `len(nums) - 1`.
    *   Multiply the running product by the new element: `p *= nums[r]`.
4.  **Shrink the Window**:
    *   While the product `p` is greater than or equal to $k$ and the window is non-empty, shrink the window from the left:
        *   Divide the running product by the leftmost element: `p /= nums[l]`.
        *   Advance the left pointer: `l++`.
5.  **Count Valid Subarrays**:
    *   Once the running product `p` is strictly less than $k$, all contiguous subarrays ending at index `r` and starting at any index from `l` to `r` are also valid (their products will be even smaller than or equal to `p`).
    *   The number of such valid subarrays is exactly equal to the length of the window: `r - l + 1`. We add this to `res`.

### Go Code

``` go
func numSubarrayProductLessThanK(nums []int, k int) int {
    if k <= 1 {
        return 0
    }
    res := 0
    l, p := 0, 1
    for r, v := range nums {
        p *= v
        for p >= k {
            p /= nums[l]
            l++
        }
        res += r-l+1
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - Although there is a nested loop, both pointers `l` and `r` only move forward. The right pointer `r` steps $n$ times, and the left pointer `l` can advance at most $n$ times across the entire function execution. Thus, the total number of operations is linear.
- **Space Complexity**: $O(1)$
    - We only track a few scalar variables, requiring constant auxiliary space.