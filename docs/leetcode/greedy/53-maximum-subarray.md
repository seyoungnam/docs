# 53. Maximum Subarray

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/maximum-subarray/description/)

## Solution: Kadane's Algorithm

To find the contiguous subarray with the largest sum, we can use **Kadane's Algorithm**. At each index, we decide whether to add the current element to the existing running subarray sum or start a new subarray beginning with the current element.

### Thought Process

1.  **State Initialization**:
    *   Initialize the running sum (`sum`) and the overall maximum sum (`res`) to the first element of the array: `nums[0]`.
2.  **Iterative Transition**:
    *   Iterate through the array starting from index 1.
    *   For each element `num`, decide whether to:
        *   Extend the current running subarray sum: `sum + num`.
        *   Start a new subarray starting at the current element: `num`.
    *   This choice is resolved by taking the maximum of the two options:
        $$\text{sum} = \max(\text{sum} + \text{num}, \text{num})$$
3.  **Update Global Max**:
    *   After updating the running sum at each index, update the overall maximum sum:
        $$\text{res} = \max(\text{res}, \text{sum})$$
4.  **Completion**:
    *   Return `res` once the entire array has been processed.

### Go Code

``` go
func maxSubArray(nums []int) int {
    res, sum := nums[0], nums[0]
    for i := 1; i < len(nums); i++ {
        num := nums[i]
        sum = max(sum + num, num)
        res = max(res, sum)
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We make a single pass through the array.
- **Space Complexity**: $O(1)$
    - The algorithm runs in-place, using only constant auxiliary variables.