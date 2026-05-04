# 79. Word Search

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/word-search/description/)

## Solution : Backtracking

### Thought Process

1.  **Iterate through the board**: Start a search from every cell $(r, c)$ in the board.
2.  **Backtracking (DFS)**:
    -   **Base Cases**:
        -   If the coordinates are out of bounds, return `false`.
        -   If the current cell's character doesn't match the word's character at index `i`, return `false`.
        -   If `i` is the last index of the word and it matches, return `true`.
    -   **Mark as visited**: To avoid using the same cell twice in one word, temporarily modify the board value (e.g., to a non-character like `'1'`).
    -   **Explore Neighbors**: Recursively call DFS for top, bottom, left, and right neighbors with `i + 1`. If any direction returns `true`, the word is found.
    -   **Backtrack**: After exploring all directions, restore the original character in `board[r][c]` so it can be used for other potential paths starting from different cells.

### Go Code

``` go
func exist(board [][]byte, word string) bool {
    ROW, COL := len(board), len(board[0])
    for r := 0; r < ROW; r++ {
        for c := 0; c < COL; c++ {
            if dfs(board, r, c, 0, word) {
               return true 
            }
        }
    }
    return false
}

func dfs(board [][]byte, r int, c int, i int, word string) bool {
    if r < 0 || r >= len(board) || c < 0 || c >= len(board[0]) {
        return false
    }
    if board[r][c] != word[i] {
        return false
    }
    if i == len(word)-1 {
        return true
    }

    original := board[r][c]
    board[r][c] = '1'

    res := dfs(board, r+1, c, i+1, word) ||
           dfs(board, r-1, c, i+1, word) ||
           dfs(board, r, c+1, i+1, word) ||
           dfs(board, r, c-1, i+1, word)

    board[r][c] = original
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N \cdot 3^L)$
    - $N$ is the total number of cells in the board. We start a DFS from each cell.
    - $L$ is the length of the word. In each DFS step, we explore up to 3 directions (excluding the one we just came from).
- **Space Complexity**: $O(L)$
    - The auxiliary space is determined by the **call stack height** during recursion, which goes up to $L$ levels deep.
