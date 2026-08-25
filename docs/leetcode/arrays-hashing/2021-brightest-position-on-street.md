# 2021. Brightest Position on Street

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/brightest-position-on-street/description/)

## Solution: Line Sweep Algorithm (Difference Array)

We are given a list of street lights where each light is specified by `[position, range]`. A light located at `position` with illumination `range` covers the closed interval `[position - range, position + range]`. We want to find the position on the street that receives the maximum total brightness (is covered by the maximum number of lights). If multiple positions share the maximum brightness, we return the smallest position.

### Thought Process

1. **Interval Representation**:
   - For each light `[pos, r]`, the illuminated range is `[left, right] = [pos - r, pos + r]`.
   - Updating every individual coordinate in `[left, right]` directly is inefficient if coordinates or ranges are large.

2. **Line Sweep / Difference Array Technique**:
   - Use a hash map `diff` to record changes in brightness at specific transition points (boundaries):
     - At coordinate `left = pos - r`, illumination starts: `diff[left]++`.
     - At coordinate `right + 1 = pos + r + 1`, illumination ends: `diff[right + 1]--`.

3. **Sorting & Prefix Sum Sweeping**:
   - Extract all unique transition points (keys) from the `diff` map into a slice `keys`.
   - Sort `keys` in ascending order.
   - Sweep through `keys` from left to right, maintaining a running total `curFreq` (prefix sum of `diff[pos]`):
     - Add `diff[pos]` to `curFreq`.
     - If `curFreq > maxFreq`, record the new maximum brightness `maxFreq = curFreq` and set `res = pos`.
     - Using strict inequality `curFreq > maxFreq` ensures that in case of a tie in maximum brightness, the smallest coordinate position is naturally preserved.

### Go Code

``` go
import "sort"

func brightestPosition(lights [][]int) int {
    if len(lights) == 0 {
        return 0
    }

    diff := map[int]int{}
    for _, light := range lights {
        left := light[0] - light[1]
        right := light[0] + light[1] + 1
        diff[left]++
        diff[right]--
    }

    keys := make([]int, 0, len(diff))
    for key := range diff {
        keys = append(keys, key)
    }
    sort.Ints(keys)

    curFreq, maxFreq := 0, 0
    res := keys[0]
    for _, pos := range keys {
        curFreq += diff[pos]
        if curFreq > maxFreq {
            maxFreq = curFreq
            res = pos
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N \log N)$
    - Where $N$ is the number of street lights. Creating boundary points takes $O(N)$ time. Sorting the $O(N)$ unique transition points takes $O(N \log N)$ time. Iterating through the sorted keys takes $O(N)$ time.
- **Space Complexity**: $O(N)$
    - The hash map `diff` and the slice `keys` store at most $2N$ boundary entries.