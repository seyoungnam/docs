# 241. Different Ways to Add Parentheses

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/different-ways-to-add-parentheses/description/)

## Solution: Divide and Conquer / Backtracking (DFS Closure with Memoization)

Adding parentheses to group arithmetic operations can be modeled as selecting which operator is evaluated last. By splitting the expression at each operator index, we can recursively solve the left and right sub-expressions using a **Divide and Conquer** approach. We also use a cache map to memoize subproblem evaluations to avoid redundant calculations.

### Thought Process

1.  **Divide and Conquer Splits**:
    *   Iterate through the string `expr` character by character.
    *   When we encounter an operator (`+`, `-`, or `*`) at index `i`, we divide the problem:
        *   Compute all possible evaluation values for the left sub-expression: `left := dfs(expr[:i])`.
        *   Compute all possible evaluation values for the right sub-expression: `right := dfs(expr[i+1:])`.
    *   Combine results: For each value `l` in `left` and `r` in `right`, evaluate `l [operator] r` and append it to our results slice `res`.
2.  **Base Case (Operand Values)**:
    *   If no operators are found in `expr` after scanning (meaning `isNumber` remains `true`), the sub-expression is a pure number. Convert it to an integer using `strconv.Atoi(expr)` and add it to `res`.
3.  **Memoization Tracking**:
    *   To avoid recalculating the same sub-expression multiple times, we maintain a map `memo := make(map[string][]int)` mapping sub-expressions to their lists of possible outputs.
    *   If `expr` is already present in `memo`, return it immediately.

### Go Code

``` go
import "strconv"

func diffWaysToCompute(expression string) []int {
    memo := make(map[string][]int)

    var dfs func(expr string) []int
    dfs = func(expr string) []int {
        if res, ok := memo[expr]; ok {
            return res
        }

        var res []int
        isNumber := true

        for i := 0; i < len(expr); i++ {
            ch := expr[i]
            if ch == '+' || ch == '-' || ch == '*' {
                isNumber = false

                left := dfs(expr[:i])
                right := dfs(expr[i+1:])

                for _, l := range left {
                    for _, r := range right {
                        switch ch {
                        case '+':
                            res = append(res, l+r)
                        case '-':
                            res = append(res, l-r)
                        case '*':
                            res = append(res, l*r)
                        }
                    }
                }
            }
        }
        if isNumber {
            num, _ := strconv.Atoi(expr)
            res = append(res, num)
        }
        memo[expr] = res
        return res
    }

    return dfs(expression)
}
```

### Code Efficiency

- **Time Complexity**: $O(C_N)$ where $C_N$ is the $N$-th Catalan number (representing the number of valid parenthesization formats of $N$ operators). $C_N = \frac{1}{N+1}\binom{2N}{N}$, which grows exponentially but remains small for the problem constraints. Memoization ensures each unique sub-expression is calculated exactly once.
- **Space Complexity**: $O(C_N)$ auxiliary space. The memoization map stores lists of output integer combinations for unique sub-expressions. The recursion call stack depth goes up to a maximum of $N$ levels.