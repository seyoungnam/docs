# 62. Unique Paths

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/unique-paths/description/)

There is a robot on an `m x n` grid. The robot is initially located at the **top-left corner** (i.e., `grid[0][0]`). The robot tries to move to the **bottom-right corner** (i.e., `grid[m - 1][n - 1]`). The robot can only move either down or right at any point in time.

Given the two integers `m` and `n`, return *the number of possible unique paths that the robot can take to reach the bottom-right corner*.

The test cases are generated so that the answer will be less than or equal to $2 \times 10^9$.

---

## Solution 1: Bottom-Up 2D Dynamic Programming

We can solve this problem using 2D Dynamic Programming. To reach any cell `(r, c)`, the robot must have come from either the cell directly above it `(r - 1, c)` or the cell directly to the left of it `(r, c - 1)`.

### Thought Process

1.  **DP Grid Setup**:
    *   Create a 2D slice `grid` of size $M \times N$.
2.  **Base Cases**:
    *   For cells in the first row (`r == 0`) or the first column (`c == 0`), there is only 1 unique path to reach them (by traveling strictly right or strictly down, respectively). Set `grid[r][c] = 1`.
3.  **State Transition**:
    *   For all other cells `(r, c)` where $r > 0$ and $c > 0$:
        *   The number of paths is the sum of the paths to the cell above it and the cell to its left:
            $$\text{grid}[r][c] = \text{grid}[r-1][c] + \text{grid}[r][c-1]$$
4.  **Result**:
    *   Return `grid[m-1][n-1]`.

### Go Code

``` go
func uniquePaths(m int, n int) int {
    grid := make([][]int, m)
    for r := range grid {
        grid[r] = make([]int, n)
    }
    
    for r := range grid {
        for c := range grid[r] {
            if r == 0 || c == 0 {
                // First row/column only have 1 path
                grid[r][c] = 1
            } else {
                // Sum of path counts from top and left neighbors
                grid[r][c] = grid[r-1][c] + grid[r][c-1]
            }
        }
    }
    return grid[m-1][n-1]
}
```

### Code Efficiency

- **Time Complexity**: $O(M \times N)$
    - We iterate through all cells of the $M \times N$ grid exactly once.
- **Space Complexity**: $O(M \times N)$
    - We allocate a 2D slice of size $M \times N$ to store the path counts.

---

## Solution 2: Space-Optimized Dynamic Programming

Since the state at `grid[r][c]` only depends on the current row's left neighbor `(r, c - 1)` and the previous row's top neighbor `(r - 1, c)`, we do not need to store the entire 2D grid. We can optimize our space complexity to $O(N)$ by keeping track of only two rows: `prevRow` and `currRow`.

### Thought Process

1.  **Optimization**:
    *   Maintain `prevRow` representing the previous row's results and `currRow` representing the current row's results, both of size $N$.
2.  **Transitions**:
    *   Initialize `prevRow` with all `1`s (representing the base case for the first row).
    *   For each row $r$ from $1$ to $m-1$:
        *   Initialize `currRow[0] = 1` (the first column always has exactly 1 path).
        *   For each column $c$ from $1$ to $n-1$:
            $$\text{currRow}[c] = \text{currRow}[c-1] + \text{prevRow}[c]$$
        *   Update pointers: set `prevRow = currRow` and reinitialize `currRow` for the next row.
3.  **Result**:
    *   Return `prevRow[n-1]`.

### Go Code

``` go
func uniquePaths(m int, n int) int {
    prevRow := make([]int, n)
    for c := range prevRow {
        prevRow[c] = 1
    }
    
    currRow := make([]int, n)
    currRow[0] = 1

    for r := 1; r < m; r++ {
        for c := 1; c < n; c++ {
            currRow[c] = currRow[c-1] + prevRow[c]
        }
        prevRow = currRow
        currRow = make([]int, n)
        currRow[0] = 1
    }
    return prevRow[n-1]
}
```

### Code Efficiency

- **Time Complexity**: $O(M \times N)$
    - We iterate through the virtual grid elements in $O(M \times N)$ steps.
- **Space Complexity**: $O(N)$
    - We only store two rows of size $N$, reducing our auxiliary space complexity to $O(N)$.