# 2272. Substring With Largest Variance

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/substring-with-largest-variance/description/)

## What is Kadane's Algorithm?

**Kadane's Algorithm** is a classic dynamic programming algorithm used to find the **Maximum Subarray Sum** in $O(n)$ time. 
* At each element `x`, we decide whether to add `x` to our current running subarray or start a new subarray beginning at `x`.
* **State Transition**: `currSum = max(x, currSum + x)`. If `currSum` becomes negative, we reset it to `0`.

---

## Solution: Kadane's Algorithm on Character Pairs

For this problem, we want to maximize the variance between any two characters `a` and `b` in a substring. Since there are only 26 lowercase English letters, we can try all possible ordered pairs `(a, b)` where we treat `a` as `+1` (character to maximize) and `b` as `-1` (character to minimize). 

However, standard Kadane's fails because a valid substring **must contain at least one `b`** (we cannot have a variance of only `a`s). To fix this, we track whether a `b` has been discarded in a previous reset, allowing us to conceptually "prepend" it to our current subarray.

### Thought Process

1.  **Count Frequencies**: Record which characters are present in `s` to avoid checking pairs that do not exist.
2.  **Try All Pairs**: Iterate through all ordered pairs `(a, b)` where `a != b`:
    *   Treat `a` as `+1`.
    *   Treat `b` as `-1`.
3.  **Run Modified Kadane's**: Traverse the string:
    *   If char is `a`, increment `variance`.
    *   If char is `b`, decrement `variance` and set `hasB = true`.
    *   If `hasB` is `true`, update `maxVar = max(maxVar, variance)`.
    *   If `hasPrevB` is `true` but `hasB` is `false` (current subarray has no `b` yet), we can conceptually prepend the previously discarded `b`, yielding a variance of `variance - 1`. Update `maxVar = max(maxVar, variance - 1)`.
    *   **Reset**: If `variance < 0`, reset `variance = 0`, `hasB = false`, and set `hasPrevB = true` (since we just discarded a `b`).

### Go Code

``` go
func largestVariance(s string) int {
    // Count total occurrences of each character
    freq := make([]int, 26)
    for i := 0; i < len(s); i++ {
        freq[s[i]-'a']++
    }
    
    maxVar := 0
    
    // Try all ordered pairs of characters (a, b)
    for a := 0; a < 26; a++ {
        for b := 0; b < 26; b++ {
            if a == b || freq[a] == 0 || freq[b] == 0 {
                continue
            }
            
            variance := 0
            hasB := false
            hasPrevB := false
            
            for i := 0; i < len(s); i++ {
                c := int(s[i] - 'a')
                if c == a {
                    variance++
                } else if c == b {
                    variance--
                    hasB = true
                } else {
                    continue // Ignore other characters
                }
                
                // If the current subarray contains at least one 'b'
                if hasB {
                    maxVar = max(maxVar, variance)
                }
                
                // If we can prepend a previously discarded 'b'
                if hasPrevB && !hasB {
                    maxVar = max(maxVar, variance-1)
                }
                
                // Reset if variance drops below 0 (classic Kadane's)
                if variance < 0 {
                    variance = 0
                    hasB = false
                    hasPrevB = true
                }
            }
        }
    }
    return maxVar
}
```

### Code Efficiency

- **Time Complexity**: $O(26 \cdot 25 \cdot n) = O(n)$
    - We try $26 \times 25 = 670$ character pairs. For each pair, we perform a single linear scan of `s`.
- **Space Complexity**: $O(1)$
    - We use a fixed-size frequency array of size 26 and a few tracking variables.
