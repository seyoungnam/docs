# 567. Permutation in String

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/permutation-in-string/description/)

## Solution: Sliding Window with Frequency Arrays

A permutation of `s1` must be a substring in `s2` with the exact same length and character frequencies.

### Thought Process

1.  **Fixed Window**: Any permutation of `s1` in `s2` must have length `len(s1)`. Use a sliding window of this fixed size.
2.  **Frequency Arrays**: Use two arrays of size 26 to store character counts for `s1` and the current window in `s2`.
3.  **Sliding**:
    - Initialize the window with the first `len(s1)` characters.
    - Slide the window one character at a time.
    - At each step, update the window's frequency array by adding the incoming character and removing the outgoing one.
4.  **Comparison**: If the frequency arrays match (`freqS1 == freqS2`), a permutation exists.

### Go Code

``` go
func checkInclusion(s1 string, s2 string) bool {
    if len(s1) > len(s2) {
        return false
    } 
    var freqS1, freqS2 [26]int
    n, m := len(s1), len(s2)
    for i := 0; i < n; i++ {
        freqS1[s1[i]-'a']++
        freqS2[s2[i]-'a']++
    }
    if freqS1 == freqS2 {
        return true
    }
    for r := n; r < m; r++ {
        freqS2[s2[r-n]-'a']--
        freqS2[s2[r]-'a']++
        if freqS1 == freqS2 {
            return true
        }
    }
    return false
}
```

### Code Efficiency

- **Time Complexity**: $O(m)$
    - We iterate through `s2` once. The array comparison `freqS1 == freqS2` takes $O(26)$ time, which is constant.
- **Space Complexity**: $O(1)$
    - We use two fixed-size arrays of length 26, regardless of the input size.


---

## Optimized Solution: Sliding window + delta comparison

``` go
func checkInclusion(s1 string, s2 string) bool {
    if len(s1) > len(s2) {
        return false
    } 
    var freqS1, freqS2 [26]int
    n, m := len(s1), len(s2)
    for i := 0; i < n; i++ {
        freqS1[s1[i]-'a']++
        freqS2[s2[i]-'a']++
    }
    match := 0
    for i := 0; i < 26; i++ {
        if freqS1[i] == freqS2[i] {
            match++
        }
    }
    l := 0
    for r := n; r < m; r++ {
        if match == 26 {
            return true
        }

        idxR := s2[r]-'a'
        freqS2[idxR]++
        if freqS1[idxR] == freqS2[idxR] {
            match++
        } else if freqS1[idxR]+1 == freqS2[idxR] {
            match--
        }

        idxL := s2[l]-'a'
        freqS2[idxL]--
        if freqS1[idxL] == freqS2[idxL] {
            match++
        } else if freqS1[idxL]-1 == freqS2[idxL] {
            match--
        }
        l++
    }
    return match == 26
}
```