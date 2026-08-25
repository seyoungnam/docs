# 329. Longest Increasing Path in a Matrix

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/longest-increasing-path-in-a-matrix/description/)

Given an `m x n` integers `matrix`, return *the length of the longest increasing path in* `matrix`.

From each cell, you can either move in four directions: left, right, up, or down. You **may not** move diagonally or move outside the boundary (i.e., wrap-around is not allowed).

---

## Solution: DFS with Memoization (DAG Traversal)

Because any valid path must be strictly increasing, it is impossible to have cycles. This allows us to treat the grid as a Directed Acyclic Graph (DAG) and find the longest path using Depth-First Search (DFS) with Memoization.

### Thought Process

1.  **DP State**:
    *   Let `memo[r][c]` represent the length of the longest increasing path starting from cell `(r, c)`.
    *   Create a 2D slice `memo` of size $M \times N$ initialized to `-1`.
2.  **DFS Helper Function**:
    *   Define `dfs(r, c, currVal)` which calculates the longest increasing path from `(r, c)` given that the previous cell's value was `currVal`.
    *   **Base Cases**:
        *   If coordinates `(r, c)` are out of bounds, or if `matrix[r][c] <= currVal`, the path cannot be extended. Return `0`.
        *   If `memo[r][c] != -1`, return the cached result.
    *   **Recursion**:
        *   If the current cell is valid, the minimum path length starting here is 1. We explore the 4 cardinal neighbors (down, up, right, left) and take the maximum path length:
            $$\text{memo}[r][c] = \max(\text{memo}[r][c], 1 + \text{dfs}(r \pm 1, c \pm 1, \text{matrix}[r][c]))$$
        *   Return `memo[r][c]`.
3.  **Simulation**:
    *   Iterate through all coordinates `(r, c)` in the matrix.
    *   Call `dfs(r, c, -1)` to get the longest path starting at `(r, c)` (we use `-1` as the initial previous value since all matrix values are non-negative).
    *   Maintain the global maximum path length `res`.

### Go Code

``` go
func longestIncreasingPath(matrix [][]int) int {
    ROWS, COLS := len(matrix), len(matrix[0])
    
    // Initialize memoization table
    memo := make([][]int, ROWS)
    for r := range memo {
        memo[r] = make([]int, COLS)
        for c := range memo[r] {
            memo[r][c] = -1
        }
    }

    var dfs func(r int, c int, curr int) int
    dfs = func(r int, c int, curr int) int {
        // Base case: out of bounds or not strictly increasing
        if r < 0 || r >= ROWS || c < 0 || c >= COLS || matrix[r][c] <= curr {
            return 0
        }
        
        // Return cached result if already calculated
        if memo[r][c] != -1 {
            return memo[r][c]
        }
        
        next := matrix[r][c]
        
        // Explore 4 directions and store the maximum path length
        memo[r][c] = max(memo[r][c], 1 + dfs(r+1, c, next)) 
        memo[r][c] = max(memo[r][c], 1 + dfs(r-1, c, next)) 
        memo[r][c] = max(memo[r][c], 1 + dfs(r, c+1, next)) 
        memo[r][c] = max(memo[r][c], 1 + dfs(r, c-1, next)) 
        
        return memo[r][c]
    }
    
    res := 0
    for r := range matrix {
        for c := range matrix[r] {
            res = max(res, dfs(r, c, -1))
        }
    }
    
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(M \times N)$
    - Where $M$ is the number of rows and $N$ is the number of columns. Thanks to memoization, the DFS is calculated at most once for each cell. Each cell explores its 4 neighbors, taking $O(1)$ operations per state.
- **Space Complexity**: $O(M \times N)$
    - We use $O(M \times N)$ space for the 2D memoization slice and $O(M \times N)$ space for the recursion call stack in the worst case (when the matrix is sorted in a snake-like increasing path).