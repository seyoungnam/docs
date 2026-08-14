# 1143. Longest Common Subsequence

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/longest-common-subsequence/description/)

Given two strings `text1` and `text2`, return the length of their longest common subsequence (LCS). A subsequence of a string is a new string generated from the original string with some characters (can be none) deleted without changing the relative order of the remaining characters.

---

## Solution 1: Top-Down Dynamic Programming (DFS with Memoization)

We can solve this problem recursively by comparing characters one by one starting from the beginning of both strings, using a 2D memoization table to cache intermediate results.

### Thought Process

1.  **State Definition**:
    *   Let `dfs(r, c)` represent the length of the longest common subsequence between suffixes `text1[r:]` and `text2[c:]`.
2.  **Base Case**:
    *   If `r == len(text1)` or `c == len(text2)`, we have reached the end of at least one string. No more characters can match, so return `0`.
3.  **State Transitions**:
    *   **Matching Characters (`text1[r] == text2[c]`)**: We include this character as part of the common subsequence and advance both pointers:
        $$dfs(r, c) = 1 + dfs(r+1, c+1)$$
    *   **Mismatching Characters (`text1[r] != text2[c]`)**: We take the maximum result of either skipping the character in `text1` or skipping the character in `text2`:
        $$dfs(r, c) = \max(dfs(r+1, c), dfs(r, c+1))$$
4.  **Memoization**:
    *   Create a 2D matrix `memo` of size $m \times n$ initialized with `-1`. Return `memo[r][c]` if it has already been computed.

### Go Code

``` go
func longestCommonSubsequence(text1 string, text2 string) int {
    m, n := len(text1), len(text2)
    memo := make([][]int, m)
    for r := 0; r < m; r++ {
        memo[r] = make([]int, n)
        for c := 0; c < n; c++ {
            memo[r][c] = -1
        }
    }

    var dfs func(r, c int) int
    dfs = func(r, c int) int {
        if r == m || c == n {
            return 0
        }
        if memo[r][c] != -1 {
            return memo[r][c]
        }
        if text1[r] == text2[c] {
            memo[r][c] = 1 + dfs(r+1, c+1)
        } else {
            memo[r][c] = max(dfs(r+1, c), dfs(r, c+1))
        }
        return memo[r][c]
    }
    return dfs(0, 0)
}
```

### Code Efficiency

- **Time Complexity**: $O(m \cdot n)$
    - Where $m$ and $n$ are the lengths of `text1` and `text2`. Each $(r, c)$ state is visited and computed at most once.
- **Space Complexity**: $O(m \cdot n)$
    - The 2D `memo` table takes $O(m \cdot n)$ space, plus $O(m + n)$ auxiliary space for the recursion call stack.

---

## Solution 2: Bottom-Up Dynamic Programming (2D DP)

We can convert the top-down recursion into a bottom-up iterative approach by computing states from the end of both strings towards the beginning.

### Thought Process

1.  **State Definition**:
    *   Define `dp[r][c]` as the length of the longest common subsequence of suffixes `text1[r:]` and `text2[c:]`.
2.  **DP Table Initialization**:
    *   Create a 2D table `dp` of size $(m+1) \times (n+1)$ initialized to `0`. The extra $(m+1)$-th row and $(n+1)$-th column serve as base cases representing empty string suffixes.
3.  **State Transitions**:
    *   Iterate backwards with $r$ from $m-1$ down to $0$ and $c$ from $n-1$ down to $0$:
        *   If `text1[r] == text2[c]`:
            $$dp[r][c] = 1 + dp[r+1][c+1]$$
        *   If `text1[r] != text2[c]`:
            $$dp[r][c] = \max(dp[r+1][c], dp[r][c+1])$$
4.  **Result**:
    *   The answer for the full strings `text1[0:]` and `text2[0:]` will be stored in `dp[0][0]`.

### Go Code

``` go
func longestCommonSubsequence(text1 string, text2 string) int {
    m, n := len(text1), len(text2)
    dp := make([][]int, m+1)
    for r := 0; r < m+1; r++ {
        dp[r] = make([]int, n+1)
    }

    for r := m-1; r >= 0; r-- {
        for c := n-1; c >= 0; c-- {
            if text1[r] == text2[c] {
                dp[r][c] = 1 + dp[r+1][c+1]
            } else {
                dp[r][c] = max(dp[r+1][c], dp[r][c+1])
            }
        }
    }
    return dp[0][0]
}
```

### Code Efficiency

- **Time Complexity**: $O(m \cdot n)$
    - We fill an $(m+1) \times (n+1)$ grid with two nested loops, doing constant work at each cell.
- **Space Complexity**: $O(m \cdot n)$
    - We allocate a 2D slice of size $(m+1) \times (n+1)$ to store the DP table. This bottom-up approach eliminates recursion stack overhead.
