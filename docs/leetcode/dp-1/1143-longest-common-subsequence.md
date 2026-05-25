# 1143. Longest Common Subsequence

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/longest-common-subsequence/description/)

## Solution: 2D Dynamic Programming

We can solve this by building a 2D `dp` table where `dp[r][c]` represents the length of the longest common subsequence of `text1[0:r]` and `text2[0:c]`. Prepending helper characters simplifies boundary condition checks.

### Thought Process

1.  **Prepending Sentinel**: Prepend dummy characters (`"1"` and `"2"`) to the strings to map indices to 1-based indexing, avoiding manual out-of-bounds checks for row `r-1` and column `c-1`.
2.  **Initialize DP Table**: Create a 2D grid `dp` of size `ROWS x COLS` filled with `0`.
3.  **State Transition**:
    *   If `text1[r] == text2[c]`, the characters match: `dp[r][c] = dp[r-1][c-1] + 1` (extend the LCS length from the diagonal).
    *   If they do not match, take the maximum of excluding either character: `dp[r][c] = max(dp[r-1][c], dp[r][c-1])`.
4.  **Result**: Return the value at the bottom-right corner: `dp[ROWS-1][COLS-1]`.

### Go Code

``` go
func longestCommonSubsequence(text1 string, text2 string) int {
    text1 = "1" + text1
    text2 = "2" + text2
    ROWS, COLS := len(text1), len(text2)
    dp := make([][]int, ROWS)
    for r := range dp {
        dp[r] = make([]int, COLS)
    } 
    for r := 1; r < ROWS; r++ {
        for c := 1; c < COLS; c++ {
            if text1[r] == text2[c] {
                dp[r][c] = dp[r-1][c-1] + 1
            } else {
                dp[r][c] = max(dp[r-1][c], dp[r][c-1])
            }
        }
    }
    return dp[ROWS-1][COLS-1]
}
```

### Code Efficiency

- **Time Complexity**: $O(m \cdot n)$
    - Where $m$ and $n$ are the lengths of `text1` and `text2`.
- **Space Complexity**: $O(m \cdot n)$
    - We allocate a 2D slice of size $(m+1) \times (n+1)$.

---

## Optimized Solution: 1D Space-Optimized DP

Since `dp[r][c]` only depends on the previous row (`dp[r-1]`) and the current row's left element (`dp[r][c-1]`), we can optimize the space complexity to $O(\min(m, n))$ by storing only the previous and current rows' states.

### Thought Process

1.  **Reduce Dimensions**: Maintain a single 1D array `dp` representing the results of the previous row.
2.  **Track Diagonal**: Use a `prev` variable to store the diagonal value (`dp[r-1][c-1]`) before it gets overwritten.
3.  **Optimize Space**: Ensure the DP array is initialized to the length of the shorter string, reducing space usage further.

### Go Code

``` go
func longestCommonSubsequence(text1 string, text2 string) int {
    m, n := len(text1), len(text2)
    if m < n {
        text1, text2 = text2, text1
        m, n = n, m
    }
    
    dp := make([]int, n+1)
    
    for i := 0; i < m; i++ {
        prev := 0 // Tracks dp[i-1][j-1]
        for j := 0; j < n; j++ {
            temp := dp[j+1] // Save current value (which acts as dp[i-1][j] for next column)
            if text1[i] == text2[j] {
                dp[j+1] = prev + 1
            } else {
                dp[j+1] = max(dp[j+1], dp[j])
            }
            prev = temp
        }
    }
    return dp[n]
}
```

### Code Efficiency

- **Time Complexity**: $O(m \cdot n)$
    - We still run nested loops of sizes $m$ and $n$.
- **Space Complexity**: $O(\min(m, n))$
    - The DP array size is determined by the shorter string length, reducing space to $O(\min(m, n))$.
