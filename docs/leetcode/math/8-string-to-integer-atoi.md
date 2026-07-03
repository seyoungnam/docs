# 8. String to Integer (atoi)

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/string-to-integer-atoi/description/)

## Solution: State Machine Parsing

To implement a robust `atoi` converter, we process the string sequentially in several stages:
1.  **Discard Leading Whitespaces**: Skip any leading spaces.
2.  **Determine Sign**: Check for an optional `+` or `-` sign.
3.  **Read Digits**: Convert contiguous digit characters to an integer. Stop reading when a non-digit character is encountered.
4.  **Clamping Overflow**: Detect if the number goes outside the 32-bit signed integer range $[-2^{31}, 2^{31} - 1]$ and clamp it to the minimum or maximum boundary.

### Thought Process

1.  **Skip Whitespace**:
    *   Initialize pointer `i = 0`.
    *   Iterate while `i < len(s)` and `s[i] == ' '`, incrementing `i`.
2.  **Check for Sign**:
    *   If `s[i] == '+'`, increment `i` and keep sign as `1`.
    *   If `s[i] == '-'`, increment `i` and set `sign = -1`.
3.  **Construct Integer & Prevent Overflow**:
    *   For each digit character (where `'0' <= s[i] <= '9'`):
        *   Extract the integer value: `digit = int(s[i] - '0')`.
        *   Before shifting: `res = res * 10 + digit`, check for 32-bit signed integer overflow.
        *   **Overflow Check**: If `res > MaxInt32 / 10` or (`res == MaxInt32 / 10` and `digit > MaxInt32 % 10`), the operation will overflow.
            *   If `sign == 1`, clamp and return `MaxInt32` ($2147483647$).
            *   If `sign == -1`, clamp and return `MinInt32` ($-2147483648$).
        *   Otherwise, update `res = res * 10 + digit` and increment `i`.
4.  **Result**:
    *   Return `sign * res`.

### Go Code

``` go
import "math"

func myAtoi(s string) int {
    if len(s) == 0 {
        return 0
    }
    i := 0
    for i < len(s) && s[i] == ' ' {
        i++
    }
    sign := 1
    if i < len(s) && s[i] == '+' {
        i++
    } else if i < len(s) && s[i] == '-' {
        sign = -1
        i++
    }

    MAX, MIN := math.MaxInt32, math.MinInt32
    res := 0
    for i < len(s) && s[i] >= '0' && s[i] <= '9' {
        digit := int(s[i] - '0')
        if res > MAX/10 || (res == MAX/10 && digit > MAX%10) {
            if sign == 1 {
                return MAX
            }
            return MIN
        }
        res = res*10 + digit
        i++
    }
    return sign * res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We traverse the string of length $n$ at most once.
- **Space Complexity**: $O(1)$
    - The algorithm processes the characters in-place without any extra space, requiring constant auxiliary memory.