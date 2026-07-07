# 140. Word Break II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/word-break-ii/description/)

## Solution 1: Backtracking (DFS Closure)

We can solve this problem using recursive depth-first backtracking. We traverse the string character-by-character, deciding at each step whether to extend the current word or (if it exists in our dictionary) split and record it as a completed word.

### Thought Process

1.  **Key Idea**:
    *   Iterate through the string, building a candidate word character-by-character.
    *   We track `maxWordLen` (the maximum length of any word in `wordDict`). If our building word exceeds `maxWordLen`, we prune the branch immediately because it can never form a valid dictionary word.
2.  **Branching Decisions**:
    *   Add `s[i]` to `word`: `word += string(s[i])`.
    *   **Branch 1 (Word Splitting)**: If `word` exists in our dictionary, we can choose to record it:
        1. Append `word` to the path slice `words`.
        2. Recurse to process the next character, resetting the current word: `dfs(i+1, "", words)`.
        3. Backtrack: Remove `word` from the `words` path.
    *   **Branch 2 (Word Extension)**: We can always choose to continue extending the word:
        1. Recurse: `dfs(i+1, word, words)`.
    *   **Backtrack**: Restore the building word by slicing off the last character before returning: `word = word[:len(word)-1]`.
3.  **Base Case**:
    *   When the index `i` reaches `len(s)`:
        *   If the current building `word` is empty (meaning the entire string was successfully partitioned with no residue), we join our collected `words` with a space using `strings.Join(words, " ")` and append it to our results slice `res`.
        *   Return.

### Go Code

``` go
import "strings"

func wordBreak(s string, wordDict []string) []string {
    maxWordLen := 0
    dicts := make(map[string]bool)
    for _, word := range wordDict {
        maxWordLen = max(maxWordLen, len(word))
        dicts[word] = true
    }

    res := make([]string, 0)
    var dfs func(i int, word string, words []string) 
    dfs = func(i int, word string, words []string) {
        if i == len(s) {
            if len(word) == 0 {
                res = append(res, strings.Join(words, " "))
            }
            return
        }
        word += string(s[i])
        if len(word) > maxWordLen {
            return
        }
        if dicts[word] {
            words = append(words, word)
            dfs(i+1, "", words)
            words = words[:len(words)-1]
        }
        dfs(i+1, word, words)
        word = word[:len(word)-1]
    }
    dfs(0, "", []string{})
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(2^N)$ in the worst case (where $N$ is the length of string $s$). We branch up to 2 ways at each character. Pruning against `maxWordLen` significantly limits the actual search depth.
- **Space Complexity**: $O(N^2)$
    - The recursion stack can go up to $N$ levels deep. The temporary path slice `words` can store up to $N$ strings of length at most $N$.

---

## Solution 2: Top-Down Dynamic Programming (Memoized DFS)

Instead of searching forward character-by-character, we can solve top-down by calculating all valid sentences that can be formed from the suffix starting at index `start`, caching the results.

### Thought Process

1.  **State Definition & Memoization**:
    *   Let `dfs(start)` return a list of all valid sentences that can be formed from the suffix `s[start:]`.
    *   We use a map `memo := make(map[int][]string)` to cache these suffix sentences. If `start` is already computed, we return `memo[start]` in $O(1)$ time.
2.  **DP Transitions**:
    *   For the suffix starting at `start`, try partitioning a prefix of length up to `maxWordLen` (i.e., `end` from `start+1` to `len(s)` such that `end - start <= maxWordLen`).
    *   Extract the prefix `prefix := s[start:end]`.
    *   If `prefix` is in the dictionary, recursively find all valid sentences for the remaining suffix `dfs(end)`.
    *   Combine `prefix` with each returned suffix sentence, adding a space in between (unless the suffix is empty).
3.  **Base Case**:
    *   When `start == len(s)`, we have reached the end of the string. Return a slice containing a single empty string `[]string{""}` representing a valid empty suffix.

### Go Code

``` go
func wordBreak(s string, wordDict []string) []string {
    maxWordLen := 0
    dicts := make(map[string]bool)
    for _, word := range wordDict {
        maxWordLen = max(maxWordLen, len(word))
        dicts[word] = true
    }

    memo := make(map[int][]string)

    var dfs func(start int) []string 
    dfs = func(start int) []string {
        if val, ok := memo[start]; ok {
            return val
        }
        if start == len(s) {
            return []string{""}
        }
        var currRes []string
        for end := start+1; end <= len(s) && end-start <= maxWordLen; end++ {
            prefix := s[start:end]
            if dicts[prefix] {
                suffixSentences := dfs(end)

                for _, suffix := range suffixSentences {
                    if suffix == "" {
                        currRes = append(currRes, prefix)
                    } else {
                        currRes = append(currRes, prefix + " " + suffix)
                    }
                }
            }
        }
        memo[start] = currRes
        return currRes
    }
    return dfs(0)
}
```

### Code Efficiency

- **Time Complexity**: $O(N \cdot 2^N)$ in the worst case (where $N$ is the length of string $s$). There are at most $2^{N-1}$ partitions. Using a memoization cache avoids redundant evaluations of identical suffix indexes.
- **Space Complexity**: $O(N \cdot 2^N)$
    - The space required to store the memoization table mapping suffix indexes to their respective list of sentences.