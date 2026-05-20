# 20. Valid Parentheses

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/valid-parentheses/description/)

## Solution: Stack

To validate balanced parentheses, we use a stack to track open brackets and ensure they are closed in the correct order.

### Thought Process

1.  **Early Exit**: If the string length is odd, it cannot be balanced.
2.  **Stack Usage**:
    - **Open Brackets**: Push `(`, `{`, or `[` onto the stack.
    - **Close Brackets**: When encountering `)`, `}`, or `]`, check the top of the stack.
    - **Validation**: If the stack is empty or the top doesn't match the current closing bracket, return `false`. Otherwise, pop the top.
3.  **Final Check**: After processing all characters, the string is valid only if the stack is empty.

### Go Code

``` go
func isValid(s string) bool {
    if len(s)%2 != 0 {
        return false
    }
    var stack []byte
    for i := 0; i < len(s); i++ {
        c := s[i]
        switch c {
        case '(', '{', '[':
            stack = append(stack, c)
        case ')':
            if len(stack) == 0 || stack[len(stack)-1] != '(' {
                return false
            }
            stack = stack[:len(stack)-1]
        case '}':
            if len(stack) == 0 || stack[len(stack)-1] != '{' {
                return false
            }
            stack = stack[:len(stack)-1]
        case ']':
            if len(stack) == 0 || stack[len(stack)-1] != '[' {
                return false
            }
            stack = stack[:len(stack)-1]
        }
    }
    return len(stack) == 0
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We traverse the string once, and each stack operation (`append` and slicing) takes $O(1)$ time.
- **Space Complexity**: $O(n)$
    - In the worst case (e.g., all open brackets), the stack stores all $n$ characters.

