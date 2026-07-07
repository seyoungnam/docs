# 2305. Fair Distribution of Cookies

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/fair-distribution-of-cookies/description/)

## Solution: Backtracking (DFS Closure with Advanced Pruning)

The **unfairness** of a cookie distribution is defined as the maximum total number of cookies obtained by any single child. We want to distribute all cookie bags to $k$ children such that this maximum sum is minimized. We can solve this minimax partitioning problem using recursive backtracking optimized with advanced pruning techniques.

### Thought Process

1.  **Minimax Formulation**:
    *   Let `children` be an array of size $k$, where `children[i]` tracks the total cookies assigned to child `i`.
    *   Our objective is to minimize the maximum sum in `children`:
        $$\text{unfairness} = \max_{0 \le i < k} (\text{children}[i])$$
2.  **Pruning 1: Descending Sort**:
    *   Sort the `cookies` array in descending order. Distributing larger cookie bags first builds up the children's cookie sums quickly, allowing our branch bounding checks to trigger much earlier in the search.
3.  **Pruning 2: Unfairness Bound Check**:
    *   As we distribute bags, we track `maxCookies` (the maximum sum of any child so far). If `maxCookies >= minUnfairness`, it is impossible for the current branch to yield a better (strictly smaller) unfairness score. We prune this branch immediately.
4.  **Pruning 3: Empty Child Symmetry**:
    *   When trying to assign `cookies[idx]` to child `i`:
        *   If the assignment fails and we backtrack, leaving `children[i] == 0`, we `break` the loop immediately.
        *   *Why?* If child `i` has $0$ cookies, then child `i` and any subsequent empty child `j > i` are symmetric. Trying to assign the bag to child `j` instead of child `i` would explore identical states. Breaking the loop eliminates these redundant branches.
5.  **Base Case**:
    *   When `idx == len(cookies)`, all bags are successfully distributed. We update the global minimum unfairness: `minUnfairness = maxCookies`.

### Go Code

``` go
import (
    "math"
    "sort"
)

func distributeCookies(cookies []int, k int) int {
    sort.Slice(cookies, func(i, j int) bool {
        return cookies[i] > cookies[j]
    })

    children := make([]int, k)
    minUnfairness := math.MaxInt32

    var dfs func(idx int, maxCookies int)
    dfs = func(idx int, maxCookies int) {
        if maxCookies >= minUnfairness {
            return
        }

        if idx == len(cookies) {
            minUnfairness = maxCookies
            return
        }

        for i := 0; i < k; i++ {
            children[i] += cookies[idx]
            dfs(idx+1, max(maxCookies, children[i]))
            children[i] -= cookies[idx]
            if children[i] == 0 {
                break
            } 
        }
    }
    dfs(0, 0)
    return minUnfairness
}
```

### Code Efficiency

- **Time Complexity**: $O(k^N)$
    - Where $N$ is the number of cookie bags ($N \le 8$) and $k$ is the number of children. For each bag, we have $k$ possible children to assign it to. Due to the very small size of $N$ and our aggressive pruning heuristics, the recursion tree is extremely small, executing in under a millisecond.
- **Space Complexity**: $O(N + k)$
    - The auxiliary space is determined by the recursion call stack depth (at most $N$) and the `children` slice of size $k$.