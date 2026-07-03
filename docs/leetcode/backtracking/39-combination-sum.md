# 39. Combination Sum

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/combination-sum/description/)

## Solution: Backtracking (DFS Closure with Sorted Pruning)

### Thought Process

- **Sorting & Early Pruning**: 
    *   Sort the `candidates` array in ascending order. 
    *   During traversal, if adding the current candidate `num` to `sum` exceeds `target` (`sum + num > target`), we can immediately `break` the loop. Since the array is sorted, any subsequent candidates will also exceed the target.
- **Recursive Branching**:
    *   Iterate `i` from `start` to `n - 1` to process candidates.
    *   To allow reusing the same number multiple times, we pass `i` (the current index) as the `start` parameter in the recursive call `dfs(i, sum, curr)`.
    *   **Backtrack**: Subtract the chosen number from `sum` and slice it off from `curr` (`curr = curr[:len(curr)-1]`) before moving to the next element in the loop.
- **Base Case**:
    *   If `sum == target`, a valid combination is found. We make a deep copy of the `curr` slice and append it to `res`.

### Go Code

``` go
import "sort"

func combinationSum(candidates []int, target int) [][]int {
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
            num := candidates[i]
            if sum + num > target {
                break
            }
            
            curr = append(curr, num)
            sum += num
            dfs(i, sum, curr)
            curr = curr[:len(curr)-1]
            sum -= num
        } 
    }

    dfs(0, 0, []int{})
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N^{\frac{T}{M}})$
    - $N$ is the number of candidates, $T$ is the target, and $M$ is the minimum candidate value.
    - The search space can be modeled as an $N$-ary tree with a maximum depth of $T/M$. In the worst case, the number of nodes explored is exponential relative to the depth.
- **Space Complexity**: $O(\frac{T}{M})$
    - Excluding the result list, the auxiliary space is determined by the recursion stack, which can go up to $T/M$ levels deep.

