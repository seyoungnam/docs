# 518. Coin Change II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/coin-change-ii/description/)

This problem is a classic **Unbounded Knapsack Problem** where each coin denomination can be chosen an unlimited number of times. We need to find the total number of distinct **combinations** (where order does not matter) that sum up to `amount`.

---

## Solution 1: Top-Down Dynamic Programming (DFS with Memoization)

We can explore the decision tree by either taking the current coin or skipping to the next coin denomination, using memoization to avoid redundant branch evaluations.

### Thought Process

1.  **State Definition**:
    *   Let `dfs(i, curr)` represent the number of ways to reach `amount` starting from coin index `i` with current sum `curr`.
2.  **Decisions at Index `i`**:
    *   **Option 1 (Take the coin)**: Add `coins[i]` to `curr`. Because we have an infinite supply of each coin, we remain at index `i`:
        $$\text{take} = dfs(i, curr + coins[i])$$
    *   **Option 2 (Skip the coin)**: Move to the next coin denomination without adding to `curr`:
        $$\text{skip} = dfs(i+1, curr)$$
    *   The total combinations from this state is $\text{take} + \text{skip}$.
3.  **Base Cases**:
    *   If `curr == amount`: A valid combination is completed; return `1`.
    *   If `curr > amount` or `i == len(coins)`: Exceeded amount or no more coins available; return `0`.
4.  **Memoization**:
    *   Cache computed states in a `memo` map where keys are of type `State{idx, sum}`.

### Go Code

``` go
type State struct {
    idx int
    sum int
}

func change(amount int, coins []int) int {
    memo := make(map[State]int)
    
    var dfs func(i int, curr int) int
    dfs = func(i int, curr int) int {
        if curr == amount {
            return 1
        }
        if curr > amount || i == len(coins) {
            return 0
        }
        state := State{i, curr}
        if val, ok := memo[state]; ok {
            return val
        }
        memo[state] = dfs(i, curr+coins[i]) + dfs(i+1, curr)
        return memo[state]
    }
    return dfs(0, 0)
}
```

### Code Efficiency

- **Time Complexity**: $O(N \cdot \text{amount})$
    - Where $N$ is the number of coins. There are at most $N \cdot \text{amount}$ unique `(index, sum)` states evaluated.
- **Space Complexity**: $O(N \cdot \text{amount})$
    - The space required for the `memo` map containing unique states, plus the recursion call stack depth of $O(\text{amount})$.

---

## Solution 2: Bottom-Up Dynamic Programming (1D DP)

To optimize space and eliminate recursion stack overhead, we can solve this iteratively using a 1D DP table.

### Thought Process

1.  **State Definition**:
    *   Define `dp[i]` as the number of combinations that sum up to amount `i`.
2.  **Base Case**:
    *   `dp[0] = 1`: There is exactly 1 way to make an amount of 0 (by choosing no coins).
3.  **Combinations vs. Permutations (Loop Order)**:
    *   To count **combinations** (e.g., `[1, 2]` is identical to `[2, 1]`), we iterate through the coins in the **outer loop** and amounts in the **inner loop**. This guarantees that coins are processed in a fixed order.
    *   Since coins can be reused indefinitely, we update `i` in **forward order** (from `coin` up to `amount`):
        $$dp[i] = dp[i] + dp[i - coin]$$
4.  **Result**:
    *   The total number of valid combinations will be stored in `dp[amount]`.

### Go Code

``` go
func change(amount int, coins []int) int {
    dp := make([]int, amount+1)
    dp[0] = 1
    for _, coin := range coins {
        for i := coin; i <= amount; i++ {
            dp[i] += dp[i-coin] 
        }
    }
    return dp[amount]
}
```

### Code Efficiency

- **Time Complexity**: $O(N \cdot \text{amount})$
    - The outer loop runs $N$ times (for each coin) and the inner loop runs $\text{amount} - \text{coin} + 1$ times.
- **Space Complexity**: $O(\text{amount})$
    - We only need a 1D slice of size $\text{amount} + 1$, optimizing space complexity from $O(N \cdot \text{amount})$ to $O(\text{amount})$.