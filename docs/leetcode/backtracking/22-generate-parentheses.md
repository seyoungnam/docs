# 22. Generate Parentheses

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/generate-parentheses/description/)

## Solution: Backtracking (DFS Closure with In-Place Overwriting)

### Thought Process

- **Balanced String Constraint**: A valid parentheses string must satisfy two conditions at any point during construction:
    1. The number of opening parentheses `open` cannot exceed `n`.
    2. The number of closing parentheses `close` cannot exceed the number of opening parentheses currently in the string (`close < open`).
- **In-Place Byte Slice Overwriting**:
    *   We pre-allocate a single byte slice `curr` of length `2 * n` at the start.
    *   The write index `i` is determined dynamically by the sum of open and close parenthesis counts: `i := open + close`.
    *   When exploring paths:
        *   **Add `(`**: If `open < n`, set `curr[i] = '('` and recurse with `open + 1`.
        *   **Add `)`**: If `close < open`, set `curr[i] = ')'` and recurse with `close + 1`.
        *   Overwriting `curr[i]` directly in place avoids slice resizing, dynamic appending, or backtrack pop operations.
- **Base Case**:
    *   When `open == n` and `close == n`, the byte slice is fully populated. We convert `curr` to a string and append it to our results slice `res`.

### Go Code

``` go
func generateParenthesis(n int) []string {
    res := make([]string, 0)
    curr := make([]byte, 2*n)
    
    var dfs func(open, close int)
    dfs = func(open int, close int) {
        if open == n && close == n {
            res = append(res, string(curr))
            return
        }
        
        i := open + close
        
        if open < n {
            curr[i] = '('
            dfs(open+1, close)
        }
        
        if close < open {
            curr[i] = ')'
            dfs(open, close+1)
        }
    }
    dfs(0, 0)
    return res
}
```

### Code Efficiency

- **Time Complexity**: $\mathcal{O}(\frac{4^n}{\sqrt{n}})$
    - The number of valid parentheses sequences of length $2n$ is given by the $n$-th **Catalan number**: $C_n = \frac{1}{n+1}\binom{2n}{n}$.
    - Asymptotically, $C_n$ grows as $\mathcal{O}(\frac{4^n}{n\sqrt{n}})$.
    - Since we spend $\mathcal{O}(n)$ time to convert each valid byte slice to a string and append it to the result, the total complexity is $\mathcal{O}(\frac{4^n}{\sqrt{n}})$.
- **Space Complexity**: $\mathcal{O}(n)$
    - The auxiliary space is determined by the maximum depth of the recursion stack, which is $2n$.
    - This excludes the space required to store the final output.