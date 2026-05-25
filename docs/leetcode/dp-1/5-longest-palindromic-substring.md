# 5. Longest Palindromic Substring

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/longest-palindromic-substring/description/)

## Solution: Expand Around Center

We can find the longest palindromic substring by treating each index (and pair of adjacent indices) in the string as the potential center of a palindrome, expanding outwards as long as the boundary characters match.

### Thought Process

1.  **Iterate Centers**: For each character `i` in the string:
    *   **Odd-Length Palindromes**: Treat `i` as the sole center node, expanding outwards using `l = i - 1` and `r = i + 1`.
    *   **Even-Length Palindromes**: Treat `i` and `i + 1` as the shared center nodes, expanding outwards using `l = i` and `r = i + 1`.
2.  **Outward Expansion**: While `l >= 0` and `r < len(s)` and `s[l] == s[r]`:
    *   The substring `s[l:r+1]` is a valid palindrome.
    *   If `len(s[l:r+1]) > len(res)`, update the longest palindrome found (`res`).
    *   Decrement `l` and increment `r` to expand further.

### Go Code

``` go
func longestPalindrome(s string) string {
    res := string(s[0])
    for i := range s {
        // odd
        l, r := i-1, i+1
        for l >= 0 && r < len(s) && s[l] == s[r] {
            if len(s[l:r+1]) > len(res) {
                res = s[l:r+1]
            }
            l--
            r++
        }
        // even
        l, r = i, i+1
        for l >= 0 && r < len(s) && s[l] == s[r] {
            if len(s[l:r+1]) > len(res) {
                res = s[l:r+1]
            }
            l--
            r++
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n^2)$
    - We loop through the string $n$ times. For each character, expanding outwards takes up to $O(n)$ time in the worst case (e.g., all identical characters).
- **Space Complexity**: $O(1)$ auxiliary space
    - We only use a few pointer variables, requiring $O(1)$ extra space.

---

## Alternative Solution: 2D Dynamic Programming

We can also solve this using Dynamic Programming. Let `dp[i][j]` be `true` if the substring `s[i:j+1]` is a palindrome.

### Thought Process

1.  **Subproblem Definition**: `dp[i][j]` is a palindrome if:
    *   The boundary characters match: `s[i] == s[j]`.
    *   AND the inner substring is also a palindrome: `dp[i+1][j-1]` is `true` (or the substring length is 3 or less: `j - i < 3`).
2.  **Iterative Fill**: We iterate the string backwards for `i` (from `n-1` to `0`) and forwards for `j` (from `i` to `n-1`) to ensure `dp[i+1][j-1]` is computed before we need it.
3.  **Track Maximum**: Keep track of the longest indices `[start, end]` where `dp[i][j]` is `true`, and slice the string at the end.

### Go Code

``` go
func longestPalindrome(s string) string {
    n := len(s)
    if n <= 1 {
        return s
    }
    
    dp := make([][]bool, n)
    for i := range dp {
        dp[i] = make([]bool, n)
    }
    
    start, maxLen := 0, 1
    
    for i := n - 1; i >= 0; i-- {
        for j := i; j < n; j++ {
            // Match boundaries and check inner substring
            if s[i] == s[j] && (j-i < 3 || dp[i+1][j-1]) {
                dp[i][j] = true
                
                // Track longest palindromic substring
                if j-i+1 > maxLen {
                    maxLen = j - i + 1
                    start = i
                }
            }
        }
    }
    
    return s[start : start+maxLen]
}
```

### Code Efficiency

- **Time Complexity**: $O(n^2)$
    - We use two nested loops to fill out the $n \times n$ table.
- **Space Complexity**: $O(n^2)$
    - We allocate an $n \times n$ boolean table to store the state.
