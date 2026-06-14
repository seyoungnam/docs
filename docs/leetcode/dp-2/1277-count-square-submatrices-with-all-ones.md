# 1277. Count Square Submatrices with All Ones

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/count-square-submatrices-with-all-ones/description/)

## Solution: 2D Dynamic Programming

### Thought Process

1.  **Understand the Problem**: We want to count the total number of square submatrices with all ones. The key realization is that the number of squares ending at cell `(r, c)` as their bottom-right corner is exactly equal to the side length of the largest square ending at `(r, c)`.
    *   *For example:* If the largest square ending at `(r, c)` has a side length of `3`, it means there is a `1x1` square, a `2x2` square, and a `3x3` square all ending at `(r, c)`. So, we add `3` to our total count.
2.  **Define DP State**: Let `dp[r][c]` be the side length of the largest square ending at cell `(r-1, c-1)` of the matrix.
3.  **Recurrence Relation**: If `matrix[r-1][c-1] == 1`:
    *   `dp[r][c] = min(dp[r-1][c], dp[r][c-1], dp[r-1][c-1]) + 1`
    *   Add this value to the running total `count`.
4.  **Base Case**: If `matrix[r-1][c-1] == 0`, `dp[r][c] = 0`.

### Go Code

``` go
func countSquares(matrix [][]int) int {
    if len(matrix) == 0 || len(matrix[0]) == 0 {
        return 0
    }
    count := 0
    ROWS, COLS := len(matrix), len(matrix[0])
    dp := make([][]int, ROWS+1)
    for r := 0; r < ROWS+1; r++ {
        dp[r] = make([]int, COLS+1)
    }
    for r := 1; r < ROWS+1; r++ {
        for c := 1; c < COLS+1; c++ {
            if matrix[r-1][c-1] == 1 {
                dp[r][c] = min(
                    dp[r-1][c],
                    dp[r][c-1],
                    dp[r-1][c-1],
                ) + 1
                count += dp[r][c]
            }
        }
    }
    return count
}
```

### Code Efficiency

- **Time Complexity**: $O(m \times n)$
    - We iterate through all cells of the $m \times n$ matrix once.
- **Space Complexity**: $O(m \times n)$
    - We allocate an $(m+1) \times (n+1)$ state array.

---

## Alternative Solution: Space-Optimized Dynamic Programming (1D Array)


### Thought Process

1.  **State Reduction**: Since `dp[r][c]` only depends on the current row's left element (`dp[r][c-1]`), the previous row's top element (`dp[r-1][c]`), and the top-left diagonal element (`dp[r-1][c-1]`), we can optimize the space complexity by collapsing the 2D array into a 1D array `dp` of size `COLS + 1`.
2.  **Tracking Diagonal State**: 
    *   To calculate the new `dp[c]`, we need the top-left diagonal neighbor `dp[r-1][c-1]`.
    *   We store this value in a variable `diag`.
    *   Before updating `dp[c]`, we save its current value (which represents the top element `dp[r-1][c]`) in a temporary variable `temp`.
    *   After the cell calculation, `diag` is updated to `temp` to serve as the diagonal state for the next column (`c + 1`).
3.  **Row Transition**: At the start of each new row, we reset `diag = 0` because there is no diagonal state for the first column.

### Go Code

``` go
func countSquares(matrix [][]int) int {
    if len(matrix) == 0 || len(matrix[0]) == 0 {
        return 0
    }
    count := 0
    ROWS, COLS := len(matrix), len(matrix[0])
    dp := make([]int, COLS+1)
    diag := 0
    for r := 1; r < ROWS+1; r++ {
        for c := 1; c < COLS+1; c++ {
            temp := dp[c] // Capture the old top value before update
            if matrix[r-1][c-1] == 1 {
                dp[c] = min(
                    dp[c],
                    dp[c-1],
                    diag,
                ) + 1
                count += dp[c]
            } else {
                dp[c] = 0
            }
            diag = temp // Update diagonal for the next column c+1
        }
        diag = 0 // Reset diagonal state at the end of each row
    }
    return count
}
```

### Code Efficiency

- **Time Complexity**: $O(m \times n)$
    - We still visit each cell of the matrix once.
- **Space Complexity**: $O(n)$
    - We only allocate a single 1D array of size $\text{COLS}+1$, where $\text{COLS}$ is the number of columns.
