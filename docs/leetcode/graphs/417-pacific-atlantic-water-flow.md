# 417. Pacific Atlantic Water Flow

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/pacific-atlantic-water-flow/description/)

There is an `m x n` rectangular island that borders both the **Pacific Ocean** and **Atlantic Ocean**. The **Pacific Ocean** touches the island's left and top edges, and the **Atlantic Ocean** touches the island's right and bottom edges.

The island is partitioned into a grid of square cells. You are given an `m x n` integer matrix `heights` where `heights[r][c]` represents the height above sea level of the cell at coordinate `(r, c)`.

Rain water can flow to neighboring cells in four directions (north, south, east, and west) if the neighboring cell's height is **less than or equal to** the current cell's height. Water can flow from any cell adjacent to an ocean into that ocean.

Return *a 2D list of grid coordinates* `result` *where* `result[i] = [r_i, c_i]` *denotes that rain water can flow from cell* `(r_i, c_i)` *to **both** the Pacific and Atlantic oceans*.

---

## Solution: Reverse Flow Depth-First Search (DFS)

Instead of checking if water can flow *downward* from each cell to both oceans (which is computationally expensive), we can invert the problem. We start at the ocean borders and simulate water flowing **uphill** (where `currentHeight >= previousHeight`). Any cell visited from both oceans is a valid starting point.

### Thought Process

1.  **Reachable Grids**:
    *   Maintain two 2D boolean slices `pac` and `atl` of size $M \times N$ to track which cells can reach the Pacific and Atlantic oceans respectively.
2.  **Ocean Boundaries**:
    *   The Pacific Ocean touches the top row (`r == 0`) and the left column (`c == 0`).
    *   The Atlantic Ocean touches the bottom row (`r == ROWS - 1`) and the right column (`c == COLS - 1`).
3.  **DFS Uphill Flow**:
    *   Implement a recursive DFS helper: `dfs(r, c, prevHeight, isPacific)`.
    *   **Base Cases**:
        *   If the coordinates $(r, c)$ are out of bounds, return.
        *   If the cell has already been marked as reachable for the current ocean (`pac[r][c]` or `atl[r][c]`), return to avoid cycles.
    *   **Uphill Condition**: If `heights[r][c] >= prevHeight`, water can flow from this cell to the previous cell (and eventually to the ocean).
        *   Mark the cell as reachable.
        *   Recursively visit its 4 neighbors: `(r+1, c)`, `(r-1, c)`, `(r, c+1)`, and `(r, c-1)`.
4.  **Border Scans**:
    *   Run DFS from all cells on the top and left borders for the Pacific Ocean.
    *   Run DFS from all cells on the bottom and right borders for the Atlantic Ocean.
5.  **Find Intersection**:
    *   Iterate through all coordinates $(r, c)$. If `pac[r][c]` and `atl[r][c]` are both `true`, add `[r, c]` to the result slice.

### Go Code

``` go
func pacificAtlantic(heights [][]int) [][]int {
    ROWS, COLS := len(heights), len(heights[0])
    
    // Matrices to track cells reachable from each ocean
    pac := make([][]bool, ROWS)
    atl := make([][]bool, ROWS)
    for r := range heights {
        pac[r] = make([]bool, COLS)
        atl[r] = make([]bool, COLS)
    }

    var dfs func(r int, c int, prev int, isPac bool) 
    dfs = func(r int, c int, prev int, isPac bool) {
        // Base Case: out of bounds
        if r < 0 || r >= ROWS || c < 0 || c >= COLS {
            return
        } 
        
        // Base Case: already visited
        if (isPac && pac[r][c]) || (!isPac && atl[r][c]) {
            return 
        }
        
        curr := heights[r][c]
        
        // Water flows from high to low, so we flow "uphill" from the oceans (curr >= prev)
        if curr >= prev {
            if isPac {
                pac[r][c] = true
            } else {
                atl[r][c] = true
            }
            
            // Recurse in 4 directions
            dfs(r+1, c, curr, isPac)
            dfs(r-1, c, curr, isPac)
            dfs(r, c+1, curr, isPac)
            dfs(r, c-1, curr, isPac)
        }
    }

    // Trigger DFS from the ocean borders
    for r := range heights {
        for c := range heights[0] {
            // Pacific: top row and left column
            if r == 0 || c == 0 {
                dfs(r, c, 0, true)
            }
            // Atlantic: bottom row and right column
            if r == ROWS-1 || c == COLS-1 {
                dfs(r, c, 0, false)
            }
        }
    }

    // Collect cells that can reach both oceans
    res := make([][]int, 0)
    for r := 0; r < ROWS; r++ {
        for c := 0; c < COLS; c++ {
            if pac[r][c] && atl[r][c] {
                res = append(res, []int{r, c})
            }
        }
    }
    
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(M \times N)$
    - Where $M$ is the number of rows and $N$ is the number of columns. Each cell is visited at most once for the Pacific scan and at most once for the Atlantic scan.
- **Space Complexity**: $O(M \times N)$
    - We use $O(M \times N)$ auxiliary space for the `pac` and `atl` matrices, plus $O(M \times N)$ space for the recursion stack in the worst case (e.g. if the heights strictly increase from the ocean borders).