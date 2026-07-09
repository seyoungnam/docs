# 1552. Magnetic Force Between Two Balls

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/magnetic-force-between-two-balls/description/)

## Solution: Binary Search on Answer

This is a classic "maximize the minimum distance" optimization problem. The feasibility of placing $m$ balls with a minimum magnetic force (distance) $d$ is monotonic: if we can place them with a force $d$, we can also place them with any force $< d$. If we cannot place them with a force $d$, we cannot do so with any force $> d$. This monotonic behavior allows us to perform binary search on the answer.

### Thought Process

1.  **Sorting**:
    *   First, sort the `position` array. This allows us to greedily place balls from left to right and calculate the distance between adjacent balls easily.
2.  **Define Search boundaries**:
    *   **Minimum possible distance (`l`)**: The minimum possible force is $1$.
    *   **Maximum possible distance (`r`)**: The maximum possible force is the distance between the first and last bucket: `position[n-1] - position[0]`.
3.  **Feasibility Check (`canPlaceMballs`)**:
    *   We want to verify if it is possible to place $m$ balls such that the distance between any two adjacent balls is at least `minDist`.
    *   Place the first ball in the first bucket `pos[0]`.
    *   Iterate through the remaining buckets. If the distance between the current bucket `pos[i]` and the `lastBall` is greater than or equal to `minDist`, place a ball there (`cnt++`) and update `lastBall = pos[i]`.
    *   If at any point the ball count reaches $m$ (`cnt >= m`), return `true`.
    *   If we finish iterating and `cnt < m`, return `false`.
4.  **Binary Search Loop (`l <= r`)**:
    *   Compute midpoint distance: `mid = l + (r - l) / 2`.
    *   If `canPlaceMballs(position, m, mid)` is `true`, then `mid` is a valid force. We record it as a candidate result (`res = mid`) and try to find a larger minimum force: `l = mid + 1`.
    *   If `canPlaceMballs(position, m, mid)` is `false`, the force `mid` is too large to place all $m$ balls. We must search for a smaller minimum force: `r = mid - 1`.

### Go Code

``` go
import "sort"

func maxDistance(position []int, m int) int {
    sort.Ints(position)
    n := len(position)

    l := 1
    r := position[n-1] - position[0]
    res := 0

    for l <= r {
        mid := l + (r-l)/2
        if canPlaceMballs(position, m, mid) {
            res = mid
            l = mid+1
        } else {
            r = mid-1
        }
    }
    return res
}

func canPlaceMballs(pos []int, m int, minDist int) bool {
    lastBall := pos[0]
    cnt := 1

    for i := 1; i < len(pos); i++ {
        if pos[i] - lastBall >= minDist {
            lastBall = pos[i]
            cnt++
            if cnt >= m {
                return true
            }
        }
    }
    return cnt >= m
}
```

### Code Efficiency

- **Time Complexity**: $O(n \log n + n \log S)$
    - Sorting the `position` array of size $n$ takes $O(n \log n)$ time.
    - The binary search takes $\log_2(S)$ iterations, where $S = \text{position}[n-1] - \text{position}[0]$. Each iteration performs an $O(n)$ greedy check, contributing $O(n \log S)$ time.
- **Space Complexity**: $O(1)$ auxiliary space
    - The sorting is done in-place, and we only use a few tracking variables.