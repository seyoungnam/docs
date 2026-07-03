# 3635. Earliest Finish Time for Land and Water Rides II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/earliest-finish-time-for-land-and-water-rides-ii/description/)

## Solution: Greedy Optimization ($O(N + M)$ time)

To handle larger constraints where a brute-force $O(N \times M)$ pairwise comparison would Time Limit Exceeded (TLE), we can optimize our search using a **Greedy** strategy.

### Thought Process

1.  **Reduce Search Space**:
    *   The total process must follow one of two possible orders:
        1.  **Land ride first**, followed by a **water ride**.
        2.  **Water ride first**, followed by a **land ride**.
    *   We can calculate the optimal finish time for both ordering configurations independently and return the minimum.
2.  **Greedy Order Optimization (`calcFinishTime`)**:
    *   Assume we must do ride type 1 first, then ride type 2 second.
    *   For the second ride $j$, its start time is bounded by:
        $$\text{boardingTime} = \max(\text{firstRideFinish}, \text{start2}[j])$$
    *   To minimize this boarding time (and thus the final finish time), we should greedily make $\text{firstRideFinish}$ as small as possible. Since any valid ride of type 1 can serve as the first ride, we can find the absolute earliest finish time of any single ride of type 1:
        $$\text{minFirstFinish} = \min_{i} (\text{start1}[i] + \text{dur1}[i])$$
    *   Once we find this global minimum finish time `minFirstFinish` (requiring a single pass of size $N$), we can treat it as a constant constraint.
    *   We then iterate through all rides of type 2 (a single pass of size $M$) to find the minimum finish time:
        $$\text{finalFinish} = \max(\text{minFirstFinish}, \text{start2}[j]) + \text{dur2}[j]$$
3.  **Result**:
    *   Run this greedy procedure twice: `calcFinishTime(Land, Water)` and `calcFinishTime(Water, Land)`.
    *   Return the minimum of the two results.

### Go Code

``` go
import "math"

func earliestFinishTime(landStartTime []int, landDuration []int, waterStartTime []int, waterDuration []int) int {
    landFirst := calcFinishTime(landStartTime, landDuration, waterStartTime, waterDuration)
    waterFirst := calcFinishTime(waterStartTime, waterDuration, landStartTime, landDuration)
    return min(landFirst, waterFirst)
}

func calcFinishTime(start1, dur1, start2, dur2 []int) int {
    minFirstFinish := math.MaxInt32
    for i := 0; i < len(start1); i++ {
        minFirstFinish = min(minFirstFinish, start1[i]+dur1[i])
    }

    minSecondFinish := math.MaxInt32
    for j := 0; j < len(start2); j++ {
        boardingTime := max(minFirstFinish, start2[j])
        finalFinish := boardingTime + dur2[j]

        minSecondFinish = min(minSecondFinish, finalFinish)
    }
    return minSecondFinish
}
```

### Code Efficiency

- **Time Complexity**: $O(N + M)$
    - We run `calcFinishTime` twice. Inside `calcFinishTime`, we perform two sequential linear scans of sizes $N$ and $M$. This eliminates the need for pairwise nesting, making it optimal for large inputs.
- **Space Complexity**: $O(1)$
    - We only track scalar variables, utilizing constant auxiliary space.