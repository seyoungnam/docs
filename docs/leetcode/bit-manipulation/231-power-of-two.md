# 231. Power of Two

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/power-of-two/description/)

## Solution: Bitwise Clearing Check

An integer $n$ is a power of two if it is strictly positive ($n > 0$) and has exactly one set bit in its binary representation. We can determine this in $O(1)$ time by clearing the lowest set bit.

### Thought Process

1.  **Binary Structure of Powers of Two**:
    *   $1_{10} = 0001_2$ (one set bit)
    *   $2_{10} = 0010_2$ (one set bit)
    *   $4_{10} = 0100_2$ (one set bit)
    *   $8_{10} = 1000_2$ (one set bit)
2.  **Clearing the Lowest Set Bit**:
    *   The expression `n & (n - 1)` clears the lowest set bit of $n$.
    *   If $n$ is a power of two, clearing its single set bit must result in $0$:
        *   Example: $4_{10} = 0100_2$
        *   $n - 1 = 3_{10} = 0011_2$
        *   $0100_2 \ \& \ 0011_2 = 0000_2$
3.  **Handling Edge Cases**:
    *   Any integer less than or equal to $0$ cannot be a power of two. We check if `n == 0` (and return `false`).
    *   If `n & (n - 1) == 0` is true, then $n$ is a power of two.

### Go Code

``` go
func isPowerOfTwo(n int) bool {
    if n == 0 {
        return false
    }
    return n & (n-1) == 0
}
```

### Code Efficiency

- **Time Complexity**: $O(1)$
    - The check uses simple arithmetic and bitwise AND operations, which execute in constant time.
- **Space Complexity**: $O(1)$
    - Only a constant amount of memory is used.