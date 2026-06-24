# 48. Rotate Image

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/rotate-image/description/)

## Solution: Transpose and Reverse Rows (In-Place)

Rotating a matrix by 90 degrees clockwise in-place can be achieved by combining two simple geometric transformations:

1.  **Transpose**: Mirror the matrix across its main diagonal (top-left to bottom-right). This converts the rows of the matrix into columns.
2.  **Reverse Rows (Flip)**: Reverse the elements in each row horizontally. This completes the clockwise rotation.

### Thought Process

1.  **Matrix Transposition**:
    - Iterate through each row `r` from `0` to `n - 1`.
    - Swap elements across the diagonal: `matrix[r][c]` with `matrix[c][r]`.
    - **Note**: The inner loop for columns `c` must start at `r` (not `0`). If we started at `0`, we would swap elements twice, returning the matrix to its original state.
2.  **Horizontal Row Reversal**:
    - For each row, use a two-pointer approach (`c` starting at `0` and `k` starting at `n - 1`).
    - Swap `matrix[r][c]` with `matrix[r][k]`, then increment `c` and decrement `k` until the pointers meet in the middle.

### Go Code

``` go
func rotate(matrix [][]int)  {
    n := len(matrix)
    for r := range matrix {
        // transpose
        for c := r; c < n; c++ {
            matrix[r][c], matrix[c][r] = matrix[c][r], matrix[r][c]
        }
        // flip the row
        for c, k := 0, n-1; c < k; c, k = c+1, k-1 {
            matrix[r][c], matrix[r][k] = matrix[r][k], matrix[r][c]
        }
    }    
}
```

### Code Efficiency

- **Time Complexity**: $O(n^2)$
    - Where $n \times n$ is the size of the matrix. We visit each cell in the matrix a constant number of times (once for transposition, and once for reversal).
- **Space Complexity**: $O(1)$
    - The rotation is performed entirely in-place, requiring no additional auxiliary memory.