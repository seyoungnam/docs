# 39. Combination Sum

[LeetCode Problem Description](https://leetcode.com/problems/combination-sum/description/)

## Solution 1: Backtracking

### Thought Process

- **Decision Tree**: Each element in `candidates` presents a binary choice: either include it in the current subset or exclude it.
- **Recursive Branching**:
    - **Include**: Add the current element to the path and recurse to the current index.
    - **Exclude**: Recurse to the next index without adding the current element.
- **Base Case**: 
    - If the current `sum` surpasses `target` or the current index surpassess the `candidate` length, return.
    - If the current `sum` reaches the `target`, add the current subset to `res`.

### Go Code

``` go
func combinationSum(candidates []int, target int) [][]int {
    res := [][]int{}
    sort.Ints(candidates)
    backtrack(candidates, []int{}, 0, target, 0, &res)
    return res
}

func backtrack(candidates []int, curr []int, sum int, target int, i int, res *[][]int) {
    if sum > target || i == len(candidates) {
        return
    }
    if sum == target {
        copied := make([]int, len(curr))
        copy(copied, curr)
        *res = append(*res, copied)
        return
    }
    backtrack(candidates, append(curr, candidates[i]), sum+candidates[i], target, i, res)
    backtrack(candidates, curr, sum, target, i+1, res)
    return
}
```

### Code Efficiency

- **Time Complexity**: $O(N^{\frac{T}{M}})$
    - $N$ is the number of candidates, $T$ is the target, and $M$ is the minimum candidate value.
    - The search space can be modeled as an $N$-ary tree with a maximum depth of $T/M$. In the worst case, the number of nodes explored is exponential relative to the depth.
- **Space Complexity**: $O(\frac{T}{M})$
    - Excluding the result list, the auxiliary space is determined by the recursion stack, which can go up to $T/M$ levels deep.

