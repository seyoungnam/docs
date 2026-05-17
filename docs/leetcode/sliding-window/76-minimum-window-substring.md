# 76. Minimum Window Substring

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/minimum-window-substring/description/)

## Solution: Sliding Window

To find the minimum window containing all characters of `t`, we use a sliding window with a frequency map to track the "satisfied" state of required characters.

### Thought Process

1.  **Requirement Mapping**: Count the frequency of each character in `t` and track the number of unique characters (`charCnt`) that must be satisfied.
2.  **Expand Window**: Move the right pointer `r`. Decrement the frequency of `s[r]` in the map. If a character's count drops to 0, it is satisfied; decrement `charCnt`.
3.  **Shrink Window**: When `charCnt == 0` (all requirements met), a valid window is found. Update the minimum size and start position.
4.  **Optimize**: Advance the left pointer `l` to minimize the window. If removing `s[l]` causes its requirement to become positive again, increment `charCnt` and continue expanding.

### Go Code

``` go
func minWindow(s string, t string) string {
    if len(s) == 0 || len(t) == 0 || len(s) < len(t) {
        return ""
    }
    m, n := len(s), len(t)
    freq := map[byte]int{}
    for i := 0; i < n; i++ {
        freq[t[i]]++
    }
    charCnt := len(freq)
    l := 0
    minLeft, minSize := 0, math.MaxInt64
    for r := 0; r < m; r++ {
        if cnt, exists := freq[s[r]]; exists {
            freq[s[r]]--
            if cnt == 1 {
                charCnt--
            }
        }
        for charCnt == 0 {
            if r-l+1 < minSize {
                minLeft = l
                minSize = r-l+1
            }
            if cnt, exists := freq[s[l]]; exists {
                freq[s[l]]++
                if cnt == 0 {
                    charCnt++
                }
            }
            l++
        }
    }
    if minSize == math.MaxInt64 {
        return ""
    }
    return s[minLeft:minLeft+minSize]
}
```

### Code Efficiency

- **Time Complexity**: $O(m + n)$
    - We iterate through `t` once ($O(n)$) and traverse `s` with two pointers, each visiting each character at most once ($O(m)$).
- **Space Complexity**: $O(K)$
    - The frequency map stores at most $K$ unique characters from `t` (e.g., 52 for English letters). Since $K$ is bounded, this is often considered $O(1)$ auxiliary space.

