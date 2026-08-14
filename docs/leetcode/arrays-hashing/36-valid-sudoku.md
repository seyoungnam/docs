# 36. Valid Sudoku

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/valid-sudoku/description/)

## Solution: Array-based Tracking (One Pass)

To validate the Sudoku board, we must ensure that there are no duplicate digits (1-9) within any row, column, or 3x3 sub-box. Since the board size and the set of possible digits are fixed, we can efficiently track seen digits using fixed-size boolean arrays instead of hash maps.

### Thought Process

1.  **State Tracking**:
    *   Create three separate 2D boolean arrays `var rows, cols, boxs [9][9]bool`.
    *   The first dimension represents the specific row `r`, column `c`, or 3x3 sub-box `j` (0 to 8).
    *   The second dimension represents the 0-indexed digit `i` (mapped from `'1'`–`'9'` to `0`–`8`).
2.  **Board Traversal**:
    *   Iterate over every cell `(r, c)` on the 9x9 board. Skip any empty cells (`'.'`).
3.  **Digit Mapping**:
    *   Convert the character digit into a 0-indexed integer: `i := board[r][c] - '1'`.
4.  **3x3 Sub-box Indexing**:
    *   Calculate the 1D box index `j` (from 0 to 8) using integer division:
        $$j = \left(\lfloor r / 3 \rfloor \times 3\right) + \lfloor c / 3 \rfloor$$
    *   `r/3` determines the vertical box block (0, 1, or 2).
    *   `c/3` determines the horizontal box block (0, 1, or 2).
5.  **Conflict Checking**:
    *   If the digit `i` has already been seen in the current row (`rows[r][i]`), column (`cols[c][i]`), or sub-box (`boxs[j][i]`), return `false`.
    *   Otherwise, mark `rows[r][i] = true`, `cols[c][i] = true`, and `boxs[j][i] = true`.
6.  **Result**:
    *   If all cells are checked without conflicts, return `true`.

### Go Code

``` go
func isValidSudoku(board [][]byte) bool {
    var rows, cols, boxs [9][9]bool
    for r := range board {
        for c := range board[r] {
            if board[r][c] == '.' {
                continue
            }
            i := board[r][c] - '1'
            j := (r/3)*3 + c/3
            if rows[r][i] || cols[c][i] || boxs[j][i] {
                return false
            }
            rows[r][i] = true
            cols[c][i] = true
            boxs[j][i] = true
        }
    }
    return true
}
```

### Code Efficiency

- **Time Complexity**: $O(1)$
    - The board is always fixed at 9x9 ($81$ cells). Iterating through every cell takes a constant $81$ iterations with $O(1)$ lookups and updates.
- **Space Complexity**: $O(1)$
    - We use three fixed-size `[9][9]bool` arrays, which take exactly $3 \times 9 \times 9 = 243$ bytes of memory. Auxiliary space is constant.
