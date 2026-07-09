# 1011. Capacity To Ship Packages Within D Days

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/description/)

## Solution: Binary Search on Answer

We need to find the minimum ship capacity that allows all packages to be shipped in order within `days` days. Since the feasibility of shipping under a given capacity follows a monotonic pattern (if capacity $C$ works, all capacities $> C$ also work), we can perform binary search on the range of possible capacities.

### Thought Process

1.  **Define Search Boundaries**:
    *   **Minimum Capacity (`l`)**: The ship must be able to carry the single heaviest package in the list. Thus, the lower bound is the maximum single package weight:
        $$l = \max(\text{weights})$$
    *   **Maximum Capacity (`r`)**: The absolute upper bound is carrying all packages in a single day:
        $$r = \sum \text{weights}$$
2.  **Feasibility Check (`canShip`)**:
    *   For a target `capacity`, iterate through the `weights` array.
    *   Maintain a running sum `currWeight` of packages loaded onto the ship for the current day.
    *   If adding the next package `w` exceeds `capacity` (`currWeight + w > capacity`), we must ship the current batch. This increments `requiredDays` by $1$ and resets `currWeight = 0`.
    *   Add `w` to `currWeight`.
    *   After loading all packages, check if the total days required satisfies the constraint: `requiredDays <= days`.
3.  **Binary Search Loop (`l < r`)**:
    *   Compute the midpoint capacity: `m = l + (r - l) / 2`.
    *   If `canShip(m)` is `true`, `m` is a feasible capacity. We try to find a smaller capacity to minimize it, so we search the left half: `r = m`.
    *   If `canShip(m)` is `false`, the capacity `m` is too small. We must increase the capacity, so we search the right half: `l = m + 1`.
4.  **Termination**:
    *   The loop terminates when `l == r`. The index `l` represents the minimum feasible shipping capacity.

### Go Code

``` go
func shipWithinDays(weights []int, days int) int {
    l, r := 0, 0
    for _, w := range weights {
        l = max(l, w)
        r += w
    }

    canShip := func(capacity int) bool {
        requiredDays := 1
        currWeight := 0
        
        for _, w := range weights {
            if currWeight+w > capacity {
                requiredDays++
                currWeight = 0
            }
            currWeight += w
        }
        return requiredDays <= days
    }

    for l < r {
        m := l + (r-l)/2
        if canShip(m) {
            r = m
        } else {
            l = m+1
        }
    }
    return l
}
```

### Code Efficiency

- **Time Complexity**: $O(n \log S)$
    - Where $n$ is the number of elements in `weights` and $S$ is the search space range, which is $\sum \text{weights} - \max(\text{weights})$. The binary search takes $\log_2(S)$ iterations, and each iteration performs a feasibility check that traverses the array of size $n$, taking $O(n)$ time.
- **Space Complexity**: $O(1)$
    - The algorithm operates with constant extra space.