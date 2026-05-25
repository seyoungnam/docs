# 33. Search in Rotated Sorted Array

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/search-in-rotated-sorted-array/description/)

## Solution: One-Pass Binary Search (Optimal)

Instead of finding the peak/pivot first in a separate pass (which can introduce tricky boundary bugs), we can solve this in a **single pass** using a single binary search. In a rotated sorted array, for any midpoint `m`, **at least one of the two halves (left or right) is guaranteed to be normally sorted**.

### Thought Process

1.  **Identify the Sorted Half**:
    *   If `nums[l] <= nums[m]`, the left half `[l, m]` is normally sorted.
    *   Otherwise, the right half `[m, r]` is normally sorted.
2.  **Range Checking**:
    *   If the left half is sorted: Check if `target` lies within the sorted range `[nums[l], nums[m])`. If so, shrink the window to the left (`r = m - 1`); otherwise, search the right (`l = m + 1`).
    *   If the right half is sorted: Check if `target` lies within the sorted range `(nums[m], nums[r]]`. If so, shrink the window to the right (`l = m + 1`); otherwise, search the left (`r = m - 1`).
3.  **Loop Condition**: Since we are searching for a **specific target** with a direct equality check (`nums[m] == target`), we use `l <= r`.

### Go Code

``` go
func search(nums []int, target int) int {
    l, r := 0, len(nums)-1
    for l <= r {
        m := l + (r-l)/2
        if nums[m] == target {
            return m
        }
        
        // Left half is sorted
        if nums[l] <= nums[m] {
            // Check if target lies within the sorted left half
            if nums[l] <= target && target < nums[m] {
                r = m - 1
            } else {
                l = m + 1
            }
        } else { // Right half is sorted
            // Check if target lies within the sorted right half
            if nums[m] < target && target <= nums[r] {
                l = m + 1
            } else {
                r = m - 1
            }
        }
    }
    return -1
}
```

### Code Efficiency

- **Time Complexity**: $O(\log n)$
    - We perform a single-pass binary search, halving the search space in each step.
- **Space Complexity**: $O(1)$
    - We only use a few tracking index variables, requiring $O(1)$ auxiliary space.
