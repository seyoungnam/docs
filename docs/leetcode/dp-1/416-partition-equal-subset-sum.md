# 416. Partition Equal Subset Sum

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/partition-equal-subset-sum/description/)

## Solution: 0-1 Knapsack Dynamic Programming (1D Array)

This problem can be transformed into the classic **0-1 Knapsack Problem**. We want to find a subset of numbers from `nums` that sums up to exactly half of the total sum of all elements in the array. 

If the total sum is odd, it is mathematically impossible to split the elements into two subsets with equal integer sums, so we can return `false` immediately. Otherwise, our target knapsack capacity is `bagSize = sum / 2`.

### Thought Process

1.  **Mathematical Formulations**:
    *   Let $S$ be the total sum of `nums`.
    *   If $S \pmod 2 \neq 0$, return `false`.
    *   Otherwise, we set our target capacity `bagSize = S / 2`.
2.  **DP State Definition**:
    *   Define a 1D boolean array `dp` of size `bagSize + 1`.
    *   `dp[j]` will be `true` if we can form a subset whose elements sum up to exactly `j`, and `false` otherwise.
    *   **Base Case**: `dp[0] = true` (an empty subset has a sum of $0$).
3.  **DP Transitions (Backwards Loop)**:
    *   For each element `num` in `nums`:
        *   Iterate `j` backwards from `bagSize` down to `num`:
            *   `dp[j] = dp[j] || dp[j - num]`
        *   *Note: We must iterate the capacity `j` backwards to prevent using the same element `num` multiple times (which would correspond to the Unbounded Knapsack problem instead of 0-1 Knapsack).*
4.  **Result**:
    *   `dp[bagSize]` indicates whether we can form a subset sum equal to exactly `sum / 2`.

### Go Code

``` go
func canPartition(nums []int) bool {
    sum := 0
    for _, num := range nums {
        sum += num
    }
    if sum%2 == 1 {
        return false
    }
    bagSize := sum/2
    dp := make([]bool, bagSize+1)
    dp[0] = true
    for _, num := range nums {
        for j := bagSize; j >= num; j-- {
            dp[j] = dp[j] || dp[j-num]
        }
    }
    return dp[bagSize]
}
```

### Code Efficiency

- **Time Complexity**: $O(n \times S)$
    - Where $n$ is the number of elements in `nums` and $S$ is the sum of the array. The outer loop runs $n$ times and the inner loop runs at most `bagSize` ($S / 2$) times.
- **Space Complexity**: $O(S)$
    - The size of the DP slice is $\frac{S}{2} + 1$, requiring space proportional to the target subset sum.