# 154. Find Minimum in Rotated Sorted Array II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array-ii/description/)

## Solution: Binary Search with Duplicate Contraction

This problem is an extension of **Find Minimum in Rotated Sorted Array**, adding the possibility of duplicate elements. The presence of duplicates requires an extra step to handle cases where we cannot determine which half of the array contains the minimum element.

### Thought Process

1.  **Comparison Reference**:
    *   Unlike finding a target, when finding the minimum element in a rotated sorted array, we compare the middle element `nums[m]` with the right element `nums[r]` to determine where the rotation inflection point lies.
2.  **Binary Search Strategy (`l < r`)**:
    *   Calculate midpoint `m = l + (r - l) / 2`.
    *   **Case 1: $\text{nums}[m] > \text{nums}[r]$**:
        *   The middle element is strictly larger than the right element. This indicates the pivot (inflection point) and the minimum element must reside in the right half: `l = m + 1`.
    *   **Case 2: $\text{nums}[m] < \text{nums}[r]$**:
        *   The middle element is strictly smaller than the right element. This means the right half is sorted, and the minimum element is either at `m` or lies in the left half: `r = m`.
    *   **Case 3: $\text{nums}[m] == \text{nums}[r]$**:
        *   Because duplicates are allowed, when the elements at `m` and `r` are equal, we cannot identify which half is sorted.
        *   However, we can safely shrink the search boundary from the right: `r--`. This is safe because even if `nums[r]` was the minimum, `nums[m]` (which is equal to `nums[r]`) remains in the search space, so we do not lose the minimum element.
3.  **Termination**:
    *   The loop terminates when `l == r`. The minimum element of the array is located at `nums[l]`.

### Go Code

``` go
func findMin(nums []int) int {
    l, r := 0, len(nums)-1

    for l < r {
        m := l + (r-l)/2
        if nums[m] > nums[r] {
            l = m+1
        } else if nums[m] < nums[r] {
            r = m
        } else {
            r--
        }
    }
    return nums[l]
}
```

### Code Efficiency

- **Time Complexity**:
    - **Average Case**: $O(\log n)$ — The search space is halved in most iterations.
    - **Worst Case**: $O(n)$ — If the array is filled with duplicate elements (e.g. $[2, 2, 2, 1, 2]$), we must decrement `r` one-by-one, resulting in a linear scan.
- **Space Complexity**: $O(1)$
    - Only a constant number of tracking pointers are used.