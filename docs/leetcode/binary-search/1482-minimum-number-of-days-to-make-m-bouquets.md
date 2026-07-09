# 1482. Minimum Number of Days to Make m Bouquets

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/minimum-number-of-days-to-make-m-bouquets/description/)

## Solution: Binary Search on Answer

We want to find the minimum number of days $D$ required to make $m$ bouquets, where each bouquet requires $k$ adjacent bloomed flowers. Since flowers that have bloomed remain bloomed, the capacity to make $m$ bouquets is monotonic relative to time: if we can make them on day $D$, we can also make them on any day $> D$. This allows us to use binary search on the answer.

### Thought Process

1.  **Impossible Case**:
    *   Each bouquet requires $k$ flowers, meaning we need a total of $m \times k$ flowers. If $m \times k > \text{len(bloomDay)}$, we do not have enough flowers in the garden. Return `-1` immediately.
2.  **Define Search Boundaries**:
    *   **Minimum possible day (`l`)**: The earliest possible day is $1$.
    *   **Maximum possible day (`r`)**: The absolute upper bound is the maximum value in `bloomDay`, representing the day when every single flower in the garden has bloomed.
3.  **Feasibility Check (`canMakeBouquets`)**:
    *   For a target number of `days`, iterate through `bloomDay`.
    *   Maintain a counter `currCnt` to track adjacent flowers that have bloomed on or before the target day (`bloom <= days`).
    *   If `currCnt` reaches $k$, we successfully form a bouquet. Increment `totalCnt` and reset `currCnt = 0` to start a new adjacent group.
    *   If a flower has not bloomed yet (`bloom > days`), the adjacency sequence is broken. Reset `currCnt = 0`.
    *   Check if we can form at least $m$ bouquets: `totalCnt >= m`.
4.  **Binary Search Loop (`l <= r`)**:
    *   Compute the midpoint day: `m = l + (r - l) / 2`.
    *   If `canMakeBouquets(m)` is `true`, `m` is a feasible day. We record `m` as our current best result (`res = m`) and attempt to find a smaller feasible day by searching the left half: `r = m - 1`.
    *   If `canMakeBouquets(m)` is `false`, the day `m` is too early to form enough bouquets. We must search for a later day: `l = m + 1`.

### Go Code

``` go
func minDays(bloomDay []int, m int, k int) int {
    if m*k > len(bloomDay) {
        return -1
    }
    l, r := 1, 0
    for _, day := range bloomDay {
        r = max(r, day)
    }

    canMakeBouquets := func(days int) bool {
        totalCnt, currCnt := 0, 0

        for _, bloom := range bloomDay {
            if bloom <= days {
                currCnt++
                if currCnt == k {
                    totalCnt++
                    currCnt = 0
                }
            } else {
                currCnt = 0
            }
        }
        return totalCnt >= m
    }

    res := -1
    for l <= r {
        mid := l + (r-l)/2
        if canMakeBouquets(mid) {
            res = mid
            r = mid-1
        } else {
            l = mid+1
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n \log(\max(\text{bloomDay})))$
    - Let $n$ be the number of flowers in `bloomDay`. The binary search runs $\log_2(\max(\text{bloomDay}))$ times. In each iteration, we run `canMakeBouquets` which does a single pass of size $n$, taking $O(n)$ time.
- **Space Complexity**: $O(1)$
    - Only a constant number of helper variables are allocated in memory.