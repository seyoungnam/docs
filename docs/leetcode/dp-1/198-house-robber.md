# 198. House Robber

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/house-robber/description/)

You are a professional robber planning to rob houses along a street. Each house has a certain amount of money stashed, the only constraint stopping you from robbing each of them is that adjacent houses have security systems connected and **it will automatically contact the police if two adjacent houses were broken into on the same night**.

Given an integer array `nums` representing the amount of money of each house, return *the maximum amount of money you can rob tonight **without alerting the police***.

---

## Solution 1: Top-Down DFS with Memoization

We can view the choice at each house as a decision to either rob it or skip it. To optimize the search, we memoize the results of the subproblems.

### Thought Process

1.  **State Definition**:
    *   Let `dfs(i)` represent the maximum amount of money we can rob starting from house `i` to the end of the street.
2.  **Recursive Choices**:
    *   At house `i`, we can either:
        *   **Skip it**: Move to the next house. The value is `dfs(i + 1)`.
        *   **Rob it**: Add its money `nums[i]` and jump to `i + 2` (skipping the adjacent house). The value is `nums[i] + dfs(i + 2)`.
    *   The recurrence relation is:
        $$\text{dfs}(i) = \max(\text{dfs}(i+1), \text{nums}[i] + \text{dfs}(i+2))$$
3.  **Base Case**:
    *   If `i >= len(nums)`, we have gone past the last house. Return `0`.
4.  **Memoization**:
    *   Use a slice `memo` initialized to `-1` to cache computed values.

### Go Code

``` go
func rob(nums []int) int {
    memo := make([]int, len(nums))
    for i := range memo {
        memo[i] = -1
    }
    
    var dfs func(i int) int
    dfs = func(i int) int {
        if i >= len(nums) {
            return 0
        }
        if memo[i] != -1 {
            return memo[i]
        }
        
        memo[i] = max(dfs(i+1), nums[i] + dfs(i+2))
        return memo[i]
    }
    
    return dfs(0)
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Where $N$ is the number of houses. We compute `dfs(i)` for each house index at most once due to memoization.
- **Space Complexity**: $O(N)$
    - We use $O(N)$ space for the memoization slice and $O(N)$ space for the recursion stack.

---

## Solution 2: Bottom-Up DP (with Memo Array)

We can formulate the problem bottom-up by defining a state array where each index stores the maximum money robbable up to that point.

### Thought Process

1.  **DP Array Setup**:
    *   Let `dp[i]` represent the maximum money we can rob from houses `0` through `i`.
2.  **Base Cases**:
    *   `dp[0] = nums[0]`
    *   `dp[1] = max(nums[0], nums[1])` (since we cannot rob both, we pick the larger one).
3.  **State Transition**:
    *   For each house `i` from `2` to `n-1`:
        *   If we skip house `i`, the maximum money is `dp[i-1]`.
        *   If we rob house `i`, the maximum money is `nums[i] + dp[i-2]`.
        *   The transition is:
            $$\text{dp}[i] = \max(\text{dp}[i-1], \text{nums}[i] + \text{dp}[i-2])$$
4.  **Result**:
    *   Return `dp[n-1]`.

### Go Code

``` go
func rob(nums []int) int {
    n := len(nums)
    if n == 0 {
        return 0
    }
    if n == 1 {
        return nums[0]
    }
    
    dp := make([]int, n)
    dp[0] = nums[0]
    dp[1] = max(nums[0], nums[1])

    for i := 2; i < n; i++ {
        dp[i] = max(dp[i-1], nums[i] + dp[i-2])
    }
    return dp[n-1]
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - We loop from step $2$ to $N-1$ exactly once.
- **Space Complexity**: $O(N)$
    - We allocate a `dp` slice of size $N$ to store the subproblem solutions.

---

## Solution 3: Space-Optimized DP

Notice that to compute the maximum value at house `i`, we only need the results of the two immediately preceding states (`dp[i-1]` and `dp[i-2]`). We can optimize the space to $O(1)$ by tracking these two states in variables.

### Thought Process

1.  **Optimization**:
    *   Let `rob1` represent `dp[i-2]` and `rob2` represent `dp[i-1]`.
2.  **Transition**:
    *   For each `num` in `nums`:
        *   The new maximum if we include `num` is `next = max(num + rob1, rob2)`.
        *   Update the state pointers: `rob1 = rob2`, `rob2 = next`.
3.  **Result**:
    *   Return `rob2`, which stores the final answer.

### Go Code

``` go
func rob(nums []int) int {
    rob1, rob2 := 0, 0
    for _, num := range nums {
        next := max(num + rob1, rob2)
        rob1, rob2 = rob2, next
    }
    return rob2
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - We iterate through the `nums` slice exactly once.
- **Space Complexity**: $O(1)$
    - We only use constant auxiliary variables, achieving optimal space efficiency.