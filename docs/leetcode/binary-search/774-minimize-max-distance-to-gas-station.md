# 774. Minimize Max Distance to Gas Station

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/minimize-max-distance-to-gas-station/description/)

## Solution: Binary Search on Floating-Point Answer

We want to find the minimum possible value of the maximum distance between adjacent gas stations after adding $k$ new stations. The feasibility of achieving a maximum distance $d$ follows a monotonic property: if a distance $d$ is achievable, any distance $> d$ is also achievable. Since the answer is a floating-point number, we perform binary search on a continuous floating-point range.

### Thought Process

1.  **Define Search Boundaries**:
    *   **Minimum Distance (`l`)**: The lower bound is `0.0`.
    *   **Maximum Distance (`r`)**: The absolute upper bound is the distance between the first and last station: `stations[len(stations)-1] - stations[0]`.
2.  **Feasibility Check (`canAchieve`)**:
    *   For a target distance `maxDist`, calculate how many additional stations are needed to ensure the distance between any adjacent stations is at most `maxDist`.
    *   For each adjacent pair of stations, compute the `gap = stations[i] - stations[i-1]`.
    *   If `gap > maxDist`, the number of new stations we must insert in this gap is:
        $$\text{stations\_needed} = \left\lceil \frac{\text{gap}}{\text{maxDist}} \right\rceil - 1$$
    *   Sum these requirements across all gaps. If the total `count <= k`, then the target `maxDist` is achievable.
3.  **Floating-Point Binary Search**:
    *   Instead of loop condition like `r - l > 1e-6` which can run into precision issues or slow convergence, we perform a fixed number of iterations (e.g. `80` iterations).
    *   Each iteration halves the interval. After $80$ iterations, the search space is scaled down by $2^{-80} \approx 8.27 \times 10^{-25}$, guaranteeing extreme precision and preventing infinite loops.
    *   In each step, find `m = l + (r - l) / 2`.
    *   If `canAchieve(m)` is `true`, then `m` works. We try to find a smaller maximum distance: `r = m`.
    *   Otherwise, `m` is too small. We search for a larger maximum distance: `l = m`.
4.  **Termination**:
    *   Return `l` (or `r`, as they are practically identical after $80$ iterations).

### Go Code

``` go
import "math"

func minmaxGasDist(stations []int, k int) float64 {
    l := 0.0
    r := float64(stations[len(stations)-1] - stations[0])
    n := len(stations)

    canAchieve := func(maxDist float64) bool {
        count := 0
        for i := 1; i < n; i++ {
            gap := float64(stations[i] - stations[i-1])
            if gap > maxDist {
                count += int(math.Ceil(gap/maxDist)) - 1
            }
        }
        return count <= k
    }

    for range 80 {
        m := l + (r-l)/2
        if canAchieve(m) {
            r = m
        } else {
            l = m
        }
    }
    return l
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - Since the binary search loop runs for a fixed number of iterations (specifically, 80 times), the loop complexity is $O(1)$. Inside the loop, we iterate through the `stations` array of size $n$, which takes $O(n)$ time. Thus, the total time complexity is linear, $O(n)$.
- **Space Complexity**: $O(1)$
    - The algorithm runs using constant auxiliary memory.