# 7. Reverse Integer

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/reverse-integer/description/)

## Solution: Digit Extraction with Overflow Prevention

To reverse the digits of an integer, we can repeatedly "pop" the last digit from the input integer and "push" it to the end of a new reversed integer. 

However, because we must prevent 32-bit signed integer overflow (returning `0` if the value exceeds the range $[-2^{31}, 2^{31} - 1]$), we must check for overflow/underflow *before* multiplying our current reversed result by $10$ and adding the popped digit.

### Thought Process

1.  **Digit Manipulation (Pop & Push)**:
    *   In a loop that runs while `x != 0`, we pop the last digit of `x`: `pop := x % 10`.
    *   Truncate `x` to remove its last digit: `x /= 10`.
    *   Normally, we would append it by doing: `reversed = reversed * 10 + pop`.
2.  **Overflow and Underflow Bounds**:
    *   The range of a 32-bit signed integer is $[-2147483648, 2147483647]$.
    *   Let $\text{MaxInt32} = 2147483647$ and $\text{MinInt32} = -2147483648$.
    *   Before updating `reversed = reversed * 10 + pop`:
        *   **Overflow Check**: If `reversed > MaxInt32 / 10`, multiplying by $10$ will overflow. If `reversed == MaxInt32 / 10`, it will overflow if `pop > 7` (the last digit of `2147483647`).
        *   **Underflow Check**: If `reversed < MinInt32 / 10`, multiplying by $10$ will underflow. If `reversed == MinInt32 / 10`, it will underflow if `pop < -8` (the last digit of `-2147483648`).
3.  **Sign Handling in Go**:
    *   In Go, the `%` operator on a negative number retains the negative sign (e.g., `-123 % 10` is `-3`, and `-123 / 10` is `-12`). This allows the exact same code and boundaries to work seamlessly for both positive and negative inputs without requiring absolute value conversions.

### Go Code

``` go
import "math"

func reverse(x int) int {
    reversed := 0

    for x != 0 {
        pop := x%10
        x /= 10
        
        // math.MaxInt32 is 2147483647 and math.MinInt32 is -2147483648
        if reversed > math.MaxInt32/10 || (reversed == math.MaxInt32/10 && pop > 7) {
            return 0
        }
        if reversed < math.MinInt32/10 || (reversed == math.MinInt32/10 && pop < -8) {
            return 0
        }

        reversed = reversed*10 + pop
    }
    return reversed
}
```

### Code Efficiency

- **Time Complexity**: $O(\log_{10} |x|)$
    - The number of digits in $x$ is proportional to $\log_{10} |x|$. The loop executes exactly once for each digit.
- **Space Complexity**: $O(1)$
    - We only allocate a few primitive integer variables, using constant auxiliary space.