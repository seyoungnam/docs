# 342. Power of Four

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/power-of-four/description/)

## Solution: Bitmasking Power of Two

An integer $n$ is a power of four if it is strictly positive ($n > 0$), is a power of two, and has its single set bit located at an even-indexed position (starting from index 0 on the right).

### Thought Process

1.  **Relation to Powers of Two**:
    *   Any power of four ($4^k$) can be written as $2^{2k}$, which is a power of two.
    *   Thus, we must first verify that $n$ is a power of two using $n > 0$ and the bit-clearing check:
        $$n \ \& \ (n - 1) == 0$$
2.  **Bit Position Constraints**:
    *   Let's look at the binary representations of the first few powers of four:
        *   $4^0 = 1_{10} = 00000001_2$ (bit index 0)
        *   $4^1 = 4_{10} = 00000100_2$ (bit index 2)
        *   $4^2 = 16_{10} = 00010000_2$ (bit index 4)
        *   $4^3 = 64_{10} = 01000000_2$ (bit index 6)
    *   The single set bit must reside on an **even bit index** (index 0, 2, 4, 6, etc.).
3.  **Even-Bit Masking**:
    *   To check if the set bit is in an even position, we mask the 32-bit integer with a binary pattern containing `1`s only at even indices:
        $$01010101010101010101010101010101_2 = \text{0x55555555}_{16}$$
    *   If `(n & 0x55555555) != 0`, then the single set bit is at an even index.

### Go Code

``` go
func isPowerOfFour(n int) bool {
    // 0x55555555 is 01010101010101010101010101010101 in binary (even bits set)
    return n > 0 && (n & (n - 1)) == 0 && (n & 0x55555555) != 0
}
```

### Code Efficiency

- **Time Complexity**: $O(1)$
    - Only a fixed number of constant-time bitwise and comparison operations are performed.
- **Space Complexity**: $O(1)$
    - The algorithm runs in-place with no extra memory allocation.