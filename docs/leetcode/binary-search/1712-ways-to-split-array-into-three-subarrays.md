# 1712. Ways to Split Array Into Three Subarrays

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/ways-to-split-array-into-three-subarrays/description/)

## Solution 1: Prefix Sum + Binary Search

We are tasked with dividing an array of non-negative integers into three non-empty contiguous subarrays `left`, `mid`, and `right` such that:

$$\text{sum}(\text{left}) \le \text{sum}(\text{mid}) \le \text{sum}(\text{right})$$

Because all elements in `nums` are non-negative, the prefix sum array `prefix` is monotonically non-decreasing. This monotonic property allows us to fix the first split point $i$ and use **Binary Search** to find the valid range for the second split point $j$.

---

### Thought Process

1. **Prefix Sum Formulation**:
    * Let $\text{prefix}[k] = \sum_{m=0}^k \text{nums}[m]$.
    * For a chosen index $i$ ($0 \le i \le n - 3$) and index $j$ ($i + 1 \le j \le n - 2$):
        * $\text{sum}(\text{left}) = \text{prefix}[i]$
        * $\text{sum}(\text{mid}) = \text{prefix}[j] - \text{prefix}[i]$
        * $\text{sum}(\text{right}) = \text{prefix}[n-1] - \text{prefix}[j]$

2. **Deriving the Constraints on $j$**:
    * **Condition 1**: $\text{sum}(\text{left}) \le \text{sum}(\text{mid})$
      $$\text{prefix}[i] \le \text{prefix}[j] - \text{prefix}[i] \implies \text{prefix}[j] \ge 2 \cdot \text{prefix}[i]$$
    * **Condition 2**: $\text{sum}(\text{mid}) \le \text{sum}(\text{right})$
      $$\text{prefix}[j] - \text{prefix}[i] \le \text{prefix}[n-1] - \text{prefix}[j] \implies 2 \cdot \text{prefix}[j] \le \text{prefix}[n-1] + \text{prefix}[i]$$

3. **Binary Search for Bounds of $j$**:
    * For each fixed $i$, valid split indices $j$ form a contiguous interval $[\text{minJ}, \text{maxJ}] \subseteq [i + 1, n - 2]$.
    * **Find $\text{minJ}$**: The smallest index $j \in [i+1, n-2]$ satisfying $\text{prefix}[j] \ge 2 \cdot \text{prefix}[i]$.
    * **Find $\text{maxJ}$**: The largest index $j \in [i+1, n-2]$ satisfying $2 \cdot \text{prefix}[j] \le \text{prefix}[n-1] + \text{prefix}[i]$.
    * If $\text{minJ} \le \text{maxJ}$, there are $(\text{maxJ} - \text{minJ} + 1)$ valid split options for this $i$.

4. **Early Termination**:
    * If $\text{prefix}[i] \cdot 3 > \text{prefix}[n-1]$, the left subarray alone accounts for more than one-third of the total sum, making $\text{sum}(\text{left}) \le \text{sum}(\text{mid}) \le \text{sum}(\text{right})$ impossible for this and any subsequent $i$. We can immediately terminate the loop.

---

### Go Code

```go
const MOD = 1_000_000_007

func waysToSplit(nums []int) int {
    n := len(nums)

    prefix := make([]int, n)
    prefix[0] = nums[0]
    for i := 1; i < n; i++ {
        prefix[i] = prefix[i-1] + nums[i]
    }

    res := 0

    // i is the end of the left subarray: nums[0...i]
    for i := 0; i < n-2; i++ {
        leftSum := prefix[i]

        // Early stopping: left sum cannot exceed 1/3 of the total sum
        if leftSum*3 > prefix[n-1] {
            break
        }

        // 1. Binary search for lower bound minJ: prefix[j] >= 2 * leftSum
        l, r := i+1, n-2
        minJ := -1
        for l <= r {
            m := l + (r-l)/2
            if prefix[m] >= 2*leftSum {
                minJ = m
                r = m - 1
            } else {
                l = m + 1
            }
        }

        // 2. Binary search for upper bound maxJ: 2 * prefix[j] <= prefix[n-1] + leftSum
        l, r = i+1, n-2
        maxJ := -1
        for l <= r {
            m := l + (r-l)/2
            if 2*prefix[m] <= prefix[n-1]+leftSum {
                maxJ = m
                l = m + 1
            } else {
                r = m - 1
            }
        }

        // Add valid split count for this i
        if minJ != -1 && maxJ != -1 && maxJ >= minJ {
            res = (res + (maxJ - minJ + 1)) % MOD
        }
    }

    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n \log n)$
    - Constructing the prefix sum array takes $O(n)$ time.
    - We iterate through $i$ from $0$ to $n-3$. In each step, we perform two binary searches over an array of size $O(n)$, taking $O(\log n)$ per iteration. Total time is $O(n \log n)$.
- **Space Complexity**: $O(n)$
    - $O(n)$ space is used to store the prefix sum array (which can be reduced to $O(1)$ auxiliary space if modified in-place).

---

## Solution 2: Three Pointers (Two Pointers / Sliding Window)

Because the prefix sum array is monotonically non-decreasing, as $i$ increases, both the lower bound threshold $2 \cdot \text{prefix}[i]$ and the upper bound threshold $\frac{\text{prefix}[n-1] + \text{prefix}[i]}{2}$ are non-decreasing. This monotonicity allows us to maintain two running pointers $j$ and $k$ that only move forward, achieving an optimal $O(n)$ time complexity.

---

### Thought Process

1. Maintain pointer $j$ representing the first valid position where $\text{prefix}[j] \ge 2 \cdot \text{prefix}[i]$.
2. Maintain pointer $k$ representing the first position where the right sum is strictly smaller than the middle sum ($2 \cdot \text{prefix}[k] > \text{prefix}[n-1] + \text{prefix}[i]$).
3. For each $i$:
    * Advance $j$ until $\text{prefix}[j] \ge 2 \cdot \text{prefix}[i]$ (or $j = n-1$).
    * Advance $k$ until $2 \cdot \text{prefix}[k] > \text{prefix}[n-1] + \text{prefix}[i]$ (or $k = n-1$).
    * If $k > j$, all indices in $[j, k-1]$ are valid split points, contributing $(k - j)$ valid combinations.

---

### Go Code

```go
const MOD = 1_000_000_007

func waysToSplit(nums []int) int {
    n := len(nums)

    prefix := make([]int, n)
    prefix[0] = nums[0]
    for i := 1; i < n; i++ {
        prefix[i] = prefix[i-1] + nums[i]
    }

    res := 0
    j, k := 0, 0

    for i := 0; i < n-2; i++ {
        leftSum := prefix[i]
        if leftSum*3 > prefix[n-1] {
            break
        }

        // Ensure j starts at least at i + 1
        j = max(j, i+1)
        for j < n-1 && prefix[j] < 2*leftSum {
            j++
        }

        // Ensure k starts at least at j
        k = max(k, j)
        for k < n-1 && 2*prefix[k] <= prefix[n-1]+leftSum {
            k++
        }

        if k > j {
            res = (res + (k - j)) % MOD
        }
    }

    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - Pointers $i$, $j$, and $k$ only advance forward from $0$ to $n-1$, traversing the array at most a constant number of times.
- **Space Complexity**: $O(n)$
    - Storing the prefix sums requires $O(n)$ memory.