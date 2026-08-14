# 485. Max Consecutive Ones

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/max-consecutive-ones/description/)

Given a binary array `nums`, return the maximum number of consecutive `1`'s in the array.

---

## Solution: Single-Pass Linear Scan

We can solve this problem in a single pass by maintaining a counter for the current streak of `1`s and updating the maximum streak encountered so far.

### Thought Process

1.  **Core Intuition**:
    *   As we traverse the binary array from left to right, we keep track of how many consecutive `1`s we have encountered in the current sequence.
2.  **Tracking Variables**:
    *   `cnt`: Tracks the length of the current streak of consecutive `1`s.
    *   `res`: Tracks the maximum streak observed across the entire array.
3.  **Iteration Steps**:
    *   For each element `num` in `nums`:
        *   **If `num == 0`**: The consecutive sequence of `1`s is broken. Reset `cnt = 0`.
        *   **If `num == 1`**: Increment the current streak `cnt++` and update the global maximum `res = max(res, cnt)`.
4.  **Result**:
    *   After traversing all elements, `res` will hold the maximum consecutive `1`s found.

### Go Code

``` go
func findMaxConsecutiveOnes(nums []int) int {
    res, cnt := 0, 0
    for _, num := range nums {
        if num == 0 {
            cnt = 0
        } else {
            cnt++
            res = max(res, cnt)
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - Where $n$ is the length of `nums`. We perform a single pass through the array.
- **Space Complexity**: $O(1)$
    - We only maintain two integer variables (`res` and `cnt`), using constant auxiliary memory.