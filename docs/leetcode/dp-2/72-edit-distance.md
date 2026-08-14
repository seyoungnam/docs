# 72. Edit Distance

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/edit-distance/description/)

Given two strings `word1` and `word2`, return the minimum number of operations required to convert `word1` to `word2`.

You have the following three operations permitted on a word:
1.  **Insert** a character
2.  **Delete** a character
3.  **Replace** a character

---

## Solution 1: Top-Down Dynamic Programming (DFS with Memoization)

We can solve this problem recursively by comparing the strings character by character from left to right. When characters mismatch, we branch into the three allowed edit operations (insert, delete, replace) and choose the option with the minimum cost.

### Thought Process

1.  **State Definition**:
    *   Let `dfs(i1, i2)` represent the minimum edit distance to convert suffix `word1[i1:]` to suffix `word2[i2:]`.
2.  **Base Cases**:
    *   If `i1 == m`: `word1` is exhausted. We must insert all remaining characters of `word2`, requiring $n - i2$ operations.
    *   If `i2 == n`: `word2` is exhausted. We must delete all remaining characters of `word1`, requiring $m - i1$ operations.
3.  **State Transitions**:
    *   **Matching Characters (`word1[i1] == word2[i2]`)**:
        *   No operation needed:
            $$dfs(i1, i2) = dfs(i1+1, i2+1)$$
    *   **Mismatching Characters (`word1[i1] != word2[i2]`)**:
        *   **Insert**: Insert `word2[i2]` before `word1[i1]`. This matches `word2[i2]`, so we advance `i2`: `dfs(i1, i2+1)`.
        *   **Delete**: Delete `word1[i1]`. We advance `i1`: `dfs(i1+1, i2)`.
        *   **Replace**: Replace `word1[i1]` with `word2[i2]`. Both characters match, so advance both: `dfs(i1+1, i2+1)`.
        *   Take the minimum cost among all three operations plus 1:
            $$dfs(i1, i2) = 1 + \min(dfs(i1, i2+1), \min(dfs(i1+1, i2), dfs(i1+1, i2+1)))$$
4.  **Memoization**:
    *   Use a 2D matrix `memo` of size $m \times n$ initialized to `-1` to cache intermediate subproblem results.

### Go Code

``` go
func minDistance(word1 string, word2 string) int {
    m, n := len(word1), len(word2)
    memo := make([][]int, m)
    for r := 0; r < m; r++ {
        memo[r] = make([]int, n)
        for c := 0; c < n; c++ {
            memo[r][c] = -1
        }
    }

    var dfs func(i1, i2 int) int
    dfs = func(i1, i2 int) int {
        if i1 == m {
            return n - i2
        }
        if i2 == n {
            return m - i1
        }
        if memo[i1][i2] != -1 {
            return memo[i1][i2]
        }
        if word1[i1] == word2[i2] {
            memo[i1][i2] = dfs(i1+1, i2+1)
        } else {
            memo[i1][i2] = 1 + min(dfs(i1, i2+1), min(dfs(i1+1, i2), dfs(i1+1, i2+1)))
        }
        return memo[i1][i2]
    }
    return dfs(0, 0)
}
```

### Code Efficiency

- **Time Complexity**: $O(m \cdot n)$
    - Where $m = \text{len}(word1)$ and $n = \text{len}(word2)$. Each $(i1, i2)$ subproblem is visited and computed once.
- **Space Complexity**: $O(m \cdot n)$
    - The `memo` matrix takes $O(m \cdot n)$ space, plus $O(m + n)$ auxiliary space for the recursion call stack.

---

## Solution 2: Bottom-Up Dynamic Programming (2D DP)

We can convert the top-down recursion into a 2D iterative DP table, evaluating suffixes in reverse order.

### Thought Process

1.  **State Definition**:
    *   Define `dp[r][c]` as the minimum edit distance to convert suffix `word1[r:]` to suffix `word2[c:]`.
2.  **Base Cases**:
    *   `dp[r][n] = m - r` for all $0 \le r \le m$: Converting `word1[r:]` to an empty string requires deleting all remaining $m - r$ characters.
    *   `dp[m][c] = n - c` for all $0 \le c \le n$: Converting an empty string to `word2[c:]` requires inserting all remaining $n - c$ characters.
3.  **State Transitions**:
    *   Iterate backwards with $r$ from $m-1$ down to $0$ and $c$ from $n-1$ down to $0$:
        *   If `word1[r] == word2[c]`:
            $$dp[r][c] = dp[r+1][c+1]$$
        *   If `word1[r] != word2[c]`:
            $$dp[r][c] = 1 + \min(dp[r][c+1], \min(dp[r+1][c], dp[r+1][c+1]))$$
4.  **Result**:
    *   The minimum edit distance between the full strings `word1` and `word2` is stored in `dp[0][0]`.

### Go Code

``` go
func minDistance(word1 string, word2 string) int {
    m, n := len(word1), len(word2)
    dp := make([][]int, m+1)
    for r := 0; r < m+1; r++ {
        dp[r] = make([]int, n+1)
        dp[r][n] = m-r
        if r == m {
            for c := 0; c < n+1; c++ {
                dp[r][c] = n-c
            }
        }
    }
    
    for r := m-1; r >= 0; r-- {
        for c := n-1; c >= 0; c-- {
            if word1[r] == word2[c] {
                dp[r][c] = dp[r+1][c+1]
            } else {
                dp[r][c] = 1 + min(dp[r][c+1], min(dp[r+1][c], dp[r+1][c+1]))
            }
        }
    }
    return dp[0][0]
}
```

### Code Efficiency

- **Time Complexity**: $O(m \cdot n)$
    - We fill an $(m+1) \times (n+1)$ table with two nested loops.
- **Space Complexity**: $O(m \cdot n)$
    - The 2D table requires $O(m \cdot n)$ auxiliary space. This bottom-up approach eliminates recursion stack overhead.