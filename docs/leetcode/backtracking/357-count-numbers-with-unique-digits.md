# 357. Count Numbers with Unique Digits

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/count-numbers-with-unique-digits/description/)

## Solution: Backtracking (DFS Closure)

To count all numbers with unique digits in the range $0 \le x < 10^n$, we can use recursive backtracking. We construct valid numbers digit-by-digit up to length $n$, keeping track of visited digits in the active search path to enforce uniqueness.

### Thought Process

1.  **Iterative Digit Construction**:
    *   We build numbers digit-by-digit.
    *   Maintain a boolean slice `visited` of size 10 to track which digits from `0` to `9` are already in the current number.
2.  **Initial State & Base Case**:
    *   Initialize `count = 1` (to account for the single-digit number `0`).
    *   The recursive closure is `dfs(length)`. If `length == n`, we have reached the maximum allowed number length: return.
3.  **Recursive Branching & Pruning**:
    *   At the current `length`, iterate through candidate digits `num` from `0` to `9`:
        *   **Leading Zeros**: A multi-digit number cannot start with `0` (which would correspond to a duplicate count of a shorter number). If `length == 0` and `num == 0`, we `continue` the loop.
        *   **Digit Uniqueness**: If `visited[num]` is `true`, skip this digit.
        *   If valid:
            1. Mark the digit as visited: `visited[num] = true`.
            2. Increment the unique numbers counter: `count++`.
            3. Recurse to place the next digit: `dfs(length+1)`.
            4. Backtrack: Mark the digit as unvisited: `visited[num] = false`.

### Go Code

``` go
func countNumbersWithUniqueDigits(n int) int {
    if n == 0 {
        return 1
    }
    count := 1
    visited := make([]bool, 10)
    
    var dfs func(length int)
    dfs = func(length int) {
        if length == n {
            return
        }
        for num := 0; num <= 9; num++ {
            if length == 0 && num == 0 {
                continue
            }
            if visited[num] {
                continue
            }
            visited[num] = true
            count++
            dfs(length+1)
            visited[num] = false
        }
    }
    dfs(0)
    return count
}
```

### Code Efficiency

- **Time Complexity**: $O(P(10, n))$
    - Where $P(10, n) = \frac{10!}{(10-n)!}$ is the number of permutations of 10 digits taken $n$ at a time. Since the problem specifies $n \le 8$, the maximum number of recursive calls is small (under $10^7$ operations in the worst case), executing in a few milliseconds.
    - *Note*: While a pure permutation math-based approach can solve this in $O(n)$ time, this backtracking search serves as a clear template for generating and traversing digit permutations.
- **Space Complexity**: $O(n)$ auxiliary space. The recursion call stack depth goes up to a maximum of $n$. The visited tracker array uses $O(1)$ constant space (size 10).