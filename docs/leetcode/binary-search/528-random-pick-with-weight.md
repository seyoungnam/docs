# 528. Random Pick with Weight

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/random-pick-with-weight/description/)

## Solution: Prefix Sums and Binary Search

To pick an index randomly with a probability proportional to its weight, we can map each index to a contiguous interval of integers whose length corresponds to its weight. We construct a **Prefix Sum** array representing the interval boundaries, generate a random target value, and perform a binary search to find which interval the target falls into.

### Thought Process

1.  **Interval Mapping (Prefix Sums)**:
    *   Suppose weights are $w = [1, 3]$. The total weight sum is $4$.
    *   We map:
        *   Index `0` (weight 1) to the range $[1, 1]$.
        *   Index `1` (weight 3) to the range $[2, 4]$.
    *   The upper bounds of these intervals are the prefix sums of $w$: $\text{prefixSums} = [1, 4]$.
2.  **Pick Index Strategy**:
    *   To pick a weighted random index, generate a random number `target` in the range $[1, \text{totalSum}]$.
    *   In Go, `rand.Intn(this.totalSum)` returns a value in $[0, \text{totalSum}-1]$. Adding $1$ shifts it to the desired range $[1, \text{totalSum}]$.
3.  **Binary Search (Boundary Search)**:
    *   Find the first index `i` in `prefixSums` where $\text{prefixSums}[i] \ge \text{target}$.
    *   Since all weights are positive ($w[i] \ge 1$), the `prefixSums` array is strictly increasing (sorted).
    *   Initialize `l = 0` and `r = len(prefixSums) - 1`.
    *   While `l < r`:
        *   Find midpoint `m = l + (r - l) / 2`.
        *   If `prefixSums[m] >= target`, then `m` is a candidate index. We narrow our search to the left half (including `m`): `r = m`.
        *   Otherwise (`prefixSums[m] < target`), the target lies further to the right: `l = m + 1`.
    *   Return the converged index `l`.

### Go Code

``` go
import "math/rand"

type Solution struct {
    prefixSums  []int
    totalSum    int
}

func Constructor(w []int) Solution {
    prefixSums := make([]int, len(w))
    sum := 0

    for i, weight := range w {
        sum += weight
        prefixSums[i] = sum
    }
    return Solution{
        prefixSums: prefixSums,
        totalSum: sum,
    }
}

func (this *Solution) PickIndex() int {
    // rand.Intn : [0, n)
    target := rand.Intn(this.totalSum) + 1
    l, r := 0, len(this.prefixSums)-1
    for l < r {
        m := l + (r-l)/2
        if this.prefixSums[m] >= target {
            r = m
        } else {
            l = m+1
        }
    }
    return l
}
```

### Code Efficiency

- **Time Complexity**:
    - **Constructor**: $O(n)$ — We iterate through the array of weights once to compute the prefix sums.
    - **PickIndex**: $O(\log n)$ — Each pick makes a single binary search call on the prefix sum array of size $n$.
- **Space Complexity**: $O(n)$
    - We store the prefix sums array of size $n$ in the struct.