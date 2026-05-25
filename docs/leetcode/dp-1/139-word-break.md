# 139. Word Break

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/word-break/description/)

## Solution: Bottom-Up Dynamic Programming (Dictionary Scan)

Your solution is correct and highly optimal. By using a `dp` array of size `n` (where `dp[i]` tracks whether `s[0:i+1]` is segmentable), you scan through all words in the dictionary to see if any word can complete the current suffix ending at `i`.

### Thought Process

1.  **DP Array**: Create a `dp` array of size `n` where `dp[i]` represents if the prefix of length `i+1` can be segmented.
2.  **Iterate and Match**: For each index `i` in the string `s`:
    *   Iterate through each `word` in `wordDict` of length `m`.
    *   Verify if the word fits within the current prefix boundary: `i-m >= -1`.
    *   Check if the substring matches the word: `s[i-m+1:i+1] == word`.
    *   Check if the remaining prefix before this word was segmentable: `i-m == -1` (covers entire prefix) or `dp[i-m] == true`.
3.  **Result**: Return `dp[n-1]`.

### Go Code

``` go
func wordBreak(s string, wordDict []string) bool {
    n := len(s)
    dp := make([]bool, n)
    
    for i := 0; i < n; i++ {
        for _, word := range wordDict {
            m := len(word)
            if i-m < -1 {
                continue
            }            
            
            // Current substring matches AND (it is the start of string OR the previous prefix is segmentable)
            if s[i-m+1:i+1] == word && (i-m == -1 || dp[i-m]) {
                dp[i] = true
                break // Found a valid segmentation for s[0:i+1], can move to next i
            }
        }
    }
    return dp[n-1]
}
```

### Code Efficiency

- **Time Complexity**: $O(n \cdot d \cdot m)$
    - Where $n$ is the length of `s`, $d$ is the number of words in `wordDict`, and $m$ is the maximum length of a word in `wordDict` (due to substring comparison).
- **Space Complexity**: $O(n)$
    - We use a 1D `dp` array of size $n$.

---

## Alternative Solution: Top-Down DP (DFS + Memoization)

Instead of scanning all dictionary words, we can think recursively: "Can the suffix starting at index `start` be segmented?" By using a Hash Set for dictionary lookups, we can try all possible prefix splits and cache the results to prevent duplicate work.

### Thought Process

1.  **Dictionary Lookup**: Convert `wordDict` into a hash set for $O(1)$ lookups.
2.  **Recursive DFS**: Let `dfs(start)` return whether `s[start:n]` can be segmented:
    *   If `start == n`, return `true` (reached the end successfully).
    *   If `memo[start]` is already calculated, return it.
    *   Try all split indexes `end` from `start+1` to `n`. If the prefix `s[start:end]` is a valid word, recursively call `dfs(end)`.
3.  **Memoization**: Store the results in a `memo` array of size `n` (`1` for true, `-1` for false) to avoid redundant recursive paths.

### Go Code

``` go
func wordBreak(s string, wordDict []string) bool {
    n := len(s)
    memo := make([]int, n) // 0: unvisited, 1: true, -1: false
    
    // Hash set for O(1) word lookups
    wordSet := make(map[string]bool)
    for _, w := range wordDict {
        wordSet[w] = true
    }
    
    var dfs func(start int) bool
    dfs = func(start int) bool {
        if start == n {
            return true
        }
        if memo[start] != 0 {
            return memo[start] == 1
        }
        
        // Try all possible split positions
        for end := start + 1; end <= n; end++ {
            if wordSet[s[start:end]] && dfs(end) {
                memo[start] = 1
                return true
            }
        }
        
        memo[start] = -1
        return false
    }
    
    return dfs(0)
}
```

### Code Efficiency

- **Time Complexity**: $O(n^2 \cdot m)$
    - There are $n$ recursive states. For each state, the loop runs up to $n$ times (or up to $m$ times, where $m$ is the maximum word length), and slicing a string takes $O(m)$ time.
- **Space Complexity**: $O(n)$
    - The hash set takes $O(d \cdot m)$ space, and the `memo` array and recursion stack take $O(n)$ space.
