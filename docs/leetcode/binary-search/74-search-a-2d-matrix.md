# 74. Search a 2D Matrix

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/search-a-2d-matrix/description/)

## Solution: Two-Pass Binary Search

Since both the rows and columns are sorted, we can search for the target using two consecutive binary searches:
1.  **Row Binary Search**: Find the correct row that could potentially contain the target by comparing the target with the first and last elements of each row.
2.  **Column Binary Search**: Once the correct row is identified, run a standard binary search on that row to find the target.

### Thought Process

1.  **Row Search**: Set `top = 0` and `bot = ROWS - 1`. Loop while `top <= bot`:
    *   Find the middle row: `row = top + (bot - top)/2`.
    *   If the target is larger than the last element of this row (`matrix[row][COLS-1] < target`), the target must be in a lower row: `top = row + 1`.
    *   If the target is smaller than the first element of this row (`matrix[row][0] > target`), the target must be in a higher row: `bot = row - 1`.
    *   Otherwise, the target is bounded within this row; break the loop.
2.  **Row Verification**: If the row search bounds cross (`!(top <= bot)`), the target is out of bounds; return `false`.
3.  **Column Search**: Set the target row: `row = top + (bot - top)/2`. Run a standard binary search between columns `l = 0` and `r = COLS - 1`:
    *   If `matrix[row][col] == target`, return `true`.
    *   Otherwise, narrow the range by shifting `l` or `r`.

### Go Code

``` go
func searchMatrix(matrix [][]int, target int) bool {
    ROWS, COLS := len(matrix), len(matrix[0])
    
    top, bot := 0, ROWS-1
    for top <= bot {
        row := top + (bot - top)/2
        if matrix[row][COLS-1] < target {
            top = row + 1
        } else if matrix[row][0] > target {
            bot = row - 1
        } else {
            break
        }
    }
    if !(top <= bot) {
        return false
    }
    row := top + (bot - top)/2
    l, r := 0, COLS-1
    for l <= r {
        col := l + (r-l)/2
        if matrix[row][col] < target {
            l = col + 1
        } else if matrix[row][col] > target {
            r = col - 1
        } else {
            return true
        }
    }
    return false
}
```

### Code Efficiency

- **Time Complexity**: $O(\log m + \log n)$
    - Where $m$ is `ROWS` and $n$ is `COLS`. The first binary search takes $O(\log m)$ and the second takes $O(\log n)$, yielding an overall logarithmic time.
- **Space Complexity**: $O(1)$
    - We only use a few tracking pointer variables, requiring $O(1)$ auxiliary space.

---

## Alternative Solution: Single-Pass Virtual 1D Binary Search

We can optimize the code significantly by treating the entire $m \times n$ 2D matrix as a **single virtual 1D sorted array** of size $m \cdot n$. This allows us to find the target in a single binary search.

### Thought Process

1.  **Virtual Indexing**: For any virtual 1D index `m` in the range `[0, ROWS * COLS - 1]`:
    *   The corresponding row is: `row = m / COLS`.
    *   The corresponding column is: `col = m % COLS`.
2.  **Binary Search**: Run a standard binary search on the interval `[0, ROWS * COLS - 1]`. For each midpoint `m`, retrieve the value at `matrix[row][col]` and compare it to the target.

### Go Code

``` go
func searchMatrix(matrix [][]int, target int) bool {
    ROWS, COLS := len(matrix), len(matrix[0])
    l, r := 0, ROWS*COLS-1
    
    for l <= r {
        m := l + (r-l)/2
        row := m / COLS
        col := m % COLS
        
        if matrix[row][col] == target {
            return true
        } else if matrix[row][col] < target {
            l = m + 1
        } else {
            r = m - 1
        }
    }
    return false
}
```

### Code Efficiency

- **Time Complexity**: $O(\log(m \cdot n))$
    - Logarithmic search over $m \cdot n$ virtual elements. Note that mathematically, $\log(m \cdot n) = \log m + \log n$, so both approaches have the same time complexity, but this version requires only one loop.
- **Space Complexity**: $O(1)$
    - Only a few tracking variables are used.
