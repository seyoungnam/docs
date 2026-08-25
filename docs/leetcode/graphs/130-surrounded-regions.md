# 130. Surrounded Regions

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/surrounded-regions/description/)

Given an `m x n` matrix `board` containing `'X'` and `'O'`, *capture all regions that are 4-directionally surrounded by* `'X'`.

A region is **captured** by flipping all `'O'`s into `'X'`s in that surrounded region.

---

## Solution: Border-to-Interior Depth-First Search (DFS)

An `'O'` cell (or a region of `'O'`s) is surrounded if it has no path to any of the four borders of the board. Instead of searching from each interior region to check if it can reach the borders, we can work backwards from the borders.

### Thought Process

We solve the problem in three sequential stages:

1.  **Phase 1: Mark Border-Connected Cells (Unsurroundable)**:
    *   Iterate along all four borders (first/last rows, first/last columns).
    *   If we find an `'O'`, it is connected to the border and cannot be captured.
    *   We run a DFS to find all connected `'O'` cells and temporarily mark them with a sentinel character `'*'`.
2.  **Phase 2: Capture Surrounded Cells**:
    *   Iterate through the entire board. Any remaining `'O'` in the interior was not reachable from the borders.
    *   Trigger a DFS to flip these surrounded `'O'` cells to `'X'`.
3.  **Phase 3: Restore Border-Connected Cells**:
    *   Iterate through the board and locate the sentinel `'*'` cells.
    *   Trigger a DFS to restore them back to their original `'O'` state.

### Go Code

``` go
func solve(board [][]byte)  {
    ROWS, COLS := len(board), len(board[0])

    // DFS helper that replaces contiguous "target" bytes with "goal" bytes
    var dfs func(r int, c int, target byte, goal byte)
    dfs = func(r int, c int, target byte, goal byte) {
        if r < 0 || r >= ROWS || c < 0 || c >= COLS || board[r][c] != target {
            return
        }
        board[r][c] = goal
        dfs(r+1, c, target, goal)
        dfs(r-1, c, target, goal)
        dfs(r, c+1, target, goal)
        dfs(r, c-1, target, goal)
    }

    // Step 1: Mark border-connected 'O's as '*'
    for r := 0; r < ROWS; r++ {
        for c := 0; c < COLS; c++ {
            if (r == 0 || r == ROWS-1 || c == 0 || c == COLS-1) && board[r][c] == 'O' {
                dfs(r, c, 'O', '*')
            }
        }
    }
    
    // Step 2: Flip remaining surrounded 'O's to 'X'
    for r := 0; r < ROWS; r++ {
        for c := 0; c < COLS; c++ {
            if board[r][c] == 'O' {
                dfs(r, c, 'O', 'X')
            }
        }
    }
    
    // Step 3: Revert '*' back to 'O'
    for r := 0; r < ROWS; r++ {
        for c := 0; c < COLS; c++ {
            if board[r][c] == '*' {
                dfs(r, c, '*', 'O')
            }
        }
    }
}
```

### Code Efficiency

- **Time Complexity**: $O(M \times N)$
    - Where $M$ is the number of rows and $N$ is the number of columns. Each cell is traversed a constant number of times during the scans and DFS recursive steps.
- **Space Complexity**: $O(M \times N)$
    - In the worst case (e.g., the entire board is filled with `'O'`), the recursion stack for DFS can go as deep as $M \times N$.