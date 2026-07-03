# 54. Spiral Matrix

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/spiral-matrix/description/)

## Solution: Simulation (Direction Arrays with Visited Tracking)

We can traverse the matrix in spiral order by simulating the movement directly. We start from the top-left cell $(0, 0)$ and move clockwise: **Right $\rightarrow$ Down $\rightarrow$ Left $\rightarrow$ Up**. Whenever we hit a matrix boundary or a cell we have already visited, we change our direction clockwise.

### Thought Process

1.  **Direction Representation**:
    *   Define two arrays representing the step increments for row (`dr`) and column (`dc`) transitions:
        *   **Right**: `(0, 1)` $\rightarrow$ `dr = 0, dc = 1`
        *   **Down**: `(1, 0)` $\rightarrow$ `dr = 1, dc = 0`
        *   **Left**: `(0, -1)` $\rightarrow$ `dr = 0, dc = -1`
        *   **Up**: `(-1, 0)` $\rightarrow$ `dr = -1, dc = 0`
    *   We use a direction index `i` (from `0` to `3`) to determine the current direction.
2.  **Visited Tracking**:
    *   Use a hash map `visited` with keys of type `[2]int` (storing row and column coordinates) to record which cells have been processed.
3.  **Iteration**:
    *   Iterate exactly $m \times n$ times (using Go's integer range loop `for range m*n`).
    *   At each step, add the current cell `matrix[r][c]` to the output slice `res` and mark it as visited.
    *   Calculate the candidate next cell: `nr, nc = r + dr[i], c + dc[i]`.
    *   If `(nr, nc)` goes out of matrix bounds or has already been visited, turn $90^{\circ}$ clockwise by updating the direction index: `i = (i + 1) % 4`. Re-calculate the next cell coordinates with the updated direction.
    *   Update `r, c = nr, nc`.

> [!TIP]
> This simulation approach can also be implemented in $O(1)$ auxiliary space (excluding the output array) by shrinking the boundaries (`top`, `bottom`, `left`, `right`) of the active matrix slice dynamically. This avoids the overhead of the `visited` hash map.

### Go Code

``` go
import "fmt"

func spiralOrder(matrix [][]int) []int {
    res := make([]int, 0)
    visited := make(map[[2]int]bool)
    m, n := len(matrix), len(matrix[0])
    
    dr := [4]int{0, 1, 0, -1}
    dc := [4]int{1, 0, -1, 0}

    r, c, i := 0, 0, 0
    for range m*n {
        res = append(res, matrix[r][c])
        visited[[2]int{r,c}] = true

        nr, nc := r+dr[i], c+dc[i]
        if !isValid(matrix, nr, nc) || visited[[2]int{nr, nc}] {
            i = (i+1)%4
            nr, nc = r+dr[i], c+dc[i]
        }
        r, c = nr, nc
    }
    return res
}

func isValid(m [][]int, r int, c int) bool {
    if r < 0 || r >= len(m) || c < 0 || c >= len(m[0]) {
        return false
    }
    return true
}
```

### Code Efficiency

- **Time Complexity**: $O(m \times n)$
    - We visit every cell of the $m \times n$ matrix exactly once. All map lookups/insertions and boundary checks take $O(1)$ constant time.
- **Space Complexity**: $O(m \times n)$
    - We allocate a `visited` hash map of size $m \times n$ to track visited cells.