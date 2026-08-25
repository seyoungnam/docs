# 746. Min Cost Climbing Stairs

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/min-cost-climbing-stairs/description/)

You are given an integer array `cost` where `cost[i]` is the cost of $i$-th step on a staircase. Once you pay the cost, you can either climb one or two steps.

You can either start from the step with index `0`, or the step with index `1`.

Return *the minimum cost to reach the top of the floor*.

---

## Solution 1: Bottom-Up DP (with Memo Array)

We can solve this problem using bottom-up 1D Dynamic Programming. By iterating backwards from the top of the staircase, we can determine the minimum cost to reach the top from each step.

### Thought Process

1.  **DP Array Setup**:
    *   Let `minCost[i]` represent the minimum cost to reach the top of the staircase starting from step `i`.
    *   Create a slice `minCost` of size `n + 1` (where `n` is the length of `cost`).
2.  **Base Case**:
    *   `minCost[n] = 0` (cost to reach the top from the top is 0).
3.  **State Transition**:
    *   Iterate backwards from $i = n - 1$ down to $0$.
    *   From step `i`, we pay `cost[i]` and can choose to climb either 1 step to `i+1` or 2 steps to `i+2` (if `i+2 <= n`).
    *   Thus, the recurrence relation is:
        $$\text{minCost}[i] = \text{cost}[i] + \min(\text{minCost}[i+1], \text{minCost}[i+2])$$
4.  **Result**:
    *   Since we can start from either index `0` or index `1`, we return the minimum of the two:
        $$\min(\text{minCost}[0], \text{minCost}[1])$$

### Go Code

``` go
import "math"

func minCostClimbingStairs(cost []int) int {
    n := len(cost)
    minCost := make([]int, n+1)
    for i := range minCost {
        minCost[i] = math.MaxInt32
    }
    minCost[n] = 0
    
    for i := n-1; i >= 0; i-- {
        minCost[i] = min(minCost[i], cost[i]+minCost[i+1])
        if i+2 <= n {
            minCost[i] = min(minCost[i], cost[i]+minCost[i+2])
        }
    }
    return min(minCost[0], minCost[1])
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - We compute the minimum cost for each step exactly once in a single linear pass from $n-1$ down to $0$.
- **Space Complexity**: $O(N)$
    - We allocate a `minCost` slice of size $N + 1$ to store the DP states.

---

## Solution 2: Space-Optimized DP (In-Place)

We can optimize the space complexity to $O(1)$ by using the input `cost` array to store the DP solutions in-place.

### Thought Process

1.  **Optimization**:
    *   Instead of allocating a new slice, we modify the `cost` array directly.
2.  **State Transition**:
    *   The last two steps, `cost[n-1]` and `cost[n-2]`, do not need any modifications because we can jump directly to the top from them.
    *   We iterate backwards starting from step $i = n - 3$ down to $0$:
        *   The minimum cost to reach the top from step `i` is the cost at step `i` plus the minimum cost of the next two steps:
            $$\text{cost}[i] += \min(\text{cost}[i+1], \text{cost}[i+2])$$
3.  **Result**:
    *   Return the minimum cost to start from either step `0` or step `1`: `min(cost[0], cost[1])`.

### Go Code

``` go
func minCostClimbingStairs(cost []int) int {
    n := len(cost)
    for i := n - 3; i >= 0; i-- {
        cost[i] += min(cost[i+1], cost[i+2])
    }
    return min(cost[0], cost[1])
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - We iterate backwards through the `cost` array from $n-3$ down to $0$ exactly once.
- **Space Complexity**: $O(1)$
    - We modify the input array in-place, requiring no extra auxiliary memory.
