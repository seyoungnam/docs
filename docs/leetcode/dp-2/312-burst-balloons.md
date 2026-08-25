# 312. Burst Balloons

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/burst-balloons/description/)

You are given `n` balloons, indexed from `0` to `n - 1`. Each balloon is painted with a number on it represented by an array `nums`. You are asked to burst all the balloons.

If you burst the $i$-th balloon, you will get `nums[i - 1] * nums[i] * nums[i + 1]` coins. If `i - 1` or `i + 1` goes out of bounds of the array, then treat it as if there is a balloon painted with a `1` there.

Return *the maximum coins you can collect by bursting the balloons wisely*.

---

## Solution: Top-Down DFS with Memoization (Divide & Conquer)

Directly simulating which balloon to burst first is difficult because bursting a balloon changes the adjacency of all remaining balloons. Instead, we can invert the problem: we choose which balloon `i` in the subrange `[l, r]` to burst **last**.

### Thought Process

1.  **Inverting the Choice**:
    *   If balloon `i` is the last balloon to be burst in the range `[l, r]`, then all other balloons in `[l, i-1]` and `[i+1, r]` must have already been burst.
    *   Therefore, at the moment balloon `i` is burst, its adjacent neighbors are the boundary elements `l-1` and `r+1`.
    *   The coins gained by bursting `i` last in the range `[l, r]` is:
        $$\text{coins} = \text{nums}[l-1] \times \text{nums}[i] \times \text{nums}[r+1]$$
    *   The total coins collected from the range `[l, r]` is the sum of:
        *   Coins from bursting `i` last.
        *   Maximum coins from bursting the left subproblem `dfs(l, i-1)`.
        *   Maximum coins from bursting the right subproblem `dfs(i+1, r)`.
2.  **Boundary Padding**:
    *   Add a sentinel `1` to the beginning and the end of the `nums` array to simplify boundary conditions.
    *   The original array is now shifted to indices `1` through `len(nums) - 2`.
3.  **DFS with Memoization**:
    *   Use a 2D slice `dp` where `dp[l][r]` stores the maximum coins from subrange `[l, r]`.
    *   **Base Case**: If `l > r`, there are no balloons to burst; return `0`.
    *   **Transition**: Iterate `i` from `l` to `r`, and find the maximum profit by treating each `i` as the last balloon burst:
        $$\text{dp}[l][r] = \max_{i \in [l, r]} \left( \text{nums}[l-1] \cdot \text{nums}[i] \cdot \text{nums}[r+1] + \text{dfs}(l, i-1) + \text{dfs}(i+1, r) \right)$$
4.  **Result**:
    *   Return `dfs(1, len(nums) - 2)`.

### Go Code

``` go
func maxCoins(nums []int) int {
    // Pad boundaries with 1
    nums = append([]int{1}, nums...)
    nums = append(nums, 1)
    n := len(nums)

    // dp[l][r] stores the max coins from index l to r
    dp := make([][]int, n)
    for i := 0; i < n; i++ {
        dp[i] = make([]int, n)
    }

    var dfs func(l, r int) int
    dfs = func(l, r int) int {
        // Base case: no balloons in the range
        if l > r {
            return 0
        }
        
        // Return cached result if already computed
        if dp[l][r] > 0 {
            return dp[l][r]
        }

        // Try bursting each balloon i last in the range [l, r]
        for i := l; i <= r; i++ {
            coins := nums[l-1] * nums[i] * nums[r+1]
            coins += dfs(l, i-1) + dfs(i+1, r)
            dp[l][r] = max(dp[l][r], coins)
        }
        
        return dp[l][r]
    }
    
    return dfs(1, len(nums)-2)
}
```

### Code Efficiency

- **Time Complexity**: $O(N^3)$
    - Where $N$ is the number of balloons in the input. There are $O(N^2)$ combinations of subranges `[l, r]`. For each subrange, we run a loop of length `r - l + 1` (which is $O(N)$) to try every possible last balloon, resulting in an overall time complexity of $O(N^3)$.
- **Space Complexity**: $O(N^2)$
    - We use $O(N^2)$ auxiliary space to store the memoization table `dp`. The recursion stack also takes $O(N)$ space.