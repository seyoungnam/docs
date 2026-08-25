# 300. Longest Increasing Subsequence

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/longest-increasing-subsequence/description/)

Given an integer array `nums`, return *the length of the longest strictly increasing subsequence*.

---

## Solution 1: Top-Down DFS with Memoization

We can model this problem as a sequence of decisions. At each index, we decide whether to include the current element in our increasing subsequence or skip it.

### Thought Process

1.  **State Definition**:
    *   Let `dfs(i, j)` represent the length of the longest increasing subsequence starting from index `i` given that the last selected element's index is `j`.
2.  **Recursive Choices**:
    *   At index `i`, we always have the choice to **skip** the current element:
        $$\text{LIS} = \text{dfs}(i + 1, j)$$
    *   If `j == -1` (meaning no element has been selected yet) or `nums[j] < nums[i]` (maintaining the increasing order), we can also choose to **include** the current element:
        $$\text{LIS} = \max(\text{LIS}, 1 + \text{dfs}(i + 1, i))$$
3.  **Base Case**:
    *   If `i == len(nums)`, we have processed all elements. Return `0`.
4.  **Memoization**:
    *   Since `j` can range from `-1` to `n - 1`, we map the second dimension of the memoization table as `j + 1` (offsetting `-1` to index `0`).
    *   Create a 2D slice `memo` of size $N \times (N + 1)$ initialized to `-1`.

### Go Code

``` go
func lengthOfLIS(nums []int) int {
    n := len(nums)
    memo := make([][]int, n)
    for i := range memo {
        memo[i] = make([]int, n+1)
        for j := range memo[i] {
            memo[i][j] = -1
        }
    }
    
    var dfs func(i, j int) int
    dfs = func(i, j int) int {
        // Base case: no more elements left
        if i == len(nums) {
            return 0
        }
        
        // Return cached result if already calculated (j + 1 offset)
        if memo[i][j+1] != -1 {
            return memo[i][j+1]
        }
        
        // Option 1: Skip the current element
        LIS := dfs(i+1, j) 

        // Option 2: Include the current element (if valid)
        if j == -1 || nums[j] < nums[i] {
            LIS = max(LIS, 1+dfs(i+1, i))
        }
        
        memo[i][j+1] = LIS
        return LIS
    }
    
    return dfs(0, -1)
}
```

### Code Efficiency

- **Time Complexity**: $O(N^2)$
    - Where $N$ is the number of elements in `nums`. There are $N$ choices for `i` and $N+1$ choices for `j`, resulting in a total state space of $O(N^2)$. Each state takes $O(1)$ time to compute.
- **Space Complexity**: $O(N^2)$
    - We use $O(N^2)$ space for the 2D memoization slice and $O(N)$ space for the recursion stack.

---

## Solution 2: Bottom-Up Dynamic Programming

We can also solve this problem bottom-up by defining a 1D DP table where each element stores the LIS ending at that index.

### Thought Process

1.  **DP Array Setup**:
    *   Let `dp[i]` represent the length of the longest increasing subsequence that ends at index `i`.
    *   Initialize `dp[i] = 1` for all $0 \le i < n$ (since any single element is an increasing subsequence of length 1).
2.  **State Transition**:
    *   Iterate through each element at index `i` from left to right.
    *   For each element `nums[i]`, iterate through all preceding elements at index `j` (where $0 \le j < i$):
        *   If `nums[j] < nums[i]`, we can extend the LIS ending at `j` by appending `nums[i]`:
            $$\text{dp}[i] = \max(\text{dp}[i], 1 + \text{dp}[j])$$
3.  **Result**:
    *   The overall maximum length will be the maximum value inside the `dp` array: `max(dp[0], dp[1], ..., dp[n-1])`.

### Go Code

``` go
func lengthOfLIS(nums []int) int {
    n := len(nums)
    if n == 0 {
        return 0
    }
    
    dp := make([]int, n)
    for i := range dp {
        dp[i] = 1
    }
    
    res := 0
    for i := 0; i < n; i++ {
        for j := 0; j < i; j++ {
            if nums[j] < nums[i] {
                dp[i] = max(dp[i], 1 + dp[j])
            }
        }
        res = max(res, dp[i])
    }
    
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N^2)$
    - We use nested loops where the outer loop runs $N$ times and the inner loop runs up to $i$ times, resulting in $O(N^2)$ total operations.
- **Space Complexity**: $O(N)$
    - We allocate a 1D slice of size $N$ to store LIS values for each index.