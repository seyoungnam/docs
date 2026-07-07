# 27. Remove Element

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/remove-element/description/)

## Solution: Two Pointers (In-Place Overwriting)

To remove all instances of a value `val` from `nums` in-place, we can use a two-pointer technique. One pointer reads elements sequentially, while a second pointer writes non-target elements to the front of the array.

### Thought Process

1.  **In-Place Modification**:
    *   We must modify the input slice `nums` in-place without allocating extra space for another array.
2.  **Two Pointers Setup**:
    *   We maintain a write pointer `k` initialized to `0`. This pointer tracks the next index where a valid element (not equal to `val`) should be placed.
3.  **Iteration**:
    *   Iterate through the slice `nums` using a loop.
    *   For each element `num`:
        *   **Valid Element (`num != val`)**: Overwrite `nums[k]` with `num` and increment `k` by 1.
        *   **Target Element (`num == val`)**: Skip it.
4.  **Result**:
    *   Once the iteration is complete, the first `k` elements of `nums` will contain all elements not equal to `val`. Return `k`.

### Go Code

``` go
func removeElement(nums []int, val int) int {
    k := 0
    for _, num := range nums {
        if num != val {
            nums[k] = num
            k++
        }
    }
    return k
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We iterate through the array of length $n$ exactly once, performing constant-time comparisons and assignments.
- **Space Complexity**: $O(1)$
    - The array is modified in-place, requiring constant auxiliary memory.