# 322. Coin Change

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/coin-change/description/)

## Solution: Bottom-Up Dynamic Programming

To make up `amount` with the minimum number of coins, we can build the solution iteratively from `0` to `amount`. 

### How to Recognize This is a DP Problem

1.  **Optimization Goal**: The problem asks for the "fewest" (minimum) number of coins. Questions asking for "min/max", "longest/shortest", or "total ways" are classic candidates for DP.
2.  **Greedy Fails**: A greedy approach (taking the largest coin first) does not work here. For example, if `coins = [1, 3, 4]` and `amount = 6`:
    *   Greedy: `4 + 1 + 1` (3 coins)
    *   Optimal: `3 + 3` (2 coins)
    *   Because local choices don't lead to the global optimum, we must check all choices.
3.  **Overlapping Subproblems**: To find the optimal way to make `6`, we can try taking each coin and recursively find the optimal way to make the remaining amounts (`5`, `3`, `2`). Finding these remainders will repeatedly recalculate the same smaller amounts.
4.  **Optimal Substructure**: The optimal solution for a larger amount `i` is constructed using optimal solutions of smaller amounts: `dp[i] = min(dp[i], dp[i - coin] + 1)`.

### Thought Process

1.  **Initialize DP Table**: Create a `dp` array of size `amount + 1` filled with a sentinel value (`MaxInt32`). Set `dp[0] = 0` (0 coins are needed to make amount 0).
2.  **Iterate Coins**: For each coin, iterate through all amounts `i` from `coin` up to `amount`.
3.  **Transition Rule**: Update the minimum coins needed: `dp[i] = min(dp[i], dp[i - coin] + 1)`.
4.  **Result**: If `dp[amount]` remains `MaxInt32`, it is impossible to form the amount; return `-1`. Otherwise, return `dp[amount]`.

### Go Code

``` go
import "math"

func coinChange(coins []int, amount int) int {
    dp := make([]int, amount+1)
    for i := 1; i <= amount; i++ {
        dp[i] = math.MaxInt32
    }
    
    // Outer loop through coins for Knapsack-style bottom-up DP
    for _, coin := range coins {
        for i := coin; i <= amount; i++ {
            if dp[i-coin] != math.MaxInt32 {
                dp[i] = min(dp[i], dp[i-coin]+1)
            }
        }
    }
    
    if dp[amount] == math.MaxInt32 {
        return -1
    }
    return dp[amount]
}
```

### Code Efficiency

- **Time Complexity**: $O(n \cdot a)$
    - Where $n$ is the number of coins and $a$ is the target amount.
- **Space Complexity**: $O(a)$
    - We use a 1D `dp` array of size `amount + 1`.

---

## Alternative Solution: Top-Down DP (DFS + Memoization)

Instead of building bottom-up, we can think recursively (DFS) and save calculated remainder results in a memoization map to avoid redundant calculations.

### Thought Process

1.  **Recursive DFS**: To find the minimum coins for a remainder `rem`:
    *   Try subtracting each coin: `dfs(rem - coin)`.
    *   The result for `rem` is `1 + min(all child dfs results)`.
2.  **Base Cases**:
    *   `rem == 0`: Return `0` (0 coins needed).
    *   `rem < 0`: Return `-1` (invalid path).
3.  **Memoization**: Before computing, check if `rem` is in the `memo` map. If so, return it immediately to prevent exponential DFS branching.

### Go Code

``` go
import "math"

func coinChange(coins []int, amount int) int {
    memo := make(map[int]int)
    
    var dfs func(rem int) int
    dfs = func(rem int) int {
        if rem < 0 {
            return -1
        }
        if rem == 0 {
            return 0
        }
        if val, exists := memo[rem]; exists {
            return val
        }
        
        minCoins := math.MaxInt32
        for _, coin := range coins {
            res := dfs(rem - coin)
            if res >= 0 && res < minCoins {
                minCoins = res + 1
            }
        }
        
        if minCoins == math.MaxInt32 {
            memo[rem] = -1
        } else {
            memo[rem] = minCoins
        }
        return memo[rem]
    }
    
    return dfs(amount)
}
```

### Code Efficiency

- **Time Complexity**: $O(n \cdot a)$
    - There are $a$ unique subproblems, and for each, we iterate through $n$ coins.
- **Space Complexity**: $O(a)$
    - The memo map and recursion stack take up to $O(a)$ space.
