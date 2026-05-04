# 51. N-Queens

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/n-queens/)

## Solution : Backtracking

### Thought Process

1.  **Objective**: Place n queens on an n x n chessboard such that no two queens attack each other.
2.  **Constraint Satisfaction**: A queen attacks others in the same row, column, or diagonal. 
    -   **Row-by-Row Placement**: By placing exactly one queen per row and moving to the next row, we automatically satisfy the row constraint.
    -   **Column Tracking**: Use a set (or map/boolean array) to track which columns are already occupied.
    -   **Diagonal Tracking**: 
        -   **Positive Diagonals** (bottom-left to top-right): The sum of row and column indices (r + c) is constant for all cells on the same diagonal.
        -   **Negative Diagonals** (top-left to bottom-right): The difference between row and column indices (r - c) is constant for all cells on the same diagonal.
3.  **Backtracking**:
    -   **Base Case**: If we reach row n, a valid configuration is found. Convert the current board state to the required string format and add it to the result.
    -   **Recursive Step**: For each column in the current row:
        -   Check if the column or either diagonal is already occupied.
        -   If not, place a queen, mark the column and diagonals as occupied, and recurse to the next row (r + 1).
        -   After returning, remove the queen and unmark the column/diagonals (backtrack) to explore other possibilities.

### Go Code

``` go
func solveNQueens(n int) [][]string {
    var res [][]string
    
    board := make([][]byte, n)
    for i := 0; i < n; i++ {
        board[i] = make([]byte, n)
        for j := 0; j < n; j++ {
            board[i][j] = '.'
        }
    }
    
    cols := make(map[int]bool)
    posDiag := make(map[int]bool) // r + c
    negDiag := make(map[int]bool) // r - c
    
    var backtrack func(r int)
    backtrack = func(r int) {
        if r == n {
            subset := make([]string, n)
            for i := 0; i < n; i++ {
                subset[i] = string(board[i])
            }
            res = append(res, subset)
            return
        }
        
        for c := 0; c < n; c++ {
            if cols[c] || posDiag[r+c] || negDiag[r-c] {
                continue
            }
            
            cols[c], posDiag[r+c], negDiag[r-c] = true, true, true
            board[r][c] = 'Q'
            
            backtrack(r + 1)
            
            cols[c], posDiag[r+c], negDiag[r-c] = false, false, false
            board[r][c] = '.'
        }
    }
    backtrack(0)
    return res
}
```

### Code Efficiency

- **Time Complexity**: O(N!)
    - In the first row, we have N options. In the second row, at most N-2. The number of valid placements is bounded by N!.
- **Space Complexity**: O(N^2)
    - We use an N x N board to track the state (O(N^2)).
    - The recursion stack depth is O(N).
    - The sets for tracking columns and diagonals use O(N) space.
