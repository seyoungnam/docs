# 424. Longest Repeating Character Replacement

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/longest-repeating-character-replacement/description/)

## Solution: Sliding Window

A window is valid if the number of characters that need to be replaced (window length minus the frequency of the most frequent character) is less than or equal to `k`.

### Thought Process

1.  **Sliding Window**: Expand the window by moving the right pointer `r` and track character frequencies in a map.
2.  **Track Max Frequency**: Maintain `maxFreq`, the count of the most frequent character in the current window.
3.  **Validate Window**: If `(window_length - maxFreq) > k`, the window is invalid. Shrink it by moving the left pointer `l` until it becomes valid again.
4.  **Result**: The maximum valid window size encountered is the answer.

### Go Code

``` go
func characterReplacement(s string, k int) int {
    freq := map[byte]int{}
    res, l, maxFreq := 0, 0, 0
    for r := range s {
        c := s[r]
        freq[c]++
        maxFreq = max(maxFreq, freq[c])

        // If characters to replace > k, shrink window
        for (r-l+1) - maxFreq > k {
            freq[s[l]]--
            l++
        }
        res = max(res, r-l+1)
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We perform a single pass over the string with the right pointer. The left pointer also moves at most $n$ times total.
- **Space Complexity**: $O(1)$
    - The frequency map stores at most 26 characters (assuming uppercase English letters), which is constant auxiliary space.

---

## Optimized Solution

``` go
func characterReplacement(s string, k int) int {
    var freq [26]int
    maxFreq := 0
    l := 0

    for r := 0; r < len(s); r++ {
        idx := s[r]-'A'
        freq[idx]++
        
        maxFreq = max(maxFreq, freq[idx])

        if (r-l+1) - maxFreq > k {
            freq[s[l]-'A']--
            l++
        }
    }

    return len(s) - l
}
```