# 1092. Shortest Common Supersequence

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/shortest-common-supersequence/description/)

Given two strings `str1` and `str2`, return the shortest string that has both `str1` and `str2` as subsequences. If there are multiple valid answers, return any of them.

---

## Approach 1: Recursive DFS (Time Limit Exceeded)

A naive recursive approach generates all possible supersequence choices by comparing characters one by one.

### Thought Process

1.  **Base Cases**:
    *   If `i == len(str1)`, return the remaining suffix of `str2` (`str2[j:]`).
    *   If `j == len(str2)`, return the remaining suffix of `str1` (`str1[i:]`).
2.  **Decisions**:
    *   If `str1[i] == str2[j]`, include the character once and advance both pointers: `string(str1[i]) + dfs(i+1, j+1)`.
    *   If `str1[i] != str2[j]`, branch into two choices:
        *   Take `str1[i]` and advance `i`: `take1 = string(str1[i]) + dfs(i+1, j)`
        *   Take `str2[j]` and advance `j`: `take2 = string(str2[j]) + dfs(i, j+1)`
        *   Return the shorter string between `take1` and `take2`.

### Go Code

``` go
func shortestCommonSupersequence(str1 string, str2 string) string {
    m, n := len(str1), len(str2)

    var dfs func(i int, j int) string
    dfs = func(i int, j int) string {
        if i == m {
            return str2[j:]
        }
        if j == n {
            return str1[i:]
        }
        if str1[i] == str2[j] {
            return string(str1[i]) + dfs(i+1, j+1)
        }
        take1 := string(str1[i]) + dfs(i+1, j)
        take2 := string(str2[j]) + dfs(i, j+1)
        if len(take1) <= len(take2) {
            return take1
        }
        return take2
    }

    return dfs(0, 0)
}
```

### Code Efficiency

- **Time Complexity**: $O(2^{m+n})$
    - Exponential branching causes TLE on larger inputs.
- **Space Complexity**: $O(m + n)$
    - Recursion call stack depth.

---

## Approach 2: Memoized DFS & String DP (Memory Limit Exceeded / OOM)

Attempting to memoize string objects directly in a 2D table results in Out of Memory (OOM) / Memory Limit Exceeded (MLE) because storing string copies across an $m \times n$ grid takes excessive space.

### Go Code (DFS with Memoization)

``` go
func shortestCommonSupersequence(str1 string, str2 string) string {
    m, n := len(str1), len(str2)
    memo := make([][]string, m)
    for i := range memo {
        memo[i] = make([]string, n)
    }

    var dfs func(i int, j int) string
    dfs = func(i int, j int) string {
        if i == m {
            return str2[j:]
        }
        if j == n {
            return str1[i:]
        }
        if memo[i][j] != "" {
            return memo[i][j]
        }
        if str1[i] == str2[j] {
            memo[i][j] = string(str1[i]) + dfs(i+1, j+1)
            return memo[i][j]
        }
        take1 := string(str1[i]) + dfs(i+1, j)
        take2 := string(str2[j]) + dfs(i, j+1)
        if len(take1) <= len(take2) {
            memo[i][j] = take1
        } else {
            memo[i][j] = take2
        }
        return memo[i][j]
    }
    return dfs(0, 0)
}
```

### Go Code (Bottom-Up String DP)

``` go
func shortestCommonSupersequence(str1 string, str2 string) string {
    m, n := len(str1), len(str2)
    dp := make([][]string, m+1)
    for r := range dp {
        dp[r] = make([]string, n+1)
    }

    for r := 0; r <= m; r++ {
        for c := 0; c <= n; c++ {
            if r == 0 {
                dp[0][c] = str2[:c]
            } else if c == 0 {
                dp[r][0] = str1[:r]
            } else if str1[r-1] == str2[c-1] {
                dp[r][c] = dp[r-1][c-1] + string(str1[r-1])
            } else {
                if len(dp[r-1][c]) < len(dp[r][c-1]) {
                    dp[r][c] = dp[r-1][c] + string(str1[r-1])
                } else {
                    dp[r][c] = dp[r][c-1] + string(str2[c-1])
                }
            }
        }
    }
    return dp[m][n]
}
```

### Code Efficiency

- **Time Complexity**: $O(m \cdot n \cdot (m+n))$
    - String concatenations copy up to $m+n$ characters at each step.
- **Space Complexity**: $O(m \cdot n \cdot (m+n))$
    - Storing $m \times n$ strings of length up to $m+n$ causes OOM / MLE.

---

## Approach 3: Integer Length DP + Backtracking (Optimal)

To avoid storing large strings in memory, we compute only the **length** of the SCS in a 2D integer table, and then reconstruct the actual string by **backtracking** through the DP table.

### Thought Process

1.  **DP Table of Lengths**:
    *   Define `dp[i][j]` as the length of the Shortest Common Supersequence of prefixes `str1[:i]` and `str2[:j]`.
    *   **Base Cases**:
        *   `dp[0][j] = j`: An empty string and a string of length $j$ have an SCS of length $j$.
        *   `dp[i][0] = i`: A string of length $i$ and an empty string have an SCS of length $i$.
    *   **State Transitions**:
        *   If `str1[i-1] == str2[j-1]`: Characters match, so we take it once:
            $$dp[i][j] = 1 + dp[i-1][j-1]$$
        *   If `str1[i-1] != str2[j-1]`: Take the minimum of excluding either character plus 1:
            $$dp[i][j] = 1 + \min(dp[i-1][j], dp[i][j-1])$$
2.  **Backtracking to Reconstruct the String**:
    *   Start pointers at $i = n$ and $j = m$ (the bottom-right of the table) and walk back to $(0, 0)$:
        *   If `str1[i-1] == str2[j-1]`: Character is shared; append `str1[i-1]` and decrement both $i$ and $j$.
        *   Else if `dp[i-1][j] < dp[i][j-1]`: The optimal choice came from `str1[i-1]`; append `str1[i-1]` and decrement $i$.
        *   Else: The optimal choice came from `str2[j-1]`; append `str2[j-1]` and decrement $j$.
    *   Append any remaining characters from `str1` (if $i > 0$) or `str2` (if $j > 0$).
    *   Reverse the collected byte slice to obtain the forward SCS string.

### Go Code

``` go
func shortestCommonSupersequence(str1 string, str2 string) string {
    n, m := len(str1), len(str2)
    dp := make([][]int, n+1)
    for i := 0; i <= n; i++ {
        dp[i] = make([]int, m+1)
    }

    for i := 0; i <= n; i++ {
        for j := 0; j <= m; j++ {
            if i == 0 {
                dp[i][j] = j
            } else if j == 0 {
                dp[i][j] = i
            } else if str1[i-1] == str2[j-1] {
                dp[i][j] = 1 + dp[i-1][j-1]
            } else {
                dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1])
            }
        }
    }

    var res []byte
    i, j := n, m
    for i > 0 && j > 0 {
        if str1[i-1] == str2[j-1] {
            res = append(res, str1[i-1])
            i--
            j--
        } else if dp[i-1][j] < dp[i][j-1] {
            res = append(res, str1[i-1])
            i--
        } else {
            res = append(res, str2[j-1])
            j--
        }
    }

    for i > 0 {
        res = append(res, str1[i-1])
        i--
    }

    for j > 0 {
        res = append(res, str2[j-1])
        j--
    }

    for l, r := 0, len(res)-1; l < r; l, r = l+1, r-1 {
        res[l], res[r] = res[r], res[l]
    }
    return string(res)
}
```

### Code Efficiency

- **Time Complexity**: $O(m \cdot n)$
    - Populating the $(n+1) \times (m+1)$ integer DP table takes $O(m \cdot n)$ time.
    - Backtracking traverses at most $m + n$ steps to reconstruct the string.
- **Space Complexity**: $O(m \cdot n)$
    - We only allocate an integer matrix of size $(n+1) \times (m+1)$, completely avoiding string object allocation in the DP table.