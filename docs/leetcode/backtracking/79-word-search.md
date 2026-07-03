# 79. Word Search

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/word-search/description/)

## Solution: Backtracking (DFS Closure)

### Thought Process

1.  **Starting Points Search**: Iterate through every cell $(r, c)$ on the board. We start our DFS search path only if the cell's character matches the first character of the word (`board[r][c] == word[0]`).
2.  **Backtracking DFS**:
    *   **Base Cases**:
        *   If the coordinates $(r, c)$ are out of bounds, return `false`.
        *   If the character at `board[r][c]` does not match `word[i]`, return `false`.
        *   If index `i` reaches the last character of the word (`i == n - 1`), we have successfully matched the entire word; return `true`.
    *   **In-Place Visited Marking**: To prevent revisiting the same cell during the current search path, we temporarily mark `board[r][c]` with a non-alphabetic character, such as `'*'`.
    *   **Four-Directional Search**: Recursively call DFS on all four adjacent cells (down, up, right, left) with index `i + 1`. If any of these paths return `true`, we propagate the success by returning `true`.
    *   **Backtrack (Restore State)**: Restore the original character to `board[r][c]` before returning `res`, freeing up the cell for subsequent path configurations starting elsewhere.

### Go Code

``` go
func exist(board [][]byte, word string) bool {
    ROWS, COLS := len(board), len(board[0])
    n := len(word)

    var dfs func(r int, c int, i int) bool
    dfs = func(r int, c int, i int) bool {
        if r < 0 || r >= ROWS || c < 0 || c >= COLS {
            return false
        }
        if word[i] != board[r][c] {
            return false
        }
        if i == n-1 {
            return true
        }

        char := board[r][c]
        board[r][c] = '*'
        
        res := dfs(r+1, c, i+1) || 
               dfs(r-1, c, i+1) ||
               dfs(r, c+1, i+1) ||
               dfs(r, c-1, i+1)
        
        board[r][c] = char
        return res
    }
    for r := 0; r < ROWS; r++ {
        for c := 0; c < COLS; c++ {
            if board[r][c] == word[0] {
                if dfs(r, c, 0) {
                    return true
                }
            }
        }
    }
    return false
}
```

### Code Efficiency

- **Time Complexity**: $O(N \cdot 3^L)$
    - $N$ is the total number of cells in the board. We start a DFS from each cell.
    - $L$ is the length of the word. In each DFS step, we explore up to 3 directions (excluding the one we just came from).
- **Space Complexity**: $O(L)$
    - The auxiliary space is determined by the **call stack height** during recursion, which goes up to $L$ levels deep.
