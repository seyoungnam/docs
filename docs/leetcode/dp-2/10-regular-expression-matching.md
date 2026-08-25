# 10. Regular Expression Matching

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/regular-expression-matching/description/)

Given an input string `s` and a pattern `p`, implement regular expression matching with support for `'.'` and `'*'` where:
*   `'.'` Matches any single character.
*   `'*'` Matches zero or more of the preceding element.

The matching should cover the **entire** input string (not partial).

---

## Solution 1: Recursive DFS (Naive)

We can model the regular expression matching as a decision tree search. At each state, we compare the current character of the string `s` and the pattern `p`.

### Thought Process

1.  **DFS Indices**:
    *   `i`: Pointer index in string `s`.
    *   `j`: Pointer index in pattern `p`.
2.  **Base Case**:
    *   If `j == len(p)` (the end of the pattern has been reached), the string is matched successfully if and only if `i == len(s)` (the entire string has been consumed).
3.  **Recursive Matching**:
    *   First, determine if the current character matches:
        $$\text{match} = i < \text{len}(s) \land (s[i] == p[j] \lor p[j] == '.')$$
    *   **Case 1: Handling `*`**: If the next character in the pattern is `'*'` (`p[j+1] == '*'`), we have two choices:
        1.  **Skip the `*` pattern**: Match zero occurrences of $p[j]$. This shifts the pattern pointer by 2: `dfs(i, j + 2)`.
        2.  **Use the `*` pattern**: Match one (or more) occurrence of $p[j]$. This is only valid if `match` is true. We consume the character in `s` by incrementing `i`, but keep `j` unchanged to allow matching subsequent occurrences: `dfs(i + 1, j)`.
        *   Return `dfs(i, j+2) || (match && dfs(i+1, j))`.
    *   **Case 2: Single Character Match**: If no `*` follows and `match` is true, we consume one character from both `s` and `p`: `dfs(i + 1, j + 1)`.
    *   If neither case applies, return `false`.

### Go Code

``` go
func isMatch(s string, p string) bool {
    m, n := len(s), len(p)

    var dfs func(i, j int) bool
    dfs = func(i, j int) bool {
        // Base case: end of pattern
        if j == n {
            return i == m
        }

        // Check if current characters match
        match := i < m && (s[i] == p[j] || p[j] == '.')

        // Case 1: Handle '*' wildcard
        if (j+1) < n && p[j+1] == '*' {
            return dfs(i, j+2) || (match && dfs(i+1, j))
        }

        // Case 2: Match single character
        if match {
            return dfs(i+1, j+1)
        }

        return false
    }
    return dfs(0, 0)
}
```

### Code Efficiency

- **Time Complexity**: $O(2^{M + N})$ in the worst case
    - Without caching, states are evaluated repeatedly, leading to exponential time complexity.
- **Space Complexity**: $O(M + N)$
    - The auxiliary space is determined by the recursion call stack, which can go up to depth $M + N$.

---

## Solution 2: DFS with Memoization

We can optimize Solution 1 by caching the results of computed state coordinates `(i, j)` to avoid duplicate calculations.

### Thought Process

1.  **Memoization Key**:
    *   We use a map `memo` with key `[2]int{i, j}` to store the boolean match result of the state.
2.  **Logic**:
    *   Before performing recursive evaluations, check if the state `[2]int{i, j}` exists in `memo`. If it does, return the cached result immediately.
    *   Otherwise, compute the result, cache it in `memo`, and return.

### Go Code

``` go
func isMatch(s string, p string) bool {
    m, n := len(s), len(p)
    memo := map[[2]int]bool{}

    var dfs func(i, j int) bool
    dfs = func(i, j int) bool {
        if j == n {
            return i == m
        }

        // Check cache
        if val, ok := memo[[2]int{i, j}]; ok {
            return val
        }
        
        match := i < m && (s[i] == p[j] || p[j] == '.')

        // Case 1: Handle '*' wildcard
        if (j+1) < n && p[j+1] == '*' {
            memo[[2]int{i, j}] = dfs(i, j+2) || (match && dfs(i+1, j))
            return memo[[2]int{i, j}]
        }

        // Case 2: Match single character
        if match {
            memo[[2]int{i, j}] = dfs(i+1, j+1)
        } else {
            memo[[2]int{i, j}] = false
        }
        return memo[[2]int{i, j}]
    }
    return dfs(0, 0)
}
```

### Code Efficiency

- **Time Complexity**: $O(M \times N)$
    - Where $M$ is the length of string `s` and $N$ is the length of pattern `p`. Since there are at most $(M+1) \times (N+1)$ unique state combinations, and each state is computed in $O(1)$ time, the time complexity is bounded by $O(M \times N)$.
- **Space Complexity**: $O(M \times N)$
    - We use $O(M \times N)$ auxiliary space to store results in the `memo` map, and $O(M + N)$ space for the DFS recursion call stack.