# 980. Unique Paths III

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/unique-paths-iii/description/)

## Solution: Backtracking (DFS Closure)

To find the number of unique paths that visit every non-obstacle square exactly once, we can use recursive depth-first backtracking. We perform a pre-scan to count the total number of walk-able squares, and then explore paths starting from the start square, checking at the end square if we have visited all required locations.

### Thought Process

1.  **Walk-able Squares Pre-calculation**:
    *   Scan the grid to find the start square coordinates `(startR, startC)`.
    *   Count the total number of walk-able squares `emptySquares` (including the starting square `1`, empty squares `0`, and the ending square `2`).
2.  **Backtracking DFS**:
    *   We track the number of remaining squares to visit using `remain`.
    *   **Pruning & Boundaries**:
        *   If the coordinates $(r, c)$ are out of bounds, or if `grid[r][c] == -1` (obstacle or already visited), return `0` paths.
    *   **Base Case (Ending Square)**:
        *   If we reach the ending square `grid[r][c] == 2`:
            *   If `remain == 1` (meaning this destination is the final required square to complete the path), we return `1` path.
            *   Otherwise, if `remain > 1` (meaning we reached the end early without covering all empty squares), return `0` paths.
    *   **In-Place Visited Marking**:
        *   Temporarily mark `grid[r][c] = -1` to prevent revisiting the current cell.
    *   **Recursive Exploration**:
        *   Recursively search the 4 cardinal directions, passing `remain - 1`. Sum up the paths:
            $$\text{paths} = dfs(r+1, c, remain-1) + dfs(r-1, c, remain-1) + dfs(r, c+1, remain-1) + dfs(r, c-1, remain-1)$$
    *   **Backtrack**:
        *   Restore the original grid value `grid[r][c] = original` before returning `paths`.

### Go Code

``` go
func uniquePathsIII(grid [][]int) int {
    ROWS, COLS := len(grid), len(grid[0])

    startR, startC := 0, 0
    emptySquares := 0

    for r := 0; r < ROWS; r++ {
        for c := 0; c < COLS; c++ {
            if grid[r][c] == 1 {
                startR, startC = r, c
                emptySquares++
            } else if grid[r][c] == 0 {
                emptySquares++
            } else if grid[r][c] == 2 {
                emptySquares++
            }
        }
    }

    var dfs func(r int, c int, remain int) int
    dfs = func(r int, c int, remain int) int {
        if r < 0 || r >= ROWS || c < 0 || c >= COLS || grid[r][c] == -1 {
            return 0
        }

        if grid[r][c] == 2 {
            if remain == 1 {
                return 1
            }
            return 0
        }

        original := grid[r][c]
        grid[r][c] = -1

        paths := dfs(r+1, c, remain-1) +
                 dfs(r-1, c, remain-1) +
                 dfs(r, c+1, remain-1) +
                 dfs(r, c-1, remain-1)
        grid[r][c] = original
        return paths
    }
    return dfs(startR, startC, emptySquares)
}
```

### Code Efficiency

- **Time Complexity**: $O(4^{R \cdot C})$
    - Where $R$ is the number of rows and $C$ is the number of columns. At each step, we search in at most 3 directions. Since $R \times C \le 20$, the grid size is extremely small, and the total states are well within execution limits.
- **Space Complexity**: $O(R \cdot C)$
    - The auxiliary space is determined by the recursion stack depth (at most $R \times C$ levels deep in the worst case). No extra space is required for tracking visits, as cells are marked in-place on the grid.