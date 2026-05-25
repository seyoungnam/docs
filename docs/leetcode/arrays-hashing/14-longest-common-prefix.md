# 14. Longest Common Prefix

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/longest-common-prefix/description/)

## Solution: Vertical Scanning (Optimal)

### Thought Process

1.  **Reference String**: Use the first string `strs[0]` as our prefix reference.
2.  **Vertical Character Scan**: Loop through each character index `i` of the reference string:
    *   Let `c` be the character at `strs[0][i]`.
    *   Compare `c` with the character at index `i` for all other strings.
    *   If index `i` is out of bounds for any string (`i == len(word)`), or if the characters do not match (`word[i] != c`), return the prefix: `strs[0][:i]`.
3.  **Complete Match**: If we complete the loop, the entire first word is the common prefix.

### Go Code

``` go
func longestCommonPrefix(strs []string) string {
    if len(strs) == 0 {
        return ""
    }
    
    // Scan character by character vertically using the first string as reference
    for i := 0; i < len(strs[0]); i++ {
        c := strs[0][i]
        for j := 1; j < len(strs); j++ {
            // If we exceed any string's length or find a mismatch, return prefix
            if i == len(strs[j]) || strs[j][i] != c {
                return strs[0][:i]
            }
        }
    }
    
    return strs[0]
}
```

### Code Efficiency

- **Time Complexity**: $O(S)$
    - Where $S$ is the sum of all characters in all strings. In the worst case, we do at most $n \cdot \min(L)$ character comparisons.
- **Space Complexity**: $O(1)$ auxiliary space
    - We only use index pointers and slice the original string, requiring no extra memory.

---

## Alternative Solution: The Sorting Hack (High-Yield Interview Trick)

You can showcase creative problem-solving by using a **sorting-based approach**. By sorting the array of strings lexicographically:

* The most different strings will be pushed to the absolute ends: `strs[0]` (lexicographically smallest) and `strs[len(strs)-1]` (lexicographically largest).
* The common prefix of the *entire* array must be the common prefix of just these **first and last** strings.

### Thought Process

1.  **Lexicographical Sort**: Sort the string slice `strs`.
2.  **Compare Extremes**: Compare the first string (`first`) and the last string (`last`) character by character.
3.  **Return Common**: Return the matched slice of the `first` string.

### Go Code

``` go
import "sort"

func longestCommonPrefix(strs []string) string {
    if len(strs) == 0 {
        return ""
    }
    if len(strs) == 1 {
        return strs[0]
    }
    
    // Sort strings lexicographically
    sort.Strings(strs)
    
    first := strs[0]
    last := strs[len(strs)-1]
    
    i := 0
    // We only need to compare the first and last strings
    for i < len(first) && i < len(last) && first[i] == last[i] {
        i++
    }
    
    return first[:i]
}
```

### Code Efficiency

- **Time Complexity**: $O(n \cdot L \log n)$
    - Where $n$ is the number of strings and $L$ is the maximum string length. Sorting $n$ strings takes $O(n \log n)$ string comparisons, and each comparison takes $O(L)$ time. While technically slower than vertical scanning for massive arrays, it is extremely elegant.
- **Space Complexity**: $O(1)$ or $O(\log n)$
    - Auxiliary space is determined by Go's sort implementation, which requires $O(\log n)$ stack space.
