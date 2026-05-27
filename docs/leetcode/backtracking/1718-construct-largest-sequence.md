# 1718. Construct the Lexicographically Largest Valid Sequence

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/construct-the-lexicographically-largest-valid-sequence/description/)

## Solution: Greedy Backtracking

Since $n \le 20$, we can solve this problem using **backtracking**. To guarantee that the first complete sequence we find is the **lexicographically largest** one, we can greedily try to place the largest available numbers (from $n$ down to $1$) at each empty index.

### Thought Process

1.  **Sequence Structure**: The resulting sequence has a fixed size of `2n - 1`.
2.  **Number Rules**:
    *   The number `1` is placed exactly once.
    *   For any number `num > 1`, it must be placed exactly twice: once at the current index `idx` and again at `idx + num`.
3.  **Greedy Backtracking DFS**:
    *   If `idx == size`, we have successfully filled the sequence; return `true` (success).
    *   If `res[idx] != 0` (already filled by a previous placement), skip and recursively call `dfs(idx + 1)`.
    *   Otherwise, try to place `num` from `n` down to `1` (which guarantees the lexicographically largest sequence):
        *   If `num == 1`: Place `1` at `res[idx]`, mark it visited, and recurse. If it fails, backtrack.
        *   If `num > 1`: Check if the second slot `idx + num` is within bounds and empty. If so, place `num` at both indices, mark it visited, and recurse. If it fails, backtrack.

### Go Code

``` go
func constructDistancedSequence(n int) []int {
    size := 2*n - 1
    res := make([]int, size)
    visited := make([]bool, n+1)
    
    var dfs func(idx int) bool
    dfs = func(idx int) bool {
        // Base case: successfully filled all slots
        if idx == size {
            return true
        }
        // If the current slot is already occupied, move to next slot
        if res[idx] != 0 {
            return dfs(idx + 1)
        }
        
        // Try placing numbers greedily from n down to 1
        for num := n; num >= 1; num-- {
            if visited[num] {
                continue
            }
            
            if num == 1 {
                res[idx] = 1
                visited[1] = true
                if dfs(idx + 1) {
                    return true
                }
                res[idx] = 0
                visited[1] = false
            } else {
                nextIdx := idx + num
                // Ensure second slot is in bounds and currently empty
                if nextIdx < size && res[nextIdx] == 0 {
                    res[idx] = num
                    res[nextIdx] = num
                    visited[num] = true
                    
                    if dfs(idx + 1) {
                        return true
                    }
                    
                    // Backtrack
                    res[idx] = 0
                    res[nextIdx] = 0
                    visited[num] = false
                }
            }
        }
        return false
    }
    
    dfs(0)
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n!)$
    - In the absolute worst case, backtracking takes exponential time. However, due to the tight distance constraints between duplicates (e.g., placing `20` immediately eliminates the slot at `idx + 20`), the search space is heavily pruned and the algorithm completes in a few milliseconds for $n \le 20$.
- **Space Complexity**: $O(n)$
    - The recursion call stack can reach a depth of at most `2n - 1`, and the visited array has size `n + 1`, requiring $O(n)$ auxiliary space.
