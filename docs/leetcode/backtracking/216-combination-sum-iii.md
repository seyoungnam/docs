# 216. Combination Sum III

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/combination-sum-iii/description/)

## Solution: Backtracking (DFS Closure with Candidate Pruning)

To find all valid combinations of $k$ unique numbers chosen from the digits $1$ through $9$ that sum up to exactly $n$, we can use recursive depth-first backtracking. We prune branches early when the running sum exceeds $n$ or the number of selected digits exceeds $k$.

### Thought Process

1.  **Candidate Pruning**:
    *   Iterate `num` from `start` up to `9`.
    *   During loop iteration, we check if adding `num` is feasible:
        *   **Sum Limit**: If `num + sum > n`, we `break` the loop immediately. Since we iterate `num` in ascending order, any subsequent numbers will be even larger and will also exceed $n$.
        *   **Count Limit**: If `len(curr) + 1 > k`, we cannot add any more numbers without exceeding the size limit $k$, so we `break` the loop.
2.  **Recursive Decisions**:
    *   To avoid duplicates and ensure we choose only unique digits in ascending order, we pass `num + 1` as the `start` boundary to the recursive `dfs` call.
    *   **Backtrack**: Before transitioning to the next number in the loop, we subtract `num` from the running `sum` and slice off the last element from `curr`: `curr = curr[:len(curr)-1]`.
3.  **Base Case**:
    *   If `sum == n` and `len(curr) == k`, a valid combination is found. We make a deep copy of `curr` and append it to our results slice `res`.

### Go Code

``` go
func combinationSum3(k int, n int) [][]int {
    res := make([][]int, 0)
    
    var dfs func(int, int, []int)
    dfs = func(start int, sum int, curr []int) {
        if sum == n && len(curr) == k {
            copied := make([]int, k)
            copy(copied, curr)
            res = append(res, copied)
            return
        }
        
        for num := start; num <= 9; num++ {
            if num+sum > n || len(curr) + 1 > k {
                break
            }
            curr = append(curr, num)
            sum += num
            dfs(num+1, sum, curr)
            curr = curr[:len(curr)-1]
            sum -= num
        } 
    }
    dfs(1, 0, []int{})
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O\left(k \cdot \binom{9}{k}\right)$
    - The search space is extremely small and bounded. There are at most $\binom{9}{k}$ unique combinations of size $k$ chosen from the digits $1$ through $9$ (the maximum number of leaf states is $\binom{9}{5} = 126$ when $k=5$). Generating and copying each combination takes $O(k)$ time.
- **Space Complexity**:
    - **With Output**: $O\left(k \cdot \binom{9}{k}\right)$ to store the list of all valid combinations.
    - **Auxiliary Space**: $O(k)$ for the recursion stack depth (at most $k$) and the temporary `curr` slice of size $k$.