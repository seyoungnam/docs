# 93. Restore IP Addresses

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/restore-ip-addresses/description/)

## Solution: Backtracking (DFS Closure with Constraints Pruning)

A valid IPv4 address consists of exactly 4 numeric segments, each between $0$ and $255$ with no leading zeros. We can generate all valid combinations using recursive depth-first backtracking while enforcing these structural constraints at each step to prune invalid branches early.

### Thought Process

1.  **Initial Boundary Checks**:
    *   Since a valid IP address contains 4 segments, and each segment contains between $1$ and $3$ digits, the length of the input string $s$ must be between $4$ and $12$. If this constraint is violated, return an empty list immediately.
2.  **Constraints-Based Pruning**:
    *   **Length Constraint**: At each state, calculate the number of remaining segments needed (`remSlots = 4 - len(segments)`) and the number of unused characters (`remChars = n - start`). If `remChars` cannot fit into the remaining slots (i.e., `remChars < remSlots` or `remChars > remSlots * 3`), prune the branch.
    *   **Leading Zeros**: A segment cannot have leading zeros (e.g., `"01"` is invalid, but `"0"` is valid). If the segment length is greater than 1 and the first character is `'0'`, stop extending the segment (`break` the loop).
    *   **Range Limit**: The numeric value of the segment must be $\le 255$. Build the numeric value incrementally and `break` the loop immediately if it exceeds $255$.
3.  **Recursive Decisions**:
    *   Iterate the end pointer `end` of the current segment from `start` up to `start + 2` (length at most 3).
    *   If the segment is valid, append it to `segments` and recursively explore the remaining string by calling `dfs(end + 1, segments)`.
    *   Backtrack by slicing off the last segment before trying other configurations.
4.  **Base Case**:
    *   If `len(segments) == 4`:
        *   If the entire string has been consumed (`start == n`), we have successfully restored a valid IP address. Join the segments with `"."` using `strings.Join` and append it to our results slice `res`.
        *   Return.

### Go Code

``` go
import "strings"

func restoreIpAddresses(s string) []string {
    res := make([]string, 0)
    n := len(s)

    if n < 4 || n > 12 {
        return res
    }

    var dfs func(start int, segments []string)
    dfs = func(start int, segments []string) {
        if len(segments) == 4 {
            if start == n {
                res = append(res, strings.Join(segments, "."))
            }
            return
        }

        remSlots := 4 - len(segments)
        remChars := n - start
        if remChars < remSlots || remChars > remSlots*3 {
            return
        }
        
        val := 0
        for end := start; end < start+3 && end < n; end++ {
            if end > start && s[start] == '0' {
                break
            }
            val = val*10 + int(s[end]-'0')
            if val > 255 {
                break
            }
            segments = append(segments, s[start:end+1])
            dfs(end+1, segments)
            segments = segments[:len(segments)-1]
        }
    }
    dfs(0, make([]string, 0, 4))
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(1)$
    - The height of the recursion tree is at most $4$. Each node has at most $3$ branches. The total number of states explored is bounded by a constant ($3^4 = 81$ states in the absolute worst case). Thus, execution time is constant and extremely fast.
- **Space Complexity**: $O(1)$
    - Excluding the space needed to store the output results, the auxiliary space is constant. The recursion stack goes at most $4$ levels deep, and the temporary `segments` slice holds at most $4$ strings.