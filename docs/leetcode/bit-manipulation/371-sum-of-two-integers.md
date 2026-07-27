# 371. Sum of Two Integers

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/sum-of-two-integers/description/)

## Solution: Bitwise Half Adder Logic

To add two integers $a$ and $b$ without using the arithmetic operators `+` or `-`, we can simulate the behavior of a hardware **Half Adder** using bitwise operations. 

### Thought Process

1.  **XOR for Sum Without Carry**:
    *   The XOR (`^`) operator performs addition on individual bit columns without considering any carries:
        *   `0 ^ 0 = 0`
        *   `0 ^ 1 = 1`
        *   `1 ^ 0 = 1`
        *   `1 ^ 1 = 0` (this column overflows, generating a carry)
2.  **AND + Shift for Carry**:
    *   A carry is generated only when both bits at a given position are `1`. We identify these positions using the AND (`&`) operator.
    *   Since a carry propagates to the next column on the left, we shift the carry bit pattern left by 1: `(a & b) << 1`.
3.  **Iterative Addition**:
    *   We add the sum-without-carry (`a ^ b`) and the shifted carry `(a & b) << 1` by updating:
        *   `carry = (a & b) << 1`
        *   `a = a ^ b` (running sum)
        *   `b = carry` (carry to add in the next iteration)
    *   Repeat this process until the carry becomes zero (`b == 0`).

### Go Code

``` go
func getSum(a int, b int) int {
    for b != 0 {
        carry := (a & b) << 1
        a = a ^ b
        b = carry
    }
    return a
}
```

### Code Efficiency

- **Time Complexity**: $O(1)$
    - The loop terminates when there are no more carry bits to propagate. For 32-bit or 64-bit integers, the loop executes at most 32 or 64 times, resulting in a constant time complexity.
- **Space Complexity**: $O(1)$
    - Only a single temporary variable `carry` is allocated, requiring constant auxiliary space.