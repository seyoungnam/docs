# 639. Decode Ways II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/decode-ways-ii/description/)

## Solution: Top-Down Dynamic Programming (Memoized DFS)

We can find the total number of ways to decode the string `s` (containing digits and the wildcard `'*'`) by exploring partition choices top-down. To prevent Time Limit Exceeded (TLE), we use a 1D array to memoize subproblem results.

### Thought Process

1.  **State Definition & Memoization**:
    *   Let $dfs(i)$ be the number of ways to decode the suffix starting at index $i$.
    *   Initialize a 1D array `memo` of size $n$ with `-1`. If `memo[i] != -1`, return the cached value immediately.
2.  **Single-Digit Transitions**:
    *   If $s[i] == '0'$, it cannot start any valid decoding: return $0$.
    *   If $s[i] == '*'$: It can represent any digit from `'1'` to `'9'`, yielding 9 choices:
        $$\text{res} = 9 \times dfs(i+1) \pmod{10^9+7}$$
    *   Otherwise, $s[i]$ is a fixed digit (from `'1'` to `'9'`), yielding 1 choice:
        $$\text{res} = dfs(i+1)$$
3.  **Two-Digit Transitions**:
    *   If $i + 1 < n$, we check if we can form a double-digit block (values from $10$ to $26$):
        *   **Case A ($s[i] == '*'$)**:
            *   If $s[i+1] == '*'$, the block is `**`. The first star can be `'1'` (giving 9 ways: `11`-`19`) or `'2'` (giving 6 ways: `21`-`26`), totaling 15 ways:
                $$\text{res} = (\text{res} + 15 \times dfs(i+2)) \pmod{10^9+7}$$
            *   If $s[i+1] \le '6'$, the first star can be `'1'` or `'2'` (e.g. `16` and `26` are valid), totaling 2 ways.
            *   If $s[i+1] > '6'$, the first star can only be `'1'` (e.g. `17` is valid, but `27` is invalid), totaling 1 way.
        *   **Case B ($s[i] == '1'$)**:
            *   If $s[i+1] == '*'$, the block is `1*`, which can be `11`-`19` (9 ways).
            *   Otherwise, $s[i+1]$ is a fixed digit, yielding 1 way.
        *   **Case C ($s[i] == '2'$)**:
            *   If $s[i+1] == '*'$, the block is `2*`, which can only be `21`-`26` (6 ways).
            *   If $s[i+1] \le '6'$, we have 1 way (e.g., `20`-`26`).
4.  **Base Case**:
    *   When $i == n$, the entire string has been successfully decoded: return $1$.

### Go Code

``` go
const MOD = 1e9+7

func numDecodings(s string) int {
    n := len(s)
    memo := make([]int, n)
    for i := range memo {
        memo[i] = -1
    }

    var dfs func(i int) int
    dfs = func(i int) int {
        if i == n {
            return 1
        }
        if memo[i] != -1 {
            return memo[i]
        }
        if s[i] == '0' {
            return 0
        }

        var res int
        if s[i] == '*' {
            res = (9 * dfs(i+1)) % MOD
        } else {
            res = dfs(i+1)
        }

        if i+1 < n {
            if s[i] == '*' {
                if s[i+1] == '*' {
                    res = (res + 15*dfs(i+2)) % MOD
                } else if s[i+1] <= '6' {
                    res = (res + 2*dfs(i+2)) % MOD
                } else {
                    res = (res + dfs(i+2)) % MOD
                }
            } else if s[i] == '1' {
                if s[i+1] == '*' {
                    res = (res + 9*dfs(i+2)) % MOD
                } else {
                    res = (res + dfs(i+2)) % MOD
                }
            } else if s[i] == '2' {
                if s[i+1] == '*' {
                    res = (res + 6*dfs(i+2)) % MOD
                } else if s[i+1] <= '6' {
                    res = (res + dfs(i+2)) % MOD
                }
            }
        }
        memo[i] = res
        return res
    }
    return dfs(0)
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - There are at most $n + 1$ unique subproblems. Since each subproblem is resolved in $O(1)$ constant time operations and computed at most once, the total time complexity is linear.
- **Space Complexity**: $O(n)$
    - The auxiliary space is determined by the `memo` array of size $n$ and the recursion stack, which can go up to $n$ levels deep.