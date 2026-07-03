# 50. Pow(x, n)

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/powx-n/description/)

## Solution: Binary Exponentiation (Iterative)

Calculating $x^n$ by multiplying $x$ by itself $n$ times takes $O(n)$ time, which will result in a Time Limit Exceeded (TLE) error for large values of $n$ (up to $2^{31} - 1$). 

We can optimize this to logarithmic time $O(\log n)$ using **Binary Exponentiation** (exponentiation by squaring). By analyzing the binary representation of the exponent $n$, we can build the result by repeatedly squaring the base and multiplying it into the result only when the corresponding binary bit of $n$ is set.

### Thought Process

1.  **Negative Exponent Handling**:
    *   If $n < 0$, we know that $x^{-n} = \left(\frac{1}{x}\right)^{n}$.
    *   We adjust the inputs by setting `x = 1 / x` and `n = -n` to handle all exponents uniformly as positive numbers.
2.  **Iterative Squaring**:
    *   Initialize `res = 1.0`, `base = x`, and `power = n`.
    *   While `power > 0`:
        *   If the current bit of `power` is odd (i.e., `power & 1 == 1`):
            *   Multiply `res` by the current `base`: `res *= base`.
        *   Square the base: `base *= base`. This represents transitioning from $x^p$ to $x^{2p}$.
        *   Right-shift `power` by 1 bit: `power >>= 1` (equivalent to dividing `power` by $2$).
3.  **Result**:
    *   Return `res`.

### Go Code

``` go
func myPow(x float64, n int) float64 {
    if n < 0 {
        x = 1/x
        n = -n
    }
    res := float64(1)
    base, power := x, n
    for power > 0 {
        if power & 1 == 1 {
            res *= base
        }
        base *= base
        power >>= 1
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(\log n)$
    - The loop runs at most $\approx \log_2(n)$ times because the exponent `power` is halved (right-shifted) in each iteration.
- **Space Complexity**: $O(1)$
    - We only allocate a constant number of scalar variables (`res`, `base`, `power`), utilizing constant auxiliary space.