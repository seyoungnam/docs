# 864. Shortest Path to Get All Keys

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/shortest-path-to-get-all-keys/description/)

## Solution: State Space BFS (Bitmasking)

### Thought Process

1.  **Shortest Path via BFS**:
    *   We want to find the shortest path in an unweighted grid, which makes Breadth-First Search (BFS) the natural algorithm choice.
2.  **State Representation (State Space)**:
    *   A simple grid coordinate `(row, col)` is insufficient for tracking visited states because our ability to pass through locks depends on the keys we have collected.
    *   To represent this, we expand our state to include the set of collected keys: `State{Row, Col, Keys}`.
3.  **Bitmasking for Keys**:
    *   Since there are at most 6 keys (`a` to `f`), we can represent the set of collected keys as a 6-bit integer bitmask (ranging from `0` to `2^6 - 1 = 63`).
    *   If we find key `char`, we set its bit: `keys |= (1 << (char - 'a'))`.
    *   If we find lock `CHAR`, we check if we have the key: `(keys & (1 << (CHAR - 'A'))) != 0`.
4.  **Visited Tracking**:
    *   We use a 3D visited table `visited[ROWS][COLS][64]` to check if we have reached `(row, col)` with a specific set of keys. 
    *   Visiting the same cell with *more* keys is allowed and necessary to unlock locks elsewhere.
5.  **BFS Execution**:
    *   Find the start position `@` and the target bitmask containing all keys (`allKeys`).
    *   Initialize the queue with `State{startRow, startCol, 0}`.
    *   Traverse neighbors:
        *   Ignore walls (`#`).
        *   If the cell is a lock (`A-F`), only pass if we hold the matching key bit.
        *   If the cell is a key (`a-f`), update the bitmask.
        *   If the new state has not been visited, mark it visited and enqueue.
    *   Return the number of `steps` as soon as we pop a state where `keys == allKeys`. If the queue empties without finding all keys, return `-1`.

### Go Code

``` go
type State struct {
    Row, Col, Keys int
}

func shortestPathAllKeys(grid []string) int {
    ROWS, COLS := len(grid), len(grid[0])
    
    startRow, startCol := -1 , -1 
    
    // get allKeys, startRow, startCol
    var allKeys int = 0
    for r := 0; r < ROWS; r++ {
        for c := 0; c < COLS; c++ {
            char := grid[r][c]
            if char >= 'a' && char <= 'f' {
                allKeys |= (1 << (char - 'a'))
            } else if char == '@' {
                startRow, startCol = r, c
            }
        }
    }

    queue := make([]State, 0)
    queue = append(queue, State{Row:startRow, Col:startCol, Keys:0})

    dr := [4]int{-1, 0, 1, 0}
    dc := [4]int{0, 1, 0, -1}

    visited := make([][][]bool, ROWS)
    for r := range visited {
        visited[r] = make([][]bool, COLS)
        for c := range visited[r] {
            visited[r][c] = make([]bool, 64)
        }
    }
    visited[startRow][startCol][0] = true
    steps := 0

    for len(queue) > 0 {
        size := len(queue)
        for range size {
            curr := queue[0]
            queue = queue[1:]

            row, col, keys := curr.Row, curr.Col, curr.Keys
            if keys == allKeys {
                return steps
            }

            for i := 0; i < 4; i++ {
                nr := row + dr[i]
                nc := col + dc[i]
                nk := curr.Keys

                if nr < 0 || nr >= ROWS || nc < 0 || nc >= COLS {
                    continue
                }

                cell := grid[nr][nc]

                if cell == '#' {
                    continue
                } else if cell >= 'A' && cell <= 'F' {
                    lockBit := 1 << (cell - 'A')
                    if (nk & lockBit) == 0 {
                        continue
                    } 
                } else if cell >= 'a' && cell <= 'f' {
                    nk |= 1 << (cell - 'a')
                }

                if !visited[nr][nc][nk] {
                    visited[nr][nc][nk] = true
                    queue = append(queue, State{Row:nr, Col:nc, Keys:nk})
                }
            } 
        }
        steps++    
    }
    return -1
}
```

### Code Efficiency

- **Time Complexity**: $O(R \times C \times 2^K)$
    - $R$ and $C$ are the rows and columns of the grid.
    - $K$ is the number of keys ($K \le 6$).
    - The state space size is $R \times C \times 2^K \le 64 \times R \times C$. Since we visit each state at most once and perform $O(1)$ operations per state, the time complexity is linear with respect to the grid size.
- **Space Complexity**: $O(R \times C \times 2^K)$
    - Required for the 3D `visited` array of size $R \times C \times 64$ and the BFS queue.
