# 704. Binary Search

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/binary-search/description/)

## Solution: Standard Binary Search (Closed Interval)

We can find the target in a sorted array by repeatedly dividing the search space in half. Using a closed interval `[l, r]` (inclusive of both boundaries) ensures that we check every potential element.

### Thought Process

1.  **Initialize Boundaries**: Set `l = 0` (left boundary) and `r = len(nums) - 1` (right boundary).
2.  **Search Loop**: Iterate while the search space is non-empty: `l <= r`.
3.  **Find Midpoint**: Calculate the mid index `m = l + (r - l) / 2` (using subtraction to prevent integer overflow).
4.  **Narrow Search Space**:
    *   If `nums[m] < target`: The target lies to the right. Shift the left boundary `l = m + 1`.
    *   If `nums[m] > target`: The target lies to the left. Shift the right boundary `r = m - 1`.
    *   If `nums[m] == target`: We found the target! Return the index `m`.
5.  **Not Found**: If the loop ends without a match (i.e. `l > r`), the target is not present in the array. Return `-1`.

### Go Code

``` go
func search(nums []int, target int) int {
    l, r := 0, len(nums)-1
    for l <= r {
        m := l + (r-l)/2
        if nums[m] < target {
            l = m + 1
        } else if nums[m] > target {
            r = m - 1
        } else {
            return m
        }
    }
    return -1
}
```

### Code Efficiency

- **Time Complexity**: $O(\log n)$
    - The search space is divided in half during each iteration, taking logarithmic time.
- **Space Complexity**: $O(1)$
    - We only use three integer variables (`l`, `r`, `m`), requiring $O(1)$ auxiliary space.
