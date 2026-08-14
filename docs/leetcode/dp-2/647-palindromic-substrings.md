# 647. Palindromic Substrings

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/palindromic-substrings/description/)

Given a string `s`, return the number of palindromic substrings in it. A substring is a contiguous sequence of characters within the string.

---

## Solution 1: Expand Around Center (Two Pointers)

A palindrome reads the same forwards and backwards, which means it expands symmetrically from its center. We can find all palindromes by testing all possible center positions.

### Thought Process

1.  **Centers of Palindromes**:
    *   For a string of length $n$, there are $2n - 1$ possible centers:
        *   $n$ centers of odd length: centered at single character $(i, i)$.
        *   $n - 1$ centers of even length: centered between two adjacent characters $(i, i+1)$.
2.  **Expanding Outward**:
    *   For each center $(l, r)$, expand outward while $l \ge 0$, $r < n$, and $s[l] == s[r]$.
    *   Each successful character match confirms another valid palindromic substring, so we increment `res++`, decrement $l$, and increment $r$.
3.  **Result**:
    *   Return the total accumulated count `res`.

### Go Code

``` go
func countSubstrings(s string) int {
    n := len(s)
    res := 0

    for i := range s {
        // Odd-length palindromes
        l, r := i, i
        for l >= 0 && r < n && s[l] == s[r] {
            res++
            l--
            r++
        }
        // Even-length palindromes
        l, r = i, i+1
        for l >= 0 && r < n && s[l] == s[r] {
            res++
            l--
            r++
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n^2)$
    - There are $2n - 1$ centers. Expanding from each center takes $O(n)$ time in the worst case (e.g., `"aaaaa"`).
- **Space Complexity**: $O(1)$
    - We only use pointers and an accumulator variable, requiring $O(1)$ auxiliary space.

---

## Solution 2: Bottom-Up Dynamic Programming (2D DP)

We can also determine whether each substring `s[i...j]` is a palindrome using dynamic programming by reusing results from smaller inner substrings.

### Thought Process

1.  **State Definition**:
    *   Define `dp[i][j]` as a boolean indicating whether the substring `s[i...j]` is a palindrome.
2.  **State Transitions**:
    *   A substring `s[i...j]` is a palindrome if:
        1.  `s[i] == s[j]` (the outer boundary characters match), **AND**
        2.  Either the substring length is $\le 2$ ($j - i + 1 \le 2$), or the inner substring is a palindrome ($dp[i+1][j-1] == true$).
3.  **Evaluation Order**:
    *   Since `dp[i][j]` depends on `dp[i+1][j-1]` (the row below), we must iterate $i$ in reverse from $n-1$ down to $0$, and $j$ forward from $i$ up to $n-1$.
4.  **Result**:
    *   Increment `res` each time a cell `dp[i][j]` is marked `true`.

### Go Code

``` go
func countSubstrings(s string) int {
    n := len(s)
    dp := make([][]bool, n)
    for i := range dp {
        dp[i] = make([]bool, n)
    }
    res := 0
    for i := n-1; i >= 0; i-- {
        for j := i; j < n; j++ {
            if s[i] == s[j] && (j-i+1 <= 2 || dp[i+1][j-1]) {
                dp[i][j] = true
                res++
            }
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n^2)$
    - We fill an $n \times n$ table with two nested loops, taking $O(1)$ time per cell.
- **Space Complexity**: $O(n^2)$
    - The 2D boolean array requires $O(n^2)$ auxiliary space.