# 40. Combination Sum II

:simple-leetcode:[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/combination-sum-ii/description/)

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
    sort.Ints(candidates)
    res := make([][]int, 0)
    backtrack(candidates, []int{}, 0, target, 0, &res)
    return res
}

func backtrack(nums []int, curr []int, sum int, target int, i int, res *[][]int) {
    if sum == target {
        copied := make([]int, len(curr))
        copy(copied, curr)
        *res = append(*res, copied)
        return
    }
    if sum > target || i >= len(nums) {
        return
    }
    backtrack(nums, append(curr, nums[i]), sum+nums[i], target, i+1, res)
    for i+1 < len(nums) && nums[i] == nums[i+1] {
        i++
    }
    backtrack(nums, curr, sum, target, i+1, res)
    return
}
```

### Code Efficiency

- **Time Complexity**: $O(2^n)$
    - In the worst case, we explore every possible subset of the $n$ candidates. While the target sum prunes many branches, the theoretical upper bound remains exponential.
- **Space Complexity**: $O(n)$
    - Excluding the output list, the auxiliary space is dominated by the recursion stack, which reaches a maximum depth of $n$.

