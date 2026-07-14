# 1901. Find a Peak Element II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/find-a-peak-element-ii/description/)

## Solution: Binary Search on Columns

To find a peak element (an element that is strictly greater than its adjacent top, bottom, left, and right neighbors) in $O(\text{ROWS} \log(\text{COLS}))$ time, we perform a binary search on the columns of the matrix. 

### Thought Process

1.  **Reduce to 1D Comparison**:
    *   For any selected column `mid`, we find the row index `maxRow` containing the maximum value in that column.
    *   By definition, `mat[maxRow][mid]` is greater than or equal to its vertical neighbors (above and below it) in column `mid`.
    *   This reduces the problem of checking 4 directions to checking only **horizontal neighbors** (left and right).
2.  **Binary Search on Columns (`l <= r`)**:
    *   Set `l = 0` and `r = COLS - 1`.
    *   Calculate midpoint column: `mid = l + (r - l) / 2`.
    *   Find the row `maxRow` that contains the largest element in column `mid` by scanning from top to bottom.
    *   **Compare with horizontal neighbors**:
        *   Check if the left neighbor is larger: `leftIsLarger = mid > 0 && mat[maxRow][mid-1] > mat[maxRow][mid]`.
        *   Check if the right neighbor is larger: `rightIsLarger = mid < COLS-1 && mat[maxRow][mid+1] > mat[maxRow][mid]`.
    *   **Decision Boundaries**:
        *   **Peak Found**: If neither neighbor is larger (`!leftIsLarger && !rightIsLarger`), we have found a 2D peak at `[maxRow, mid]`. Return its coordinates.
        *   **Move Right**: If the right neighbor is larger (`rightIsLarger`), a peak is guaranteed to exist in the right half. Shift left bound: `l = mid + 1`.
        *   **Move Left**: Otherwise, a peak must exist in the left half. Shift right bound: `r = mid - 1`.

### Go Code

``` go
func findPeakGrid(mat [][]int) []int {
    ROWS, COLS := len(mat), len(mat[0])

    l, r := 0, COLS-1
    for l <= r {
        mid := l + (r-l)/2
        maxRow := 0
        for row := 1; row < ROWS; row++ {
            if mat[row][mid] > mat[maxRow][mid] {
                maxRow = row
            }
        }

        leftIsLarger := false
        rightIsLarger := false

        if mid > 0 && mat[maxRow][mid-1] > mat[maxRow][mid] {
            leftIsLarger = true
        }
        if mid < COLS-1 && mat[maxRow][mid+1] > mat[maxRow][mid] {
            rightIsLarger = true
        }
        if !leftIsLarger && !rightIsLarger {
            return []int{maxRow, mid}
        }

        if rightIsLarger {
            l = mid+1
        } else {
            r = mid-1
        }
    }
    return []int{-1, -1}
}
```

### Code Efficiency

- **Time Complexity**: $O(\text{ROWS} \log(\text{COLS}))$
    - The binary search divides the columns in half at each step, running for $\log_2(\text{COLS})$ iterations. In each iteration, we perform a linear scan of height $\text{ROWS}$ to locate the maximum value in column `mid`.
- **Space Complexity**: $O(1)$
    - Only a constant number of tracking index variables are allocated in memory.