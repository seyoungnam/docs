# 529. Minesweeper

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/minesweeper/description/)

## Solution 1: Depth-First Search (DFS)

We can simulate the Minesweeper click mechanics using Depth-First Search (DFS) across the 8 neighboring directions (horizontal, vertical, and diagonal).

---

### Thought Process

1. **Direct Mine Hit**:
    * If `board[click[0]][click[1]] == 'M'`, change it to `'X'` and return immediately (game over).
2. **Empty Square Processing**:
    * If the clicked cell is `'E'`, count how many of its 8 neighboring cells contain an unrevealed mine (`'M'`).
3. **State Transition Rules**:
    * **Rule 1 (Adjacent mines > 0)**:
        * Update the current cell to the corresponding digit character: `'0' + byte(count)`.
        * Stop further recursion from this cell.
    * **Rule 2 (Adjacent mines == 0)**:
        * Update the current cell to `'B'` (Blank).
        * Recursively invoke DFS on all adjacent unrevealed empty squares (`'E'`).
4. **Boundary and Visited Checks**:
    * Only recurse into neighboring cells that are within matrix bounds and currently marked as `'E'`. Setting a cell to `'B'` or a digit automatically acts as the visited marker.

---

### Go Code

```go
func updateBoard(board [][]byte, click []int) [][]byte {
    r, c := click[0], click[1]

    // Rule 1: Mine clicked
    if board[r][c] == 'M' {
        board[r][c] = 'X'
        return board
    }

    m, n := len(board), len(board[0])
    dirs := [][2]int{
        {-1, -1}, {-1, 0}, {-1, 1},
        {0, -1},           {0, 1},
        {1, -1},  {1, 0},  {1, 1},
    }

    var dfs func(row, col int)
    dfs = func(row, col int) {
        // Step 1: Count adjacent mines in 8 directions
        mineCount := 0
        for _, d := range dirs {
            nr, nc := row+d[0], col+d[1]
            if nr >= 0 && nr < m && nc >= 0 && nc < n && board[nr][nc] == 'M' {
                mineCount++
            }
        }

        if mineCount > 0 {
            // Rule 2: Cell has adjacent mines -> set number and stop
            board[row][col] = '0' + byte(mineCount)
        } else {
            // Rule 3: Blank square -> set 'B' and recursively reveal neighbors
            board[row][col] = 'B'
            for _, d := range dirs {
                nr, nc := row+d[0], col+d[1]
                if nr >= 0 && nr < m && nc >= 0 && nc < n && board[nr][nc] == 'E' {
                    dfs(nr, nc)
                }
            }
        }
    }

    dfs(r, c)
    return board
}
```

---

### Code Efficiency

- **Time Complexity**: $O(M \times N)$
    - Where $M$ is the number of rows and $N$ is the number of columns. Each cell is visited at most once, and checking 8 neighbors takes $O(1)$ constant time per cell.
- **Space Complexity**: $O(M \times N)$
    - In the worst case (e.g. an entire board of empty cells with no mines), the recursion stack can grow up to $M \times N$.

---

## Solution 2: Breadth-First Search (BFS)

We can also implement the reveal process iteratively using a queue.

---

### Thought Process

1. If the initial click is a mine (`'M'`), set to `'X'` and return.
2. Initialize a queue with `[click[0], click[1]]`.
3. To prevent duplicate enqueuing of the same cell, mark cells when they are enqueued or when popped:
    * Dequeue `[r, c]`.
    * Count adjacent mines in all 8 directions.
    * If `mineCount > 0`, set `board[r][c] = '0' + byte(mineCount)`.
    * If `mineCount == 0`, set `board[r][c] = 'B'`, and enqueue all valid neighboring `'E'` cells, immediately setting them to `'B'` to avoid re-enqueuing.

---

### Go Code

```go
func updateBoard(board [][]byte, click []int) [][]byte {
    r, c := click[0], click[1]

    if board[r][c] == 'M' {
        board[r][c] = 'X'
        return board
    }

    m, n := len(board), len(board[0])
    dirs := [][2]int{
        {-1, -1}, {-1, 0}, {-1, 1},
        {0, -1},           {0, 1},
        {1, -1},  {1, 0},  {1, 1},
    }

    queue := [][2]int{{r, c}}

    for len(queue) > 0 {
        curr := queue[0]
        queue = queue[1:]
        row, col := curr[0], curr[1]

        // Count adjacent mines
        mineCount := 0
        for _, d := range dirs {
            nr, nc := row+d[0], col+d[1]
            if nr >= 0 && nr < m && nc >= 0 && nc < n && board[nr][nc] == 'M' {
                mineCount++
            }
        }

        if mineCount > 0 {
            board[row][col] = '0' + byte(mineCount)
        } else {
            board[row][col] = 'B'
            for _, d := range dirs {
                nr, nc := row+d[0], col+d[1]
                if nr >= 0 && nr < m && nc >= 0 && nc < n && board[nr][nc] == 'E' {
                    board[nr][nc] = 'B' // Mark immediately to prevent duplicate queue entries
                    queue = append(queue, [2]int{nr, nc})
                }
            }
        }
    }

    return board
}
```

---

### Code Efficiency

- **Time Complexity**: $O(M \times N)$
    - Each cell is enqueued and processed at most once.
- **Space Complexity**: $O(M \times N)$
    - The queue holds at most $O(M \times N)$ coordinates in memory.