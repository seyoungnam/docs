# 540. Single Element in a Sorted Array

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/single-element-in-a-sorted-array/description/)

## Solution: Binary Search on Even Indexes (Boundary Search)

We are given a sorted array where every element appears exactly twice, except for one element which appears exactly once. To find this single element in $O(\log n)$ time and $O(1)$ space, we can exploit the index pattern of the duplicates.

### Thought Process

1.  **Index Pattern of Duplicates**:
    *   Since every number (except one) appears exactly twice and the array is sorted, the elements form pairs.
    *   **Before the single element**: Each duplicate pair starts at an **even index** ($2k$) and ends at an **odd index** ($2k + 1$). That is, $\text{nums}[2k] == \text{nums}[2k+1]$.
    *   **After the single element**: The single element shifts the alignment, so pairs start at an **odd index** ($2k - 1$) and end at an **even index** ($2k$). That is, $\text{nums}[2k] \neq \text{nums}[2k+1]$.
2.  **Binary Search Strategy**:
    *   Our goal is to find the first even index `m` where $\text{nums}[m] \neq \text{nums}[m+1]$. The element at this index will be the single non-duplicate.
    *   Initialize `l = 0` and `r = len(nums) - 1`.
    *   While `l < r`:
        *   Find the midpoint `m = l + (r - l) / 2`.
        *   If `m` is odd (`m % 2 == 1`), decrement `m` by 1 to make it even. This ensures we are always comparing the first element of a potential pair at an even index.
        *   **Check alignment**:
            *   If `nums[m] == nums[m+1]`: The alignment is correct, meaning the single element lies strictly to the right. Adjust the left boundary to search the right half: `l = m + 2`.
            *   If `nums[m] != nums[m+1]`: The alignment is broken, meaning the single element is at `m` or lies to the left. Adjust the right boundary: `r = m`.
3.  **Termination**:
    *   The loop terminates when `l == r`. The single non-duplicate element is located at `nums[l]`.

### Go Code

``` go
func singleNonDuplicate(nums []int) int {
    l, r := 0, len(nums)-1
    for l < r {
        m := l + (r-l)/2
        if m%2 == 1 {
            m--
        }
        if nums[m] == nums[m+1] {
            l = m+2
        } else {
            r = m
        }
    }
    return nums[l]
}
```

### Code Efficiency

- **Time Complexity**: $O(\log n)$
    - The search space is halved in each iteration of the binary search, requiring logarithmic time.
- **Space Complexity**: $O(1)$
    - Only a constant number of pointer variables (`l`, `r`, `m`) are stored, requiring $O(1)$ auxiliary space.