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

## Solution 3: Bottom-Up Dynamic Programming (2D Map-Based DP)

We can solve this problem iteratively without recursion overhead by computing the number of ways to form each possible sum at each step.

### Thought Process

1.  **State Definition**:
    *   Define `dp[i][sum]` as the number of ways to reach a running total of `sum` using the first `i` elements from the `nums` array.
    *   To handle dynamic ranges of sums, we represent this as a slice of maps in Go: `dp := make([]map[int]int, n+1)`.
2.  **Base Case**:
    *   `dp[0][0] = 1`: There is exactly one way to form a sum of 0 using zero elements (the empty set).
3.  **State Transition**:
    *   For each element `nums[i]` (from `i = 0` to `n-1`):
        *   For each existing `sum` and its frequency `cnt` that can be formed using the first `i` elements (`dp[i]`):
            *   **Option 1 (Subtract)**: We can subtract `nums[i]` to get a new sum `sum - nums[i]`. Increment its count:
                `dp[i+1][sum-nums[i]] += cnt`
            *   **Option 2 (Add)**: We can add `nums[i]` to get a new sum `sum + nums[i]`. Increment its count:
                `dp[i+1][sum+nums[i]] += cnt`
4.  **Result**:
    *   The final answer is the number of ways to reach the `target` sum using all `n` elements: `dp[n][target]`. If `target` is not reachable, the map lookup returns `0` by default.

### Go Code

``` go
func findTargetSumWays(nums []int, target int) int {
    n := len(nums)
    dp := make([]map[int]int, n+1)
    for i := 0; i <= n; i++ {
        dp[i] = make(map[int]int)
    }
    dp[0][0] = 1
    for i := 0; i < n; i++ {
        for sum, cnt := range dp[i] {
            dp[i+1][sum-nums[i]] += cnt
            dp[i+1][sum+nums[i]] += cnt
        }
    }
    return dp[n][target]
}
```

### Code Efficiency

- **Time Complexity**: $O(N \cdot S)$
    - Where $N$ is the number of elements in `nums` and $S$ is the number of unique sums we can form. At each step `i`, we iterate over all unique running sums from the previous step. Since $S \le 2 \cdot \text{totalSum}$, the total time is bounded by $O(N \cdot S)$.
- **Space Complexity**: $O(N \cdot S)$
    - We store a slice of maps of length $N+1$, where each map contains up to $S$ unique running sums, requiring $O(N \cdot S)$ auxiliary space. This bottom-up approach avoids recursion stack space overhead.