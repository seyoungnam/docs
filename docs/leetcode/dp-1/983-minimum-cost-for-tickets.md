# 983. Minimum Cost For Tickets

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/minimum-cost-for-tickets/description/)

This problem asks us to find the minimum cost to cover a list of travel days using three types of tickets: a 1-day pass, a 7-day pass, and a 30-day pass. We can solve this efficiently using **Dynamic Programming**.

---

## Solution 1: Top-Down Dynamic Programming (DFS with Memoization)

We define a recursive search to compute the minimum cost starting from a given travel day index, utilizing a cache to store the results of subproblems.

### Thought Process

1.  **State Definition**:
    *   Let `dfs(i)` represent the minimum cost to cover all travel days starting from index `i` (from `0` to `n-1`).
2.  **Decisions at Index `i`**:
    *   **1-Day Pass**: We buy a 1-day pass at cost `costs[0]`. This covers `days[i]`. The next day we need to purchase a ticket for is `i+1`:
        $$\text{cost}_1 = costs[0] + dfs(i+1)$$
    *   **7-Day Pass**: We buy a 7-day pass at cost `costs[1]`. This covers all travel days in the range $[days[i], days[i] + 6]$. We search forward for the first travel day index `j` that is not covered (i.e., `days[j] >= days[i] + 7`):
        $$\text{cost}_7 = costs[1] + dfs(j)$$
    *   **30-Day Pass**: We buy a 30-day pass at cost `costs[2]`. This covers all travel days in the range $[days[i], days[i] + 29]$. We search forward for the first travel day index `k` that is not covered (i.e., `days[k] >= days[i] + 30`):
        $$\text{cost}_{30} = costs[2] + dfs(k)$$
3.  **Base Case**:
    *   If `i == len(days)`, we have covered all travel days, so we return `0`.
4.  **Memoization**:
    *   We store computed results in a `memo[i]` map to avoid redundant calculations.

### Go Code

``` go
func mincostTickets(days []int, costs []int) int {
    n := len(days)
    memo := make(map[int]int)
    var dfs func(i int) int
    dfs = func(i int) int {
        if i == n {
            return 0
        }
        if val, ok := memo[i]; ok {
            return val
        }
        res := costs[0] + dfs(i+1)
        j := i
        for j < n && days[j] < days[i]+7 {
            j++
        }
        res = min(res, costs[1] + dfs(j))
        k := j
        for k < n && days[k] < days[i]+30 {
            k++
        }
        memo[i] = min(res, costs[2] + dfs(k))
        return memo[i]
    }
    return dfs(0)
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Where $N$ is the number of travel days. There are $N$ unique states. In each state, the forward scans for indices `j` and `k` are bounded and do not repeat work, leading to an overall linear time complexity.
- **Space Complexity**: $O(N)$
    - The space required for the `memo` map containing the unique states, plus the recursion call stack depth of $O(N)$.

---

## Solution 2: Bottom-Up Dynamic Programming (1D DP)

To optimize space and avoid recursion overhead, we can solve the subproblems iteratively in reverse order.

### Thought Process

1.  **State Definition**:
    *   Define `dp[i]` as the minimum cost to cover all travel days starting from travel day index `i`.
2.  **DP Transitions**:
    *   We iterate `i` backwards from `n-1` down to `0`.
    *   For each day `i`, we try all three ticket durations `dur = []int{1, 7, 30}`:
        *   Find the first travel day index `j` that is not covered by the current pass: `days[j] >= days[i] + dur[k]`.
        *   The transition is:
            $$dp[i] = \min_{k} (costs[k] + dp[j])$$
3.  **Result**:
    *   The final minimum cost is stored in `dp[0]`.

### Go Code

``` go
import "math"

func mincostTickets(days []int, costs []int) int {
    n := len(days)
    dp := make([]int, n+1)
    dur := []int{1, 7, 30}

    for i := n-1; i >= 0; i-- {
        dp[i] = math.MaxInt32
        j := i
        for k := 0; k < 3; k++ {
            for j < n && days[j] < days[i]+dur[k] {
                j++
            }
            dp[i] = min(dp[i], costs[k]+dp[j])
        }
    }
    return dp[0]
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - The outer loop runs $N$ times. The inner loops perform linear forward scans, which aggregate to a constant number of steps per day.
- **Space Complexity**: $O(N)$
    - We use a single 1D slice of size $N+1$ to store the subproblem answers, requiring $O(N)$ auxiliary space.