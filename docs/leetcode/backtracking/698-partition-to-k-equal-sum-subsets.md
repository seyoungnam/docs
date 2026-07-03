# 698. Partition to K Equal Sum Subsets

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/partition-to-k-equal-sum-subsets/description/)

## Solution: Backtracking (DFS Closure with Advanced Pruning)

To partition an array into $k$ subsets of equal sum, we can use recursive depth-first backtracking. Since the search space for this problem is exponential ($O(k \cdot 2^N)$), we must apply critical pruning optimizations to pass within the time limit.

### Thought Process

1.  **Initial Feasibility Checks**:
    *   Find the sum of all elements. If `sum % k != 0`, we cannot partition the array into $k$ equal subsets; return `false`.
    *   Define the target sum of each subset (bucket) as `target = sum / k`.
    *   If the largest element in the array is greater than `target`, return `false`.
2.  **Optimization 1: Descending Sort**:
    *   Sort `nums` in descending order. Placing larger elements first reduces the search branching factor since they are harder to fit and cause search paths to fail much earlier.
3.  **Optimization 2: Duplicate Pruning**:
    *   If `nums[i] == nums[i-1]` and `nums[i-1]` is not used (`!used[i-1]`), it means we just attempted to place `nums[i-1]` at this position in the current bucket and it failed. Trying the same value `nums[i]` will also fail, so we skip it.
4.  **Optimization 3: First-Element Failure Pruning**:
    *   If `currSum == 0` (we are attempting to choose the first element to start a new subset bucket) and the recursive search fails after picking `nums[i]`, we immediately `break` the loop and return `false`.
    *   *Why?* Since all buckets are symmetric, if we cannot start a bucket using `nums[i]`, then `nums[i]` can never be placed in any bucket. There is no point in trying smaller elements as the first element of this bucket.
5.  **Recursive Branching**:
    *   **Base Case 1**: If `buckets == k - 1`, we have successfully filled $k-1$ buckets. The remaining unused elements must sum up to `target`, so we return `true`.
    *   **Base Case 2**: If the current subset reaches `currSum == target`, we successfully completed one bucket. We recursively start building the next bucket from the beginning by calling `dfs(0, 0, buckets+1)`.
    *   **Loop**: Try to fill the current bucket by iterating `i` from `start` to $n - 1$. If `nums[i]` fits, mark it as `used`, recurse, and backtrack if it fails.

### Go Code

``` go
import "sort"

func canPartitionKSubsets(nums []int, k int) bool {
    sum := 0
    for _, num := range nums {
        sum += num
    }

    if sum%k != 0 {
        return false
    }
    target := sum / k

    sort.Slice(nums, func(i, j int) bool {
        return nums[i] > nums[j]
    })

    if nums[0] > target {
        return false
    }
    n := len(nums)
    used := make([]bool, n)

    var dfs func(start int, currSum int, buckets int) bool
    dfs = func(start int, currSum int, buckets int) bool {
        if buckets == k-1 {
            return true
        }

        if currSum == target {
            return dfs(0, 0, buckets+1)
        }

        for i := start; i < n; i++ {
            if used[i] || currSum + nums[i] > target {
                continue
            }
            if i > start && nums[i-1] == nums[i] && !used[i-1] {
                continue
            }
            used[i] = true
            if dfs(i+1, currSum+nums[i], buckets) {
                return true
            }
            used[i] = false
            if currSum == 0 {
                break
            }
        }
        return false
    }
    return dfs(0, 0, 0)
}
```

### Code Efficiency

- **Time Complexity**: $O(k \cdot 2^N)$ in the worst case. However, with the sorting and advanced pruning heuristics, the search space is pruned drastically, making it run in a fraction of a millisecond for typical inputs.
- **Space Complexity**: $O(N)$ auxiliary space for the recursion stack (which goes at most $N$ levels deep) and the `used` visited slice of size $N$.