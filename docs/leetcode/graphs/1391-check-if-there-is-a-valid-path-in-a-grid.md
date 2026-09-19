# 1391. Check if There is a Valid Path in a Grid

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/check-if-there-is-a-valid-path-in-a-grid/description/)

Given an `m x n` `grid`. Each cell of `grid` represents a street. The street of `grid[i][j]` can be:
- `1` which means a street connecting the left cell and the right cell.
- `2` which means a street connecting the upper cell and the lower cell.
- `3` which means a street connecting the left cell and the lower cell.
- `4` which means a street connecting the right cell and the lower cell.
- `5` which means a street connecting the left cell and the upper cell.
- `6` which means a street connecting the right cell and the upper cell.

You will start at the upper left cell `(0, 0)` and you want to reach the lower right cell `(m - 1, n - 1)`. You can only move along the street.

Return `true` *if there is a valid path in the grid, or* `false` *otherwise*.

---

## Solution 1: DFS (Depth-First Search)

### Thought Process

1. **Direction Mapping**:
    * Represent the 4 cardinal directions using indices:
        * `0`: Up (`[-1, 0]`)
        * `1`: Right (`[0, 1]`)
        * `2`: Down (`[1, 0]`)
        * `3`: Left (`[0, -1]`)
    * The opposite direction when entering an adjacent cell is given by `(d + 2) % 4`.
2. **Street Port Configuration**:
    * Create a lookup table `pipeDirs` where index `1` to `6` represents each street's two open directions:
        * Street 1: `{3, 1}` (Left, Right)
        * Street 2: `{0, 2}` (Up, Down)
        * Street 3: `{3, 2}` (Left, Down)
        * Street 4: `{1, 2}` (Right, Down)
        * Street 5: `{0, 3}` (Up, Left)
        * Street 6: `{0, 1}` (Up, Right)
3. **Connectivity Verification**:
    * When moving from `(r, c)` along direction `d` to `(nr, nc)`:
        * `(nr, nc)` must be within grid boundaries and unvisited.
        * The neighboring street at `(nr, nc)` must have an opening in the `oppositeDir = (d + 2) % 4` to complete the pipe connection.
4. **DFS Traversal**:
    * Start at `(0, 0)`.
    * Mark `visited[r][c] = true`.
    * If `r == m - 1 && c == n - 1`, return `true`.
    * Recursively explore all valid connected neighbors.

### Go Code

```go
func hasValidPath(grid [][]int) bool {
    m, n := len(grid), len(grid[0])

    // Directions: 0 = Up, 1 = Right, 2 = Down, 3 = Left
    dirs := [4][2]int{{-1, 0}, {0, 1}, {1, 0}, {0, -1}}

    // pipeDirs maps each street type (1-indexed) to its two open directions
    pipeDirs := [7][2]int{
        {},       // 0 (unused)
        {3, 1},   // 1: Left <-> Right
        {0, 2},   // 2: Up <-> Down
        {3, 2},   // 3: Left <-> Down
        {1, 2},   // 4: Right <-> Down
        {0, 3},   // 5: Up <-> Left
        {0, 1},   // 6: Up <-> Right
    }

    visited := make([][]bool, m)
    for i := range visited {
        visited[i] = make([]bool, n)
    }

    var dfs func(r, c int) bool
    dfs = func(r, c int) bool {
        if r == m-1 && c == n-1 {
            return true
        }

        visited[r][c] = true

        for _, d := range pipeDirs[grid[r][c]] {
            nr, nc := r+dirs[d][0], c+dirs[d][1]

            // Boundary and visited check
            if nr < 0 || nr >= m || nc < 0 || nc >= n || visited[nr][nc] {
                continue
            }

            // The adjacent cell must have an opening in the opposite direction
            oppositeDir := (d + 2) % 4
            canConnect := false
            for _, nd := range pipeDirs[grid[nr][nc]] {
                if nd == oppositeDir {
                    canConnect = true
                    break
                }
            }

            if canConnect && dfs(nr, nc) {
                return true
            }
        }

        return false
    }

    return dfs(0, 0)
}
```

### Code Efficiency

- **Time Complexity**: $O(M \times N)$
    - Each cell has at most 2 outgoing street directions. Each cell is visited at most once, and connection checks take $O(1)$ time.
- **Space Complexity**: $O(M \times N)$
    - The `visited` 2D array requires $O(M \times N)$ space. In the worst case (e.g., a winding path visiting all cells), the DFS call stack takes $O(M \times N)$ space.

---

## Solution 2: BFS (Breadth-First Search)

### Thought Process

1. **Queue Traversal**:
    * Use a queue `queue := [][2]int{{0, 0}}` to explore reachable cells level by level.
    * Mark `visited[0][0] = true` initially.
2. **Step-by-Step Expansion**:
    * Dequeue `[r, c]`. If `r == m - 1 && c == n - 1`, target is reached, return `true`.
    * For each direction `d` allowed by `grid[r][c]`, compute `(nr, nc)`.
    * If `(nr, nc)` is valid, unvisited, and connects via `oppositeDir = (d + 2) % 4`:
        * Mark `visited[nr][nc] = true`.
        * Enqueue `[nr, nc]`.
3. **Iterative Advantage**:
    * Replaces recursion with an iterative queue, preventing potential call stack overflow.

### Go Code

```go
func hasValidPath(grid [][]int) bool {
    m, n := len(grid), len(grid[0])

    // Directions: 0 = Up, 1 = Right, 2 = Down, 3 = Left
    dirs := [4][2]int{{-1, 0}, {0, 1}, {1, 0}, {0, -1}}

    // pipeDirs maps each street type (1-indexed) to its two open directions
    pipeDirs := [7][2]int{
        {},       // 0 (unused)
        {3, 1},   // 1: Left <-> Right
        {0, 2},   // 2: Up <-> Down
        {3, 2},   // 3: Left <-> Down
        {1, 2},   // 4: Right <-> Down
        {0, 3},   // 5: Up <-> Left
        {0, 1},   // 6: Up <-> Right
    }

    visited := make([][]bool, m)
    for i := range visited {
        visited[i] = make([]bool, n)
    }

    queue := [][2]int{{0, 0}}
    visited[0][0] = true

    for len(queue) > 0 {
        curr := queue[0]
        queue = queue[1:]
        r, c := curr[0], curr[1]

        if r == m-1 && c == n-1 {
            return true
        }

        for _, d := range pipeDirs[grid[r][c]] {
            nr, nc := r+dirs[d][0], c+dirs[d][1]

            // Boundary and visited check
            if nr < 0 || nr >= m || nc < 0 || nc >= n || visited[nr][nc] {
                continue
            }

            // The adjacent cell must have an opening in the opposite direction
            oppositeDir := (d + 2) % 4
            canConnect := false
            for _, nd := range pipeDirs[grid[nr][nc]] {
                if nd == oppositeDir {
                    canConnect = true
                    break
                }
            }

            if canConnect {
                visited[nr][nc] = true
                queue = append(queue, [2]int{nr, nc})
            }
        }
    }

    return false
}
```

### Code Efficiency

- **Time Complexity**: $O(M \times N)$
    - Each cell is pushed and popped from the queue at most once. Connectivity checks for each direction take $O(1)$ time.
- **Space Complexity**: $O(M \times N)$
    - The `visited` matrix and BFS queue require $O(M \times N)$ auxiliary space.