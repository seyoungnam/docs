# 1231. Divide Chocolate

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/divide-chocolate/description/)

## Solution: Binary Search on Answer (Maximize the Minimum)

We want to divide a chocolate bar into $k + 1$ pieces (making $k$ cuts) such that the minimum sweetness of any piece is maximized. This is a classic "maximize the minimum" optimization problem. Since the feasibility of dividing the chocolate bar with a minimum piece sweetness $s$ is monotonic (if we can achieve $s$, we can achieve any value $< s$), we can perform binary search on the answer.

### Thought Process

1.  **Define Search Boundaries**:
    *   **Minimum Sweetness (`l`)**: The minimum possible sweetness of a piece is $1$.
    *   **Maximum Sweetness (`r`)**: The absolute upper bound is the sum of all sweetness values (e.g. if $k=0$ and we get the entire chocolate bar):
        $$r = \sum \text{sweetness}$$
2.  **Feasibility Check (`canKcuts`)**:
    *   For a target sweetness `minSweet`, we greedily group adjacent chunks of chocolate.
    *   Maintain a running sweetness sum `sum`.
    *   Iterate through the `sweetness` array. Add the current chunk's sweetness to `sum`.
    *   If `sum >= minSweet`, we successfully form a piece. Increment the piece count (`count++`) and reset the running sum: `sum = 0`.
    *   Once the iteration is complete, we check if we were able to create at least $k + 1$ pieces (one for each of the $k$ friends and one for ourselves): `count >= k + 1`.
3.  **Binary Search Loop (`l <= r`)**:
    *   Compute the midpoint sweetness: `m = l + (r - l) / 2`.
    *   If `canKcuts(m)` is `true`, then `m` is a feasible sweetness. We record `m` as our current best candidate (`res = m`) and search for a larger minimum sweetness in the right half: `l = m + 1`.
    *   If `canKcuts(m)` is `false`, the target sweetness `m` is too high to form $k + 1$ pieces. We search for a smaller minimum sweetness in the left half: `r = m - 1`.

### Go Code

``` go
func maximizeSweetness(sweetness []int, k int) int {
    l, r := 1, 0
    for _, s := range sweetness {
        r += s
    }

    canKcuts := func(minSweet int) bool {
        count, sum := 0, 0
        for _, s := range sweetness {
            sum += s
            if sum >= minSweet {
                count++
                sum = 0
            }
        }
        return count >= k+1
    }
    
    res := 0
    for l <= r {
        m := l + (r-l)/2
        if canKcuts(m) {
            res = m
            l = m+1
        } else {
            r = m-1
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n \log S)$
    - Where $n$ is the number of elements in `sweetness` and $S$ is the sum of sweetness values. The binary search runs $\log_2(S)$ times, and each iteration performs an $O(n)$ greedy check traversing the array.
- **Space Complexity**: $O(1)$
    - Only a constant number of helper variables are allocated in memory.