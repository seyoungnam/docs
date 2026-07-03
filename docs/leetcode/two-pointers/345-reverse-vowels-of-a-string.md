# 345. Reverse Vowels of a String

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/reverse-vowels-of-a-string/description/)

## Solution: Two Pointers (Left and Right)

We can reverse only the vowels of a string in-place by converting the string to a mutable byte slice and applying the **Two Pointers** technique. One pointer starts at the beginning of the string and moves right, while the other starts at the end of the string and moves left. 

### Thought Process

1.  **Fast Vowel Lookup**:
    *   To determine if any ASCII character is a vowel in $O(1)$ time, we initialize a fixed-size boolean array `isVowel` of size `256`.
    *   We populate it by setting `true` for both lowercase and uppercase vowels: `"aeiouAEIOU"`.
2.  **Mutability Conversion**:
    *   Since strings are immutable in Go, we convert the string into a mutable byte slice: `res := []byte(s)`.
3.  **Two-Pointer Swap**:
    *   Initialize `l = 0` (left pointer) and `r = len(res) - 1` (right pointer).
    *   While `l < r`:
        *   Increment `l` until `res[l]` is a vowel or `l >= r`.
        *   Decrement `r` until `res[r]` is a vowel or `l >= r`.
        *   If `l < r`, swap the vowels: `res[l], res[r] = res[r], res[l]`.
        *   Increment `l` and decrement `r` to step inside the window.
4.  **Result**:
    *   Convert the byte slice `res` back to a string and return.

### Go Code

``` go
func reverseVowels(s string) string {
    var isVowel [256]bool
    for _, ch := range "aeiouAEIOU" {
        isVowel[ch] = true
    }

    res := []byte(s)
    l, r := 0, len(res)-1

    for l < r {
        for l < r && !isVowel[res[l]] {
            l++
        }
        for l < r && !isVowel[res[r]] {
            r--
        }

        if l < r {
            res[l], res[r] = res[r], res[l]
            l++
            r--
        }
    }
    return string(res)
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We traverse the string of length $n$ exactly once. Vowel lookups in the boolean array are done in $O(1)$ constant time.
- **Space Complexity**: $O(n)$
    - Since Go strings are immutable, converting the string to a byte slice requires allocating $O(n)$ space. The lookup array uses a fixed-size allocation of $256$ bytes, which is $O(1)$ auxiliary space.