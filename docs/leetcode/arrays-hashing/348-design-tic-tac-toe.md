# 348. Design Tic-Tac-Toe

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/design-tic-tac-toe/description/)

## Solution: Row and Column Counters

Instead of maintaining a complete $n \times n$ grid and scanning it on every move (which would take $O(n)$ time), we can optimize the game state representation. We track the running sums of each row, column, and the two diagonals. Since Player 1 adds $+1$ and Player 2 adds $-1$ to these sums, a player wins when any sum's absolute value reaches $n$.

### Thought Process

1.  **State Representation**:
    - Assign the value `1` to Player 1 and `-1` to Player 2.
    - Keep two slices of size $n$: `rows` and `cols` to store the sum of the values in each row and column.
    - Keep two integer counters `diag` and `antiDiag` to store the sum of values on the main diagonal and anti-diagonal.
2.  **Move Execution**:
    - On a move at `(row, col)` by `player`:
        - Update `rows[row]` and `cols[col]` by adding the player's value.
        - If `row == col`, update the main diagonal `diag`.
        - If `row + col == n - 1`, update the anti-diagonal `antiDiag`.
3.  **Win Detection**:
    - Check if the absolute value of `rows[row]`, `cols[col]`, `diag`, or `antiDiag` equals $n$.
    - If any of these conditions are met, the current player has successfully occupied the entire line and wins. Return the `player`.
    - Otherwise, return `0` to indicate the game continues.

### Go Code

``` go
type TicTacToe struct {
    rows        []int
    cols        []int
    diag        int
    antiDiag    int
    n           int
}


func Constructor(n int) TicTacToe {
    return TicTacToe{
        rows: make([]int, n),
        cols: make([]int, n),
        n:  n,
    }
}


func (this *TicTacToe) Move(row int, col int, player int) int {
    value := 1
    if player == 2 {
        value = -1
    }
    this.rows[row] += value
    this.cols[col] += value

    if row == col {
        this.diag += value
    }

    if row+col == this.n - 1 {
        this.antiDiag += value
    }

    target := this.n
    if abs(this.rows[row]) == target ||
       abs(this.cols[col]) == target ||
       abs(this.diag) == target ||
       abs(this.antiDiag) == target {
        return player
    }
    return 0
}

func abs(a int) int {
    if a < 0 {
        return -a
    }
    return a
}
```

### Code Efficiency

- **Time Complexity**:
    - **`Constructor`**: $O(n)$ to allocate space for the rows and columns arrays.
    - **`Move`**: $O(1)$ constant time per move. We only perform a fixed number of slice updates and comparison checks.
- **Space Complexity**: $O(n)$
    - We use two slices of size $n$ to track row and column sums, plus a few constant-space integers.
