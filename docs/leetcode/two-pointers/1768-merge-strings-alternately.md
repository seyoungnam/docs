# 1768. Merge Strings Alternately

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/merge-strings-alternately/description/)

## Solution: Two Pointers

We can merge the two strings by iterating through both of them in parallel using two separate pointers. We alternate appending characters from `word1` and `word2` to our result builder, and once one string is exhausted, we append the remainder of the other.

### Thought Process

1.  **Pointers Setup**:
    *   Let `i` track the index in `word1` and `j` track the index in `word2`, both initialized to `0`.
    *   Use a byte slice `res` to build the resulting merged string.
2.  **Alternating Loop**:
    *   Iterate while both pointers are within bounds (`i < len(word1)` and `j < len(word2)`).
    *   In each step, append `word1[i]` and increment `i`, then append `word2[j]` and increment `j`.
3.  **Append Remaining**:
    *   If `word1` is longer, a separate loop appends the remaining characters from `i` to the end.
    *   If `word2` is longer, a separate loop appends the remaining characters from `j` to the end.
4.  **Result**:
    *   Convert the byte slice `res` to a string and return it.

> [!TIP]
> To optimize execution speed and reduce memory allocations, we could pre-allocate the slice capacity since we know the final length will be exactly $m + n$: `res := make([]byte, 0, m+n)`.

### Go Code

``` go
func mergeAlternately(word1 string, word2 string) string {
    m, n := len(word1), len(word2)
    i, j := 0, 0
    res := make([]byte, 0)
    for i < m && j < n {
        res = append(res, word1[i])
        i++
        res = append(res, word2[j])
        j++
    }
    for i < m {
        res = append(res, word1[i])
        i++
    }
    for j < n {
        res = append(res, word2[j])
        j++
    }
    return string(res)
}
```

### Code Efficiency

- **Time Complexity**: $O(m + n)$
    - We iterate through every character of both strings exactly once, where $m$ and $n$ are the lengths of `word1` and `word2` respectively.
- **Space Complexity**: $O(m + n)$
    - The output byte slice/string requires $O(m + n)$ memory to store the merged result.