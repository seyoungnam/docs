# 37. Sudoku Solver

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/sudoku-solver/description/)

## Solution: Backtracking (DFS Closure with Array State Lookups)

Solving a Sudoku board requires filling in empty cells with digits $1$ to $9$ such that each row, column, and $3 \times 3$ sub-grid contains all digits without repetition. We can solve this using recursive backtracking, optimized with constant-time lookup arrays to track placed digits.

### Thought Process

1.  **State Representation ($O(1)$ Lookups)**:
    *   Rather than scanning rows, columns, or box grids repeatedly to check if a digit is valid, we use three tracking arrays:
        *   `rows [9][10]bool`: `rows[r][d]` is `true` if digit `d` exists in row `r`.
        *   `cols [9][10]bool`: `cols[c][d]` is `true` if digit `d` exists in column `c`.
        *   `boxs [9][10]bool`: `boxs[i][d]` is `true` if digit `d` exists in box `i`.
    *   The sub-box index $i$ ($0 \le i < 9$) is mapped from grid cell $(r, c)$ using:
        $$i = \left(\lfloor r / 3 \rfloor \times 3\right) + \lfloor c / 3 \rfloor$$
2.  **State Initialization**:
    *   Perform a single pass of the board to pre-populate the `rows`, `cols`, and `boxs` state tables with the numbers already present.
3.  **Recursive DFS (Cell-by-Cell)**:
    *   **Base Cases**:
        *   If column `c == 9`, we have finished the current row. Recurse to the next row starting at column 0: `dfs(r + 1, 0)`.
        *   If row `r == 9`, we have successfully filled the entire board. Return `true` to stop searching.
        *   If `board[r][c]` is not empty (i.e. not `'.'`), skip it and recursively call `dfs(r, c + 1)`.
    *   **Exploring Choices**:
        *   For an empty cell, loop digit `d` from $1$ to $9$:
            *   If `d` is already used in row `r`, column `c`, or sub-box `i`, skip it.
            *   Otherwise, place `d`:
                1. Update the cell: `board[r][c] = byte(d + '0')`.
                2. Mark `d` as used in `rows`, `cols`, and `boxs`.
                3. Recurse: `dfs(r, c + 1)`. If it returns `true`, return `true` immediately.
                4. Backtrack: Reset `board[r][c] = '.'` and mark `d` as unused in the state arrays.
4.  **Termination**:
    *   If no digit leads to a successful layout, return `false`.

### Go Code

``` go
func solveSudoku(board [][]byte)  {
    var rows, cols, boxs [9][10]bool

    for r := 0; r < 9; r++ {
        for c := 0; c < 9; c++ {
            if board[r][c] != '.' {
                d := board[r][c] - '0'
                i := (r/3)*3 + c/3

                rows[r][d] = true
                cols[c][d] = true
                boxs[i][d] = true
            }
        }
    }

    var dfs func(r, c int) bool
    dfs = func(r, c int) bool {
        if c == 9 {
            return dfs(r+1, 0)
        }
        if r == 9 {
            return true
        }
        if board[r][c] != '.' {
            return dfs(r, c+1)
        }
        i := (r/3)*3 + c/3

        for d := 1; d <= 9; d++ {
            if rows[r][d] || cols[c][d] || boxs[i][d] {
                continue
            }

            rows[r][d] = true
            cols[c][d] = true
            boxs[i][d] = true
            board[r][c] = byte(d + '0')

            if dfs(r, c+1) {
                return true
            }

            rows[r][d] = false
            cols[c][d] = false
            boxs[i][d] = false
            board[r][c] = '.'
        }
        return false
    }
    dfs(0, 0)
}
```

### Code Efficiency

- **Time Complexity**: $O(9^m)$ where $m$ is the number of empty cells on the board (at most $81$). In practice, the search space is heavily pruned due to Sudoku constraints, and using $O(1)$ tracking array lookups ensures the search performs extremely fast.
- **Space Complexity**: $O(1)$ constant auxiliary space.
    - The state arrays `rows`, `cols`, and `boxs` are of fixed size.
    - The recursion call stack depth goes up to a maximum of $81$ levels (corresponding to the cells on the board), which requires constant space.