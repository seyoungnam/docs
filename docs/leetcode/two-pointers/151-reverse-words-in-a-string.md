# 151. Reverse Words in a String

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/reverse-words-in-a-string/description/)

## Solution: Word Extraction with Two Pointers & Reversal

To reverse the words in a string while removing leading, trailing, and multiple spaces between words, we can extract each word numerically using two pointers, reverse the sequence of extracted words, and join them with a single space.

### Thought Process

1.  **Extract Words (Ignoring Spaces)**:
    *   Initialize pointer `i = 0`.
    *   Iterate through the string:
        *   If `s[i]` is a space, increment `i` to skip it.
        *   If `s[i]` is not a space, we have found the start of a word. Set `j = i` and advance `j` until we hit a space or the end of the string.
        *   Append the substring slice `s[i:j]` to our list of words: `words`.
        *   Move `i` to `j` to search for the next word.
2.  **Reverse the Word List**:
    *   Use two pointers, `l = 0` and `r = len(words) - 1`.
    *   Swap `words[l]` and `words[r]`, increment `l`, and decrement `r` until they meet.
3.  **Concatenate with Space**:
    *   Use Go's `strings.Join(words, " ")` to build the final space-separated string.

### Go Code

``` go
import "strings"

func reverseWords(s string) string {
    n := len(s)
    words := make([]string, 0)
    for i := 0; i < n; {
        if s[i] == ' ' {
            i++
            continue
        }
        j := i
        for j < n && s[j] != ' ' {
            j++
        }
        words = append(words, s[i:j])
        i = j
    }
    for l, r := 0, len(words)-1; l < r; l, r = l+1, r-1 {
        words[l], words[r] = words[r], words[l]
    }
    return strings.Join(words, " ")
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We scan the string of length $n$ exactly once to extract words. Reversing the word list takes $O(W)$ time where $W$ is the number of words ($W \le n / 2$). Joining the words takes $O(n)$ time.
- **Space Complexity**: $O(n)$
    - We allocate the `words` string slice to store the parsed words, requiring space proportional to the input string length.