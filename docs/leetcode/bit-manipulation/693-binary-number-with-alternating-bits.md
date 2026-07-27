# 693. Binary Number with Alternating Bits

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/binary-number-with-alternating-bits/description/)

Given a positive integer, we want to check if its binary representation has alternating adjacent bits (e.g. `101010` is alternating, while `1101` is not). Below are two solutions implemented in Go.

---

## Solution 1: XOR Shift Clearing (Recommended)

If a number has alternating bits, shifting the number right by 1 and XORing it with the original number will result in a value where every single bit is set to `1`.

### Thought Process

1.  **XOR Shifting**:
    *   Suppose $n = 10_{10} = 1010_2$.
    *   $n \gg 1 = 5_{10} = 0101_2$.
    *   XORing the two:
        $$x = n \oplus (n \gg 1) = 1010_2 \oplus 0101_2 = 1111_2$$
    *   If $n$ has alternating bits, $x$ is guaranteed to have all bits set to `1` ($111\dots1_2$).
2.  **All-Ones Bit Check**:
    *   To check if $x$ has all bits set to `1` (e.g., $1111_2$), we add `1` to it, which yields a power of two (e.g. $10000_2$).
    *   Taking the bitwise AND `x & (x + 1)` clears all bits and must equal `0`:
        $$1111_2 \ \& \ 10000_2 = 0000_2$$
3.  **Algorithm**:
    *   Let $x = n \oplus (n \gg 1)$.
    *   Return `x & (x + 1) == 0`.

### Go Code

``` go
func hasAlternatingBits(n int) bool {
    x := n ^ (n >> 1) // Yields all bits set to 1 (e.g., 11111) if alternating
    return x & (x+1) == 0
}
```

---

## Solution 2: Iterative Bit Toggling

We can iterate through the bits of the integer from right to left, checking if each bit matches the expected alternating pattern.

### Thought Process

1.  **Initialize**:
    *   Determine the rightmost bit of $n$: `curr = n & 1`.
2.  **Bit-by-Bit Loop**:
    *   While $n > 0$:
        *   If the current rightmost bit `n & 1` equals the expected bit value `curr`:
            *   We toggle the expected bit value for the next position using XOR: `curr ^= 1`.
        *   If it does not match, return `false` immediately.
        *   Shift the input right by 1 to process the next bit: `n >>= 1`.
3.  **Termination**:
    *   If the loop finishes without mismatches, return `true`.

### Go Code

``` go
func hasAlternatingBits(n int) bool {
    curr := n&1
    for n > 0 {
        if n&1 == curr {
            curr ^= 1 // Toggle expected bit: 0 -> 1 or 1 -> 0
        } else {
            return false
        }
        n >>= 1
    }
    return true
}
```

---

## Code Efficiency

For **Solution 1**:
- **Time Complexity**: $O(1)$
    - Constant number of bitwise operations.
- **Space Complexity**: $O(1)$
    - No extra memory allocation.

For **Solution 2**:
- **Time Complexity**: $O(\log n)$
    - Runs once per bit position in $n$.
- **Space Complexity**: $O(1)$
    - Constant auxiliary space.