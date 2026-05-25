# 26. Remove Duplicates from Sorted Array

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/remove-duplicates-from-sorted-array/description/)

## Solution: Two Pointers (Slow and Fast)

We can modify the sorted array in-place using two pointers: a slow pointer `i` that tracks the position of the last unique element found, and a fast pointer `j` that scans the array for new unique values.

### Thought Process

1.  **Two Pointers**: Initialize `i = 0` (slow pointer) to track the last unique element's slot.
2.  **Scan for Uniques**: Iterate a fast pointer `j` from `0` to the end of the array:
    *   If `nums[i] != nums[j]`, we have encountered a new unique element.
    *   Copy this new unique element to the next slot: `nums[i+1] = nums[j]`.
    *   Increment the slow pointer `i++`.
3.  **Result**: The number of unique elements in the array is `i + 1`.

### Go Code

``` go
func removeDuplicates(nums []int) int {
    i := 0
    for j := 0; j < len(nums); j++ {
        if nums[i] != nums[j] {
            nums[i+1] = nums[j]
            i++
        }
    }
    return i+1
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We traverse the array of length $n$ exactly once using the fast pointer `j`.
- **Space Complexity**: $O(1)$
    - We modify the array in-place, requiring $O(1)$ auxiliary space.
