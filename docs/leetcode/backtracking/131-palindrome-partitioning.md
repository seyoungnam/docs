# 131. Palindrome Partitioning

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/palindrome-partitioning/description/)

## Solution: Backtracking (DFS Closure)

To find all possible palindrome partitioning schemes for a string `s`, we can explore all partition configurations using recursive depth-first backtracking. We only proceed down a search path if the current prefix substring is a valid palindrome.

### Thought Process

- **Partitioning Decisions**: 
    *   Starting at index `start`, we try placing a partition divider at every index `end` from `start + 1` up to $n$ (the length of the string).
- **Recursive Branching**:
    *   Iterate `end` from `start + 1` to $n$:
        *   Extract the candidate substring `s[start:end]` and check if it is a palindrome using the helper function `isPalindrome(s, start, end)`.
        *   If it is a palindrome:
            1. Append `s[start:end]` to the path slice `curr`.
            2. Recursively explore the remaining suffix of the string starting at index `end` by calling `dfs(end, curr)`.
            3. Backtrack: Remove the last added substring from `curr` (`curr = curr[:len(curr)-1]`) to restore state before trying other divider positions.
- **Base Case**:
    *   When `start == n`, the entire string has been successfully partitioned into palindrome substrings. We make a deep copy of `curr` and append it to our results slice `res`.

### Go Code

``` go
func partition(s string) [][]string {
    res := make([][]string, 0)
    n := len(s)

    var dfs func(int, []string)
    dfs = func(start int, curr []string) {
        if start == n {
            copied := make([]string, len(curr))
            copy(copied, curr)
            res = append(res, copied)
            return
        }

        for end := start+1; end <= n; end++ {
            if isPalindrome(s, start, end) {
                curr = append(curr, s[start:end])
                dfs(end, curr)
                curr = curr[:len(curr)-1]
            }
        }
    }

    dfs(0, []string{})
    return res
}

func isPalindrome(s string, start int, end int) bool {
    for l, r := start, end-1; l < r; l, r = l+1, r-1 {
        if s[l] != s[r] {
            return false
        }
    } 
    return true
}
```

### Code Efficiency

- **Time Complexity**: $O(N \cdot 2^N)$
    - In the worst case (e.g., a string consisting of all identical characters like `"aaaa"`), there are $2^{N-1}$ possible ways to partition the string. For each partitioning configuration, validating palindromes and extracting substrings takes up to $O(N)$ time.
- **Space Complexity**:
    - **With Output**: $O(N \cdot 2^N)$ to store all valid partitioning schemes.
    - **Auxiliary Space**: $O(N)$ for the recursion stack depth (at most $N$) and the temporary `curr` slice of size at most $N$.
