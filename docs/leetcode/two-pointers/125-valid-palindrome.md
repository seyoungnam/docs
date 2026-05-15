# 125. Valid Palindrome

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/valid-palindrome/description/)

## Solution: Character Filtering and Two Pointers

The problem requires checking if a string is a palindrome after converting all uppercase letters to lowercase and removing all non-alphanumeric characters. We can solve this by first cleaning the string and then using two pointers to compare characters from both ends.

### Thought Process

1.  **String Normalization**:
    - The input string `s` may contain uppercase letters, spaces, and punctuation.
    - Create a helper function `convert` to filter out non-alphanumeric characters.
    - During filtering, convert all uppercase letters (`'A'-'Z'`) to lowercase (`'a'-'z'`).
2.  **Two-Pointer Comparison**:
    - Once we have the normalized `converted` byte slice, initialize two pointers: `i` at the start (0) and `j` at the end (`n-1`).
    - Move `i` forward and `j` backward, comparing the characters at each step.
    - **Found Mismatch**: If `converted[i] != converted[j]`, the string is not a palindrome; return `false`.
    - **Success**: If the pointers meet or cross without any mismatch, return `true`.

### Go Code

``` go
func isPalindrome(s string) bool {
    converted := convert(s)
    n := len(converted)
    for i, j := 0, n-1; i < j; i, j = i+1, j-1 {
        if converted[i] != converted[j] {
            return false
        }
    }
    return true
}

func convert(s string) []byte {
    bytes := []byte(s)
    res := []byte{}
    for i := 0; i < len(bytes); i++ {
        c := bytes[i]
        // Convert uppercase to lowercase and keep alphanumeric
        if c >= 'A' && c <= 'Z' {
            res = append(res, c - 'A' + 'a')
        } else if (c >= 'a' && c <= 'z') || (c >= '0' && c <= '9') {
            res = append(res, c)
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We iterate through the original string once to filter it ($O(n)$).
    - We iterate through the filtered string once with two pointers to check the palindrome property ($O(n)$). Total time is linear.
- **Space Complexity**: $O(n)$
    - We use an auxiliary byte slice `res` to store the normalized version of the string, which can take up to $O(n)$ space.

---

## Optimized Solution: In-Place Two Pointers

### Thought Process

1.  **Avoid Auxiliary Space**: Instead of creating a new string, we can perform the filtering and comparison in a single pass using two pointers directly on the original string.
2.  **Pointer Movement**:
    - Pointer `l` starts at the beginning, and pointer `r` starts at the end.
    - In each step, if the character at `l` is not alphanumeric, skip it and increment `l`.
    - Similarly, if the character at `r` is not alphanumeric, skip it and decrement `r`.
3.  **Case-Insensitive Comparison**: Compare the characters at `l` and `r` after converting them to lowercase.
4.  **Early Exit**: If a mismatch is found, return `false`.

### Go Code

``` go
import (
    "unicode"
)

func isPalindrome(s string) bool {
    runes := []rune(s)
    l, r := 0, len(runes)-1
    
    for l < r {
        for l < r && !unicode.IsLetter(runes[l]) && !unicode.IsDigit(runes[l]) {
            l++
        }
        for l < r && !unicode.IsLetter(runes[r]) && !unicode.IsDigit(runes[r]) {
            r--
        }
        
        if unicode.ToLower(runes[l]) != unicode.ToLower(runes[r]) {
            return false
        }
        l++
        r--
    }
    return true
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - Each character in the string is visited at most twice (once by `l` or `r`).
- **Space Complexity**: $O(1)$ auxiliary space
    - Note: In Go, strings are immutable, so converting to a slice of runes takes $O(n)$ space. If we use the string directly with byte indexing (assuming ASCII), we can achieve true $O(1)$ auxiliary space.
