# 47. Permutations II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/permutations-ii/description/)

## Solution: Backtracking (DFS Closure with Visited Slice and Duplicate Pruning)

This problem builds on the standard Permutations problem by allowing duplicate elements in the input array `nums`. To ensure we generate only unique permutations, we sort the array to group duplicates together and apply a duplicate pruning condition during recursion.

### Thought Process

1.  **Duplicate Handling (Sorting)**:
    *   First, sort the input array `nums` using `sort.Ints(nums)`. This groups identical elements together (e.g., `[1, 2, 1] -> [1, 1, 2]`), making it easy to identify duplicates by comparing adjacent indices.
2.  **Visited Slice for Tracking**:
    *   Maintain a boolean slice `used` of size $n$ to keep track of which indices are currently part of the active path `curr`.
3.  **Recursive Branching & Pruning**:
    *   Iterate through all indices from `0` to $n - 1$:
        *   If `used[i]` is `true`, skip the element since it's already in use.
        *   **Duplicate Check**: If `nums[i] == nums[i-1]` and `used[i-1]` is `false`, skip `nums[i]`.
            *   *Why?* If `used[i-1]` is `false`, it means the identical element at `i-1` was already explored in a previous branch at this recursion level and has been backtracked (restored to unused). Starting a new branch with `nums[i]` at this point would produce a redundant set of identical permutations.
        *   Otherwise, include `nums[i]`:
            1. Append `nums[i]` to `curr` and mark index `i` as used: `used[i] = true`.
            2. Explore recursively: `dfs(curr)`.
            3. Backtrack: Remove the last element (`curr = curr[:len(curr)-1]`) and mark index `i` as unused: `used[i] = false`.
4.  **Base Case**:
    *   When `len(curr) == len(nums)`, we have built a complete, unique permutation. We create a deep copy of `curr` and append it to our results slice `res`.

### Go Code

``` go
import "sort"

func permuteUnique(nums []int) [][]int {
    n := len(nums)
    res := make([][]int, 0)
    sort.Ints(nums)
    used := make([]bool, n)

    var dfs func([]int)
    dfs = func(curr []int) {
        if len(curr) == n {
            copied := make([]int, n)
            copy(copied, curr)
            res = append(res, copied)
            return
        }

        for i := 0; i < n; i++ {
            if used[i] {
                continue
            }
            if i > 0 && nums[i-1] == nums[i] && !used[i-1] {
                continue
            }
            curr = append(curr, nums[i])
            used[i] = true
            dfs(curr)
            curr = curr[:len(curr)-1]
            used[i] = false
        }
    }
    dfs([]int{})
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n \cdot n!)$
    - In the worst case (no duplicate elements), there are $n!$ unique permutations. At each leaf node of the recursion tree, we perform an $O(n)$ slice copy operation. Sorting takes $O(n \log n)$ time, which is dominated by the $O(n \cdot n!)$ backtracking search.
- **Space Complexity**:
    - **With Output**: $O(n \cdot n!)$ to store all unique permutations.
    - **Auxiliary Space**: $O(n)$ for the recursion stack depth (at most $n$), the visited slice `used` of size $n$, and the temporary slice `curr` of size $n$.