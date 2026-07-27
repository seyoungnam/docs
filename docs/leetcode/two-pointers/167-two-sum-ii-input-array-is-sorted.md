# 167. Two Sum II - Input Array Is Sorted

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/description/)

## Solution: Two Pointers

Since the input array is already sorted in non-decreasing order, we can solve this problem in linear time using a **two-pointer** approach without requiring any additional memory.

### Thought Process

1.  **Sorted Array Strategy**:
    *   Initialize two pointers: `l` at the beginning of the array (`0`) and `r` at the end of the array (`len(numbers) - 1`).
2.  **Iterative Convergence**:
    *   Compute the sum of elements at both pointers: `sum = numbers[l] + numbers[r]`.
    *   **Case 1 (Sum too large)**: If `sum > target`, we need a smaller sum. Since the array is sorted in ascending order, the only way to decrease the sum is to decrement the right pointer: `r--`.
    *   **Case 2 (Sum too small)**: If `sum < target`, we need a larger sum. We increment the left pointer to obtain a larger value: `l++`.
    *   **Case 3 (Target Found)**: If `sum == target`, return the index positions. Since the problem specifies 1-based indexing, return `[]int{l + 1, r + 1}`.
3.  **Guaranteed Output**:
    *   The problem description guarantees that each input has exactly one solution and you may not use the same element twice.

### Go Code

``` go
func twoSum(numbers []int, target int) []int {
    l, r := 0, len(numbers)-1
    for l < r {
        sum := numbers[l] + numbers[r]
        if sum > target {
            r--
        } else if sum < target {
            l++
        } else {
            return []int{l+1, r+1}
        }
    }
    return []int{-1, -1}
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We scan the array at most once, as either `l` increases or `r` decreases in each iteration.
- **Space Complexity**: $O(1)$
    - The search runs in-place, using only constant auxiliary variables.