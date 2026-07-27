# 303. Range Sum Query - Immutable

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/range-sum-query-immutable/description/)

## Solution: Prefix Sum Precomputation

The problem requires querying the sum of elements in a range $[left, right]$ multiple times. By precomputing the prefix sums of the array, we can answer each range sum query in constant $O(1)$ time.

### Thought Process

1.  **Naive Approach**:
    *   Iterate from `left` to `right` and sum the elements for each query.
    *   **Time Complexity**: $O(n)$ per query, which is inefficient when there are many queries.
2.  **Prefix Sum Technique**:
    *   Create an array `prefixSum` where `prefixSum[i]` stores the sum of all elements from index `0` to `i` (inclusive):
        $$\text{prefixSum}[i] = \sum_{j=0}^{i} \text{nums}[j]$$
    *   This precomputation takes $O(n)$ time and is done once during construction:
        *   For $i = 0$: `prefixSum[0] = nums[0]`
        *   For $i > 0$: `prefixSum[i] = prefixSum[i-1] + nums[i]`
3.  **Querying in $O(1)$**:
    *   To find the sum of range $[left, right]$:
        *   If $left = 0$, the sum is simply `prefixSum[right]`.
        *   If $left > 0$, the sum is obtained by subtracting the prefix sum just before the start of the range from the prefix sum at the end of the range:
            $$\text{SumRange}(left, right) = \text{prefixSum}[right] - \text{prefixSum}[left - 1]$$

### Go Code

``` go
type NumArray struct {
    prefixSum []int
}

func Constructor(nums []int) NumArray {
    prefixSum := make([]int, len(nums))
    for i, num := range nums {
        if i == 0 {
            prefixSum[0] = num
        } else {
            prefixSum[i] = prefixSum[i-1] + num
        }
    }
    return NumArray {
        prefixSum: prefixSum,
    }
}

func (this *NumArray) SumRange(left int, right int) int {
    if left == 0 {
        return this.prefixSum[right]
    }
    return this.prefixSum[right] - this.prefixSum[left-1]
}

/**
 * Your NumArray object will be instantiated and called as such:
 * obj := Constructor(nums);
 * param_1 := obj.SumRange(left,right);
 */
```

### Code Efficiency

- **Time Complexity**:
    - **Constructor**: $O(n)$ where $n$ is the length of the array, since we iterate through the input array once.
    - **SumRange**: $O(1)$ because it only performs a basic subtraction and slice index lookup.
- **Space Complexity**: $O(n)$
    - We use an auxiliary slice of size $n$ to store the prefix sums.