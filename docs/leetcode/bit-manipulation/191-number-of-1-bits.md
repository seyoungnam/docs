# 191. Number of 1 Bits

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/number-of-1-bits/description/)

## Solution: Brian Kernighan's Algorithm

To count the number of set bits (also known as the Hamming Weight) in an integer `n`, we can use **Brian Kernighan's Algorithm**. This approach is more efficient than checking all bits one-by-one because it jumps directly from one set bit to the next.

### Thought Process

1.  **Bit Clearing Property**:
    *   Subtracting `1` from a number $n$ reverses all bits from the rightmost set bit to the end:
        *   Example: $12_{10} = 1100_2$
        *   $12 - 1 = 11_{10} = 1011_2$
    *   Performing a bitwise AND between $n$ and $n - 1$ clears the rightmost set bit:
        *   $1100_2 \ \& \ 1011_2 = 1000_2$
2.  **Counting Iterations**:
    *   By applying $n = n \ \& \ (n - 1)$ repeatedly in a loop, we clear one set bit per iteration.
    *   We increment a counter `cnt` for each iteration.
3.  **Termination**:
    *   The loop terminates when all set bits are cleared ($n == 0$). The value of `cnt` is the total number of set bits.

### Go Code

``` go
func hammingWeight(n int) int {
    cnt := 0
    for n > 0 {
        n = n & (n-1)
        cnt++
    }    
    return cnt
}
```

### Code Efficiency

- **Time Complexity**: $O(k)$
    - Where $k$ is the number of set bits in $n$. For a 32-bit integer, the loop runs at most 32 times, making it $O(1)$ in practice.
- **Space Complexity**: $O(1)$
    - Only a single counter variable is used.