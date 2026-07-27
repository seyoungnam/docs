# 219. Contains Duplicate II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/contains-duplicate-ii/description/)

## Solution: Sliding Window with Hash Set

To determine if the array contains duplicate elements within a maximum distance of $k$, we can maintain a sliding window of size $k$ using a Hash Set.

### Thought Process

1.  **Window Boundaries**:
    *   The current element is at index `i`.
    *   The sliding window must only contain elements from index `i - k` to `i`.
    *   If the index `i - k - 1 >= 0`, the element at `i - k - 1` has fallen outside our window boundaries. We must remove it from the Hash Set.
2.  **Duplicate Detection**:
    *   Initialize a Hash Set (`map[int]bool`) called `seen` to represent the elements within our active window.
    *   For each element `num` at index `i`:
        *   Remove the element at `i - k - 1` from `seen` if it exists.
        *   If `num` is already present in `seen`, it represents a duplicate element located at most $k$ indices away. Return `true` immediately.
        *   Otherwise, add `num` to `seen`.
3.  **Completion**:
    *   If the loop finishes without detecting duplicates in the window, return `false`.

### Go Code

``` go
func containsNearbyDuplicate(nums []int, k int) bool {
    seen := map[int]bool{}
    for i, num := range nums {
        if i-k-1 >= 0 {
            seen[nums[i-k-1]] = false
        }
        if seen[num] {
            return true
        }
        seen[num] = true
    }
    return false
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We make a single pass through the array. Map lookups, insertions, and updates run in $O(1)$ time on average.
- **Space Complexity**: $O(\min(n, k))$
    - The hash map stores at most $k$ elements at any point, representing the active sliding window.