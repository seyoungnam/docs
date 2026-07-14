# 4. Median of Two Sorted Arrays

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/median-of-two-sorted-arrays/description/)

## Solution: Binary Search on Partition Index

To find the median of two sorted arrays of sizes $m$ and $n$ in $O(\log(\min(m, n)))$ time, we partition both arrays into two halves (left and right) such that the left half contains exactly the same number of elements as the right half (or one extra if the total size is odd), and all elements in the left half are less than or equal to all elements in the right half.

### Thought Process

1.  **Binary Search on the Shorter Array**:
    *   To optimize the search and prevent index out-of-bounds, we always run the binary search on the shorter array. Let `A` be the shorter array (size $m$) and `B` be the longer array (size $n$, where $m \le n$).
2.  **Partitioning Math**:
    *   Partition index $i$ in `A` splits it into `A[0...i-1]` (left) and `A[i...m-1]` (right).
    *   Partition index $j$ in `B` splits it into `B[0...j-1]` (left) and `B[j...n-1]` (right).
    *   To ensure the combined left and right halves are equal in size:
        $$\text{half} = \frac{m + n + 1}{2}$$
        $$j = \text{half} - i$$
3.  **Boundary Values**:
    *   Let $A_{\text{left}}$ be the largest element in `A`'s left partition: $A[i-1]$ (or $-\infty$ if $i = 0$).
    *   Let $A_{\text{right}}$ be the smallest element in `A`'s right partition: $A[i]$ (or $+\infty$ if $i = m$).
    *   Let $B_{\text{left}}$ be the largest element in `B`'s left partition: $B[j-1]$ (or $-\infty$ if $j = 0$).
    *   Let $B_{\text{right}}$ be the smallest element in `B`'s right partition: $B[j]$ (or $+\infty$ if $j = n$).
4.  **Validity Check**:
    *   The partition is correct if:
        $$A_{\text{left}} \le B_{\text{right}} \quad \text{and} \quad B_{\text{left}} \le A_{\text{right}}$$
5.  **Calculate Median**:
    *   If the total number of elements $m + n$ is odd, the median is the maximum of the left partition boundaries:
        $$\text{median} = \max(A_{\text{left}}, B_{\text{left}})$$
    *   If $m + n$ is even, the median is the average of the boundary elements:
        $$\text{median} = \frac{\max(A_{\text{left}}, B_{\text{left}}) + \min(A_{\text{right}}, B_{\text{right}})}{2.0}$$
6.  **Binary Search Adjustments**:
    *   If $A_{\text{left}} > B_{\text{right}}$, our partition index $i$ is too far to the right. We must move left: `r = i - 1`.
    *   If $B_{\text{left}} > A_{\text{right}}$, our partition index $i$ is too far to the left. We must move right: `l = i + 1`.

### Go Code

``` go
import "math"

func findMedianSortedArrays(nums1 []int, nums2 []int) float64 {
    A, B := nums1, nums2
    if len(A) > len(B) {
        A, B = B, A
    }
    total := len(A) + len(B)
    half := (total + 1) / 2

    l, r := 0, len(A)
    for l <= r {
        i := l + (r-l)/2
        j := half - i

        Aleft := math.MinInt32
        if i > 0 {
            Aleft = A[i-1]
        }
        Aright := math.MaxInt32
        if i < len(A) {
            Aright = A[i]
        }
        Bleft := math.MinInt32
        if j > 0 {
            Bleft = B[j-1]
        }
        Bright := math.MaxInt32
        if j < len(B) {
            Bright = B[j]
        }

        if Aleft <= Bright && Bleft <= Aright {
            if total%2 != 0 {
                return float64(max(Aleft, Bleft))
            }
            return float64(max(Aleft, Bleft)+min(Aright, Bright)) / 2.0
        }
        if Aleft > Bright {
            r = i - 1
        } else {
            l = i + 1
        }
    }
    return -1
}
```

### Code Efficiency

- **Time Complexity**: $O(\log(\min(m, n)))$
    - We perform binary search on the shorter array, which has size $\min(m, n)$.
- **Space Complexity**: $O(1)$
    - The partitioning checks are performed in-place using index boundaries, requiring no extra auxiliary memory.
