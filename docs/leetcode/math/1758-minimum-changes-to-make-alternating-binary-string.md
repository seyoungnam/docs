# 1758. Minimum Changes To Make Alternating Binary String

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/minimum-changes-to-make-alternating-binary-string/description/)

## Solution: Pattern Matching (Complement Rule)

An alternating binary string of length $n$ can only take one of two patterns:
1. **Pattern A**: `010101...` (starts with `'0'`, where characters at index `i` equal `i % 2`).
2. **Pattern B**: `101010...` (starts with `'1'`, where characters at index `i` equal `1 - (i % 2)`).

Because Pattern B is the exact inverse (bitwise NOT) of Pattern A, any character that matches Pattern A must be flipped to match Pattern B, and vice versa. If we count how many characters match Pattern A (let's call this `patternCount`), then:
- The operations required to transform `s` into Pattern A is $n - \text{patternCount}$.
- The operations required to transform `s` into Pattern B is $\text{patternCount}$.

The final answer is the minimum of these two values: $\min(\text{patternCount}, n - \text{patternCount})$.

### Thought Process

1.  **Identify Target Patterns**:
    - Observe that there are only two valid alternating patterns of length $n$.
    - Standardize the check against Pattern A: `010101...`. The expected character at index `i` is `i % 2`.
2.  **Linear Scan**:
    - Iterate through the string `s`.
    - Check if the current bit matches the expected bit for Pattern A at index `i`: `currBit == (i % 2)`.
    - If it matches, increment `patternCount`.
3.  **Result Retrieval**:
    - The number of mismatches for Pattern A is `n - patternCount`.
    - The number of mismatches for Pattern B is `patternCount`.
    - Return the smaller of these two counts.

### Go Code

``` go
func minOperations(s string) int {
    n := len(s)
    patternCount := 0 // 0101..

    for i := 0; i < n; i++ {
        currBit := int(s[i] - '0')
        if currBit == (i%2) {
            patternCount++
        }
    }
    return min(patternCount, n-patternCount)
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We iterate through the string of length $n$ exactly once.
- **Space Complexity**: $O(1)$
    - We only track state using a few integer variables, requiring constant auxiliary space.