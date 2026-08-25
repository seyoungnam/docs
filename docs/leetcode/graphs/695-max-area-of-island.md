# 695. Max Area of Island

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/max-area-of-island/description/)

You are given an `m x n` binary matrix `grid`. An island is a group of `1`s (representing land) connected **4-directionally** (horizontal or vertical). You may assume all four edges of the grid are surrounded by water.

The **area** of an island is the number of cells with a value `1` in the island.

Return *the maximum **area** of an island in* `grid`. If there is no island, return `0`.

---

## Solution: Depth-First Search (DFS)

We can solve this problem by scanning the grid cell-by-cell. When we encounter land (`1`), we trigger a Depth-First Search (DFS) to traverse the entire island, count its cells, and simultaneously sink it (set the cells to `0`) to prevent counting them again.

### Thought Process

1.  **Scanning**:
    *   Iterate through every cell $(r, c)$ in the $M \times N$ grid.
    *   When we find a cell with value `1`, it marks the discovery of an island.
2.  **Traversing and Measuring**:
    *   Call the DFS helper function starting at $(r, c)$ to compute the total area of the island.
    *   Update our global maximum area `res` with the computed area.
3.  **DFS Helper Function**:
    *   **Base Cases**: If the coordinates $(r, c)$ are out of bounds or if `grid[r][c] == 0` (water or already visited), return `0` because it contributes nothing to the area.
    *   **Mark Visited**: Set `grid[r][c] = 0` to mark it as visited and avoid infinite recursion or duplicate counting.
    *   **Recursive Step**: The total area of the island is $1$ (for the current cell) plus the recursive areas of its four neighbors (down, up, right, left):
        $$\text{area} = 1 + \text{dfs}(r+1, c) + \text{dfs}(r-1, c) + \text{dfs}(r, c+1) + \text{dfs}(r, c-1)$$
4.  **Result**:
    *   Return `res`.

### Go Code

``` go
func maxAreaOfIsland(grid [][]int) int {
    ROWS, COLS := len(grid), len(grid[0])

    var dfs func(r int, c int) int
    dfs = func(r int, c int) int {
        // Base Case: out of bounds or water
        if r < 0 || r >= ROWS || c < 0 || c >= COLS || grid[r][c] == 0 {
            return 0
        }
        
        // Sink the current cell to mark it as visited
        grid[r][c] = 0
        
        // Calculate area recursively of current cell + 4 neighbors
        return 1 + dfs(r+1, c) + dfs(r-1, c) + dfs(r, c+1) + dfs(r, c-1)
    }

    res := 0
    for r := 0; r < ROWS; r++ {
        for c := 0; c < COLS; c++ {
            if grid[r][c] == 1 {
                sum := dfs(r, c)
                res = max(res, sum)
            }
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(M \times N)$
    - Where $M$ is the number of rows and $N$ is the number of columns. Every cell in the grid is visited at most once since land cells are set to `0` immediately upon visiting.
- **Space Complexity**: $O(M \times N)$
    - In the worst case (e.g., the entire grid is a single island), the recursion stack for the DFS can grow up to $M \times N$.