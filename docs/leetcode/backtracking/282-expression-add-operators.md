# 282. Expression Add Operators

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/expression-add-operators/description/)

## Solution: Backtracking (DFS Closure with Operator Precedence Tracking)

To insert operators (`+`, `-`, `*`) into a digit string `num` so that it evaluates to a target value, we can use recursive depth-first backtracking. The main challenge is handling multiplication operator precedence correctly, which we solve by tracking the value of the previous term.

### Thought Process

1.  **Multiplication Precedence Tracking (`prev`)**:
    *   To evaluate multiplication correctly without parsing the whole string iteratively (e.g., evaluating `2 + 3 * 4` as `14` instead of `20`), we pass a `prev` signed term parameter in our DFS recursion.
    *   When we perform multiplication, we "undo" the last term's addition or subtraction from `currSum` and add the multiplied term instead:
        $$\text{newSum} = (\text{currSum} - \text{prev}) + (\text{prev} \times \text{val})$$
    *   The new `prev` term for the next recursive step becomes $\text{prev} \times \text{val}$.
    *   By tracking `prev` as a signed value, subtraction cases are also evaluated correctly. For example, in `2 - 3 * 4`, when we process the `- 3`, `prev` becomes `-3`. When processing `* 4`:
        $$\text{newSum} = (\text{currSum} - (-3)) + ((-3) \times 4) = (\text{currSum} + 3) - 12$$
2.  **In-Place Byte Buffer Optimization**:
    *   To avoid string allocation and concatenation overhead at every branch, we pre-allocate a single byte slice `path` with a capacity of `len(num) * 2`.
    *   We append operators and characters to `path` during recursion and backtrack by resetting the slice length to its entry length: `path = path[:curLen]`.
3.  **Pruning (Leading Zeros)**:
    *   Multi-digit operands cannot start with leading zeros (e.g. `"05"` is invalid, but `"0"` is valid). If `end > start` and `num[start] == '0'`, we `break` the loop immediately.
4.  **Recursive Decisions**:
    *   Iterate `end` from `start` to `len(num) - 1` to extract operand values.
    *   If `start == 0`, we are processing the first number in our expression (which has no preceding operator). We call `dfs(end + 1, val, val, ...)` directly.
    *   Otherwise, try inserting `+`, `-`, and `*` sequentially, resetting the `path` slice length after each recursive call to backtrack.
5.  **Base Case**:
    *   When `start == len(num)`, the entire string is processed. If `currSum == target`, the expression is valid; convert the `path` byte slice to a string and append it to our results slice `res`.

### Go Code

``` go
func addOperators(num string, target int) []string {
    res := make([]string, 0)
    if len(num) == 0 {
        return res
    }
    // pre-allocate
    path := make([]byte, 0, len(num)*2)

    var dfs func(start int, prev int, currSum int, path []byte)
    dfs = func(start int, prev int, currSum int, path []byte) {
        if start == len(num) {
            if currSum == target {
                res = append(res, string(path))
            }
            return
        }

        curLen := len(path)
        val := 0
        for end := start; end < len(num); end++ {
            if end > start && num[start] == '0' {
                break
            }
            val = val*10 + int(num[end]-'0')
            strNum := num[start:end+1]

            if start == 0 {
                dfs(end+1, val, val, append(path, strNum...))
            } else {
                // +
                dfs(end+1, val, currSum+val, append(append(path, '+'), strNum...))
                path = path[:curLen]

                // -
                dfs(end+1, -val, currSum-val, append(append(path, '-'), strNum...))
                path = path[:curLen]

                // *
                dfs(end+1, prev*val, (currSum-prev)+(prev*val), append(append(path, '*'), strNum...))
                path = path[:curLen]
            }
        }
    }
    
    dfs(0, 0, 0, path)
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(4^N)$
    - Let $N$ be the length of the string `num`. At each split point between digits, we have 4 choices: join digits, insert `+`, insert `-`, or insert `*`. The total number of states is bounded by $4^N$.
- **Space Complexity**:
    - **With Output**: $O(4^N)$ to store all matching expression string combinations.
    - **Auxiliary Space**: $O(N)$ for the recursion stack depth (at most $N$) and the pre-allocated temporary `path` byte slice.