# 131. Palindrome Partitioning

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/palindrome-partitioning/description/)

## Solution : Backtracking

### Thought Process

1.  **Goal**: Partition the string `s` such that every substring in the partition is a palindrome.
2.  **Backtracking Strategy**: Use a Depth-First Search (DFS) approach to explore all possible partition points.
    -   **Base Case**: When the start index `i` reaches the end of the string, it means we've successfully partitioned the entire string. Add a copy of the current partition `curr` to the results.
    -   **Recursive Step**: Iterate through the possible end positions `j` (from `i+1` to `len(s)`).
        -   Check if the substring `s[i:j]` is a palindrome.
        -   If it is, "choose" it by adding it to `curr` and recursively call DFS starting from index `j`.
        -   After the recursive call, "un-choose" the last added substring (backtrack) to explore other possibilities.
3.  **Palindrome Check**: A helper function `isPalindrome` uses two pointers to verify if a substring is the same forwards and backwards.

### Go Code

``` go
func partition(s string) [][]string {
    res := [][]string{}
    dfs(s, 0, []string{}, &res)
    return res
}

func dfs(s string, i int, curr []string, res *[][]string) {
    if i >= len(s) {
        *res = append(*res, append([]string{}, curr...))
        return
    }
    for j := i+1; j <= len(s); j++ {
        if isPalindrome(s, i, j) {
            curr = append(curr, s[i:j])
            dfs(s, j, curr, res)
            curr = curr[:len(curr)-1]
        }
    }
    return
}

func isPalindrome(s string, i int, j int) bool {
    for l, r := i, j-1; l < r; l, r = l+1, r-1 {
        if s[l] != s[r] {
            return false
        }
    }
    return true
}
```

### Code Efficiency

- **Time Complexity**: $O(N \cdot 2^N)$
    - There are $2^{N-1}$ possible ways to partition a string of length $N$.
    - For each partition, we perform palindrome checks and string slicing, which can take $O(N)$.
- **Space Complexity**: $O(N)$
    - The recursion stack depth is at most $N$.
    - This excludes the space required to store the output results.
