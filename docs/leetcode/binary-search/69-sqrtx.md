# 69. Sqrt(x)

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/sqrtx/description/)

## Solution: Binary Search on Value Range

To compute the square root of $x$ rounded down to the nearest integer, we perform a binary search on the range of possible answers: $[0, x]$.

### Thought Process

1.  **Define Search Boundaries**:
    *   The integer square root of $x$ must lie in the closed interval $[0, x]$.
    *   Set `l = 0` and `r = x`.
2.  **Binary Search Loop (`l <= r`)**:
    *   Calculate midpoint: `m = l + (r - l) / 2`.
    *   Compare $m^2$ with $x$:
        *   **Exact match ($m^2 == x$)**: We found the exact square root, return `m`.
        *   **Too large ($m^2 > x$)**: The value `m` is too large to be the square root. Search the left half: `r = m - 1`.
        *   **Too small ($m^2 < x$)**: The value `m` is a candidate for the floor integer square root, but there could be a larger value. Search the right half: `l = m + 1`.
3.  **Termination**:
    *   The loop terminates when `l > r`. At this point, `r` will point to the largest integer whose square is less than or equal to $x$ (i.e. $\lfloor \sqrt{x} \rfloor$). Return `r`.

### Go Code

``` go
func mySqrt(x int) int {
    l, r := 0, x
    for l <= r {
        m := l + (r-l)/2
        if m*m == x {
            return m
        }
        if m*m > x {
            r = m-1
        } else {
            l = m+1
        }
    }
    return r
}
```

### Code Efficiency

- **Time Complexity**: $O(\log x)$
    - The search range $[0, x]$ is halved in each step, resulting in logarithmic time complexity.
- **Space Complexity**: $O(1)$
    - Only a constant number of helper variables are tracked.