# 221. Maximal Square

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/maximal-square/description/)

## Solution: 2D Dynamic Programming

We can solve this problem by finding the maximum side length of a square ending at each cell `(i, j)`. Let `dp[i][j]` represent the side length of the largest square whose bottom-right corner is at cell `(i-1, j-1)` in the matrix.

### Thought Process

1.  **Subproblem Definition**: If `matrix[i-1][j-1] == '1'`, then the maximum square ending at this cell depends on the size of the squares ending at its top, left, and top-left neighbors:
    *   `dp[i][j] = min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) + 1`
2.  **Base Case**: If `matrix[i-1][j-1] == '0'`, then no square can end at this cell, so `dp[i][j] = 0`.
3.  **Area Calculation**: Track the maximum side length found (`maxSide`). The final answer is the area of the square, which is `maxSide * maxSide`.

### Go Code

``` go
func maximalSquare(matrix [][]byte) int {
    if len(matrix) == 0 || len(matrix[0]) == 0 {
        return 0
    }
    
    rows, cols := len(matrix), len(matrix[0])
    dp := make([][]int, rows+1)
    for i := range dp {
        dp[i] = make([]int, cols+1)
    }
    
    maxSide := 0
    for i := 1; i <= rows; i++ {
        for j := 1; j <= cols; j++ {
            if matrix[i-1][j-1] == '1' {
                dp[i][j] = min(dp[i-1][j], min(dp[i][j-1], dp[i-1][j-1])) + 1
                if dp[i][j] > maxSide {
                    maxSide = dp[i][j]
                }
            }
        }
    }
    
    return maxSide * maxSide
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```

### Code Efficiency

- **Time Complexity**: $O(m \times n)$
    - We iterate through all cells of the $m \times n$ matrix once.
- **Space Complexity**: $O(m \times n)$
    - We allocate an $(m+1) \times (n+1)$ state array.

---

## Alternative Solution: Space-Optimized Dynamic Programming (1D Array)

Instead of keeping track of the entire 2D grid of states, we can optimize space because the value `dp[i][j]` only depends on the current row and the previous row.

### Thought Process

1.  **State Reduction**: We can collapse the 2D grid into a 1D array `dp` of size `COLS + 1`.
2.  **Tracking Diagonal State**: To calculate the new `dp[c]`, we need the top-left diagonal neighbor `dp[r-1][c-1]`. We store this value in a variable `diag` before it gets overwritten by the update.
3.  **Row Transition**: At the start of each row, reset `diag = 0`.

### Go Code

``` go
func maximalSquare(matrix [][]byte) int {
    if len(matrix) == 0 || len(matrix[0]) == 0 {
        return 0
    }

    ROWS, COLS := len(matrix), len(matrix[0])
    dp := make([]int, COLS+1)
    maxLen := 0
    diag := 0 // dp[r-1][c-1]

    for r := 1; r < ROWS+1; r++ {
        for c := 1; c < COLS+1; c++ {
            temp := dp[c] // Temporarily hold dp[r-1][c]
            if matrix[r-1][c-1] == '1' {
                dp[c] = min(
                    dp[c],      // dp[r-1][c]
                    dp[c-1],    // dp[r][c-1]
                    diag,       // dp[r-1][c-1]
                    ) + 1
                maxLen = max(maxLen, dp[c])
            } else {
                dp[c] = 0
            }
            diag = temp // The old top state becomes the diagonal for the next column
        }
        diag = 0 // Reset diagonal state for the next row
    }
    return maxLen * maxLen
}
```

### Code Efficiency

- **Time Complexity**: $O(m \times n)$
    - We still visit each cell of the matrix once.
- **Space Complexity**: $O(n)$
    - We only allocate a single 1D array of size $\text{COLS}+1$, where $\text{COLS}$ is the number of columns.
