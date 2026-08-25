# 309. Best Time to Buy and Sell Stock with Cooldown

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/description/)

You are given an array `prices` where `prices[i]` is the price of a given stock on the $i$-th day.

Find the maximum profit you can achieve. You may complete as many transactions as you like (i.e., buy one and sell one share of the stock multiple times) with the following constraints:
*   After you sell your stock, you cannot buy stock on the next day (i.e., cooldown one day).

**Note**: You may not engage in multiple transactions simultaneously (i.e., you must sell the stock before you buy again).

---

## Solution 1: Top-Down DFS with Memoization

We can view the problem as a sequence of decisions made on each day. On any given day, we decide whether to buy, sell, or cooldown (do nothing).

### Thought Process

1.  **DFS Parameters**:
    *   `i`: The current day index.
    *   `isBuy`: A boolean indicating whether we are allowed to buy stock (`true`) or sell stock (`false`).
2.  **Transitions**:
    *   **Cooldown Choice**: On any day, we can choose to do nothing. This transitions to the next day with the same buy/sell state:
        `cooldown = dfs(i + 1, isBuy)`
    *   **Transaction Choice**:
        *   If `isBuy` is `true` (we are buying): We pay `prices[i]`, and the state switches to selling:
            `profit = dfs(i + 1, false) - prices[i]`
        *   If `isBuy` is `false` (we are selling): We receive `prices[i]`, and because of the 1-day cooldown, the next day we can buy is `i + 2`:
            `profit = dfs(i + 2, true) + prices[i]`
    *   The result for the current state is `max(profit, cooldown)`.
3.  **Base Case**:
    *   If `i >= len(prices)`: No more transactions can be made; return `0`.
4.  **Memoization**:
    *   Create a struct `State{index, isBuy}` to act as keys in our `memo` map to avoid recalculating the same subproblem states.

### Go Code

``` go
type State struct {
    index   int
    isBuy   bool
}

func maxProfit(prices []int) int {
    memo := map[State]int{}
    
    var dfs func(i int, isBuy bool) int
    dfs = func(i int, isBuy bool) int {
        // Base case: out of days
        if i >= len(prices) {
            return 0
        }
        
        state := State{i, isBuy}
        if val, ok := memo[state]; ok {
            return val
        }
        
        // Option 1: Cooldown (do nothing)
        cooldown := dfs(i+1, isBuy)
        
        // Option 2: Buy or Sell
        var profit int
        if isBuy {
            profit = dfs(i+1, false) - prices[i]            
        } else {
            profit = dfs(i+2, true) + prices[i]
        }
        
        memo[state] = max(profit, cooldown)
        return memo[state]
    }
    
    return dfs(0, true)
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Where $N$ is the number of days. There are $N$ choices for `index` and 2 choices for `isBuy`, giving at most $2N$ states. Each state is calculated in $O(1)$ time.
- **Space Complexity**: $O(N)$
    - We use $O(N)$ space to store the states in the `memo` map and $O(N)$ space for the recursion call stack.

---

## Solution 2: Space-Optimized State Machine DP

We can optimize the space complexity to $O(1)$ using a State Machine DP approach. On each day, we can be in one of three states:
1.  `hold`: We currently hold a stock.
2.  `sold`: We just sold a stock on this day (forces cooldown on the next day).
3.  `reset`: We are in cooldown or idle (not holding and did not sell on this day).

### Thought Process

1.  **State Transitions**:
    *   `hold = max(hold, reset - price)`: We either continue to hold the stock, or buy a new stock after being in a `reset`/cooldown state.
    *   `sold = hold + price`: We sell the stock we currently hold.
    *   `reset = max(reset, sold)`: We either remain idle or enter the reset state after selling stock on the previous day.
2.  **Base Cases**:
    *   Initialize `hold = -infinity` (since we cannot hold a stock before buying).
    *   Initialize `sold = 0` and `reset = 0`.
3.  **Result**:
    *   Return `max(sold, reset)` at the end of the simulation (since it is never optimal to end the process holding a stock).

### Go Code

``` go
import "math"

func maxProfit(prices []int) int {
    if len(prices) == 0 {
        return 0
    }
    
    hold := math.MinInt32
    sold := 0
    reset := 0
    
    for _, price := range prices {
        prevSold := sold
        
        // Transitions
        sold = hold + price
        hold = max(hold, reset - price)
        reset = max(reset, prevSold)
    }
    
    return max(sold, reset)
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - We iterate through the `prices` array exactly once.
- **Space Complexity**: $O(1)$
    - We only use constant auxiliary variables, achieving optimal space efficiency.