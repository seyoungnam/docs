# 240. Search a 2D Matrix II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/search-a-2d-matrix-ii/description/)

## Solution: Search from Top-Right Corner

We are given a 2D matrix where each row and column is sorted in ascending order. Instead of checking every element or performing separate binary searches, we can search starting from the **top-right corner** of the matrix. This approach allows us to eliminate a row or a column at each step, yielding a linear time complexity.

### Thought Process

1.  **Exploiting Sorted Boundaries**:
    *   Let's position ourselves at the top-right corner of the matrix: row $r = 0$, column $c = \text{COLS} - 1$.
    *   Let the current element be `curr = matrix[r][c]`.
    *   **If $\text{curr} == \text{target}$**: We found the target; return `true`.
    *   **If $\text{curr} > \text{target}$**: Since the columns are sorted from top to bottom, all elements below `curr` in the same column are even larger than `curr`. Therefore, the target cannot be in column `c`. We can safely eliminate the column: `c--`.
    *   **If $\text{curr} < \text{target}$**: Since the rows are sorted from left to right, all elements to the left of `curr` in the same row are even smaller than `curr`. Therefore, the target cannot be in row `r`. We can safely eliminate the row: `r++`.
2.  **Termination**:
    *   We repeat the comparison in a loop while our bounds are valid ($r < \text{ROWS}$ and $c \ge 0$). If we fall out of the bounds without finding the target, it does not exist in the matrix. Return `false`.

### Go Code

``` go
func searchMatrix(matrix [][]int, target int) bool {
    ROWS, COLS := len(matrix), len(matrix[0])
    r := 0
    c := COLS - 1
    for r < ROWS && c >= 0 {
        curr := matrix[r][c]
        if curr == target {
            return true
        }
        if curr > target {
            c--
        } else {
            r++
        }
    }
    return false
}
```

### Code Efficiency

- **Time Complexity**: $O(M + N)$
    - Where $M$ is the number of rows and $N$ is the number of columns. In each step of the loop, we either increment `r` or decrement `c`. The maximum possible steps is $M + N$, yielding a linear time complexity.
- **Space Complexity**: $O(1)$
    - Only a constant number of tracking index variables (`ROWS`, `COLS`, `r`, `c`, `curr`) are stored, requiring no extra auxiliary memory.