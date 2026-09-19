# 1711. Count Good Meals

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/count-good-meals/description/)

## Solution: Hash Map (Single-Pass Two Sum with Powers of Two)

This problem is a variation of the classic **Two Sum** problem. Instead of checking for a single fixed target, we check for a small, bounded set of possible target sums: powers of two.

---

### Thought Process

1. **Range of Target Sums**:
    * The problem states that $0 \le \text{deliciousness}[i] \le 2^{20}$.
    * The maximum possible sum of any two meals is:
      $$\max(\text{sum}) = 2^{20} + 2^{20} = 2 \times 2^{20} = 2^{21}$$
    * Therefore, the sum of any valid pair must be one of the **22 powers of two**:
      $$2^0, 2^1, 2^2, \dots, 2^{21}$$

2. **Single-Pass Hash Map (On-the-Fly Pairing)**:
    * Maintain a frequency map `freq` where `freq[x]` stores the count of items with deliciousness `x` seen so far.
    * For each meal `num` in `deliciousness`:
        * Iterate through all 22 powers of two ($p \in [2^0, 2^{21}]$).
        * Calculate the required complement: $\text{complement} = p - \text{num}$.
        * If `complement` exists in `freq`, add its frequency `freq[complement]` to `res`.
        * After checking all 22 powers, increment `freq[num]++`.
    * **Why this prevents duplicate counting**: By only pairing the current element with elements that appeared *before* it, each distinct pair of indices $(i, j)$ with $i < j$ is counted exactly once.

---

### Go Code

```go
const MOD = 1_000_000_007

func countPairs(deliciousness []int) int {
    // Precompute all 22 possible powers of two from 2^0 to 2^21
    powers := make([]int, 22)
    for i := 0; i <= 21; i++ {
        powers[i] = 1 << i
    }

    freq := make(map[int]int)
    res := 0

    for _, num := range deliciousness {
        for _, power := range powers {
            complement := power - num
            if count, exists := freq[complement]; exists {
                res = (res + count) % MOD
            }
        }
        freq[num]++
    }

    return res
}
```

---

### Code Efficiency

- **Time Complexity**: $O(22 \cdot n) = O(n)$
    - We iterate through `deliciousness` of length $n$ once. For each element, we check exactly 22 precomputed powers of two. Since map lookups take $O(1)$ average time, the overall runtime is $O(22n)$, which simplifies to linear $O(n)$ time.
- **Space Complexity**: $O(\min(n, U))$
    - Where $n$ is the number of items and $U$ is the range of unique deliciousness values ($U \le 2^{20}$). The `freq` hash map stores at most $n$ distinct deliciousness values, and the `powers` slice uses $O(1)$ fixed auxiliary space.