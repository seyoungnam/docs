# 167. Two Sum II - Input Array Is Sorted

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/description/)

## Solution: Two Pointers

Since the input array is already **sorted in non-decreasing order**, we can find the two numbers that sum up to the target in a single pass using two pointers without any extra space (like a hash map).

### Thought Process

1.  **Leveraging the Sorted Property**: 
    - In a sorted array, the smallest elements are at the beginning and the largest are at the end.
    - If we take the sum of the smallest (`left`) and largest (`right`) elements:
        - If the **sum == target**, we've found our pair.
        - If the **sum > target**, the current sum is too large. Since the array is sorted, the only way to decrease the sum is to move the `right` pointer to a smaller element (decrement `right`).
        - If the **sum < target**, the current sum is too small. To increase the sum, we must move the `left` pointer to a larger element (increment `left`).
2.  **Two-Pointer Strategy**:
    - Initialize `l = 0` and `r = len(numbers) - 1`.
    - Continue moving the pointers toward each other until they meet or the target is found.
3.  **1-indexed Requirement**: The problem asks for 1-indexed results, so we return `[l+1, r+1]`.

### Go Code

``` go
func twoSum(numbers []int, target int) []int {
    l, r := 0, len(numbers)-1
    for l < r {
        if numbers[l] + numbers[r] == target {
            return []int{l+1, r+1}
        } else if numbers[l] + numbers[r] > target {
            r--
        } else {
            l++
        }
    }
    return []int{}
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We traverse the array at most once with two pointers. Each element is visited at most once.
- **Space Complexity**: $O(1)$
    - We only use two integer variables (`l` and `r`) regardless of the size of the input array. No extra data structures are required.
