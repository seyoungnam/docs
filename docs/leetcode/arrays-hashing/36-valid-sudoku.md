# 36. Valid Sudoku

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/valid-sudoku/description/)

## Solution: Array-based Tracking (One Pass)

To validate the Sudoku board, we must ensure that there are no duplicate digits (1-9) within any row, column, or 3x3 sub-box. Since the board size and the set of possible digits are fixed, we can efficiently track seen digits using fixed-size boolean arrays instead of hash maps.

### Thought Process

1.  **State Tracking**: Create three separate 2D boolean arrays (`rows`, `cols`, `boxs`) of size `[9][9]`. 
    - The first index represents the specific row, column, or box (0 to 8).
    - The second index represents the digit encountered (mapped from 1-9 to 0-8).
2.  **Board Traversal**: Iterate over each cell `(r, c)` in the 9x9 board. Skip any empty cells (`.`).
3.  **Digit Mapping**: Convert the character digit to an integer index `val = board[r][c] - '1'`.
4.  **Box Index Calculation**: Map the 2D grid coordinates `(r, c)` to a 1D box index (0 to 8) using the formula `idx = (r/3)*3 + c/3`.
    - `r/3` determines the vertical box row (0, 1, or 2).
    - `c/3` determines the horizontal box column (0, 1, or 2).
5.  **Validation**: For the current cell:
    - Check if the digit has already been seen in its corresponding `rows`, `cols`, or `boxs`. If it has, the board is invalid; return `false`.
    - Otherwise, mark the digit as seen (`true`) in all three tracking arrays.
6.  **Completion**: If the entire board is traversed without encountering any conflicts, the Sudoku is valid; return `true`.

### Go Code

``` go
func isValidSudoku(board [][]byte) bool {
    var rows, cols, boxs [9][9]bool

    for r := 0; r < 9; r++ {
        for c := 0; c < 9; c++ {
            if board[r][c] == '.' {
                continue
            }
            val := board[r][c] - '1'
            if rows[r][val] {
                return false
            } 
            rows[r][val] = true
            
            if cols[c][val] {
                return false
            }
            cols[c][val] = true

            idx := (r/3)*3 + c/3
            if boxs[idx][val] {
                return false
            } 
            boxs[idx][val] = true
        }
    }
    return true
}
```


### Code Efficiency

- **Time Complexity**: $O(1)$
    - Since the Sudoku board is always fixed at 9x9, the number of iterations is strictly bounded to 81. Therefore, the time complexity is constant.
- **Space Complexity**: $O(1)$
    - We use three fixed-size 9x9 boolean arrays. This requires exactly 243 booleans of memory, which is a constant auxiliary space overhead regardless of the input.



