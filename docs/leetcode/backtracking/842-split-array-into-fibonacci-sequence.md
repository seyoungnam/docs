# 842. Split Array into Fibonacci Sequence

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/split-array-into-fibonacci-sequence/description/)

## Solution: Backtracking (DFS Closure with Early Termination)

To partition a numeric string into a sequence of at least 3 integers that satisfy the Fibonacci relation, we can use recursive backtracking. By checking constraints (leading zeros, 32-bit integer limits, and summation requirements) at each step, we can prune invalid branches immediately.

### Thought Process

1.  **Sequence Constraints**:
    *   Each number must fit in a 32-bit signed integer:
        $$\text{val} \le 2^{31} - 1 \quad (\text{math.MaxInt32})$$
    *   No number can have leading zeros (e.g. `"05"` is invalid, but `"0"` is valid).
    *   For any three consecutive numbers in the sequence, the third must be the sum of the first two:
        $$\text{expectedSum} = res[k-2] + res[k-1]$$
2.  **Pruning Heuristics**:
    *   **Leading Zeros**: If `end > start` and `num[start] == '0'`, we `break` the loop because no valid multi-digit number can start with `0`.
    *   **Overflow**: If `val > math.MaxInt32`, we `break` the loop immediately.
    *   **Fibonacci Sum check**: If we have at least 2 numbers in our path slice `res`:
        *   Calculate `expectedSum = res[len(res)-2] + res[len(res)-1]`.
        *   If `val > expectedSum`, `break` the loop. Since extending the digit window further will only increase `val`, it will remain larger than `expectedSum`.
        *   If `val < expectedSum`, `continue` extending the number (it is too small).
        *   If `val == expectedSum`, we proceed to append and recurse.
3.  **Recursive Branching & Early Termination**:
    *   If a candidate `val` is valid (or if `len(res) < 2`), append `val` to `res` and recursively call `dfs(end + 1)`.
    *   If `dfs` returns `true`, propagate `true` immediately to stop the search (early termination).
    *   Otherwise, backtrack by popping `val` from `res`.
4.  **Base Case**:
    *   When the index pointer `start == len(num)`:
        *   If the sequence contains at least 3 numbers (`len(res) > 2`), return `true` (success).
        *   Otherwise, return `false`.

### Go Code

``` go
import "math"

func splitIntoFibonacci(num string) []int {
    res := make([]int, 0)
    n := len(num)

    var dfs func(start int) bool
    dfs = func(start int) bool {
        if start == n && len(res) > 2 {
            return true
        }
        val := 0
        for end := start; end < n; end++ {
            if end > start && num[start] == '0' {
                break
            }
            val = val*10 + int(num[end]-'0')
            if val > math.MaxInt32 {
                break
            }
            if len(res) >= 2 {
                expectedSum := res[len(res)-2] + res[len(res)-1]
                if val > expectedSum {
                    break
                }
                if val < expectedSum {
                    continue
                }
            }
            res = append(res, val)
            if dfs(end+1) {
                return true
            }
            res = res[:len(res)-1]
        }
        return false
    }
    if dfs(0) {
        return res
    }
    return []int{}
}
```

### Code Efficiency

- **Time Complexity**: $O(N \log^2(\text{MaxInt32}))$
    - Although the theoretical search complexity is exponential, each number is bounded by $2^{31}-1$, which has at most 10 digits. Therefore, the branching factor is extremely limited (at most 10 choices for the first two elements), and subsequent elements are uniquely determined by the sum constraint. This makes the search run in virtually $O(N^2)$ time in practice.
- **Space Complexity**: $O(N)$ auxiliary space. The recursion call stack depth goes up to a maximum of $N$, and the `res` slice holds at most $N$ integers.