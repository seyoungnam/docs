# 45. Jump Game II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/jump-game-ii/description/)

You are given a **0-indexed** array of integers `nums` of length `n`. You are initially positioned at `nums[0]`.

Each element `nums[i]` represents the maximum length of a forward jump from index `i`. In other words, if you are at `nums[i]`, you can jump to any `nums[i] + j` where:
*   `0 <= j <= nums[i]` and
*   `i + j < n`

Return *the minimum number of jumps to reach* `nums[n - 1]`. The test cases are generated such that you can reach `nums[n - 1]`.

---

## Solution 1: Bottom-Up Dynamic Programming

We can solve this problem using bottom-up 1D Dynamic Programming. By iterating backwards from the end of the array, we can determine the minimum jumps needed from each index to reach the final index.

### Thought Process

1.  **DP State**:
    *   Let `jumps[i]` represent the minimum number of jumps required to reach the last index (`n - 1`) starting from index `i`.
2.  **Base Case**:
    *   `jumps[n - 1] = 0` (0 jumps are needed to reach the end when already at the end).
3.  **Initialization**:
    *   Initialize all other values in the `jumps` slice to a large number (`math.MaxInt32`) representing unreachable states.
4.  **Transition**:
    *   Iterate backwards from $i = n - 2$ down to $0$.
    *   At index `i`, we can make a jump of size up to `step = nums[i]`. Thus, we can land on any index `j` in the range `[i + 1, min(n - 1, i + step)]`.
    *   For each reachable index `j`, the cost to reach the end would be `1 + jumps[j]`. We take the minimum cost across all choices:
        $$jumps[i] = \min_{j \in [i+1, \min(n-1, i+step)]} (1 + jumps[j])$$
5.  **Result**:
    *   Return `jumps[0]`.

### Go Code

``` go
import "math"

func jump(nums []int) int {
    n := len(nums)
    jumps := make([]int, n)
    for i := range jumps {
        jumps[i] = math.MaxInt32
    }
    jumps[n-1] = 0
    
    // Bottom-up computation
    for i := n-2; i >= 0; i-- {
        step := nums[i]
        for j := i+1; j <= min(n-1, i+step); j++ {
            if jumps[j] != math.MaxInt32 {
                jumps[i] = min(jumps[i], 1 + jumps[j])
            }
        }
    }
    return jumps[0]
}
```

### Code Efficiency

- **Time Complexity**: $O(N^2)$
    - In the worst case (e.g., when each element allows jumping to the end), the inner loop runs $O(N)$ times at each step, resulting in a total time complexity of $O(N^2)$.
- **Space Complexity**: $O(N)$
    - We allocate a `jumps` slice of size $N$ to store the subproblem solutions.

---

## Solution 2: Greedy (Linear Scan, Space Optimized)

We can optimize the solution to $O(N)$ time and $O(1)$ space using a greedy approach. We view the problem as a BFS-like traversal where we keep track of the farthest reachable index for each "level" of jumps.

### Thought Process

1.  **Variables**:
    *   `jumps`: The total number of jumps made so far.
    *   `currEnd`: The boundary index of the current jump range.
    *   `farthest`: The farthest index we can reach with the current number of jumps.
2.  **Linear Sweep**:
    *   Iterate through the array from index `0` up to `n - 2` (we stop before the last element because we don't need to jump once we have reached the end).
    *   For each index `i`, update the farthest index we can reach: `farthest = max(farthest, i + nums[i])`.
    *   When we reach the boundary of our current jump range (`i == currEnd`):
        *   We must make another jump: `jumps++`.
        *   Update the current jump boundary to the farthest index we can reach: `currEnd = farthest`.
3.  **Result**:
    *   Return `jumps`.

### Go Code

``` go
func jump(nums []int) int {
    n := len(nums)
    if n <= 1 {
        return 0
    }
    
    jumps := 0
    currEnd := 0
    farthest := 0
    
    for i := 0; i < n-1; i++ {
        farthest = max(farthest, i + nums[i])
        
        // If we reach the end of the current jump range
        if i == currEnd {
            jumps++
            currEnd = farthest
            
            // If the current jump range already reaches the end
            if currEnd >= n-1 {
                break
            }
        }
    }
    
    return jumps
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - We make a single pass through the `nums` array of size $N$.
- **Space Complexity**: $O(1)$
    - We only use constant auxiliary variables, achieving optimal space efficiency.