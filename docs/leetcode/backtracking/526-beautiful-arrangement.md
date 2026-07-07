# 526. Beautiful Arrangement

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/beautiful-arrangement/description/)

## Solution: Backtracking (DFS Closure with Candidate Pruning)

A permutation of integers from $1$ to $n$ is defined as a **beautiful arrangement** if, for every position `i` (1-indexed), either the number at index `i` is divisible by `i`, or `i` is divisible by that number. We can count all valid beautiful arrangements using recursive depth-first backtracking, placing elements position-by-position and pruning invalid branches early.

### Thought Process

1.  **Beautiful Constraint Validation**:
    *   For each position `pos` from $1$ to $n$, we check if candidate number `num` is valid at that position:
        $$\text{num} \% \text{pos} == 0 \quad \text{or} \quad \text{pos} \% \text{num} == 0$$
    *   If this condition is not met, we skip `num` for `pos`, pruning the entire sub-tree early.
2.  **Visited Tracker**:
    *   We maintain a boolean slice `used` of size $n+1$ to keep track of which numbers have already been placed in the arrangement.
3.  **Recursive Decisions**:
    *   We fill the arrangement position by position starting from `pos = 1`.
    *   At position `pos`, iterate candidate `num` from $1$ to $n$:
        *   If `used[num]` is `true`, skip.
        *   If the beautiful arrangement constraint is satisfied:
            1. Mark `num` as used: `used[num] = true`.
            2. Recurse to fill the next position: `dfs(pos+1)`.
            3. Backtrack: Mark `num` as unused: `used[num] = false`.
4.  **Base Case**:
    *   When the position tracker `pos` exceeds $n$ (meaning `pos > n`), it indicates we have successfully placed valid numbers in all $n$ positions. Increment the global arrangement counter `count` and return.

### Go Code

``` go
func countArrangement(n int) int {
    count := 0
    used := make([]bool, n+1)
    
    var dfs func(pos int)
    dfs = func(pos int) {
        if pos > n {
            count++
            return
        }
        for num := 1; num <= n; num++ {
            if used[num] {
                continue
            }
            if num%pos == 0 || pos%num == 0 {
                used[num] = true
                dfs(pos+1)
                used[num] = false
            }
        }
    } 
    dfs(1)
    return count
}
```

### Code Efficiency

- **Time Complexity**: $O(k)$ where $k$ is the number of valid beautiful arrangements. 
    - Although the worst-case time complexity of generating permutations is $O(n!)$, the divisibility constraint prunes branches extremely early in the search tree. For the maximum constraints ($n \le 15$), the backtracking completes in a few milliseconds.
- **Space Complexity**: $O(n)$ auxiliary space. The recursion call stack depth goes up to a maximum of $n$, and the visited tracker slice `used` uses $O(n)$ space.