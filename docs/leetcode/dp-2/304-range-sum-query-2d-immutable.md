# 304. Range Sum Query 2D - Immutable

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/range-sum-query-2d-immutable/)

## Solution: 2D Dynamic Programming

### Thought Process

1.  **Inclusion-Exclusion Principle (2D Prefix Sum)**: To calculate the sum of any rectangular region in $O(1)$ time, we precompute a 2D prefix sum grid. Let `dp[r][c]` represent the sum of all elements in the subgrid from `(0, 0)` to `(r-1, c-1)`.
2.  **Precomputation Recurrence**:
    *   The sum of elements up to `(r, c)` is the current cell value plus the sum of the top subgrid and the left subgrid, minus the overlapping top-left subgrid (which is counted twice):
    *   `dp[r][c] = matrix[r-1][c-1] + dp[r-1][c] + dp[r][c-1] - dp[r-1][c-1]`
3.  **Querying (Sum Region)**:
    *   To find the sum of the region between `(row1, col1)` and `(row2, col2)`:
    *   `res = dp[row2+1][col2+1] - dp[row2+1][col1] - dp[row1][col2+1] + dp[row1][col1]`
    *   This subtracts the top region and the left region, and adds back the top-left region that was subtracted twice.
4.  **Optimization Note**: Storing the original `matrix` inside the `NumMatrix` struct is redundant. All boundary checks and query logic can be done using the dimensions of the `dp` array (`len(dp) - 1` and `len(dp[0]) - 1`), saving memory.

### Go Code

``` go
type NumMatrix struct {
    matrix  [][]int
    dp      [][]int
}


func Constructor(matrix [][]int) NumMatrix {
    ROWS, COLS := len(matrix), len(matrix[0])
    dp := make([][]int, ROWS+1)
    for r := 0; r < ROWS+1; r++ {
        dp[r] = make([]int, COLS+1)
    }
    for r := 1; r < ROWS+1; r++ {
        for c := 1; c < COLS+1; c++ {
            dp[r][c] = matrix[r-1][c-1] + dp[r-1][c] + dp[r][c-1] - dp[r-1][c-1] 
        }
    }
    return NumMatrix{
        matrix: matrix,
        dp: dp,
    }
}


func (this *NumMatrix) SumRegion(row1 int, col1 int, row2 int, col2 int) int {
    ROWS, COLS := len(this.matrix), len(this.matrix[0])
    if row1 >= ROWS || row2 >= ROWS || col1 >= COLS || col2 >= COLS {
        return 0
    }
    if row1 > row2 || col1 > col2 {
        return 0
    }
    return this.dp[row2+1][col2+1] - this.dp[row2+1][col1] - this.dp[row1][col2+1] + this.dp[row1][col1]
}

```

### Code Efficiency

- **Time Complexity**: $O(m \times n)$
    - We iterate through all cells of the $m \times n$ matrix once.
- **Space Complexity**: $O(m \times n)$
    - We allocate an $(m+1) \times (n+1)$ state array.

---

## Alternative Solution: Row-by-Row 1D Prefix Sum

If we want to simplify the precomputation logic or if the number of rows is small, we can store a 1D prefix sum array for each row instead of a full 2D prefix sum grid.

### Thought Process

1.  **Row-by-Row 1D Prefix Sums**: For each row `r`, we precompute the 1D prefix sum of its elements:
    *   `prefixSums[r][c+1] = prefixSums[r][c] + matrix[r][c]`
2.  **Querying**: When `SumRegion(row1, col1, row2, col2)` is called, we iterate through each row `r` from `row1` to `row2` and calculate the range sum for that row in $O(1)$ time:
    *   `rowSum = prefixSums[r][col2+1] - prefixSums[r][col1]`
3.  **Trade-Off**: 
    *   **Pros**: Simpler DP/prefix sum recurrence relation.
    *   **Cons**: Query time is $O(m)$ (where $m$ is the number of rows in the region) instead of $O(1)$.

### Go Code

``` go
type NumMatrix struct {
    prefixSums [][]int
}

func Constructor(matrix [][]int) NumMatrix {
    if len(matrix) == 0 || len(matrix[0]) == 0 {
        return NumMatrix{}
    }
    ROWS, COLS := len(matrix), len(matrix[0])
    prefixSums := make([][]int, ROWS)
    for r := 0; r < ROWS; r++ {
        prefixSums[r] = make([]int, COLS+1)
        for c := 0; c < COLS; c++ {
            prefixSums[r][c+1] = prefixSums[r][c] + matrix[r][c]
        }
    }
    return NumMatrix{prefixSums: prefixSums}
}

func (this *NumMatrix) SumRegion(row1 int, col1 int, row2 int, col2 int) int {
    sum := 0
    for r := row1; r <= row2; r++ {
        sum += this.prefixSums[r][col2+1] - this.prefixSums[r][col1]
    }
    return sum
}
```

### Code Efficiency

- **Time Complexity**:
    - **Constructor**: $O(m \times n)$ to compute the prefix sums.
    - **SumRegion**: $O(m)$ where $m$ is the number of rows (`row2 - row1 + 1`).
- **Space Complexity**: $O(m \times n)$ to store the prefix sums of all rows.
