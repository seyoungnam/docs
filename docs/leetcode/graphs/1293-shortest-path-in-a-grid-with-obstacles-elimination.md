# 1293. Shortest Path in a Grid with Obstacles Elimination

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/shortest-path-in-a-grid-with-obstacles-elimination/description/)

## Solution: State Space BFS

### Thought Process

1.  **Shortest Path via BFS**:
    *   Since we are looking for the shortest path in an unweighted grid, Breadth-First Search (BFS) is the optimal traversal algorithm.
2.  **State Representation (State Space)**:
    *   A simple cell coordinate `(row, col)` is not enough to track visited positions because our path options depend on how many obstacles we can still eliminate.
    *   Thus, we define our state as `State{Row, Col, Quota}`, where `Quota` is the remaining obstacle elimination budget.
3.  **2D Visited Optimization (Max Quota Tracking)**:
    *   Instead of a 3D visited table `visited[ROWS][COLS][k+1]`, we can optimize space by using a 2D integer array `visited[ROWS][COLS]` initialized to `-1`.
    *   `visited[r][c]` stores the **maximum remaining quota** with which we have ever visited `(r, c)`.
    *   When we reach `(nr, nc)` with a remaining quota `nq`:
        *   If `nq > visited[nr][nc]`, this is a better path (more capability to break future walls than any previous visit). We update `visited[nr][nc] = nq` and enqueue the state.
        *   If `nq <= visited[nr][nc]`, this path is suboptimal or redundant, so we prune (skip) it.
4.  **BFS Progression**:
    *   Initialize the queue with `State{0, 0, k}`.
    *   Loop level-by-level to track `steps`.
    *   If we reach the bottom-right corner `(ROWS-1, COLS-1)`, return the current `steps` (guaranteed to be the minimum).
    *   For each neighbor:
        *   Check grid boundaries.
        *   Calculate the new quota: `nq = curr.Quota - grid[nr][nc]`.
        *   If `nq >= 0` and `nq > visited[nr][nc]`, record the visit and enqueue.
    *   If the queue becomes empty without reaching the target, return `-1`.

### Go Code

``` go
type State struct {
    Row, Col, Quota int
}

func shortestPath(grid [][]int, k int) int {
    ROWS, COLS := len(grid), len(grid[0])
    
    visited := make([][]int, ROWS)
    for r := 0; r < ROWS; r++ {
        visited[r] = make([]int, COLS)
        for c := 0; c < COLS; c++ {
            visited[r][c] = -1
        }
    }

    dr := [4]int{-1, 0, 1, 0}
    dc := [4]int{0, 1, 0, -1}

    queue := make([]State, 0)
    queue = append(queue, State{0, 0, k})
    steps := 0
    for len(queue) > 0 {
        size := len(queue)
        for range size {
            curr := queue[0]
            queue = queue[1:]

            if curr.Row == ROWS-1 && curr.Col == COLS-1 {
                return steps
            }

            for i := 0; i < 4; i++ {
                nr := curr.Row + dr[i]
                nc := curr.Col + dc[i]

                if nr < 0 || nr >= ROWS || nc < 0 || nc >= COLS {
                    continue
                }

                nq := curr.Quota - grid[nr][nc]

                if nq > visited[nr][nc] {
                    visited[nr][nc] = nq
                    queue = append(queue, State{nr, nc, nq})
                }
            }
        }
        steps++
    }
    return -1
}
```

### Code Efficiency

- **Time Complexity**: $O(R \times C \times K)$
    - $R$ and $C$ are the grid dimensions, and $K$ is the maximum allowed obstacle eliminations.
    - Each cell `(r, c)` can be visited at most $K + 1$ times (since the remaining quota must strictly increase to trigger a re-visit). Thus, the state space is $R \times C \times K$ and each transition is processed in $O(1)$ time.
- **Space Complexity**: $O(R \times C \times K)$
    - In the worst case, the BFS queue can grow up to the state space size $O(R \times C \times K)$. The 2D `visited` array requires $O(R \times C)$ auxiliary space.
