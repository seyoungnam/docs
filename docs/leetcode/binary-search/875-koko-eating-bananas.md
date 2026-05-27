# 875. Koko Eating Bananas

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/koko-eating-bananas/description/)

## Solution: Binary Search on the Answer

We want to find the minimum integer speed $k$ such that Koko can eat all the bananas within $h$ hours. Since the possible eating speeds are sorted ($1, 2, \dots, \max(\text{piles})$), we can perform a binary search on the **eating speed $k$**.

### Optimizing Your Boundaries

Your original code used `l = 0` and `r = sum(piles)`. While mathematically sound due to your `d <= h` check, this introduces two interview risks:
1.  **Division by Zero Risk**: Setting `l = 0` means the midpoint `v` can theoretically become `0`, leading to a division-by-zero panic. It is safer to set `l = 1` since Koko's minimum speed is 1.
2.  **Integer Overflow Risk**: The sum of all piles can be extremely large (up to $10^{14}$), which overflows 32-bit signed integers.
3.  **The `max(piles)` Limit**: Koko never needs to eat faster than the largest pile (`max(piles)`). Eating any faster would still take 1 hour per pile because she cannot start a new pile in the same hour. Thus, `r = max(piles)` is the optimal upper bound.

### Thought Process

1.  **Search Range**: Set `l = 1` and `r = max(piles)`.
2.  **Binary Search Loop**: While `l < r`:
    *   Find the middle eating speed: `v = l + (r - l) / 2`.
    *   Calculate the total hours `t` Koko takes to eat all piles at speed `v`. For each pile:
        *   Hours taken: `t += pile / v`.
        *   If there is a remainder, add an extra hour: `if pile % v > 0 { t++ }`.
3.  **Adjust Bounds**:
    *   If `t <= h` (Koko finished in time): This speed `v` is feasible, but a slower speed might also work. Search the left half including `v`: `r = v`.
    *   If `t > h` (Koko ran out of time): This speed is too slow. Search the right half excluding `v`: `l = v + 1`.
4.  **Result**: When `l == r`, the loop terminates, and `l` (or `r`) is the minimum valid speed.

### Go Code

``` go
func minEatingSpeed(piles []int, h int) int {
    // 1. Find the upper bound (maximum pile size)
    maxPile := 0
    for _, pile := range piles {
        maxPile = max(maxPile, pile)
    }
    
    // 2. Binary search on speed range [1, maxPile]
    l, r := 1, maxPile
    for l < r {
        v := l + (r-l)/2
        t := 0
        for _, pile := range piles {
            t += pile / v
            if pile % v > 0 {
                t += 1
            }
        }
        
        // If Koko can finish in time, try a slower speed (look left)
        if t <= h {
            r = v
        } else { // Too slow, must eat faster (look right)
            l = v + 1
        }
    }
    return r
}
```

### Code Efficiency

- **Time Complexity**: $O(n \log(\max(P)))$
    - Where $n$ is the number of piles and $\max(P)$ is the maximum pile size. The binary search takes $\log(\max(P))$ steps, and in each step, we iterate through all $n$ piles.
- **Space Complexity**: $O(1)$
    - We only use a few tracking variables, requiring $O(1)$ auxiliary space.
