# 88. Merge Sorted Array

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/merge-sorted-array/description/)

## Solution: Two Pointers (Backwards Merge)

We can merge the two sorted arrays in-place by starting from the end of both arrays. Since `nums1` has enough extra space at the end to hold the final merged result, placing elements from right to left (largest to smallest) allows us to populate `nums1` without overwriting any of its elements before we have processed them.

### Thought Process

1.  **Three Pointers Setup**:
    *   `i = m + n - 1`: The write pointer starting at the very end of `nums1`.
    *   `i1 = m - 1`: The read pointer starting at the last active element of `nums1`.
    *   `i2 = n - 1`: The read pointer starting at the last element of `nums2`.
2.  **Compare and Place**:
    *   While both `i1 >= 0` and `i2 >= 0`, compare `nums1[i1]` and `nums2[i2]`.
    *   Place the larger element at `nums1[i]` and decrement the corresponding read pointer.
    *   Decrement the write pointer `i`.
3.  **Handle Leftovers**:
    *   If `i1 >= 0`, copy any remaining elements from `nums1`. (Note: In a pure in-place solution, these are already in their correct position in `nums1`, but the code explicitly copies them).
    *   If `i2 >= 0`, copy any remaining elements from `nums2` into `nums1`.

### Go Code

``` go
func merge(nums1 []int, m int, nums2 []int, n int)  {
    i := m+n-1
    i1, i2 := m-1, n-1
    for i1 >= 0 && i2 >= 0 {
        if nums1[i1] >= nums2[i2] {
            nums1[i] = nums1[i1]
            i1--
        } else {
            nums1[i] = nums2[i2]
            i2--
        }
        i--
    }
    for i1 >= 0 {
        nums1[i] = nums1[i1]
        i1--
        i--
    }
    for i2 >= 0 {
        nums1[i] = nums2[i2]
        i2--
        i--
    }
}
```

### Code Efficiency

- **Time Complexity**: $O(m + n)$
    - We place each of the $m + n$ elements into their final positions in the merged array exactly once.
- **Space Complexity**: $O(1)$
    - We modify `nums1` in-place, requiring no extra auxiliary memory.