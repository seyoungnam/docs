# 55. Jump Game

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/jump-game/description/)

## Solution 1: Dynamic Programming (Bottom-Up)

We can solve this problem using bottom-up Dynamic Programming. We maintain a boolean array `dp` where `dp[i]` represents whether it is possible to reach the last index starting from index `i`.

### Thought Process

1.  **DP Array Setup**:
    *   Create a boolean slice `dp` of size $n$.
    *   **Base Case**: Set `dp[n-1] = true` because we are already at the target index when starting from the last index.
2.  **State Transition**:
    *   Iterate backwards from $i = n-2$ down to $0$.
    *   For each index `i`, we can make jumps of length up to `step = nums[i]`.
    *   Check if any reachable index $j$ (where $i+1 \le j \le \min(n-1, i+\text{step})$) has `dp[j] == true`.
    *   If a reachable index `j` is valid to reach the end, mark `dp[i] = true` and `break` early.
3.  **Result**:
    *   The value of `dp[0]` tells us if the last index is reachable starting from index $0$.

### Go Code

``` go
func canJump(nums []int) bool {
    n := len(nums)
    dp := make([]bool, n)
    dp[n-1] = true
    for i := n-2; i >= 0; i-- {
        step := nums[i]
        for j := i+1; j <= min(n-1, i+step); j++ {
            if dp[j] {
                dp[i] = true
                break
            }
        }
    }
    return dp[0]
}
```

### Code Efficiency

- **Time Complexity**: $O(n^2)$ in the worst case
    - If every element in the array allows jumping to the end, the inner loop can check up to $n$ elements, yielding $O(n^2)$ time.
- **Space Complexity**: $O(n)$
    - We allocate a boolean slice of size $n$ to store DP states.

---

## Solution 2: Greedy Strategy (Reachable Horizon)

We can optimize both time and space by using a **Greedy** approach. We keep track of the maximum index we can reach from the beginning. If the maximum reachable index ever matches or exceeds the last index, we return `true`.

### Thought Process

1.  **Furthest Reach tracking**:
    *   Maintain a variable `maxIdx` initialized to `0` representing the furthest index we can currently reach.
2.  **Traverse and Update**:
    *   Iterate through the array from left to right.
    *   At each index `i` (excluding the last element, since reaching the last element means we are done):
        *   If `maxIdx < i`, it means we cannot even reach the current index `i` $\rightarrow$ break early.
        *   Otherwise, update `maxIdx` to be the maximum of `maxIdx` and the furthest we can jump from the current index: `i + nums[i]`.
3.  **Result**:
    *   Return whether `maxIdx` is at least `len(nums) - 1`.

### Go Code

``` go
func canJump(nums []int) bool {
    maxIdx := 0
    for i := 0; i < len(nums)-1; i++ {
        if maxIdx < i {
            break
        }
        maxIdx = max(maxIdx, i+nums[i])
    }
    return maxIdx >= len(nums)-1
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We traverse the array of length $n$ at most once.
- **Space Complexity**: $O(1)$
    - We only track a single integer variable `maxIdx`, using constant auxiliary space.