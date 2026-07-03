# 3121. Count the Number of Special Characters II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/count-the-number-of-special-characters-ii/description/)

## Solution: Single-Pass Position Tracking

A character is considered **special** if both its lowercase and uppercase forms appear in the word, and **every** occurrence of its lowercase form appears **before** the **first** occurrence of its uppercase form. 

To solve this in a single pass, we track the position of the first occurrence of each uppercase letter and the last occurrence of each lowercase letter, invalidating the lowercase letter if it ever appears after an uppercase counterpart.

### Thought Process

1.  **State Representation Arrays**:
    *   Maintain two arrays of size 26: `lowerPos` and `upperPos`.
    *   Using 1-based indices (recording `i + 1` instead of `i`) allows us to use `0` to denote that a letter has not been seen yet.
2.  **Tracking Rules**:
    *   For each character `c` at index `i` of `word`:
        *   **Uppercase (`'A' <= c <= 'Z'`)**:
            *   Calculate `idx = c - 'A'`.
            *   We only care about the **first** occurrence of the uppercase letter. If `upperPos[idx] == 0`, record its position: `upperPos[idx] = i + 1`.
        *   **Lowercase (`'a' <= c <= 'z'`)**:
            *   Calculate `idx = c - 'a'`.
            *   Check if we have already encountered the corresponding uppercase letter: `upperPos[idx] > 0`.
                *   If `true`, this lowercase letter appears after an uppercase letter, violating the rule. We invalidate it by setting `lowerPos[idx] = -1`.
                *   If `false`, this lowercase letter is still valid. Update its position to the latest index: `lowerPos[idx] = i + 1`.
3.  **Result Aggregation**:
    *   Iterate through all 26 alphabet indices.
    *   A character is special if and only if:
        *   `lowerPos[i] > 0` (it was seen and never invalidated by a late-arriving lowercase letter).
        *   `upperPos[i] > 0` (its uppercase counterpart was seen).
    *   Increment our counter if both conditions are met.

### Go Code

``` go
func numberOfSpecialChars(word string) int {
    lowerPos, upperPos := [26]int{}, [26]int{}

    for i := 0; i < len(word); i++ {
        c := word[i]
        if 'A' <= c && c <= 'Z' {
            idx := int(c-'A')

            if upperPos[idx] == 0 {
                upperPos[idx] = i+1
            }
        } else if 'a' <= c && c <= 'z' {
            idx := int(c-'a')
            
            if upperPos[idx] > 0 {
                lowerPos[idx] = -1
            } else {
                lowerPos[idx] = i+1
            }
        }
    }
    res := 0
    for i := 0; i < 26; i++ {
        if lowerPos[i] > 0 && upperPos[i] > 0 {
            res++
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We process the string of length $n$ exactly once. The final loop runs a constant 26 times, resulting in $O(n)$ overall runtime.
- **Space Complexity**: $O(1)$
    - We use two fixed-size arrays of size 26 (`[26]int`), using constant auxiliary memory.