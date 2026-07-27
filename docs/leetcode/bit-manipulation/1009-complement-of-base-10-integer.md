# 1009. Complement of Base 10 Integer

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/complement-of-base-10-integer/description/)

> [!NOTE]
> This problem is identical to [476. Number Complement](https://leetcode.com/problems/number-complement/).

## Solution: Bitmask XOR Flipped

The complement of a binary number is obtained by flipping all its bits (changing `0`s to `1`s and `1`s to `0`s). Flipping bits is mathematically equivalent to performing a bitwise XOR (`^`) with a mask containing all `1`s of the same bit length.

### Thought Process

1.  **Bitwise XOR Cancellation**:
    *   Suppose $n = 5_{10} = 101_2$.
    *   We want to obtain $2_{10} = 010_2$.
    *   If we XOR $101_2$ with a mask of all `1`s ($111_2$), we get:
        $$101_2 \oplus 111_2 = 010_2$$
2.  **Constructing the Mask**:
    *   We initialize `ones = 1`.
    *   While `ones < n`, we shift `ones` left by 1 and set the lowest bit:
        `ones = (ones << 1) + 1` (implemented as `ones <<= 1` then `ones++`).
    *   This grows the mask ($1_2 \rightarrow 11_2 \rightarrow 111_2$, etc.) until its bit length matches or exceeds that of $n$.
3.  **Return**:
    *   Return `ones ^ n`.

### Go Code

``` go
func bitwiseComplement(n int) int {
    ones := 1
    for ones < n {
        ones <<= 1
        ones++
    }
    return ones^n
}
```

### Code Efficiency

- **Time Complexity**: $O(\log n)$
    - The mask construction loop runs once per bit position in $n$, executing at most 32 times for standard integers.
- **Space Complexity**: $O(1)$
    - The algorithm runs using constant auxiliary memory.