# 1049. Last Stone Weight II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/last-stone-weight-ii/description/)

This problem can be mathematically modeled as partitioning the stones into two groups, $A$ and $B$, such that the absolute difference between their sums, $|sum(A) - sum(B)|$, is minimized. This reduces the problem to a variation of the **0/1 Knapsack Problem / Subset Sum Problem**.

Let $S$ be the total sum of all stones. We want to find a subset of stones whose sum is as close to $S/2$ as possible. If the sum of this subset is $S'$, then the remaining stones will sum up to $S - S'$, and the resulting minimum weight of the last stone is:
$$(S - S') - S' = S - 2S'$$

---

## Solution 1: Top-Down Dynamic Programming (DFS with Memoization)

We can use recursive DFS to explore whether to add or skip each stone, caching the results of unique states.

### Thought Process

1.  **Search State**:
    *   The state in our recursion is defined by the current stone index `i` and the running sum `curr` of the chosen subset.
2.  **Decisions**:
    *   **Skip**: Do not include the current stone: `dfs(i+1, curr)`.
    *   **Take**: Include the current stone: `dfs(i+1, curr + stones[i])`.
    *   We want to minimize the final difference, so we take the minimum of both choices.
3.  **Base Case**:
    *   If `curr >= target` (where `target = sum / 2`) or `i == len(stones)`, we calculate the difference between the two subset sums and return it:
        $$\text{difference} = |curr - (sum - curr)|$$
4.  **Memoization**:
    *   We cache the evaluated states using a `memo := make(map[State]int)` map, where `State` wraps `{idx, sum}`.

### Go Code

``` go
type State struct {
    idx int
    sum int
}

func lastStoneWeightII(stones []int) int {
    sum := 0
    for _, stone := range stones {
        sum += stone
    }
    target := sum/2
    memo := make(map[State]int)

    var dfs func(i int, curr int) int
    dfs = func(i int, curr int) int {
        if curr >= target || i == len(stones) {
            return abs(curr - (sum-curr)) 
        }
        state := State{i, curr}
        if val, ok := memo[state]; ok {
            return val
        }
        memo[state] = min(dfs(i+1, curr), dfs(i+1, curr+stones[i]))
        return memo[state]
    }
    return dfs(0, 0)
}

func abs(a int) int {
    if a < 0 {
        return -a
    }
    return a
}
```

### Code Efficiency

- **Time Complexity**: $O(N \cdot S)$
    - Where $N$ is the number of stones and $S$ is the total sum of all stones. The number of unique states in the recursion tree is bounded by $N \cdot S/2$.
- **Space Complexity**: $O(N \cdot S)$
    - The space required for the `memo` map containing the unique states, plus the recursion call stack depth of $O(N)$.

---

## Solution 2: Bottom-Up Dynamic Programming (1D DP)

To avoid recursion overhead and optimize space, we can solve this iteratively as a classic subset-sum knapsack problem.

### Thought Process

1.  **State Definition**:
    *   Define `dp[w]` as the maximum subset sum we can form that is less than or equal to `w`.
2.  **Initialization**:
    *   Initialize a 1D slice `dp` of size `target + 1` (where `target = sum / 2`).
3.  **State Transition**:
    *   For each `stone` in `stones`:
        *   To ensure each stone is used at most once, we iterate backwards through the `dp` slice from `target` down to `stone`:
            $$dp[w] = \max(dp[w], dp[w-\text{stone}] + \text{stone})$$
4.  **Result**:
    *   `dp[target]` will contain the maximum subset sum closest to $S/2$ (let this be $S'$).
    *   The final answer is:
        $$\text{minDiff} = S - 2 \cdot dp[target]$$

### Go Code

``` go
func lastStoneWeightII(stones []int) int {
    sum := 0
    for _, stone := range stones {
        sum += stone
    }
    target := sum/2
    dp := make([]int, target+1)
    for _, stone := range stones {
        for i := target; i >= stone; i-- {
            dp[i] = max(dp[i], dp[i-stone]+stone)
        }
    }
    return abs(dp[target] - (sum - dp[target]))
}

func abs(a int) int {
    if a < 0 {
        return -a
    }
    return a
}
```

### Code Efficiency

- **Time Complexity**: $O(N \cdot S)$
    - The outer loop runs $N$ times (for each stone), and the inner loop runs $S/2$ times.
- **Space Complexity**: $O(S)$
    - We only require a 1D slice of size $S/2 + 1$, optimizing space complexity from $O(N \cdot S)$ to $O(S)$.