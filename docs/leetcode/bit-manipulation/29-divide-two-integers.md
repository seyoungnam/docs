# 29. Divide Two Integers

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/divide-two-integers/description/)

## Solution: Exponential Subtraction via Bit Shifting

To divide two integers without using multiplication, division, or modulo operators, we can perform division bit-by-bit. This is equivalent to finding the largest quotient `quo` such that $\text{divisor} \times \text{quo} \le \text{dividend}$ using binary search / exponential subtraction.

### Thought Process

1.  **Handling Edge Cases & Overflow**:
    *   The only overflow case in a 32-bit signed integer occurs when dividing $-2^{31}$ (minimum integer) by $-1$, which results in $2^{31}$ (exceeding the maximum positive limit $2^{31} - 1$). In this case, return `math.MaxInt32`.
    *   Determine the sign of the result: `neg := (dividend < 0) != (divisor < 0)`.
    *   To prevent overflow when taking absolute values, cast both numbers to positive 64-bit integers (`int64`).
2.  **Bitwise Division Logic**:
    *   We want to subtract powers of two multiples of the divisor from the dividend.
    *   We iterate down from the most significant bit position ($i = 31$) to the least significant ($i = 0$):
        *   We want to check if $d \ge dv \times 2^i$.
        *   To avoid overflow when shifting the divisor left (`dv << i`), we instead shift the dividend right: `(d >> i) >= dv`.
        *   If the condition is met:
            *   Add $2^i$ to the quotient: `quo += (1 << i)`.
            *   Subtract $dv \times 2^i$ from the remaining dividend: `d -= (dv << i)`.
3.  **Result Reconstruction**:
    *   Apply the negative sign if `neg` is true, and return the quotient cast to a standard integer.

### Go Code

``` go
import "math"

func divide(dividend int, divisor int) int {
    if dividend == math.MinInt32 && divisor == -1 {
        return math.MaxInt32
    }

    neg := (dividend < 0) != (divisor < 0)

    d := abs(int64(dividend))
    dv := abs(int64(divisor))

    quo := int64(0)

    for i := 31; i >= 0; i-- {
        if (d >> i) >= dv {
            quo += (1 << i)
            d -= (dv << i)
        }
    }

    if neg {
        quo = -quo
    }

    return int(quo)
}

func abs(n int64) int64 {
    if n < 0 {
        return -n
    }
    return n
}
```

### Code Efficiency

- **Time Complexity**: $O(1)$
    - The loop always executes exactly 32 times, performing a constant number of shifts and additions.
- **Space Complexity**: $O(1)$
    - The algorithm runs in-place using a constant number of integer tracking variables.