# 1899. Merge Triplets to Form Target Triplet

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/merge-triplets-to-form-target-triplet/description/)

A **triplet** is an array of three integers. You are given a 2D integer array `triplets`, where `triplets[i] = [a_i, b_i, c_i]` describes the $i$-th triplet. You are also given an integer array `target = [x, y, z]` that describes the triplet you want to obtain.

To obtain `target`, you may apply the following operation on `triplets` **any number of times** (possibly zero):
*   Choose two indices `i` and `j` ($i \ne j$) and update `triplets[j]` to be `[max(a_i, a_j), max(b_i, b_j), max(c_i, c_j)]`.

Return `true` *if it is possible to obtain the* `target` *triplet, or* `false` *otherwise*.

---

## Solution: Greedy Filter & Match

Since the merge operation uses the element-wise `max` function, once an element in our running merged triplet exceeds the target's element, it can never be reduced. Therefore, we must discard any triplet containing any value greater than the corresponding target value.

### Thought Process

1.  **Discard Invalid Triplets**:
    *   For each triplet `t`, if `t[0] > target[0]`, `t[1] > target[1]`, or `t[2] > target[2]`, it is **invalid**. If we merge this triplet, we will overshoot the target. We skip it.
2.  **Match Target Positions**:
    *   For the remaining valid triplets, we check if they contain values matching the target at position `0`, `1`, or `2`.
    *   Use a map `good` (acting as a set) to record which indices ($0, 1, \text{or } 2$) have been matched.
    *   For each valid triplet `t`:
        *   If `t[i] == target[i]`, we set `good[i] = true`.
3.  **Result**:
    *   If we successfully match all three positions (`len(good) == 3`), we can merge those specific matching triplets together to form the exact target. Return `true`.
    *   Otherwise, return `false`.

### Go Code

``` go
func mergeTriplets(triplets [][]int, target []int) bool {
    good := make(map[int]bool)

    for _, t := range triplets {
        // Discard triplets that have any value greater than target
        if t[0] > target[0] || t[1] > target[1] || t[2] > target[2] {
            continue
        }
        
        // Find matching values for target indices
        for i, v := range t {
            if v == target[i] {
                good[i] = true
            }
        }
    }
    
    // If all three indices find matches, the target can be formed
    return len(good) == 3
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Where $N$ is the number of triplets. We perform a single pass over the array of triplets, doing constant time operations for each.
- **Space Complexity**: $O(1)$
    - The auxiliary space is constant because the `good` map stores at most 3 key-value pairs (for indices 0, 1, and 2).