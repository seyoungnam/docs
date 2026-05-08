# 994. Rotting Oranges

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/rotting-oranges/description/)

## Solution 1: Basic BFS (Multi-Source)

### Thought Process

1.  **Multi-Source BFS**: All initially rotten oranges (value `2`) spread rot simultaneously. We use a queue to track these sources.
2.  **Level-by-Level Simulation**: Each iteration of the BFS queue (processing one full "level") represents one minute passing.
3.  **Time Tracking**: We initialize `time` to `-1`. Each time we process a level of the queue, we increment `time`. We use `max(time, 0)` at the end to handle the case where no oranges were ever rotten or no rot could spread.
4.  **Post-Process Verification**: After the BFS finishes, we perform a second full scan of the grid to check if any fresh oranges (`1`) remain. If so, return `-1`.

### Go Code

``` go
func orangesRotting(grid [][]int) int {
    ROW, COL := len(grid), len(grid[0])
    queue := [][2]int{}
    for r := 0; r < ROW; r++ {
        for c := 0; c < COL; c++ {
            if grid[r][c] == 2 {
                queue = append(queue, [2]int{r, c})
            }
        }
    }
    
    dir := [4][2]int{{1,0}, {-1,0}, {0,1}, {0,-1}}
    time := -1
    for len(queue) > 0 {
        curLen := len(queue)
        for range curLen {
            curr := queue[0]
            queue = queue[1:]
            cr, cc := curr[0], curr[1]

            for _, d := range dir {
                nr := cr + d[0]
                nc := cc + d[1]
                if nr >= 0 && nr < ROW && nc >= 0 && nc < COL && grid[nr][nc] == 1 {
                    queue = append(queue, [2]int{nr, nc})
                    grid[nr][nc] = 2
                }
            }
        }
        time++
    }

    for r := 0; r < ROW; r++ {
        for c := 0; c < COL; c++ {
            if grid[r][c] == 1 {
                return -1
            }
        }
    }
    return max(time, 0)
}
```

### Code Efficiency

-   **Time Complexity**: $O(M \times N)$
    -   Initial scan takes $O(M \times N)$.
    -   BFS visits each cell at most once.
    -   Final verification scan takes $O(M \times N)$.
-   **Space Complexity**: $O(M \times N)$
    -   In the worst case (e.g., all oranges rot), the queue can hold up to $M \times N$ elements.

---

## Solution 2: Optimized BFS (Fresh Count Tracking)

### Thought Process (Changes vs. Solution 1)

1.  **Fresh Orange Counter**: Unlike Solution 1, we count the number of fresh oranges (`freshCount`) during the initial grid scan. 
2.  **Early Termination**: If `freshCount` is 0 initially, we return `0` immediately. During BFS, we only continue if `freshCount > 0`, which avoids an unnecessary extra minute increment when the last orange rots.
3.  **Efficiency Improvement**: Instead of a full $O(M \times N)$ scan at the end to check for remaining fresh oranges, we simply check if `freshCount == 0` ($O(1)$).
4.  **Minute Logic**: The `time` starts at `0` and is only incremented if the rot actually spreads to new oranges in a given level.

### Go Code

``` go
func orangesRotting(grid [][]int) int {
    ROWS, COLS := len(grid), len(grid[0])
    var queue [][2]int
    freshCount := 0

    // Initial scan: find rotten oranges AND count fresh ones
    for r := 0; r < ROWS; r++ {
        for c := 0; c < COLS; c++ {
            if grid[r][c] == 2 {
                queue = append(queue, [2]int{r, c})
            } else if grid[r][c] == 1 {
                freshCount++
            }
        }
    }
    
    if freshCount == 0 {
        return 0
    }

    dir := [4][2]int{{1, 0}, {-1, 0}, {0, 1}, {0, -1}}
    time := 0

    // Only continue if there's rot to spread AND fresh oranges to infect
    for len(queue) > 0 && freshCount > 0 {
        curLen := len(queue)
        for i := 0; i < curLen; i++ {
            cr, cc := queue[0][0], queue[0][1]
            queue = queue[1:]

            for _, d := range dir {
                nr, nc := cr+d[0], cc+d[1]
                
                if nr >= 0 && nr < ROWS && nc >= 0 && nc < COLS && grid[nr][nc] == 1 {
                    grid[nr][nc] = 2
                    queue = append(queue, [2]int{nr, nc})
                    freshCount-- // Update count immediately
                }
            }
        }
        time++ // One minute passes after processing a full level
    }

    if freshCount > 0 {
        return -1
    }
    return time
}
```

### Code Efficiency

-   **Time Complexity**: $O(M \times N)$
    -   Still linear relative to grid size, but performs fewer total operations by eliminating the final full scan and stopping the BFS early.
-   **Space Complexity**: $O(M \times N)$
    -   Queue size remains proportional to the number of cells.

---
