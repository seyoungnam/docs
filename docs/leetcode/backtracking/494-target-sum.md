# 494. Target Sum

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/target-sum/description/)

## Solution 1: Brute-Force Backtracking (Time Limit Exceeded)

A direct way to solve this is to explore all possible configurations by making a binary decision at each index: either assign a positive sign (`+`) or a negative sign (`-`) to the current number.

### Thought Process

1.  **Decision Tree**:
    *   For each element `nums[i]`, we branch in two directions:
        *   **Branch 1 (Add)**: Recursively call `dfs(i+1, sum + nums[i])`.
        *   **Branch 2 (Subtract)**: Recursively call `dfs(i+1, sum - nums[i])`.
2.  **Base Case**:
    *   When the index `i` reaches `len(nums)`, we have assigned signs to all numbers. If the running `sum` matches `target`, we increment our global `count` by 1.

### Go Code

``` go
func findTargetSumWays(nums []int, target int) int {
    count := 0

    var dfs func(i int, sum int)
    dfs = func(i int, sum int) {
        if i == len(nums) {
            if sum == target {
                count++
            }
            return
        }
        dfs(i+1, sum+nums[i])
        dfs(i+1, sum-nums[i])
    }
    dfs(0, 0)
    return count
}
```

### Code Efficiency

- **Time Complexity**: $O(2^N)$
    - Since we make 2 recursive calls at each of the $N$ elements, the recursion tree has $2^N$ leaf nodes. This causes TLE for larger values of $N$.
- **Space Complexity**: $O(N)$
    - The auxiliary space is determined by the recursion call stack, which goes up to $N$ levels deep.

---

## Solution 2: Top-Down Dynamic Programming (Memoized DFS)

To avoid redundant computations in overlapping search branches, we cache the results of our subproblems.

### Thought Process

1.  **State Definition**:
    *   Each unique subproblem state in the recursion tree is defined by the tuple of the current index `i` and the running `sum`: `[2]int{i, sum}`.
2.  **Memoization (Cache Map)**:
    *   We use a map `memo := make(map[[2]int]int)` to cache the number of valid paths from state `[2]int{i, sum}`.
    *   If a state has already been evaluated, we return `memo[state]` in $O(1)$ time. This optimizes the time complexity from exponential to polynomial.

### Go Code

``` go
func findTargetSumWays(nums []int, target int) int {
    memo := make(map[[2]int]int)

    var dfs func(i int, sum int) int
    dfs = func(i int, sum int) int {
        if i == len(nums) {
            if sum == target {
                return 1
            }
            return 0
        }

        state := [2]int{i, sum}
        if val, ok := memo[state]; ok {
            return val
        }

        plusPath := dfs(i+1, sum+nums[i])
        minusPath := dfs(i+1, sum-nums[i])

        memo[state] = plusPath + minusPath
        return memo[state]
    }
    return dfs(0, 0)
}
```

### Code Efficiency

- **Time Complexity**: $O(N \cdot S)$
    - Where $N$ is the number of elements in `nums` and $S$ is the sum of all elements in `nums`. The number of unique states is bounded by $2 \cdot N \cdot S$, and each state is computed once.
- **Space Complexity**: $O(N \cdot S)$
    - The space required to store the memoization cache map containing unique states, plus the recursion call stack depth of $O(N)$.

---

## Solution 3: Bottom-Up Dynamic Programming (0/1 Knapsack Reduction)

We can mathematically transform this target sum problem into a classic subset sum (0/1 Knapsack) problem, allowing for an iterative bottom-up solution with $O(1)$ recursion overhead.

### Thought Process

1.  **Mathematical Reduction**:

    *   Let $P$ be the subset of numbers assigned a positive sign, and $N$ be the subset of numbers assigned a negative sign.
    *   We want to satisfy:
        $\sum P - \sum N = \text{target}$
    *   We also know that the total sum of all elements is:
        $\sum P + \sum N = \text{totalSum}$
    *   Adding these two equations gives:
        $2 \cdot \sum P = \text{target} + \text{totalSum}$

        $$\sum P = \frac{\text{target} + \text{totalSum}}{2}$$

    *   Thus, the problem reduces to: **Find the number of subsets $P$ that sum up to exactly $\text{targetSum} = \frac{\text{target} + \text{totalSum}}{2}$**.

2.  **Edge Cases**:

    *   If the absolute value of `target` is greater than `totalSum`, it is impossible to reach the target: return `0`.
    *   If $\text{target} + \text{totalSum}$ is odd, it cannot be divided cleanly by 2 (since subset sums must be integers): return `0`.

3.  **State Transition (1D DP)**:

    *   Define `dp[sum]` as the number of subsets that sum up to `sum`.
    *   Base case: `dp[0] = 1` (exactly 1 way to form a sum of 0, using the empty subset).
    *   For each candidate `num` in `nums`, update `dp[sum]` backwards from `targetSum` down to `num` to prevent using the same element multiple times:
    
        $$dp[\text{sum}] = dp[\text{sum}] + dp[\text{sum} - \text{num}]$$

### Go Code

``` go
func findTargetSumWays(nums []int, target int) int {
    totalSum := 0
    for _, num := range nums {
        totalSum += num
    }

    // Edge Cases:
    // 1. If the target is physically out of bounds of the total sum capacity
    // 2. If (target + totalSum) is odd, it cannot be divided cleanly by 2
    if abs(target) > totalSum || (target + totalSum)%2 != 0 {
        return 0
    }

    targetSum := (target + totalSum) / 2
    dp := make([]int, targetSum+1)
    dp[0] = 1

    for _, num := range nums {
        for sum := targetSum; sum >= num; sum-- {
            dp[sum] += dp[sum-num]
        } 
    }
    return dp[targetSum]
}

func abs(a int) int {
    if a < 0 {
        return -a
    }
    return a
}
```

### Code Efficiency

- **Time Complexity**: $O(N \cdot T_{sum})$
    - Where $N$ is the number of elements in `nums` and $T_{sum}$ is the `targetSum` value. Nested loops: outer loop runs $N$ times, inner loop runs up to $T_{sum}$ times.
- **Space Complexity**: $O(T_{sum})$
    - Requires a single `dp` array of size $T_{sum} + 1$. This bottom-up approach avoids recursion stack space overhead.