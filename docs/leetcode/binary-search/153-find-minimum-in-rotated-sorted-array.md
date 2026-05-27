# 153. Find Minimum in Rotated Sorted Array

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/description/)

## Solution: Binary Search on Boundary (Half-Open Loop)

We can find the minimum element in a rotated sorted array in $O(\log n)$ time. Since we are searching for an inflection/pivot point (a boundary condition) rather than a specific target, we use a half-open loop `l < r` which terminates when the window shrinks to a single element.

### Thought Process

1.  **Initialize Pointers**: Set `l = 0` and `r = len(nums) - 1`.
2.  **Inflection Decision**: Find the midpoint `m = l + (r - l) / 2`:
    *   If `nums[m] < nums[r]`: The right half `[m, r]` is normally sorted. This implies the minimum element must be to the left of `m` (or could be `m` itself). Shrink the right boundary: `r = m`.
    *   If `nums[m] > nums[r]`: The inflection point lies to the right of `m` (the left half `[l, m]` is sorted, and the rotation boundary is in the right half). The minimum element must be strictly to the right of `m`. Shift the left boundary: `l = m + 1`.
3.  **Result**: When `l == r`, the loop terminates, and `nums[l]` (or `nums[r]`) is the minimum element.

### Go Code

``` go
func findMin(nums []int) int {
    n := len(nums)
    l, r := 0, n-1
    for l < r {
        m := l + (r-l)/2
        
        // If middle element is smaller than right, minimum is to the left (including m)
        if nums[m] < nums[r] {
            r = m
        } else { // Minimum is strictly to the right of m
            l = m + 1
        }
    }
    return nums[l]
}
```

### Code Efficiency

- **Time Complexity**: $O(\log n)$
    - We divide the search space in half during each iteration, taking logarithmic time.
- **Space Complexity**: $O(1)$
    - We only use three integer variables (`l`, `r`, `m`), requiring $O(1)$ auxiliary space.
