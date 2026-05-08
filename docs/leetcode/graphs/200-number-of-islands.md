# 200. Number of Islands

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/number-of-islands/description/)

## Solution: DFS (Depth-First Search)

### Thought Process

1.  **Understand the Grid**: The grid consists of '1's (land) and '0's (water). An "island" is a group of connected '1's (horizontally or vertically).
2.  **Counting Strategy**:
    *   Iterate through every cell in the $M \times N$ grid.
    *   When we encounter a '1', it marks the discovery of a new island. Increment our island counter (`res`).
    *   **Eliminate the Island**: Once an island is found, we need to "sink" or visit all connected land pieces so they aren't counted as separate islands later.
3.  **DFS Traversal**:
    *   From the starting '1', recursively visit all 4 neighbors (up, down, left, right).
    *   **Base Case**: If the neighbor is out of bounds or is '0' (water/already visited), stop.
    *   **Mark as Visited**: Change `grid[r][c]` from '1' to '0' to avoid redundant checks and infinite recursion.

### Go Code

``` go
func numIslands(grid [][]byte) int {
    ROW, COL := len(grid), len(grid[0])
    var res int
    for r := 0; r < ROW; r++ {
        for c := 0; c < COL; c++ {
            if grid[r][c] == '1' {
                dfs(grid, r, c)
                res++
            }
        }
    }
    return res
}

func dfs(grid [][]byte, r int, c int) {
    if r < 0 || r >= len(grid) || c < 0 || c >= len(grid[0]) || grid[r][c] == '0' {
        return
    }
    grid[r][c] = '0'
    
    dfs(grid, r+1, c)
    dfs(grid, r-1, c)
    dfs(grid, r, c+1)
    dfs(grid, r, c-1)
    return
}
```

### Code Efficiency

-   **Time Complexity**: $O(M \times N)$
    -   We iterate through every cell in the grid once. In the worst case (all land), the DFS will visit every cell exactly once.
-   **Space Complexity**: $O(M \times N)$
    -   In the worst case (e.g., the entire grid is land), the recursion stack for DFS can go as deep as $M \times N$.

---

## Solution: BFS (Breadth-First Search)

### Thought Process

1.  **Iterative Discovery**: Similar to the DFS approach, we iterate through the grid. When we encounter a '1', we increment our island count and trigger a BFS to "sink" all connected land.
2.  **Queue for Level-Order Traversal**:
    *   We use a queue to store the coordinates of land cells that need to be explored.
    *   Initialize the queue with the starting cell's coordinates.
3.  **Preventing Duplicates**:
    *   **Crucial Step**: Mark the cell as '0' (visited) **as soon as it is added to the queue**.
    *   If we wait until the cell is *popped* from the queue to mark it, the same land cell might be added to the queue multiple times by different neighbors, leading to redundant work or memory exhaustion.
4.  **Neighbor Exploration**: For each cell dequeued, check its 4 neighbors. If a neighbor is within bounds and is '1', add it to the queue and immediately mark it as '0'.

### Go Code

``` go
func numIslands(grid [][]byte) int {
    ROW, COL := len(grid), len(grid[0])
    var res int

    for r := 0; r < ROW; r++ {
        for c := 0; c < COL; c++ {
            if grid[r][c] == '1' {
                bfs(grid, r, c)
                res++
            }
        }
    }
    return res
}

func bfs(grid [][]byte, r int, c int) {
    ROW, COL := len(grid), len(grid[0])
    q := [][2]int{[2]int{r,c}}
    grid[r][c] = '0'
    dir := [4][2]int{{1,0}, {-1,0}, {0,1}, {0,-1}}

    for len(q) > 0 {
        cr, cc := q[0][0], q[0][1]
        q = q[1:]

        for _, d := range dir {
            nr := cr + d[0]
            nc := cc + d[1]
            if nr >= 0 && nr < ROW && nc >= 0 && nc < COL && grid[nr][nc] == '1' {
                q = append(q, [2]int{nr, nc})
                grid[nr][nc] = '0'
            }
        }
    }
}
```

### Code Efficiency

-   **Time Complexity**: $O(M \times N)$
    -   We visit every cell in the grid. Each '1' is added to and removed from the queue exactly once.
-   **Space Complexity**: $O(\min(M, N))$
    -   The maximum size of the queue is determined by the "frontier" of the search. In a 2D grid, the widest part of the BFS frontier is proportional to the minimum of the grid's dimensions.