# 1351. Count Negative Numbers in a Sorted Matrix

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/count-negative-numbers-in-a-sorted-matrix/description/)

## Solution: Decisive 2D Search (Top-Right Corner)

The grid is sorted in non-increasing order both row-wise and column-wise (meaning positive values come first, followed by negative values). Instead of checking all elements or doing binary searches on each row, we can traverse the matrix starting from the **top-right corner** ($r = 0, c = \text{COLS} - 1$), allowing us to eliminate either a row or count a whole column of negative values in $O(1)$ time per step.

### Thought Process

1.  **Exploiting Sorted Boundaries**:
    *   Initialize pointer $r = 0$ (top row) and $c = \text{COLS} - 1$ (rightmost column).
    *   **If $\text{grid}[r][c] < 0$**:
        *   Since column `c` is sorted in non-increasing order top-to-bottom, if $\text{grid}[r][c]$ is negative, then all elements below it in the same column must also be negative.
        *   There are $\text{ROWS} - r$ elements from row `r` to the bottom of column `c`.
        *   Add $\text{ROWS} - r$ to our `count`, and move to the left column: `c--`.
    *   **If $\text{grid}[r][c] \ge 0$**:
        *   Since row `r` is sorted in non-increasing order left-to-right, all elements to the left of column `c` in this row are greater than or equal to $\text{grid}[r][c]$, meaning they are also non-negative.
        *   Therefore, there are no negative numbers left to find in row `r`. We move down to the next row: `r++`.
2.  **Termination**:
    *   We run the loop while $r < \text{ROWS}$ and $c \ge 0$.
    *   Return the accumulated `count`.

### Go Code

``` go
func countNegatives(grid [][]int) int {
    ROWS, COLS := len(grid), len(grid[0])
    count := 0
    r, c := 0, COLS-1
    for r < ROWS && c >= 0 {
        if grid[r][c] < 0 {
            count += ROWS-r
            c--
        } else {
            r++
        }
    }
    return count
}
```

### Code Efficiency

- **Time Complexity**: $O(M + N)$
    - Where $M$ is the number of rows and $N$ is the number of columns. In each iteration, we either increment `r` (move down) or decrement `c` (move left). The path traverses from top-right to bottom-left, taking at most $M + N$ steps.
- **Space Complexity**: $O(1)$
    - The search runs in-place with no extra memory allocation.