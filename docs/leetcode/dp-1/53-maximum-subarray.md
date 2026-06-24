# 53. Maximum Subarray

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/maximum-subarray/description/)

## Solution: Kadane's Algorithm (Dynamic Programming)

To find the contiguous subarray with the largest sum, we can use Kadane's algorithm. At each index, we decide whether to add the current element to the existing running subarray sum or start a new subarray beginning with the current element.

### How to Recognize This is a DP Problem

1.  **Optimization Goal**: We want to find the "maximum" sum.
2.  **Overlapping Subproblems**: The maximum subarray sum ending at index `i` is dependent on the maximum subarray sum ending at index `i - 1`.
3.  **Optimal Substructure**: The optimal solution for index `i` is `dp[i] = max(dp[i - 1] + nums[i], nums[i])`. We can optimize this by keeping only a single running sum variable rather than a full DP table.

### Thought Process

1.  **Running Sum**: Maintain `curSum` representing the maximum sum of a subarray ending at the current position.
2.  **Global Max**: Maintain `maxSum` to track the overall maximum sum seen so far.
3.  **State Transition**: For each number `n` in `nums`, update:
    $$\text{curSum} = \max(\text{curSum} + n, n)$$
4.  **Update Max**: Update `maxSum = max(maxSum, curSum)`.

### Go Code

``` go
import "math"

func maxSubArray(nums []int) int {
    curSum, maxSum := math.MinInt32, math.MinInt32
    for _, n := range nums {
        curSum = max(curSum + n, n)
        maxSum = max(maxSum, curSum)
    }
    return maxSum
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We make a single pass through the array.
- **Space Complexity**: $O(1)$
    - We only use two integer variables (`curSum` and `maxSum`) for state tracking.

---

## Alternative Solution: Divide and Conquer

We can also solve this problem recursively using a divide and conquer strategy. We split the array into two halves, recursively compute the maximum subarray sum in each half, and compute the maximum subarray sum that crosses the midpoint.

### Thought Process

1.  **Base Case**: If the subarray has only one element (`l == r`), its maximum sum is the element itself.
2.  **Divide**: Calculate the midpoint `m = l + (r - l) / 2`.
3.  **Conquer**: Recursively find the maximum subarray sum in:
    - The left half: `leftMax` from `l` to `m`.
    - The right half: `rightMax` from `m + 1` to `r`.
4.  **Combine (Crossing Subarray)**:
    - Find the maximum sum of a contiguous subarray starting at `m` and extending left to `l`.
    - Find the maximum sum of a contiguous subarray starting at `m + 1` and extending right to `r`.
    - The maximum crossing sum is `crossMax = maxLeftSum + maxRightSum`.
5.  **Return**: The result is the maximum of the three values: $\max(\text{leftMax}, \text{rightMax}, \text{crossMax})$.

### Go Code

``` go
import "math"

func maxSubArray(nums []int) int {
    return divideAndConquer(nums, 0, len(nums)-1)
}

func divideAndConquer(nums []int, l, r int) int {
    if l == r {
        return nums[l]
    }
    m := l + (r-l)/2
    leftMax := divideAndConquer(nums, l, m)
    rightMax := divideAndConquer(nums, m+1, r)
    crossMax := findCrossMax(nums, l, m, r)
    return max(max(leftMax, rightMax), crossMax)
}

func findCrossMax(nums []int, l, m, r int) int {
    leftSum := 0
    maxLeftSum := math.MinInt32
    for i := m; i >= l; i-- {
        leftSum += nums[i]
        maxLeftSum = max(maxLeftSum, leftSum)
    }

    rightSum := 0
    maxRightSum := math.MinInt32
    for i := m+1; i <= r; i++ {
        rightSum += nums[i]
        maxRightSum = max(maxRightSum, rightSum)
    }
    return maxLeftSum + maxRightSum
}
```

### Code Efficiency

- **Time Complexity**: $O(n \log n)$
    - Splitting the array takes $O(\log n)$ recursive depth, and at each level of recursion, we perform a linear scan of size $O(n)$ to find the crossing sum. The recurrence is $T(n) = 2T(n/2) + O(n)$, which solves to $O(n \log n)$.
- **Space Complexity**: $O(\log n)$
    - The height of the recursion tree is $O(\log n)$, which determines the size of the recursion stack.