# 80. Remove Duplicates from Sorted Array II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/description/)

## Solution: Two Pointers (Slow and Fast)

We can modify the sorted array in-place using two pointers: a slow write pointer `targetIdx` that tracks where the next valid element should be placed, and a fast scan pointer `i` that iterates through the array. 

### Thought Process

1.  **Duplicate Constraint**:
    *   Each unique element is allowed to appear at most twice.
    *   Therefore, the first two elements of the array (`nums[0]` and `nums[1]`) are always valid. 
    *   If `len(nums) <= 2`, return `len(nums)` immediately.
2.  **Two Pointers Initialization**:
    *   Set the write pointer `targetIdx = 2` (since the first two elements are already processed).
    *   Scan the array starting from index `i = 2` using the fast pointer.
3.  **Scan and Filter**:
    *   For each element `nums[i]`, compare it with the element written two positions back: `nums[targetIdx - 2]`.
    *   If `nums[i] != nums[targetIdx - 2]`, writing `nums[i]` to the current target position is safe and will not violate the "at most twice" duplicate rule:
        *   Write it: `nums[targetIdx] = nums[i]`.
        *   Increment the slow pointer: `targetIdx++`.
4.  **Result**:
    *   After the scan, the valid length of the modified array is `targetIdx`. Return `targetIdx`.

### Go Code

``` go
func removeDuplicates(nums []int) int {
    if len(nums) <= 2 {
        return len(nums)
    }
    targetIdx := 2
    for i := 2; i < len(nums); i++ {
        if nums[i] != nums[targetIdx-2] {
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
    - The algorithm runs in-place, requiring constant auxiliary space.