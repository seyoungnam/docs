# 213. House Robber II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/house-robber-ii/description/)

You are a professional robber planning to rob houses along a street. Each house has a certain amount of money stashed. All houses at this place are **arranged in a circle**. That means the first house is the neighbor of the last one. Meanwhile, adjacent houses have a security system connected, and **it will automatically contact the police if two adjacent houses were broken into on the same night**.

Given an integer array `nums` representing the amount of money of each house, return *the maximum amount of money you can rob tonight **without alerting the police***.

---

## Solution 1: Space-Optimized DP (Circular Array Splitting)

Since the houses are arranged in a circle, the first and last houses are adjacent. This means we cannot rob both house `0` and house `n-1` on the same night.

### Thought Process

We can break the circular dependency by splitting the problem into two separate linear subproblems (which are identical to [198. House Robber](198-house-robber.md)):

1.  **Case 1**: Rob houses from index `0` to `n-2` (excluding the last house).
2.  **Case 2**: Rob houses from index `1` to `n-1` (excluding the first house).

The overall result will be the maximum of these two cases.

*   **Base Case**: If `n == 1`, return `nums[0]`.
*   **Linear Helper**: Implement the space-optimized linear House Robber solution using two variables `rob1` and `rob2` to track previous states.

### Go Code

``` go
func rob(nums []int) int {
    n := len(nums)
    if n == 1 {
        return nums[0]
    }
    
    // The result is the maximum of excluding the last house vs excluding the first house
    return max(helper(nums[1:]), helper(nums[:n-1]))
}

// Same as 198. House Robber (Linear version)
func helper(nums []int) int {
    rob1, rob2 := 0, 0
    for _, num := range nums {
        currMax := max(num+rob1, rob2)
        rob1, rob2 = rob2, currMax
    }
    return rob2
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Where $N$ is the number of houses. We run the linear helper function twice on subarrays of size $N-1$, yielding $O(N)$ total operations.
- **Space Complexity**: $O(1)$
    - Slicing in Go creates a view of the original slice without copying the elements. Thus, we only use constant auxiliary variables, achieving $O(1)$ space efficiency.

---

## Solution 2: Top-Down DFS with Memoization & Flag

Alternatively, we can solve the circular constraint by passing a status flag through our recursive DFS function.

### Thought Process

1.  **State Definition**:
    *   Let `dfs(i, flag)` represent the maximum amount of money we can rob starting from index `i` given the first-house status `flag`.
    *   `flag = 1`: We robbed the first house (`0`), meaning we cannot rob the last house (`n-1`).
    *   `flag = 0`: We did not rob the first house (`0`), so the last house remains eligible to be robbed.
2.  **Recursive Choices**:
    *   At house `i`, we can either:
        *   **Skip it**: Recurse to `dfs(i + 1, flag)`.
        *   **Rob it**: Add `nums[i]` and jump to `i + 2`. If `i == 0`, we rob the first house, so we set `nextFlag = 1`. Otherwise, `nextFlag = flag`.
3.  **Base Cases**:
    *   If `i >= n`: We have gone past the last house. Return `0`.
    *   If `flag == 1 && i == n - 1`: We are at the last house, but since we robbed the first house (`flag == 1`), we cannot rob this last house. Return `0`.
4.  **Memoization**:
    *   Use a 2D slice `memo` of size $N \times 2$ initialized to `-1` to cache states based on index `i` and `flag`.

### Go Code

``` go
func rob(nums []int) int {
    n := len(nums)
    if n == 1 {
        return nums[0]
    }

    // memo[i][flag] stores max robbable starting at index i
    memo := make([][2]int, n)
    for i := range memo {
        memo[i][0] = -1
        memo[i][1] = -1 
    }
    
    var dfs func(i int, flag int) int
    dfs = func(i int, flag int) int {
        // Base case: out of bounds or trying to rob the last house when first was robbed
        if i >= n || (flag == 1 && i == n-1) {
            return 0
        }
        if memo[i][flag] != -1 {
            return memo[i][flag]
        }
        
        // If we rob house 0, set flag = 1 for subsequent recursion calls
        nextFlag := flag
        if i == 0 {
            nextFlag = 1
        }
        
        memo[i][flag] = max(dfs(i+1, flag), nums[i] + dfs(i+2, nextFlag))
        return memo[i][flag]
    }
    
    return dfs(0, 0)
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Since index `i` ranges from $0$ to $N-1$ and `flag` can only be $0$ or $1$, the total number of unique states is $2N$. Each state is computed exactly once in $O(1)$ time.
- **Space Complexity**: $O(N)$
    - We use $O(N)$ space for the $N \times 2$ memoization table and $O(N)$ space for the recursion call stack.