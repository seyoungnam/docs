# 97. Interleaving String

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/interleaving-string/description/)

Given strings `s1`, `s2`, and `s3`, find whether `s3` is formed by an interleaving of `s1` and `s2`.

An interleaving of two strings `s` and `t` is a configuration where they are divided into non-empty substrings such that:
*   $s = s_1 + s_2 + \dots + s_n$
*   $t = t_1 + t_2 + \dots + t_m$
*   $|n - m| \le 1$
*   The interleaving is $s_1 + t_1 + s_2 + t_2 + \dots$ or $t_1 + s_1 + t_2 + s_2 + \dots$

---

## Solution 1: Top-Down Dynamic Programming (DFS with Memoization)

We can solve this problem recursively by matching characters of `s3` from left to right against the current characters of `s1` and `s2`.

### Thought Process

1.  **Length Validation**:
    *   If $\text{len}(s1) + \text{len}(s2) \neq \text{len}(s3)$, `s3` cannot be formed by interleaving; return `false` immediately.
2.  **State Definition**:
    *   Let `dfs(i, j, k)` return true if `s3[k:]` can be formed by interleaving suffixes `s1[i:]` and `s2[j:]`.
    *   Because the total characters used from `s1` and `s2` must equal the characters matched in `s3` ($k = i + j$), the state is uniquely defined by $(i, j)$.
3.  **Base Case**:
    *   If $k == \text{len}(s3)$, return true if we have reached the end of both strings ($i == \text{len}(s1)$ and $j == \text{len}(s2)$).
4.  **State Transitions**:
    *   **Match from `s1`**: If $i < \text{len}(s1)$ and $s1[i] == s3[k]$, recursively check `dfs(i+1, j, k+1)`.
    *   **Match from `s2`**: If not yet resolved and $j < \text{len}(s2)$ and $s2[j] == s3[k]$, recursively check `dfs(i, j+1, k+1)`.
5.  **Memoization**:
    *   Cache evaluated states in a 2D matrix `memo` of size $(m+1) \times (n+1)$ with `-1` (unvisited), `0` (false), and `1` (true).

### Go Code

``` go
func isInterleave(s1 string, s2 string, s3 string) bool {
    m, n, o := len(s1), len(s2), len(s3)
    if m + n != o {
        return false
    }
    memo := make([][]int, m+1)
    for i := range memo {
        memo[i] = make([]int, n+1)
        for j := range memo[i] {
            memo[i][j] = -1
        }
    }

    var dfs func(i, j, k int) bool
    dfs = func(i, j, k int) bool {
        if k == o {
            return i == m && j == n
        }

        if memo[i][j] != -1 {
            return memo[i][j] == 1
        }

        res := false
        if i < m && s1[i] == s3[k] {
            res = dfs(i+1, j, k+1)
        }

        if !res && j < n && s2[j] == s3[k] {
            res = dfs(i, j+1, k+1)
        }

        if res {
            memo[i][j] = 1
        } else {
            memo[i][j] = 0
        }
        return res
    }
    return dfs(0, 0, 0)
}
```

### Code Efficiency

- **Time Complexity**: $O(m \cdot n)$
    - Where $m = \text{len}(s1)$ and $n = \text{len}(s2)$. There are at most $(m+1) \times (n+1)$ unique $(i, j)$ states computed.
- **Space Complexity**: $O(m \cdot n)$
    - The `memo` matrix takes $O(m \cdot n)$ space, plus $O(m + n)$ auxiliary space for the recursion call stack.

---

## Solution 2: Bottom-Up Dynamic Programming (2D DP)

We can convert the top-down recursion into a 2D iterative DP table, evaluating suffix interleavings in reverse order.

### Thought Process

1.  **State Definition**:
    *   Define `dp[r][c]` as true if `s3[r+c:]` can be formed by interleaving suffixes `s1[r:]` and `s2[c:]`.
2.  **Base Case**:
    *   `dp[m][n] = true`: Empty suffixes of `s1` and `s2` can form an empty suffix of `s3`.
3.  **State Transitions**:
    *   Iterate backwards with $r$ from $m$ down to $0$ and $c$ from $n$ down to $0$:
        *   If $r < m$, $s1[r] == s3[r+c]$, and $dp[r+1][c]$ is true, then set $dp[r][c] = true$.
        *   If $c < n$, $s2[c] == s3[r+c]$, and $dp[r][c+1]$ is true, then set $dp[r][c] = true$.
4.  **Result**:
    *   `dp[0][0]` stores whether the complete `s3` is formed by interleaving `s1` and `s2`.

### Go Code

``` go
func isInterleave(s1 string, s2 string, s3 string) bool {
    m, n, o := len(s1), len(s2), len(s3)
    if m + n != o {
        return false
    }

    dp := make([][]bool, m+1)
    for r := range dp {
        dp[r] = make([]bool, n+1)
    }

    dp[m][n] = true
    for r := m; r >= 0; r-- {
        for c := n; c >= 0; c-- {
            if r < m && s1[r] == s3[r+c] && dp[r+1][c] {
                dp[r][c] = true
            }
            if c < n && s2[c] == s3[r+c] && dp[r][c+1] {
                dp[r][c] = true
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