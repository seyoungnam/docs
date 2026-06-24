# 242. Valid Anagram

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/valid-anagram/description/)

## Solution: Frequency Counter (Fixed-size Array)

An anagram is a word or phrase formed by rearranging the letters of a different word or phrase. This means two strings are anagrams if they have the exact same characters with the exact same frequencies.

### Thought Process

1.  **Length Check**: If the lengths of strings `s` and `t` are different, they cannot be anagrams. Return `false` immediately.
2.  **Frequency Counting**: We need to count the occurrences of each character in both strings. For lowercase English letters, fixed-size arrays of 26 integers (`[26]int`) are used: `sCounts` for string `s` and `tCounts` for string `t`.
3.  **One-Pass Counting**: Iterate through both strings simultaneously, incrementing the character counts in their respective frequency arrays.
4.  **Verification**: Return the direct comparison `sCounts == tCounts`. In Go, fixed-size arrays of the same type and size are comparable using the `==` operator, which performs a direct element-by-element equality check.

### Go Code

``` go
func isAnagram(s string, t string) bool {
    if len(s) != len(t) {
        return false
    }

    var sCounts, tCounts [26]int

    for i := 0; i < len(s); i++ {
        sCounts[s[i]-'a']++
        tCounts[t[i]-'a']++
    }
    return sCounts == tCounts
}
```


### Code Efficiency

- **Time Complexity**: $O(n)$
    - We iterate through the strings once ($O(n)$) and then perform a constant-time check on the 26-element array ($O(1)$). Total time is $O(n)$.
- **Space Complexity**: $O(1)$
    - We use a fixed-size array of size 26 regardless of the input string length $n$. This is considered constant auxiliary space.


