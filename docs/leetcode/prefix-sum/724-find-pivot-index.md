# 724. Find Pivot Index

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/find-pivot-index/description/)

## Solution: Sum Balancing (Prefix Sum on the Fly)

We can find the pivot index efficiently in $O(n)$ time using a two-pass algorithm that tracks the running left sum and compares it with the right sum calculated on the fly.

### Thought Process

1.  **Mathematical Relation**:
    *   Let $S$ be the total sum of all numbers in the array.
    *   For any index `i`, let $L_i$ represent the sum of elements strictly to the left of `i`.
    *   Let $R_i$ represent the sum of elements strictly to the right of `i`.
    *   The total sum can be written as:
        $$L_i + nums[i] + R_i = S$$
    *   Therefore, the right sum is:
        $$R_i = S - L_i - nums[i]$$
2.  **Pivot Condition**:
    *   An index `i` is a pivot if the left sum equals the right sum ($L_i = R_i$).
    *   Substituting $R_i$, the condition becomes:
        $$L_i = S - L_i - nums[i]$$
3.  **Algorithm**:
    *   **First Pass**: Compute the total sum $S$ of all elements.
    *   **Second Pass**: Iterate through the array with a running left sum $L_i$ (initialized to `0`).
        *   At each index `i`, check if $L_i == S - L_i - nums[i]$. If true, return `i` immediately. Since we check from left to right, this guarantees we return the **leftmost** pivot index.
        *   Update the running left sum: $L_i = L_i + nums[i]$.
    *   If no pivot index is found after completing the loop, return `-1`.

### Go Code

``` go
func pivotIndex(nums []int) int {
    totalSum := 0
    for _, num := range nums {
        totalSum += num
    }
    leftSum := 0
    for i, num := range nums {
        if leftSum == totalSum-leftSum-num {
            return i
        }
        leftSum += num
    }
    return -1    
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We perform two linear passes over the array: one to compute `totalSum` and one to check each index.
- **Space Complexity**: $O(1)$
    - We only use constant auxiliary variables (`totalSum` and `leftSum`).