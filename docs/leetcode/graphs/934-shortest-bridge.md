# 934. Shortest Bridge

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/shortest-bridge/description/)

## Solution: DFS + BFS (Multi-Source Breadth-First Search)

To find the minimum number of water cells (`0`) we must flip to bridge two islands of land (`1`), we can combine DFS and BFS:
1.  **DFS (Island Marking)**: Scan the grid to find the first island. Perform a DFS to label all cells of this island as `2` (to distinguish them from the second island) and add them to a queue.
2.  **BFS (Outward Expansion)**: Run a multi-source BFS starting from the cells of the first island (all elements in the queue). The first time the BFS reaches a cell with value `1` (the second island), the number of steps traversed represents the shortest bridge.

### Thought Process

1.  **Locate and Color the First Island**:
    - Iterate through the $n \times n$ grid to find the first cell containing `1`.
    - Initiate a DFS from this cell. The DFS function:
        - Changes the grid value from `1` to `2` to mark it as visited and distinct.
        - Appends the cell's coordinates as a `pair` to the BFS queue.
        - Recursively visits all adjacent 4-directional neighbors.
    - Set a flag `found = true` and terminate the outer search loops once the first island is fully mapped.
2.  **Level-Order BFS Expansion**:
    - Initialize a counter `steps = 0`.
    - Run BFS layer-by-layer:
        - For each cell in the current layer of the queue:
            - Explore its 4-directional neighbors.
            - **Target Found**: If a neighbor contains `1`, it belongs to the second island. Return `steps` immediately.
            - **Water Encountered**: If a neighbor contains `0`, it is a water cell. Mark it as `2` (to avoid visiting it again) and append it to the queue for the next layer.
        - Increment `steps` after the current layer is fully processed.

### Go Code

``` go
type pair struct {
    r, c int
}

func shortestBridge(grid [][]int) int {
    n := len(grid)
    queue := make([]pair, 0)

    dr := []int{-1, 0, 1, 0}
    dc := []int{0, 1, 0, -1}

    var dfs func(r, c int)
    dfs = func(r, c int) {
        if r < 0 || r >= n || c < 0 || c >= n || grid[r][c] != 1 {
            return
        }
        grid[r][c] = 2
        queue = append(queue, pair{r, c})

        for i := 0; i < 4; i++ {
            dfs(r+dr[i], c+dc[i])
        }
    }

    found := false
    for r := 0; r < n; r++ {
        for c := 0; c < n; c++ {
            if grid[r][c] == 1 {
                dfs(r, c)
                found = true
                break
            }
        }
        if found {
            break
        }
    }

    steps := 0
    for len(queue) > 0 {
        size := len(queue)
        for i := 0; i < size; i++ {
            curr := queue[0]
            queue = queue[1:]

            for d := 0; d < 4; d++ {
                nr := curr.r + dr[d]
                nc := curr.c + dc[d]

                if nr < 0 || nr >= n || nc < 0 || nc >= n {
                    continue
                }

                if grid[nr][nc] == 1 {
                    return steps
                }

                if grid[nr][nc] == 0 {
                    grid[nr][nc] = 2
                    queue = append(queue, pair{nr, nc})
                }
            }
        }
        steps++
    }
    return -1
}
```

### Code Efficiency

- **Time Complexity**: $O(n^2)$
    - Where $n \times n$ is the size of the grid. The DFS visits all cells of the first island. The BFS traverses the remaining cells of the grid at most once. Hence, the overall time complexity is linear with respect to the number of cells, $O(n^2)$.
- **Space Complexity**: $O(n^2)$
    - The recursion stack for DFS can go up to $O(n^2)$ deep in the worst case (e.g., a single-cell-wide island winding through the entire grid).
    - The BFS queue can store up to $O(n^2)$ coordinate pairs.