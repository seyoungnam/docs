# 26. Remove Duplicates from Sorted Array

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/remove-duplicates-from-sorted-array/description/)

## Solution: Two Pointers (Slow and Fast)

We can modify the sorted array in-place using two pointers: a slow write pointer `targetIdx` that tracks where the next unique element should be written, and a fast scan pointer `i` that iterates through the array checking for new unique values.

### Thought Process

1.  **Two Pointers**: 
    *   Initialize `targetIdx = 1` (slow pointer). Since the first element `nums[0]` is always unique by default, we start writing from index 1.
    *   Initialize a fast pointer `i = 1` to scan the rest of the array.
2.  **Scan for Uniques**:
    *   For each element `nums[i]`, compare it with the last written unique element, which resides at index `targetIdx - 1`.
    *   If `nums[i] != nums[targetIdx - 1]`, we have encountered a new unique value.
    *   Write the unique value to the target slot: `nums[targetIdx] = nums[i]`.
    *   Increment the slow pointer: `targetIdx++` to prepare for the next unique element.
3.  **Result**:
    *   After the scan, the number of unique elements is equal to `targetIdx`. Return `targetIdx`.

### Go Code

``` go
func removeDuplicates(nums []int) int {
    targetIdx := 1
    for i := 1; i < len(nums); i++ {
        if nums[i] != nums[targetIdx-1] {
            nums[targetIdx] = nums[i]
            targetIdx++
        }
    }
    return targetIdx
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We traverse the array of length $n$ exactly once using the fast scan pointer `i`.
- **Space Complexity**: $O(1)$
    - We modify the array in-place, requiring $O(1)$ auxiliary space.
