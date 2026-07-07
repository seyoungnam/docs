# 377. Combination Sum IV

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/combination-sum-iv/description/)

## Solution 1: Top-Down Dynamic Programming (Memoized DFS)

We can solve this problem using depth-first search (DFS) by recursively choosing numbers from `nums` to reduce the remaining target. To avoid Time Limit Exceeded (TLE) due to overlapping subproblems, we memoize the results using a 1D array.

### Thought Process

1.  **State Transition**:
    *   Let $dfs(\text{remain})$ be the number of combinations that sum up to $\text{remain}$.
    *   To calculate $dfs(\text{remain})$, we can transition by trying to subtract each number $x \in \text{nums}$ from $\text{remain}$:
        $$dfs(\text{remain}) = \sum_{x \in \text{nums}} dfs(\text{remain} - x)$$
2.  **Base Cases**:
    *   If $\text{remain} == 0$, we have successfully found exactly one combination path: return $1$.
    *   If $\text{remain} < 0$, the sum has exceeded the target: return $0$.
3.  **Memoization**:
    *   Initialize a 1D array `dp` of size `target + 1` with `-1`.
    *   If `dp[remain]` has already been computed (`dp[remain] != -1`), we return it immediately. This optimizes the search space by solving each subproblem only once.

### Go Code

``` go
func combinationSum4(nums []int, target int) int {
    dp := make([]int, target+1)
    for i := range dp {
        dp[i] = -1
    }

    var dfs func(remain int) int
    dfs = func(remain int) int {
        if remain == 0 {
            return 1
        }
        if remain < 0 {
            return 0
        }
        if dp[remain] != -1 {
            return dp[remain]
        }
        count := 0
        for _, curr := range nums {
            count += dfs(remain-curr)
        }
        dp[remain] = count
        return dp[remain]
    }
    return dfs(target)
}
```

### Code Efficiency

- **Time Complexity**: $O(T \cdot N)$
    - Where $T$ is the `target` value and $N$ is the number of elements in `nums`. There are $T + 1$ unique subproblems. For each subproblem, we iterate through all $N$ numbers in `nums`.
- **Space Complexity**: $O(T)$
    - The auxiliary space includes the `dp` memoization table of size $T + 1$ and the recursion stack, which can go up to $T$ levels deep.

---

## Solution 2: Bottom-Up Dynamic Programming (Iterative 1D DP)

Instead of recursion, we can solve the subproblems iteratively starting from the smallest possible target sum `0` up to `target`.

### Thought Process

1.  **State Definition**:
    *   Define `dp[sum]` as the number of combinations that sum up to `sum`.
2.  **Base Case**:
    *   `dp[0] = 1`. There is exactly one way to form a sum of 0: choosing an empty combination.
3.  **DP Transition**:
    *   For each value `sum` from $1$ to `target`:
        *   Iterate through each candidate `curr` in `nums`.
        *   If `sum - curr >= 0`, we add the number of combinations from the remaining sum to our current sum state:
            $$dp[\text{sum}] = dp[\text{sum}] + dp[\text{sum} - \text{curr}]$$
4.  **Result**:
    *   `dp[target]` will hold the total number of combinations.

### Go Code

``` go
func combinationSum4(nums []int, target int) int {
    dp := make([]int, target+1)
    dp[0] = 1

    for sum := 1; sum <= target; sum++ {
        for _, curr := range nums {
            if sum-curr >= 0 {
                dp[sum] += dp[sum-curr]
            }
        }
    }
    return dp[target]
}
```

### Code Efficiency

- **Time Complexity**: $O(T \cdot N)$
    - Nested loops: the outer loop runs $T$ times (from $1$ to $T$), and the inner loop runs $N$ times (for each candidate in `nums`).
- **Space Complexity**: $O(T)$
    - Requires a single `dp` array of size $T + 1$. This bottom-up approach avoids recursion stack space overhead.