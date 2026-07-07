# 351. Android Unlock Patterns

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/android-unlock-patterns/description/)

## Solution: Backtracking (DFS Closure with Symmetry Optimizations)

To find the total number of Android unlock patterns of length $L$ (where $m \le L \le n$), we can use recursive backtracking. We precalculate invalid digit jumps that skip over intermediate numbers and optimize our search branches using the geometric symmetry of the 3x3 grid.

### Thought Process

1.  **Android Pattern Rules**:
    *   Keys are numbered 1 to 9 in a 3x3 layout.
    *   No key can be repeated in a pattern.
    *   A line connecting key `curr` to key `next` cannot cross an unvisited key. If it crosses key `mid`, the jump is valid *only* if `mid` has already been visited.
2.  **Skipped Keys Matrix**:
    *   We pre-populate a 10x10 lookup table `skip`, where `skip[i][j]` stores the digit `mid` crossed by the segment connecting `i` and `j`:
        *   **Edges**: `1-3` (crosses `2`), `1-7` (crosses `4`), `3-9` (crosses `6`), `7-9` (crosses `8`).
        *   **Center / Diagonals**: `1-9` (crosses `5`), `3-7` (crosses `5`), `2-8` (crosses `5`), `4-6` (crosses `5`).
3.  **Recursive DFS Branching**:
    *   Define a closure `dfs(curr, remain int) int` where `remain` is the number of keys left to add to the pattern.
    *   At the current key `curr`, iterate through all possible next keys `next` from $1$ to $9$:
        *   A transition to `next` is valid if:
            1. `next` has not been visited (`!visited[next]`).
            2. There is no intermediate key (`skip[curr][next] == 0`) OR the intermediate key has already been visited (`visited[skip[curr][next]]`).
        *   If valid, mark `next` as visited, recurse with `remain - 1`, and unmark `next` to backtrack.
4.  **Symmetry Optimization**:
    *   Instead of starting the DFS from all 9 keys, we group them by geometric symmetry to reduce initial branches from 9 to 3:
        *   **Corners** (`1`, `3`, `7`, `9`): All four corners behave identically. Run DFS starting at `1` and multiply by 4.
        *   **Edges/Sides** (`2`, `4`, `6`, `8`): All four side keys behave identically. Run DFS starting at `2` and multiply by 4.
        *   **Center** (`5`): Key `5` stands alone. Run DFS starting at `5` once.

### Go Code

``` go
func numberOfPatterns(m int, n int) int {
    skip := make([][]int, 10)
    for i := range skip {
        skip[i] = make([]int, 10)
    }

    // pre-populate the invalid direct jumps
    skip[1][3], skip[3][1] = 2, 2
    skip[1][7], skip[7][1] = 4, 4
    skip[3][9], skip[9][3] = 6, 6
    skip[7][9], skip[9][7] = 8, 8

    skip[1][9], skip[9][1] = 5, 5
    skip[2][8], skip[8][2] = 5, 5
    skip[3][7], skip[7][3] = 5, 5
    skip[4][6], skip[6][4] = 5, 5

    visited := make([]bool, 10)

    var dfs func(curr, remain int) int
    dfs = func(curr, remain int) int {
        if remain == 0 {
            return 1
        }
        count := 0
        for next := 1; next <= 9; next++ {
            if !visited[next] && (skip[curr][next] == 0 || visited[skip[curr][next]]) {
                visited[next] = true
                count += dfs(next, remain-1)
                visited[next] = false
            }
        }
        return count
    }

    totalPatterns := 0
    for length := m; length <= n; length++ {
        // Core symmetry group: corners (1, 3, 7, 9)
        visited[1] = true
        totalPatterns += dfs(1, length-1)*4
        visited[1] = false

        // Core symmetry group: edges (2, 4, 6, 8)
        visited[2] = true
        totalPatterns += dfs(2, length-1)*4
        visited[2] = false

        // Core symmetry group: center (5)
        visited[5] = true
        totalPatterns += dfs(5, length-1)
        visited[5] = false
    }
    return totalPatterns
}
```

### Code Efficiency

- **Time Complexity**: $O(9!)$ in the absolute worst case (generating all permutations of length 9), but the skipped-key constraints prune the search tree heavily. Grouping by symmetry further cuts the initial search paths by two-thirds. The total search executes in under a millisecond.
- **Space Complexity**: $O(L)$ auxiliary space. The recursion call stack depth goes up to a maximum of $L \le 9$ levels. The `visited` tracker uses $O(1)$ space (a fixed size array of size 10).