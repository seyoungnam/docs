# 4. Median of Two Sorted Arrays

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/median-of-two-sorted-arrays/description/)

## Solution: Binary Search on Partition Index

To find the median in $O(\log(\min(m, n)))$ time, we can partition both sorted arrays `nums1` and `nums2` into two halves (left and right) such that the left half contains exactly the same number of elements as the right half, and all elements in the left half are less than or equal to all elements in the right half.

### Thought Process

1.  **Binary Search Shorter Array**: To avoid index out-of-bounds, we always run the binary search on the shorter array. Let `nums1` be the shorter array (length `m <= n`).
2.  **Partitioning**:
    *   Find partition index `i` in `nums1` using binary search in the range `[0, m]`.
    *   The partition index `j` in `nums2` is uniquely determined: `j = (m + n + 1) / 2 - i`.
3.  **Boundary Checks**:
    *   Let `maxLeft1` and `minRight1` be the boundary values in `nums1` around partition `i`.
    *   Let `maxLeft2` and `minRight2` be the boundary values in `nums2` around partition `j`.
    *   Use `math.MinInt32` if a left partition is empty, and `math.MaxInt32` if a right partition is empty.
4.  **Verification**: The partition is correct if:
    *   `maxLeft1 <= minRight2` AND `maxLeft2 <= minRight1`.
5.  **Compute Median**:
    *   If total size `m + n` is odd: `median = max(maxLeft1, maxLeft2)`.
    *   If total size `m + n` is even: `median = (max(maxLeft1, maxLeft2) + min(minRight1, minRight2)) / 2.0`.
6.  **Adjust Binary Search**:
    *   If `maxLeft1 > minRight2`: `i` is too far to the right, decrease partition search: `high = i - 1`.
    *   If `maxLeft2 > minRight1`: `i` is too far to the left, increase partition search: `low = i + 1`.

### Go Code

``` go
import "math"

func findMedianSortedArrays(nums1 []int, nums2 []int) float64 {
    m, n := len(nums1), len(nums2)
    // Ensure nums1 is the shorter array
    if m > n {
        return findMedianSortedArrays(nums2, nums1)
    }
    
    low, high := 0, m
    halfLen := (m + n + 1) / 2
    
    for low <= high {
        i := (low + high) / 2
        j := halfLen - i
        
        var maxLeft1, minRight1, maxLeft2, minRight2 int
        
        if i == 0 {
            maxLeft1 = math.MinInt32
        } else {
            maxLeft1 = nums1[i-1]
        }
        
        if i == m {
            minRight1 = math.MaxInt32
        } else {
            minRight1 = nums1[i]
        }
        
        if j == 0 {
            maxLeft2 = math.MinInt32
        } else {
            maxLeft2 = nums2[j-1]
        }
        
        if j == n {
            minRight2 = math.MaxInt32
        } else {
            minRight2 = nums2[j]
        }
        
        if maxLeft1 <= minRight2 && maxLeft2 <= minRight1 {
            // Correct partition found
            if (m+n)%2 == 1 {
                return float64(max(maxLeft1, maxLeft2))
            }
            return float64(max(maxLeft1, maxLeft2)+min(minRight1, minRight2)) / 2.0
        } else if maxLeft1 > minRight2 {
            // Too far right in nums1, move left
            high = i - 1
        } else {
            // Too far left in nums1, move right
            low = i + 1
        }
    }
    return 0.0
}
```

### Code Efficiency

- **Time Complexity**: $O(\log(\min(m, n)))$
    - We perform a binary search on the shorter array of length $\min(m, n)$.
- **Space Complexity**: $O(1)$
    - We only use a few integer variables, requiring $O(1)$ auxiliary space.
