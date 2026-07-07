# 35. Search Insert Position

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/search-insert-position/description/)

## Solution: Binary Search (Boundary/Lower Bound)

To find the target's index or the index where it would be inserted in sorted order, we can search for the **first element in the array that is greater than or equal to the target**. This is a classic lower bound boundary search.

### Thought Process

1.  **Monotonic Condition**:
    *   Since the input array is sorted, the condition `nums[i] >= target` is monotonic. It evaluates to `false` for elements smaller than the target, and `true` for all elements greater than or equal to the target.
2.  **Initialize Boundaries**:
    *   Set `l = 0` (left boundary) and `r = len(nums)` (right boundary).
    *   *Note: Setting `r` to `len(nums)` (instead of `len(nums) - 1`) is crucial because the insert position can be at the very end of the array if the target is larger than all existing elements.*
3.  **Iteration (`l < r`)**:
    *   Calculate midpoint `m = l + (r - l) / 2`.
    *   **Condition Met (`nums[m] >= target`)**: `m` is a valid candidate. We narrow our search space to the left half (including `m`) to find the first such occurrence: `r = m`.
    *   **Condition Failed (`nums[m] < target`)**: `m` is too small. The insertion point must be strictly to the right: `l = m + 1`.
4.  **Termination**:
    *   The loop terminates when `l == r`. At this point, the pointer `l` is positioned at the first index where `nums[l] >= target`, which is the correct insertion index.

### Go Code

``` go
func searchInsert(nums []int, target int) int {
    l, r := 0, len(nums)
    for l < r {
        m := l + (r-l)/2
        if nums[m] >= target {
            r = m
        } else {
            l = m+1
        }
    }
    return l
}
```

### Code Efficiency

- **Time Complexity**: $O(\log n)$
    - The search space of size $n$ is halved in each step, resulting in logarithmic time complexity.
- **Space Complexity**: $O(1)$
    - Only a constant number of integer variables are used, requiring no extra memory allocations.