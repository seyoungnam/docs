# 763. Partition Labels

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/partition-labels/description/)

You are given a string `s`. We want to partition the string into as many parts as possible so that each letter appears in at most one part.

Note that the partition is done so that after concatenating all the parts in order, the resultant string should be `s`.

Return *a list of integers representing the size of these parts*.

---

## Solution: Greedy Boundary Expansion

To ensure each letter appears in at most one partition, the boundary of our partition must extend to at least the last occurrence of any character we have encountered so far. We can solve this greedily in two linear passes.

### Thought Process

1.  **Record Last Occurrences**:
    *   Perform a first pass over the string `s` to record the last occurrence index of each character in a map: `charToLastIdx[c] = i`.
2.  **Determine Partition Boundaries**:
    *   Initialize `start := 0` and `end := 0` to represent the range of the current partition.
    *   Perform a second pass through the string `s` using an index pointer `i`:
        *   Update the end boundary of the current partition to be the maximum of `end` and the last index of the current character `c`:
            $$\text{end} = \max(\text{end}, \text{charToLastIdx[c]})$$
        *   If the index pointer reaches the end boundary (`i == end`):
            *   It means all characters encountered in the range `[start, end]` do not appear anywhere beyond index `end`.
            *   We have successfully found a valid partition of size `end - start + 1`. Append this size to `res`.
            *   Update `start = i + 1` to begin the next partition.
3.  **Result**:
    *   Return the accumulated list of partition sizes `res`.

### Go Code

``` go
func partitionLabels(s string) []int {
    // Record the last occurrence of each character
    charToLastIdx := map[rune]int{}
    for i, c := range s {
        charToLastIdx[c] = i
    }
    
    res := []int{}
    start, end := 0, 0
    
    // Determine partition boundaries
    for i, c := range s {
        end = max(end, charToLastIdx[c])
        
        // If we reach the end of the current partition
        if i == end {
            res = append(res, end - start + 1)
            start = i + 1
        }
    }
    
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Where $N$ is the length of string `s`. We make exactly two linear passes over the string.
- **Space Complexity**: $O(1)$
    - The auxiliary space is constant because the `charToLastIdx` map stores at most 26 key-value pairs (representing the lowercase English alphabet).