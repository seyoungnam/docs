# 190. Reverse Bits

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/reverse-bits/description/)

## Solution: Bit Shifting Accumulator

To reverse the bits of a 32-bit unsigned integer, we can iterate through the bits of the input, extracting one bit at a time from right to left, and appending it to our result accumulator from left to right.

### Thought Process

1.  **Extract and Shift Strategy**:
    *   We maintain an accumulator variable `res` initialized to `0`.
    *   We iterate exactly 32 times (once for each bit of the 32-bit integer).
2.  **Bitwise Accumulation Loop**:
    *   In each of the 32 steps:
        *   **Shift Result**: Shift `res` to the left by 1: `res <<= 1`. This moves all accumulated bits to the left, leaving the rightmost bit position vacant (`0`).
        *   **Extract Bit**: Extract the rightmost bit of `n` using `n & 1`.
        *   **Append Bit**: Place the extracted bit into the vacant rightmost position of `res` using bitwise OR: `res |= (n & 1)`.
        *   **Shift Input**: Shift the input number `n` to the right by 1: `n >>= 1`. This discards the processed bit and brings the next bit to the rightmost position for the next iteration.
3.  **Termination**:
    *   Since we loop exactly 32 times, all 32 bits are fully reversed. Return `res`.

### Go Code

``` go
func reverseBits(n int) int {
    var res int

    for range 32 {
        res <<= 1
        res |= (n & 1)
        n >>= 1
    }

    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(1)$
    - The loop runs exactly 32 times, executing a constant number of bitwise operations.
- **Space Complexity**: $O(1)$
    - The operation is performed in-place using constant auxiliary space.