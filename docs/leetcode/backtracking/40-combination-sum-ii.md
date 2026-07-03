# 40. Combination Sum II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/combination-sum-ii/description/)

## Solution: Backtracking

### Thought Process

- **Pruning through Sorting**: Sorting groups identical elements together, which is essential for identifying and skipping duplicates to avoid redundant combinations.
- **Decision Tree**: Each element in `candidates` presents a binary choice: either include it in the current combination or exclude it.
- **Recursive Branching**:
    - **Include**: Add the current element to the path and recurse to the next index ($i+1$).
    - **Exclude**: To avoid duplicate results, skip all subsequent instances of the current value before recursing to the next unique candidate.
- **Base Case**: 
    - If the current `sum` reaches the `target`, a valid combination is found; add a deep copy of the path to the result.
    - If `sum` exceeds `target` or the index reaches the end of `candidates`, terminate the current branch.

### Go Code

``` go
func combinationSum2(candidates []int, target int) [][]int {
    res := make([][]int, 0)
    sort.Ints(candidates)
    n := len(candidates)

    var dfs func(int, int, []int)
    dfs = func(start int, sum int, curr []int) {
        if sum == target {
            copied := make([]int, len(curr))
            copy(copied, curr)
            res = append(res, copied)
            return
        }
        
        for i := start; i < n; i++ {
            if i > start && candidates[i-1] == candidates[i] {
                continue
            }
            num := candidates[i]
            if sum + num > target {
                break
            }

            curr = append(curr, num)
            sum += num
            dfs(i+1, sum, curr)
            curr = curr[:len(curr)-1]
            sum -= num
        }
    }
    
    dfs(0, 0, []int{})
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(2^n)$
    - In the worst case, we explore every possible subset of the $n$ candidates. While the target sum prunes many branches, the theoretical upper bound remains exponential.
- **Space Complexity**: $O(n)$
    - Excluding the output list, the auxiliary space is dominated by the recursion stack, which reaches a maximum depth of $n$.

