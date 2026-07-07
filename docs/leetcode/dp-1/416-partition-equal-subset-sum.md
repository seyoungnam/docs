# 416. Partition Equal Subset Sum

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/partition-equal-subset-sum/description/)

## Solution 1: Top-Down Dynamic Programming (Memoized DFS)

We can solve this problem using recursion by checking all possible subset combinations. To make it efficient, we memoize subproblem states using a map to cache results of previously visited index and sum configurations.

### Thought Process

1.  **Initial Feasibility Checks**:
    *   Find the sum of all elements `total` and track the maximum element `maxNum`.
    *   **Odd Sum check**: If `total` is odd, we cannot partition it into two equal integer subsets: return `false`.
    *   **Max Number check**: If the largest element `maxNum` is strictly greater than `total / 2`, it can never fit into any subset of size `total / 2` without exceeding it: return `false`.
2.  **State Representation**:
    *   Define a struct `Info` to uniquely represent each recursive subproblem state:
        ```go
        type Info struct {
            idx int
            sum int
        }
        ```
        where `idx` is the current element index in `nums` and `sum` is the running subset sum.
3.  **Memoization Mapping**:
    *   We use a map `memo := make(map[Info]int)` to cache results:
        *   `0`: State is unvisited.
        *   `1`: State cannot reach the target sum (`false`).
        *   `2`: State can reach the target sum (`true`).
4.  **Recursive Decisions**:
    *   At each index `i`:
        *   **Base Cases**:
            *   If `sum == target`, return `true` (success).
            *   If `i >= len(nums)` or `sum > target`, return `false` (invalid path).
            *   If `memo` has cached the state, return `memo[state] == 2` in $O(1)$ time.
        *   **Branching**:
            *   We try both including and excluding `nums[i]`:
                `dfs(i+1, sum+nums[i]) || dfs(i+1, sum)`
            *   Cache the result in `memo` as `2` (if successful) or `1` (if failed), and return.

### Go Code

``` go
type Info struct {
    idx     int
    sum     int
}

func canPartition(nums []int) bool {
    maxNum, total := -1, 0
    for _, num := range nums {
        total += num
        maxNum = max(maxNum, num)
    }
    if total%2 == 1 || maxNum > total/2  {
        return false
    }
    target := total/2

    // 0 = unvisited, 1 = false, 2 = true
    memo := make(map[Info]int)
    
    var dfs func(i int, sum int) bool 
    dfs = func(i int, sum int) bool {
        if sum == target {
            return true
        }
        if i >= len(nums) || sum > target {
            return false
        }
        if memo[Info{i, sum}] != 0 {
            return memo[Info{i, sum}] == 2
        }
        if dfs(i+1, sum+nums[i]) || dfs(i+1, sum) {
            memo[Info{i, sum}] = 2
            return true
        }
        memo[Info{i, sum}] = 1
        return false
    }
    return dfs(0, 0)
}
```

### Code Efficiency

- **Time Complexity**: $O(n \times S)$
    - Where $n$ is the number of elements in `nums` and $S$ is the `total` sum. There are at most $n \times (S/2)$ unique states. Each state is calculated once.
- **Space Complexity**: $O(n \times S)$
    - The auxiliary space includes the `memo` map (storing up to $n \times (S/2)$ states) and the recursion stack, which can go up to $n$ levels deep.

---

## Solution 2: Bottom-Up Dynamic Programming (Iterative 1D DP)

This problem can also be modeled as the classic **0-1 Knapsack Problem**, where we seek a subset of items (`nums`) that sums up to exactly `target = total / 2`.

### Thought Process

1.  **State Definition**:
    *   Define a 1D boolean array `dp` of size `target + 1`.
    *   `dp[sum]` will be `true` if we can form a subset whose elements sum up to exactly `sum`, and `false` otherwise.
    *   **Base Case**: `dp[0] = true` (an empty subset has a sum of $0$).
2.  **DP Transitions (Backwards Loop)**:
    *   For each element `num` in `nums`:
        *   Iterate `sum` backwards from `target` down to `num`:
            *   `dp[sum] = dp[sum] || dp[sum - num]`
        *   *Note: We must iterate the capacity `sum` backwards to prevent using the same element `num` multiple times (which would correspond to the Unbounded Knapsack problem instead of 0-1 Knapsack).*
3.  **Result**:
    *   `dp[target]` indicates whether we can form a subset sum equal to exactly `total / 2`.

### Go Code

``` go
func canPartition(nums []int) bool {
    total := 0
    for _, num := range nums {
        total += num
    }
    if total%2 != 0 {
        return false
    }
    target := total/2
    dp := make([]bool, target+1)
    dp[0] = true
    for _, num := range nums {
        for sum := target; sum >= num; sum-- {
            dp[sum] = dp[sum] || dp[sum-num]
        }
    }
    return dp[target]
}
```

### Code Efficiency

- **Time Complexity**: $O(n \times S)$
    - The outer loop runs $n$ times, and the inner loop runs at most `target` ($S / 2$) times.
- **Space Complexity**: $O(S)$
    - Requires a single `dp` array of size $S/2 + 1$, with no recursion stack space overhead.