# 378. Kth Smallest Element in a Sorted Matrix

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/description/)

## Solution: Binary Search on Value Range (with 2D Search)

The matrix is sorted both row-wise and column-wise. Instead of using a heap, we can binary search on the **value range** of the matrix (from the minimum element at the top-left to the maximum element at the bottom-right). For each midpoint value, we count how many elements are less than or equal to it using a linear $O(n)$ search starting from the bottom-left corner.

### Thought Process

1.  **Define Search Boundaries**:
    *   Since the matrix is sorted in ascending order across both rows and columns:
        *   The smallest element is at the top-left: $l = \text{matrix}[0][0]$.
        *   The largest element is at the bottom-right: $r = \text{matrix}[n-1][n-1]$.
2.  **Counting Elements $\le$ Target (`countEqualOrLess`)**:
    *   To count how many elements in the sorted matrix are less than or equal to a value `target`, we perform a search starting from the **bottom-left corner** of the matrix ($r = n - 1, c = 0$):
        *   **If $\text{matrix}[r][c] \le \text{target}$**: Since the column is sorted from top to bottom, all elements in column `c` from row `0` to row `r` are also $\le \text{target}$. There are $r + 1$ such elements. We add $r + 1$ to our count and move to the right column: `c++`.
        *   **If $\text{matrix}[r][c] > \text{target}$**: This element is too large. We move to the row above: `r--`.
    *   This helper takes at most $2n$ steps, which runs in linear $O(n)$ time.
3.  **Binary Search Loop (`l < r`)**:
    *   Compute midpoint value: `m = l + (r - l) / 2`.
    *   If `countEqualOrLess(m) >= k`:
        *   There are at least $k$ elements less than or equal to `m`. Thus, the $k$-th smallest element is less than or equal to `m`. We adjust our right boundary: `r = m`.
    *   If `countEqualOrLess(m) < k`:
        *   There are fewer than $k$ elements less than or equal to `m`. Thus, the $k$-th smallest element must be strictly greater than `m`. Adjust left boundary: `l = m + 1`.
4.  **Termination**:
    *   The loop terminates when `l == r`. The value `l` is guaranteed to be the $k$-th smallest element present in the matrix.

### Go Code

``` go
func kthSmallest(matrix [][]int, k int) int {
    n := len(matrix)
    l, r := matrix[0][0], matrix[n-1][n-1]

    countEqualOrLess := func(target int) int {
        count := 0
        r, c := n-1, 0
        for r >= 0 && c < n {
            if matrix[r][c] <= target {
                count += r+1
                c++
            } else {
                r--
            }
        }
        return count
    }

    for l < r {
        m := l + (r-l)/2
        if countEqualOrLess(m) >= k {
            r = m
        } else {
            l = m + 1
        }
    }
    return l
}
```

### Code Efficiency

- **Time Complexity**: $O(n \log(\text{max\_val} - \text{min\_val}))$
    - Where $n$ is the matrix dimension. The binary search operates on the value difference between the maximum and minimum elements in the matrix, executing $\log_2(\text{max\_val} - \text{min\_val})$ steps. Each step calls the $O(n)$ helper.
- **Space Complexity**: $O(1)$
    - The algorithm runs using constant auxiliary memory.