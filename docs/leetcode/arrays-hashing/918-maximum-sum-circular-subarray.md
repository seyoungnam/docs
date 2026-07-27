# 918. Maximum Sum Circular Subarray

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/maximum-sum-circular-subarray/description/)

## Solution: Dual Kadane's Algorithm (Min/Max Tracking)

To find the maximum subarray sum in a circular array, we can think of the problem in two cases:
1.  **Case 1 (No Wrap-around)**: The maximum sum subarray is completely in the middle of the array (does not cross the boundary). This is calculated using standard Kadane's algorithm (`totMax`).
2.  **Case 2 (Wrap-around)**: The maximum sum subarray wraps around the end of the array to the beginning. 
    *   If the subarray wraps around, the elements left out in the middle form a contiguous, non-wrapping subarray.
    *   To maximize the wrapping subarray sum, we must **minimize** the sum of this middle subarray (`totMin`).
    *   The sum of the wrapping subarray is then:
        $$\text{Wrapping Max} = \text{totSum} - \text{totMin}$$

By running Kadane's algorithm to find both the maximum and minimum subarray sums simultaneously, we can determine the answer in a single pass.

### Thought Process

1.  **Dual State Variables**:
    *   Track the running maximum (`curMax`) and global maximum (`totMax`) initialized to `nums[0]`.
    *   Track the running minimum (`curMin`) and global minimum (`totMin`) initialized to `nums[0]`.
    *   Track the overall array sum (`totSum`).
2.  **Iterative Scanning**:
    *   For each element `num` in the array:
        *   Update the maximum running subarray sum: `curMax = max(curMax + num, num)`.
        *   Update the minimum running subarray sum: `curMin = min(curMin + num, num)`.
        *   Update the global maximum `totMax = max(totMax, curMax)` and global minimum `totMin = min(totMin, curMin)`.
        *   Add `num` to `totSum`.
3.  **Special Edge Case**:
    *   If all elements in the array are negative, the minimum subarray sum `totMin` will be equal to the total array sum `totSum`. 
    *   Subtracting `totMin` from `totSum` would yield `0` (corresponding to an empty subarray). Since the subarray must be non-empty, we cannot choose this option.
    *   Thus, if `totSum == totMin`, we return `totMax` (which will be the maximum single negative element).
4.  **Final Comparison**:
    *   Otherwise, return the maximum of Case 1 and Case 2:
        $$\max(\text{totMax}, \text{totSum} - \text{totMin})$$

### Go Code

``` go
func maxSubarraySumCircular(nums []int) int {
    curMax, totMax := 0, nums[0]
    curMin, totMin := 0, nums[0]
    totSum := 0

    for _, num := range nums {
        curMax = max(curMax+num, num)
        curMin = min(curMin+num, num)

        totMax = max(totMax, curMax)
        totMin = min(totMin, curMin)
        totSum += num
    }

    if totSum == totMin {
        return totMax
    }
    return max(totMax, totSum-totMin)
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We perform a single pass through the array.
- **Space Complexity**: $O(1)$
    - Only a constant number of tracking variables are used.