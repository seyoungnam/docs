# 678. Valid Parenthesis String

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/valid-parenthesis-string/description/)

Given a string `s` containing only three types of characters: `'('`, `')'` and `'*'`, return `true` *if* `s` *is **valid***.

The rules for a **valid** string are as follows:
*   Any left parenthesis `'('` must have a corresponding right parenthesis `')'`.
*   Any right parenthesis `')'` must have a corresponding left parenthesis `'('`.
*   Left parenthesis `'('` must go before the corresponding right parenthesis `')'`.
*   `'*'` could be treated as a single right parenthesis `')'` or a single left parenthesis `'('` or an empty string `""`.

---

## Solution 1: Top-Down DFS with Memoization

We can view the validation as a decision tree search. At each character, we decide how to treat it. If the character is `'*'`, we branch in three directions (treating it as `'('`, `')'`, or `""`). To avoid redundant computations, we use memoization.

### Thought Process

1.  **State Definition**:
    *   Let `dfs(i, open, close)` represent whether the substring starting at index `i` is valid, given that we have already seen `open` left parentheses and `close` right parentheses.
2.  **Base Cases**:
    *   If we reach the end of the string (`i == len(s)`):
        *   Return `true` if all parentheses are balanced: `open == close`.
        *   Return `false` otherwise.
    *   If at any point the number of right parentheses exceeds left parentheses (`open < close`):
        *   It is invalid, so return `false` early.
3.  **Recursive Transitions**:
    *   If `s[i] == '('`, recurse to the next state: `dfs(i + 1, open + 1, close)`.
    *   If `s[i] == ')'`, recurse to the next state: `dfs(i + 1, open, close + 1)`.
    *   If `s[i] == '*'`, try all three possibilities:
        *   Treat as `'('`: `dfs(i + 1, open + 1, close)`
        *   Treat as `')'`: `dfs(i + 1, open, close + 1)`
        *   Treat as `""`: `dfs(i + 1, open, close)`
        *   Return `true` if any of these branches succeed.
4.  **Memoization**:
    *   Store results in a hash map `memo` with a composite key of `Info{idx, open, close}`.

### Go Code

``` go
type Info struct {
    idx     int
    open    int
    close   int
}

func checkValidString(s string) bool {
    n := len(s)
    memo := map[Info]bool{}
    
    var dfs func(i, open, close int) bool
    dfs = func(i, open, close int) bool {
        if i == n {
            return open == close
        }
        if open < close {
            return false
        }
        
        entry := Info{i, open, close}
        if val, ok := memo[entry]; ok {
            return val
        }
        
        if s[i] == '(' {
            memo[entry] = dfs(i+1, open+1, close)
        } else if s[i] == ')' {
            memo[entry] = dfs(i+1, open, close+1)
        } else {
            // '*' can be '(', ')', or empty
            memo[entry] = dfs(i+1, open+1, close) || 
                          dfs(i+1, open, close+1) || 
                          dfs(i+1, open, close)
        }
        
        return memo[entry]
    }
    
    return dfs(0, 0, 0)
}
```

### Code Efficiency

- **Time Complexity**: $O(N^3)$
    - Where $N$ is the length of the string `s`. The state space is bounded by the combinations of index `i` ($O(N)$), `open` ($O(N)$), and `close` ($O(N)$). Each state takes $O(1)$ time to compute.
- **Space Complexity**: $O(N^3)$
    - To store the subproblem states in the `memo` map, plus $O(N)$ space for the recursion call stack.

---

## Solution 2: Greedy (Interval Range / Linear Scan)

We can optimize the solution to $O(N)$ time and $O(1)$ space by tracking the minimum and maximum possible number of open parentheses that could remain unmatched.

### Thought Process

1.  **Bounds Tracking**:
    *   Maintain two integers `leftMin` and `leftMax`:
        *   `leftMin`: The minimum possible number of unmatched left parentheses.
        *   `leftMax`: The maximum possible number of unmatched left parentheses.
2.  **Transitions**:
    *   If `c == '('`: Both boundaries increase: `leftMin++, leftMax++`.
    *   If `c == ')'`: Both boundaries decrease: `leftMin--, leftMax--`.
    *   If `c == '*'`:
        *   If we treat it as `')'`, `leftMin` decreases.
        *   If we treat it as `'('`, `leftMax` increases.
        *   If we treat it as `""`, the bounds remain unchanged.
        *   Thus, we update: `leftMin--, leftMax++`.
3.  **Validations**:
    *   If `leftMax < 0`: Even if we converted all `'*'` to `'('`, we still had too many right parentheses `')'`. The string is invalid; return `false`.
    *   If `leftMin < 0`: Having `leftMin < 0` implies that treating all `'*'` as `')'` was too aggressive. Since we are not forced to treat them as `')'`, we can treat some of them as `""` or `'('`. Thus, we reset `leftMin = 0`.
4.  **Final Check**:
    *   After scanning the entire string, the string is valid if and only if we can achieve exactly 0 unmatched left parentheses: `leftMin == 0`.

### Go Code

``` go
func checkValidString(s string) bool {
    leftMin, leftMax := 0, 0

    for _, c := range s {
        if c == '(' {
            leftMin, leftMax = leftMin+1, leftMax+1
        } else if c == ')' {
            leftMin, leftMax = leftMin-1, leftMax-1
        } else {
            // c == '*'
            leftMin, leftMax = leftMin-1, leftMax+1
        }
        
        // Too many right parentheses
        if leftMax < 0 {
            return false
        }
        
        // Minimum possible unmatched '(' cannot be negative
        if leftMin < 0 {
            leftMin = 0
        }
    }
    
    return leftMin == 0    
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - We make a single linear pass through the string `s`.
- **Space Complexity**: $O(1)$
    - We only use two integer variables, achieving optimal space complexity.