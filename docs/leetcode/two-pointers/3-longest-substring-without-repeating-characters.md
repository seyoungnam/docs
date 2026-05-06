# 3. Longest Substring Without Repeating Characters

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/longest-substring-without-repeating-characters/description/)

## Solution: Sliding Window

### Thought Process

1.  **Sliding Window Approach**: We use two pointers, `l` (left) and `r` (right), to define a window that contains only unique characters.
2.  **Tracking Character Frequency**: A hash map (`counter`) stores the count of each character within the current window.
3.  **Expanding the Window**: As the right pointer `r` moves forward, we add the character `s[r]` to the map.
4.  **Handling Duplicates**: If adding `s[r]` causes a duplicate (i.e., `counter[s[r]] > 1`), we must shrink the window from the left by moving the `l` pointer and decrementing the counts of characters being removed until the duplicate is gone.
5.  **Result Update**: At each step where the window is valid (no duplicates), we calculate the window size (`r - l + 1`) and update the `maxLen` if the current window is longer than the previous maximum.

### Go Code

``` go
func lengthOfLongestSubstring(s string) int {
    counter := make(map[byte]int)
    maxLen := 0
    l := 0
    n := len(s)
    for r := 0; r < n; r++ {
        counter[s[r]]++
        for counter[s[r]] > 1 {
            counter[s[l]]--
            l++
        }
        maxLen = max(maxLen, r-l+1)
    }
    return maxLen
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - Each character is visited at most twice: once by the right pointer and once by the left pointer.
- **Space Complexity**: $O(\min(m, n))$
    - The size of the hash map is bounded by the size of the character set ($m$) and the length of the string ($n$).

---

## Optimized Solution: Sliding Window with Index Mapping

### Thought Process

1.  **Direct Index Mapping**: Instead of incrementing the left pointer `l` one by one and clearing counts, we store the **next potential starting position** for each character in an array or map (`lastSeen`).
2.  **Jumping the Left Pointer**: When we encounter a character `s[r]` that we have seen before, we can instantly jump `l` to the position right after the last occurrence of `s[r]`.
3.  **Ensuring Monotonicity**: We must ensure `l` never moves backward. This happens if the duplicate character is found *outside* the current window boundaries (to the left of `l`). We use `l = max(l, lastSeen[s[r]])` to handle this.
4.  **Optimized Storage**: For ASCII strings, a fixed-size array of 128 (or 256 for extended ASCII) integers is more efficient than a hash map.
5.  **Efficiency Gain**: This eliminates the inner `for` loop, making the algorithm strictly single-pass.

### Go Code

``` go
func lengthOfLongestSubstring(s string) int {
    var lastSeen [128]int // ASCII has 128 characters.
    maxLen := 0
    l := 0
    for r := 0; r < len(s); r++ {
        l = max(l, lastSeen[s[r]])
        maxLen = max(maxLen, r - l + 1)
        lastSeen[s[r]] = r + 1
    }
    return maxLen
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - The algorithm performs a single pass over the string.
- **Space Complexity**: $O(m)$
    - We use a fixed-size array of length $m$ (128 for ASCII) to store character indices. Since the array size is constant, this is often considered $O(1)$ auxiliary space.
