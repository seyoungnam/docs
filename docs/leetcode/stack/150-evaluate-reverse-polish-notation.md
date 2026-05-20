# 150. Evaluate Reverse Polish Notation

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/evaluate-reverse-polish-notation/description/)

## Solution: Operand Stack

Reverse Polish Notation (RPN) is naturally solved using a stack. Numbers are pushed onto the stack, and operators consume the top two elements, pushing the result back.

### Thought Process

1.  **Stack Initialization**: Use a slice as a stack to store operands. Pre-allocate capacity to `len(tokens)` to prevent dynamic resizing.
2.  **Linear Traversal**: Iterate through the tokens.
3.  **Operator Handling**: When an operator (`+`, `-`, `*`, `/`) is found, perform the operation on the top two elements of the stack. Update the second-to-top element in-place and shrink the stack.
4.  **Operand Handling (Optimized)**: 
    - **Fast-Path**: If the token is a single digit or a single-digit negative number, parse it using ASCII math for performance.
    - **Fallback**: Use `strconv.Atoi` only for multi-digit numbers.
5.  **Result**: The final value remaining in the stack is the result.

### Go Code

``` go
import (
	"strconv"
)

func evalRPN(tokens []string) int {
	if len(tokens) == 0 {
		return 0
	}

	// 1. Pre-allocate capacity to completely prevent heap resizing loops
	stack := make([]int, 0, len(tokens))

	for i := 0; i < len(tokens); i++ {
		t := tokens[i]

		// 2. A switch statement speeds up branch execution over sequential if-else checks
		switch t {
		case "+":
			stack[len(stack)-2] += stack[len(stack)-1]
			stack = stack[:len(stack)-1]
		case "-":
			stack[len(stack)-2] -= stack[len(stack)-1]
			stack = stack[:len(stack)-1]
		case "*":
			stack[len(stack)-2] *= stack[len(stack)-1]
			stack = stack[:len(stack)-1]
		case "/":
			stack[len(stack)-2] /= stack[len(stack)-1]
			stack = stack[:len(stack)-1]
		default:
			// 3. Fast-Path Optimization: If it's a single digit string, resolve via ASCII math
			if len(t) == 1 {
				stack = append(stack, int(t[0]-'0'))
			} else if len(t) == 2 && t[0] == '-' {
				// Fast-Path for single digit negative numbers (e.g., "-5")
				stack = append(stack, -int(t[1]-'0'))
			} else {
				// Fallback to heavy string conversion only for multi-digit numbers
				num, _ := strconv.Atoi(t)
				stack = append(stack, num)
			}
		}
	}
	return stack[0]
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We iterate through the list of $n$ tokens exactly once. Each stack operation and number conversion takes $O(1)$ time.
- **Space Complexity**: $O(n)$
    - In the worst case (e.g., an expression with many operands before any operators), the stack can store up to $n$ elements.

