# 868. Binary Gap

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/binary-gap/description/)

## Solution: Bit Index Tracking

To find the maximum distance between any two adjacent set bits (`1`s) in the binary representation of `n`, we can iterate through the bits, keeping track of the index positions of the set bits we encounter.

### Thought Process

1.  **Tracking Indices**:
    *   Maintain a variable `currPos` to represent the index of the bit we are currently examining (starting at `0` for the rightmost bit).
    *   Maintain a variable `lastPos` (initialized to `-1`) to record the index of the most recently encountered set bit.
    *   Maintain `maxGap` to track the maximum distance found so far.
2.  **Iterative Bit Scan**:
    *   While $n > 0$:
        *   **Check rightmost bit**: If `n & 1 == 1` (the current bit is set):
            *   If `lastPos != -1` (we have encountered a set bit before), calculate the distance to the previous set bit: `gap = currPos - lastPos`.
            *   Update the maximum distance: `maxGap = max(maxGap, gap)`.
            *   Record the current position as the new last seen position: `lastPos = currPos`.
        *   **Shift & Increment**: Shift `n` right by 1 (`n >>= 1`) to examine the next bit, and increment the current index position `currPos++`.
3.  **Termination**:
    *   Once all set bits are processed ($n == 0$), return `maxGap`. If fewer than two set bits exist, `maxGap` remains `0`.

### Go Code

``` go
func binaryGap(n int) int {
    maxGap := 0
    lastPos := -1
    currPos := 0

    for n > 0 {
        if n&1 == 1 {
            if lastPos != -1 {
                gap := currPos - lastPos
                maxGap = max(maxGap, gap)
            }
            lastPos = currPos
        }
        n >>= 1
        currPos++
    }
    return maxGap
}
```

### Code Efficiency

- **Time Complexity**: $O(\log n)$
    - The loop runs once for each bit position in the binary representation of $n$, taking at most 32 iterations for a standard integer.
- **Space Complexity**: $O(1)$
    - The algorithm runs in-place with constant memory allocation.