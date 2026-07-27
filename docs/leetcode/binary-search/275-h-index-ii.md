# 275. H-Index II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/h-index-ii/description/)

## Solution: Binary Search on Citations Index

We are given a list of papers' citations sorted in ascending order. We want to find the researcher's h-index, which is the maximum value $h$ such that at least $h$ papers have each been cited at least $h$ times. Since the citations list is already sorted, we can solve this in logarithmic $O(\log n)$ time using binary search.

### Thought Process

1.  **Exploiting the Sorted Citations**:
    *   Let $n$ be the number of papers (length of `citations`).
    *   For any index $m$, there are exactly $n - m$ papers that have citation counts greater than or equal to `citations[m]`.
    *   Our goal is to find the first index $m$ where the citation count of the paper satisfies:
        $$\text{citations}[m] \ge n - m$$
    *   Once we find the first index $l$ that satisfies this condition, all papers from index $l$ to $n-1$ have at least $n-l$ citations. Thus, the maximum h-index is $n - l$.
2.  **Binary Search Strategy (`l < r`)**:
    *   Set `l = 0` and `r = n` (since the h-index can potentially be $n$ if all papers have at least $n$ citations).
    *   While `l < r`:
        *   Calculate the midpoint index: `m = l + (r - l) / 2`.
        *   Check the h-index criteria:
            *   **If $\text{citations}[m] \ge n - m$**: The paper at $m$ has enough citations to make $n - m$ a valid candidate h-index. To find a larger h-index (which corresponds to a smaller index $m$), we narrow our search to the left half: `r = m`.
            *   **Otherwise**: The paper at $m$ does not have enough citations, meaning the current candidate h-index is too large. We search strictly to the right: `l = m + 1`.
3.  **Termination**:
    *   The loop terminates when `l == r`. The resulting h-index is $n - l$.

### Go Code

``` go
func hIndex(citations []int) int {
    n := len(citations)
    l, r := 0, n
    for l < r {
        m := l + (r-l)/2
        if citations[m] >= n-m {
            r = m
        } else {
            l = m+1
        }
    }
    return n-l
}
```

### Code Efficiency

- **Time Complexity**: $O(\log n)$
    - The search interval is halved at each iteration, performing binary search on the array index.
- **Space Complexity**: $O(1)$
    - The search runs in-place with constant auxiliary variables.