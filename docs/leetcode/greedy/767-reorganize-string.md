# 767. Reorganize String

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/reorganize-string/description/)

## Solution: Greedy (Even/Odd Index Placement)

To reorganize a string so that no two adjacent characters are identical, we must prioritize placing the most frequent character first. We can distribute it across even indices (0, 2, 4, etc.). If the most frequent character occupies more than half of the slots (specifically, more than `(len(s) + 1) / 2`), it is mathematically impossible to reorganize the string, and we must return an empty string.

### Thought Process

1.  **Frequency Counting**:
    - Count the frequency of each lowercase English letter in the string `s` using an array of size 26.
    - Identify the character with the highest frequency (`maxCount`) and its index (`maxCharIdx`).
2.  **Impossibility Check**:
    - If `maxCount > (len(s) + 1) / 2`, it's impossible to avoid adjacent duplicates. Return `""` immediately.
3.  **Filling Even Indices**:
    - Initialize a byte slice `res` of the same length as `s`.
    - Place the most frequent character at index `0` and increment the index by `2` each time. This fills the even positions (`0`, `2`, `4`, ...).
4.  **Filling Remaining Indices**:
    - Iterate through all other characters and place them in the remaining slots.
    - Continue incrementing the index by `2` for each placement.
    - When the index meets or exceeds the length of the string, wrap around by setting the index to `1` (which begins filling the odd positions: `1`, `3`, `5`, ...).
5.  **Output**:
    - Convert the byte slice to a string and return.

### Go Code

``` go
func reorganizeString(s string) string {
    counts := make([]int, 26)
    maxCount := 0
    maxCharIdx := -1
    for i := 0; i < len(s); i ++ {
        idx := int(s[i]-'a')
        counts[idx]++
        if counts[idx] > maxCount {
            maxCount = counts[idx]
            maxCharIdx = idx
        }
    }

    if maxCount > (len(s)+1)/2 {
        return ""
    }

    res := make([]byte, len(s))
    idx := 0

    for counts[maxCharIdx] > 0 {
        res[idx] = byte('a' + maxCharIdx)
        idx += 2
        counts[maxCharIdx]--
    }

    for cIdx, count := range counts {
        for count > 0 {
            if idx >= len(s) {
                idx = 1
            }
            res[idx] = byte('a' + cIdx)
            idx += 2
            count--
        }
    }
    return string(res)
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - Counting character frequencies takes $O(n)$ time. Placing the characters into the result array of size $n$ also takes $O(n)$ time. Iterating over the constant size (26) frequency array is $O(1)$.
- **Space Complexity**: $O(n)$
    - We use $O(1)$ auxiliary space for the frequency counter of size 26, and $O(n)$ space for the output byte slice `res`.
