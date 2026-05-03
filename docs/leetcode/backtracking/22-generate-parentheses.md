# 22. Generate Parentheses

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/generate-parentheses/description/)

## Solution: Backtracking

### Thought Process

- **Balanced String Constraint**: A valid parentheses string must satisfy two conditions at any point during construction:
    1. The number of opening parentheses (`openParen`) cannot exceed `n`.
    2. The number of closing parentheses (`closeParen`) cannot exceed the number of opening parentheses currently in the string.
- **Recursive Building**: We build the string character by character using a byte slice, tracking the counts of `open` and `close` parentheses.
- **Decision Tree**:
    - **Add `(`**: Permitted if `open < n`.
    - **Add `)`**: Permitted if `open > close`.
- **Backtracking via Slices**: By passing the result of `append(curr, ...)` to the recursive calls, we effectively branch the state. Since slices are passed by value (but share the underlying array), each branch can safely add its respective parenthesis without interfering with other branches at the same depth.
- **Base Case**: When the number of closing parentheses reaches `n`, the string is complete and valid. We convert the byte slice to a string and add it to the result.

### Go Code

``` go
const (
    openParen  byte = '('
    closeParen byte = ')'
)

func generateParenthesis(n int) []string {
    res := []string{}
    dfs(n, 0, 0, []byte{}, &res)
    return res
}

func dfs(n int, open int, close int, curr []byte, res *[]string) {
    if close == n {
        *res = append(*res, string(curr))
        return
    }
    if open < n {
        dfs(n, open+1, close, append(curr, openParen), res)
    }
    if open > close {
        dfs(n, open, close+1, append(curr, closeParen), res)
    }
    return
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