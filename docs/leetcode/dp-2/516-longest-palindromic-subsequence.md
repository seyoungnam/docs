# 516. Longest Palindromic Subsequence

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/longest-palindromic-subsequence/description/)

Given a string `s`, find the longest palindromic subsequence's length in `s`. A subsequence is a sequence that can be derived from another sequence by deleting some or no elements without changing the order of the remaining elements.

---

## Solution 1: Top-Down Dynamic Programming (DFS with Memoization via LCS Reduction)

A key mathematical insight is that a palindromic subsequence reads the same forwards and backwards. Therefore, finding the **Longest Palindromic Subsequence (LPS)** of `s` is equivalent to finding the **Longest Common Subsequence (LCS)** between `s` and its reversed string `t = reverse(s)`.

### Thought Process

1.  **String Reversal**:
    *   Create `t = reverse(s)`.
2.  **State Definition**:
    *   Let `dfs(r, c)` represent the length of the longest common subsequence between suffixes `s[r:]` and `t[c:]`.
3.  **Base Case**:
    *   If `r == len(s)` or `c == len(t)`, at least one string has ended, so return `0`.
4.  **State Transitions**:
    *   **Matching Characters (`s[r] == t[c]`)**: Advance both pointers and increment the count:
        $$dfs(r, c) = 1 + dfs(r+1, c+1)$$
    *   **Mismatching Characters (`s[r] != t[c]`)**: Take the maximum between skipping in `s` and skipping in `t`:
        $$dfs(r, c) = \max(dfs(r+1, c), dfs(r, c+1))$$
5.  **Memoization**:
    *   Use a 2D matrix `memo` of size $n \times n$ initialized to `-1` to cache intermediate subproblem results.

### Go Code

``` go
func longestPalindromeSubseq(s string) int {
    t := reverse(s)
    n := len(s)
    memo := make([][]int, n)
    for r := range memo {
        memo[r] = make([]int, n)
        for c := range memo[r] {
            memo[r][c] = -1
        }
    }
    var dfs func(r, c int) int
    dfs = func(r, c int) int {
        if r == n || c == n {
            return 0
        }
        if memo[r][c] != -1 {
            return memo[r][c]
        }
        if s[r] == t[c] {
            memo[r][c] = 1 + dfs(r+1, c+1)
        } else {
            memo[r][c] = max(dfs(r+1, c), dfs(r, c+1))
        }
        return memo[r][c]
    }
    return dfs(0, 0)
}

func reverse(s string) string {
    arr := []byte(s)
    for i, j := 0, len(s)-1; i < j; i, j = i+1, j-1 {
        arr[i], arr[j] = arr[j], arr[i]
    }
    return string(arr)
}
```

### Code Efficiency

- **Time Complexity**: $O(n^2)$
    - Where $n$ is the length of `s`. There are $n \times n$ unique $(r, c)$ states computed once. Reversing the string takes $O(n)$ time.
- **Space Complexity**: $O(n^2)$
    - The 2D `memo` table takes $O(n^2)$ space, plus $O(n)$ auxiliary space for the recursion call stack.

---

## Solution 2: Bottom-Up Dynamic Programming (2D DP via LCS Reduction)

We can convert the LCS subproblems into a 2D bottom-up iterative DP table.

### Thought Process

1.  **State Definition**:
    *   Define `dp[r][c]` as the length of the longest common subsequence of suffixes `s[r:]` and `t[c:]` (where `t = reverse(s)`).
2.  **DP Table Initialization**:
    *   Create a 2D table `dp` of size $(n+1) \times (n+1)$ initialized to `0`.
3.  **State Transitions**:
    *   Iterate backwards with $r$ from $n-1$ down to $0$ and $c$ from $n-1$ down to $0$:
        *   If `s[r] == t[c]`:
            $$dp[r][c] = 1 + dp[r+1][c+1]$$
        *   If `s[r] != t[c]`:
            $$dp[r][c] = \max(dp[r+1][c], dp[r][c+1])$$
4.  **Result**:
    *   `dp[0][0]` stores the maximum length of the palindromic subsequence.

### Go Code

``` go
func longestPalindromeSubseq(s string) int {
    t := reverse(s)
    n := len(s)
    dp := make([][]int, n+1)
    for r := range dp {
        dp[r] = make([]int, n+1)
    }
    for r := n-1; r >= 0; r-- {
        for c := n-1; c >= 0; c-- {
            if s[r] == t[c] {
                dp[r][c] = 1 + dp[r+1][c+1]
            } else {
                dp[r][c] = max(dp[r+1][c], dp[r][c+1])
            }
        }
    }
    return dp[0][0]
}

func reverse(s string) string {
    arr := []byte(s)
    for i, j := 0, len(s)-1; i < j; i, j = i+1, j-1 {
        arr[i], arr[j] = arr[j], arr[i]
    }
    return string(arr)
}
```

### Code Efficiency

- **Time Complexity**: $O(n^2)$
    - Reversing `s` takes $O(n)$ time. Filling the $(n+1) \times (n+1)$ table takes $O(n^2)$ time.
- **Space Complexity**: $O(n^2)$
    - The 2D table requires $O(n^2)$ auxiliary space.