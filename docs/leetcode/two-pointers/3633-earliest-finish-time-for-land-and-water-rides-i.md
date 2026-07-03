# 3633. Earliest Finish Time for Land and Water Rides I

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/earliest-finish-time-for-land-and-water-rides-i/description/)

## Solution: Brute Force with Pruning (Pairwise Evaluation)

To find the earliest time we can finish both a land ride and a water ride, we can evaluate every possible pair of land and water rides. For each pair, we determine the optimal order of execution based on their start times and calculate the final finish time, returning the minimum finish time found.

### Thought Process

1.  **Iterate Over All Pairs**:
    *   Let $N$ be the number of land rides and $M$ be the number of water rides.
    *   Use a nested loop to evaluate every combination of land ride $i$ and water ride $j$.
2.  **Pruning Suboptimal Pairs**:
    *   If the start time of either the land ride (`landStart`) or the water ride (`waterStart`) is already greater than or equal to the minimum finish time found so far (`res`), we can skip this pair (`continue`). Any schedule involving them will inevitably finish at a later time.
3.  **Determine Execution Order**:
    *   To minimize the idle waiting time, we should start the ride that opens earlier first.
    *   **Case 1: Land ride starts first (`landStart < waterStart`)**:
        *   The land ride starts at `landStart` and finishes at `landStart + landDuration[i]`.
        *   The water ride can start at its designated start time `waterStart` or immediately after the land ride finishes, whichever is later: `max(landStart + landDuration[i], waterStart)`.
        *   The combined finish time is: `max(landStart + landDuration[i], waterStart) + waterDuration[j]`.
    *   **Case 2: Water ride starts first (`landStart >= waterStart`)**:
        *   By symmetry, the water ride starts at `waterStart` and finishes at `waterStart + waterDuration[j]`.
        *   The land ride starts at: `max(waterStart + waterDuration[j], landStart)`.
        *   The combined finish time is: `max(waterStart + waterDuration[j], landStart) + landDuration[i]`.
4.  **Track Minimum Finish Time**:
    *   Update `res = min(res, subRes)` in each iteration.

### Go Code

``` go
import "math"

func earliestFinishTime(landStartTime []int, landDuration []int, waterStartTime []int, waterDuration []int) int {
    res := math.MaxInt32
    for i, landStart := range landStartTime {
        for j, waterStart := range waterStartTime {
            if landStart >= res || waterStart >= res {
                continue
            }
            var subRes int
            if landStart < waterStart {
                subRes = max(landStart + landDuration[i], waterStart) + waterDuration[j]
            } else {
                subRes = max(waterStart + waterDuration[j], landStart) + landDuration[i]
            }
            res = min(res, subRes)
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N \times M)$
    - We check all pairs of land and water rides, where $N$ is the number of land rides and $M$ is the number of water rides. The prune check cuts down some computations in the search space.
- **Space Complexity**: $O(1)$
    - We only track scalar variables, utilizing constant auxiliary space.