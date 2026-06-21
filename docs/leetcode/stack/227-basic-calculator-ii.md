# 227. Basic Calculator II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/basic-calculator-ii/description/)

## Solution: Stack-Based Evaluation

To evaluate mathematical expressions containing `+`, `-`, `*`, and `/`, we can use a stack to store intermediate values. Since multiplication and division have higher precedence, we perform them immediately on the top element of the stack. Addition and subtraction operations are deferred by pushing the values (with their signs) onto the stack to be summed up at the end.

### Thought Process

1.  **Initialization**:
    - A value stack `stack` to store numbers.
    - An operator stack `opStack` to store the active operator immediately preceding a number. We initialize `opStack` with `+` so that the first parsed number is pushed onto the stack as a positive number.
2.  **Linear Traversal**:
    - Iterate through the string using a index pointer `i`.
    - **Parsing Numbers**: If `s[i]` is a digit, read forward to extract the full multi-digit integer. Pop the last operator from `opStack` and evaluate:
        - If the operator is `+`, append the parsed number to `stack`.
        - If the operator is `-`, append the negative of the parsed number to `stack`.
        - If the operator is `*`, multiply the top element of `stack` by the parsed number.
        - If the operator is `/`, divide the top element of `stack` by the parsed number.
    - **Parsing Operators**: If `s[i]` is one of `+`, `-`, `*`, or `/`, push it onto `opStack`.
    - **Spaces**: Ignore spaces and advance.
3.  **Result Aggregation**:
    - After processing the entire string, sum all elements in `stack` to get the final result.

### Go Code

``` go
import (
	"strconv"
)

func calculate(s string) int {
	stack := make([]int, 0)
	opStack := make([]byte, 0)
	opStack = append(opStack, '+')
	i := 0
	for i < len(s) {
		if s[i] >= '0' && s[i] <= '9' {
			j := i + 1
			for j < len(s) && s[j] >= '0' && s[j] <= '9' {
				j++
			}
			val, _ := strconv.Atoi(s[i:j])
			operator := opStack[len(opStack)-1]
			opStack = opStack[:len(opStack)-1]
			if operator == '+' {
				stack = append(stack, val)
			} else if operator == '-' {
				stack = append(stack, -val)
			} else if operator == '*' {
				stack[len(stack)-1] *= val
			} else if operator == '/' {
				stack[len(stack)-1] /= val
			}
			i = j
			continue
		}
		if s[i] == '+' || s[i] == '-' || s[i] == '*' || s[i] == '/' {
			opStack = append(opStack, s[i])
			i++
			continue
		}
		i++
	}
	res := 0
	for _, val := range stack {
		res += val
	}
	return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We iterate through the string of length $n$ exactly once. Each character is processed a constant number of times. Summing up the stack at the end takes $O(n)$ time in the worst case.
- **Space Complexity**: $O(n)$
    - The value stack and the operator stack can hold up to $O(n)$ elements in the worst case (e.g. `1+1+1+1...`).

---

## Optimized Solution: Stackless ($O(1)$ Space)

Instead of using a stack to defer addition and subtraction operations, we can maintain the running evaluation state using three registers: `curr` (the number currently being parsed), `last` (the value of the last term evaluated), and `res` (the accumulated sum of all fully resolved terms).

### Thought Process

1.  **Registers**:
    - `curr`: Constructed character-by-character as we traverse digits.
    - `last`: Represents the evaluated result of the most recent term. When we see `*` or `/`, we immediately apply it to `last`. When we see `+` or `-`, the previous term is fully resolved, so we add `last` to `res` and set `last` to the new term.
    - `res`: Holds the running sum of all terms that are no longer subject to multiplication or division.
    - `sign`: Keeps track of the operator immediately preceding the `curr` (initialized to `+`).
2.  **State Transitions on Operator or End-of-String**:
    - When we encounter an operator (except spaces) OR reach the end of the string:
        - If the preceding `sign` was `+`, set `last = curr`. (We also add the previous `last` to `res` first).
        - If the preceding `sign` was `-`, set `last = -curr`.
        - If the preceding `sign` was `*`, update `last = last * curr`.
        - If the preceding `sign` was `/`, update `last = last / curr`.
        - Update `sign` to the current operator, and reset `curr = 0`.
3.  **Result**:
    - At the end of the string, add the final `last` to `res` and return.

### Go Code

``` go
func calculate(s string) int {
    if len(s) == 0 {
        return 0
    }
    curr, last, res := 0, 0, 0
    var sign byte = '+'

    for i := 0; i < len(s); i++ {
        ch := s[i]
        if ch >= '0' && ch <= '9' {
            curr = curr*10 + int(ch-'0')
        }
        if (ch != ' ' && (ch < '0' || ch > '9')) || i == len(s)-1 {
            switch sign {
            case '+':
                res += last
                last = curr
            case '-':
                res += last
                last = -curr
            case '*':
                last = last*curr
            case '/':
                last = last/curr
            }
            sign = ch
            curr = 0
        }
    }
    res += last
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We traverse the string of length $n$ exactly once. Each character is processed in constant $O(1)$ time.
- **Space Complexity**: $O(1)$
    - We only use a constant number of integer variables and one byte variable for state tracking, requiring $O(1)$ auxiliary space.