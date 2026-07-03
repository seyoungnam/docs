# 77. Combinations

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/combinations/description/)

## Solution: Backtracking (DFS Closure with Descending Order Pruning)

To generate all combinations of $k$ numbers chosen from $1$ to $n$, we can use recursive depth-first backtracking. 

Because order does not matter in combinations (e.g., `[1, 2]` is identical to `[2, 1]`), we must enforce a strict sorting rule within each generated combination to avoid generating duplicates. In this implementation, we enforce a **strictly descending order** for each path.

### Thought Process

1.  **Enforcing Sorting Constraints (Pruning)**:
    *   To prevent duplicates, we only choose numbers that are strictly smaller than the previously selected number in our current combination:
        $$\text{curr}[\text{last}] > \text{num}$$
    *   If `len(curr) > 0` and the last element is less than or equal to the current number (`curr[len(curr)-1] <= num`), we skip `num`.
2.  **Visited Slice for Tracking**:
    *   We maintain a boolean slice `used` of size $n+1$ to keep track of which numbers are currently part of the active path.
3.  **Recursive Decisions**:
    *   For each recursion step, iterate `num` from $1$ to $n$:
        *   If `used[num]` is `true`, skip.
        *   If `num` violates the descending order constraint, skip.
        *   Otherwise, include `num`:
            1. Append `num` to `curr` and mark it as used: `used[num] = true`.
            2. Explore recursively: `dfs(curr)`.
            3. Backtrack: Remove the last element (`curr = curr[:len(curr)-1]`) and mark it as unused: `used[num] = false`.
4.  **Base Case**:
    *   When the length of the current path `len(curr)` reaches $k$, we have successfully formed a valid combination. We make a deep copy of `curr` and append it to our results slice `res`.

### Go Code

``` go
func combine(n int, k int) [][]int {
    res := make([][]int, 0)
    used := make([]bool, n+1)

    var dfs func([]int)
    dfs = func(curr []int) {
        if len(curr) == k {
            copied := make([]int, len(curr))
            copy(copied, curr)
            res = append(res, copied)
            return
        }
        for num := 1; num <= n; num++ {
            if used[num] {
                continue
            }
            if len(curr) > 0 && curr[len(curr)-1] <= num {
                continue
            }
            used[num] = true
            curr = append(curr, num)
            dfs(curr)
            curr = curr[:len(curr)-1]
            used[num] = false
        }
    }
    dfs([]int{})
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O\left(k \cdot \binom{n}{k}\right)$
    - The total number of valid combinations generated is $\binom{n}{k}$ (i.e., $n$ choose $k$). At each of the leaf nodes in the recursion tree, we perform an $O(k)$ slice copy operation to save the permutation.
- **Space Complexity**:
    - **With Output**: $O\left(k \cdot \binom{n}{k}\right)$ to store the list of all combinations.
    - **Auxiliary Space**: $O(k)$ for the recursion stack (which goes at most $k$ levels deep) and the temporary `curr` slice of size $k$.