# 2597. The Number of Beautiful Subsets

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/the-number-of-beautiful-subsets/description/)

## Solution: Backtracking (DFS Closure with Frequency Array Pruning)

A subset of `nums` is defined as **beautiful** if it does not contain any two integers with an absolute difference equal to `k`. We can count all valid non-empty beautiful subsets using recursive depth-first backtracking.

### Thought Process

1.  **Sorting & Conflict Simplification**:
    *   First, sort `nums` in ascending order using `sort.Ints(nums)`.
    *   By sorting, when we consider `num := nums[i]`, any conflicting element in our subset must be smaller than `num`. That is:
        $$\text{conflict} = \text{num} - k$$
        Any conflicting element like $\text{num} + k$ is larger and hasn't been processed yet. Therefore, we only need to verify if the element $\text{num} - k$ is currently present in our active subset path.
2.  **State Tracking (Frequency Array)**:
    *   To check for conflicts in $O(1)$ constant time, we maintain a frequency tracker. Given the constraint $1 \le \text{nums}[i] \le 1000$, we can use a pre-allocated array `freq := make([]int, 1001)` of size 1001.
3.  **Recursive Decisions**:
    *   At each index `i`:
        *   **Branch 1 (Include)**: We can include `num := nums[i]` only if no conflict exists:
            *   Conflict check: If `num < k` (meaning `num - k` is negative and cannot exist in the array) or if `freq[num-k] == 0` (meaning `num - k` is not currently in our subset).
            *   If valid, increment `freq[num]`, recursively call `dfs(i+1)`, and decrement `freq[num]` to backtrack.
        *   **Branch 2 (Exclude)**: Leave `nums[i]` out of the subset and recurse by calling `dfs(i+1)`.
4.  **Base Case**:
    *   When the index `i` reaches `len(nums)`, we have completed a subset path. Increment the global `res` and return.
5.  **Excluding the Empty Subset**:
    *   Since the backtracking generates all possible subsets (including the empty subset `[]`), and the problem specifies **non-empty** beautiful subsets, we return `res - 1`.

### Go Code

``` go
func beautifulSubsets(nums []int, k int) int {
    sort.Ints(nums)
    freq := make([]int, 1001)
    res := 0

    var dfs func(i int)
    dfs = func(i int) {
        if i == len(nums) {
            res++
            return
        }
        num := nums[i]
        if num < k || freq[num-k] == 0 {
            freq[num]++
            dfs(i+1)
            freq[num]--
        }
        dfs(i+1)
    }

    dfs(0)
    // Subtract 1 because the standard subset generator counts the empty subset []
    return res - 1
}
```

### Code Efficiency

- **Time Complexity**: $O(2^N)$ in the worst case (where $N$ is the length of `nums`). At each element, we branch at most twice. Sorting takes $O(N \log N)$ which is dominated by the $O(2^N)$ search tree. In practice, the execution is much faster because the search space is heavily pruned when conflicts are detected.
- **Space Complexity**: $O(N)$ auxiliary space. The recursion call stack depth goes up to a maximum of $N$, and the frequency array `freq` uses a constant size of 1001.

### Reference

<iframe width="560" height="315" src="https://www.youtube.com/embed/Dle_SpjHTio?si=qgkqFlEYHHENDnkS" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>