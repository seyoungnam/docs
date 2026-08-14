# 474. Ones and Zeroes

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/ones-and-zeroes/description/)

This problem is a multi-constraint variation of the classic **0/1 Knapsack Problem**. Instead of a single weight constraint, we have two independent constraints: the maximum number of zeroes ($m$) and the maximum number of ones ($n$). We want to find the size of the largest subset of strings we can form within these limits.

---

## Solution 1: Top-Down Dynamic Programming (DFS with Memoization)

A natural way to solve this is to make a binary decision for each string (either include or exclude it) and cache the results of the subproblems.

### Thought Process

1.  **DFS Decisions**:
    *   For the string at index `i`, we first count the number of zeroes and ones it contains.
    *   **Skip Option**: We do not include the string: `dfs(i+1, m, n)`.
    *   **Take Option**: If `zeros <= m` and `ones <= n`, we can include the string: `1 + dfs(i+1, m-zeros, n-ones)`.
    *   We take the maximum of these two decisions.
2.  **Memoization State**:
    *   The state of our search is uniquely identified by the index `i` and the remaining capacity limits `m` and `n`.
    *   We define a `State` struct:
        ```go
        type State struct {
            idx     int
            zeros   int
            ones    int
        }
        ```
    *   We store computed results in a `memo := make(map[State]int)` map to avoid redundant calculations.

### Go Code

``` go
type State struct {
    idx     int
    zeros   int
    ones    int
}

func findMaxForm(strs []string, m int, n int) int {
    memo := make(map[State]int)
    var dfs func(i, m, n int) int
    dfs = func(i, m, n int) int {
        if i == len(strs) {
            return 0
        }
        state := State{i, m, n}
        if val, ok := memo[state]; ok {
            return val
        }
        zeros, ones := 0, 0
        for j := range strs[i] {
            if strs[i][j] == '0' {
                zeros++
            } else {
                ones++
            }
        }
        memo[state] = dfs(i+1, m, n)
        if zeros <= m && ones <= n {
            memo[state] = max(memo[state], 1+dfs(i+1, m-zeros, n-ones))
        }        
        return memo[state]
    }
    return dfs(0, m, n)
}
```

### Code Efficiency

- **Time Complexity**: $O(L \cdot m \cdot n)$
    - Where $L$ is the number of strings in `strs`. There are at most $L \cdot (m+1) \cdot (n+1)$ unique states. Counting the characters of a string of length $S_{len}$ takes $O(S_{len})$ time. Overall time is bounded by $O(L \cdot m \cdot n + \sum S_{len})$.
- **Space Complexity**: $O(L \cdot m \cdot n)$
    - The size of the memoization map can grow up to $O(L \cdot m \cdot n)$, plus $O(L)$ stack depth for recursion.

---

## Solution 2: Bottom-Up Dynamic Programming (2D DP)

To optimize space and avoid recursion overhead, we can solve the subproblems iteratively.

### Thought Process

1.  **State Definition**:
    *   Define `dp[i][j]` as the maximum number of strings we can form using at most `i` zeroes and `j` ones.
2.  **DP Transitions**:
    *   For each string `s` in `strs`:
        *   Count the number of `zeros` and `ones` it contains.
        *   To avoid using the same string multiple times (since it is a 0/1 knapsack variation), we update the table in **reverse order** (from `m` down to `zeros` and `n` down to `ones`):
            $$dp[M][N] = \max(dp[M][N], 1 + dp[M-\text{zeros}][N-\text{ones}])$$
3.  **Result**:
    *   The answer will be stored in `dp[m][n]` representing the maximum subset size using at most `m` zeroes and `n` ones.

### Go Code

``` go
func findMaxForm(strs []string, m int, n int) int {
    dp := make([][]int, m+1)
    for i := range dp {
        dp[i] = make([]int, n+1)
    }

    for _, s := range strs {
        zeros, ones := 0, 0
        for i := 0; i < len(s); i++ {
            if s[i] == '0' {
                zeros++
            } else {
                ones++
            }
        }
        for M := m; M >= zeros; M-- {
            for N := n; N >= ones; N-- {
                dp[M][N] = max(dp[M][N], 1 + dp[M-zeros][N-ones])
            }
        }
    }
    return dp[m][n]
}
```

### Code Efficiency

- **Time Complexity**: $O(L \cdot m \cdot n)$
    - Where $L$ is the number of strings. The outer loop runs $L$ times. For each string, we iterate backwards through the `dp` matrix of size $(m+1) \times (n+1)$, performing a constant-time transition.
- **Space Complexity**: $O(m \cdot n)$
    - We only need a 2D array of size $(m+1) \times (n+1)$, eliminating the $O(L)$ dependency from the state space.