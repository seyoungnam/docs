# 242. Valid Anagram

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/valid-anagram/description/)

## Solution: Frequency Counter (Fixed-size Array)

An anagram is a word or phrase formed by rearranging the letters of a different word or phrase. This means two strings are anagrams if they have the exact same characters with the exact same frequencies.

### Thought Process

1.  **Length Check**: If the lengths of strings `s` and `t` are different, they cannot be anagrams. Return `false` immediately.
2.  **Frequency Counting**: We need to count the occurrences of each character in both strings. 
    - For lowercase English letters, a fixed-size array of 26 integers (`[26]int`) is more performant than a hash map.
3.  **One-Pass Update**: Iterate through both strings simultaneously:
    - Increment the count for the character found in `s`.
    - Decrement the count for the character found in `t`.
4.  **Verification**: After the loop, check the frequency array.
    - If all values are `0`, the characters in `s` and `t` canceled each other out perfectly, meaning they are anagrams.
    - If any value is non-zero, the strings have different character compositions.

### Go Code

``` go
func isAnagram(s string, t string) bool {
    if len(s) != len(t) {
        return false
    }
    // Fixed-size array for 26 lowercase English letters
    cnt := [26]int{}
    for i := range s {
        cnt[s[i]-'a']++
        cnt[t[i]-'a']--
    }

    for _, count := range cnt {
        if count != 0 {
            return false
        }
    }
    return true
}
```


### Code Efficiency

- **Time Complexity**: $O(n)$
    - We iterate through the strings once ($O(n)$) and then perform a constant-time check on the 26-element array ($O(1)$). Total time is $O(n)$.
- **Space Complexity**: $O(1)$
    - We use a fixed-size array of size 26 regardless of the input string length $n$. This is considered constant auxiliary space.


