# 115. Distinct Subsequences

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/distinct-subsequences/description/)

Given two strings `s` and `t`, return the number of distinct subsequences of `s` which equals `t`.

---

## Solution 1: Top-Down Dynamic Programming (DFS with Memoization)

We can solve this recursively by comparing characters from left to right. When two characters match, we have two choices: use the character in `s` to match `t`, or skip it to look for another match further along `s`.

### Thought Process

1.  **State Definition**:
    *   Let `dfs(i1, i2)` represent the number of distinct subsequences of `s[i1:]` that equal `t[i2:]`.
2.  **Base Cases**:
    *   If `i2 == len(t)`: We have successfully matched all characters in `t`, so return `1`.
    *   If `i1 == len(s)`: We have run out of characters in `s` without completing `t`, so return `0`.
3.  **State Transitions**:
    *   **Matching Characters (`s[i1] == t[i2]`)**:
        *   **Option 1 (Match)**: Use `s[i1]` to match `t[i2]` and advance both pointers: `dfs(i1+1, i2+1)`.
        *   **Option 2 (Skip)**: Skip `s[i1]` and try to match `t[i2]` with a later occurrence in `s`: `dfs(i1+1, i2)`.
        *   Total combinations:
            $$dfs(i1, i2) = dfs(i1+1, i2+1) + dfs(i1+1, i2)$$
    *   **Mismatching Characters (`s[i1] != t[i2]`)**:
        *   We cannot match `s[i1]` with `t[i2]`, so we must skip it:
            $$dfs(i1, i2) = dfs(i1+1, i2)$$
4.  **Memoization**:
    *   Use a 2D matrix `memo` of size `len(s) x len(t)` initialized to `-1` to cache intermediate results.

### Go Code

``` go
func numDistinct(s string, t string) int {
    memo := make([][]int, len(s))
    for i := 0; i < len(s); i++ {
        memo[i] = make([]int, len(t))
        for j := 0; j < len(t); j++ {
            memo[i][j] = -1
        }
    }
    var dfs func(i1, i2 int) int
    dfs = func(i1, i2 int) int {
        if i2 == len(t) {
            return 1
        }
        if i1 == len(s) {
            return 0
        }
        if memo[i1][i2] != -1 {
            return memo[i1][i2]
        }
        if s[i1] == t[i2] {
            memo[i1][i2] = dfs(i1+1, i2+1) + dfs(i1+1, i2)
        } else {
            memo[i1][i2] = dfs(i1+1, i2)
        }
        return memo[i1][i2]
    }
    return dfs(0, 0)
}
```

### Code Efficiency

- **Time Complexity**: $O(m \cdot n)$
    - Where $m = \text{len}(s)$ and $n = \text{len}(t)$. Each $(i1, i2)$ subproblem is visited and computed once.
- **Space Complexity**: $O(m \cdot n)$
    - The `memo` matrix takes $O(m \cdot n)$ space, plus $O(m)$ auxiliary space for the recursion call stack.

---

## Solution 2: Bottom-Up Dynamic Programming (2D DP)

We can convert the top-down recursion into a 2D bottom-up iterative DP table by evaluating states backwards from the end of both strings.

### Thought Process

1.  **State Definition**:
    *   Define `dp[r][c]` as the number of distinct subsequences of suffix `s[r:]` that equal suffix `t[c:]`.
2.  **Base Cases**:
    *   `dp[r][n] = 1` for all $0 \le r \le m$: An empty suffix `t[n:]` can be formed in exactly 1 way from any suffix of `s` (by deleting all remaining characters).
    *   `dp[m][c] = 0` for all $0 \le c < n$: An empty string `s[m:]` cannot form any non-empty suffix `t[c:]`.
3.  **State Transitions**:
    *   Iterate backwards with $r$ from $m-1$ down to $0$ and $c$ from $n-1$ down to $0$:
        *   Always carry forward the skip branch:
            $$dp[r][c] += dp[r+1][c]$$
        *   If `s[r] == t[c]`, also add the match branch:
            $$dp[r][c] += dp[r+1][c+1]$$
4.  **Result**:
    *   The total number of distinct subsequences is stored in `dp[0][0]`.

### Go Code

``` go
func numDistinct(s string, t string) int {
    m, n := len(s), len(t)
    dp := make([][]int, m+1)
    for r := 0; r < m+1; r++ {
        dp[r] = make([]int, n+1)
        dp[r][n] = 1
    }

    for r := m-1; r >= 0; r-- {
        for c := n-1; c >= 0; c-- {
            dp[r][c] += dp[r+1][c]
            if s[r] == t[c] {
                dp[r][c] += dp[r+1][c+1]
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
    - The 2D table requires $O(m \cdot n)$ auxiliary space.