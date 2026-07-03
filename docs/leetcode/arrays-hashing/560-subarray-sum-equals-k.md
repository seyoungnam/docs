# 560. Subarray Sum Equals K

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/subarray-sum-equals-k/description/)

## Solution: Prefix Sum with Hash Map (Frequency Map)

We can solve this problem in linear time by using a running **Prefix Sum** combined with a **Hash Map** that stores the frequency of each prefix sum seen so far.

### Thought Process

1.  **Mathematical Relation**:
    *   The sum of elements in a subarray `nums[i...j]` is calculated as:
        $$\text{sum}(i, j) = \text{prefixSum}[j] - \text{prefixSum}[i-1]$$
    *   We want to find subarrays where $\text{sum}(i, j) = k$. Rearranging the equation:
        $$\text{prefixSum}[j] - \text{prefixSum}[i-1] = k \implies \text{prefixSum}[i-1] = \text{prefixSum}[j] - k$$
    *   This means that as we compute the running `prefixSum` at index $j$, we need to check how many times the value `prefixSum - k` has occurred at some previous index $i-1$.
2.  **Frequency Map Setup**:
    *   Maintain a hash map `count` mapping each prefix sum to its frequency: `map[int]int`.
    *   **Base Case**: Initialize `count[0] = 1`. This accounts for subarrays starting from index $0$ that sum up to exactly $k$ (where `prefixSum - k == 0`).
3.  **Iteration**:
    *   Initialize `prefixSum = 0` and `res = 0`.
    *   For each element in `nums`:
        *   Accumulate it into `prefixSum`.
        *   Look up `prefixSum - k` in the map and add its frequency to `res` (since Go map lookups return the zero-value `0` if the key is missing, this is safe).
        *   Increment the frequency of `prefixSum` in our map: `count[prefixSum]++`.

### Go Code

``` go
func subarraySum(nums []int, k int) int {
    count := make(map[int]int)
    count[0] = 1
    prefixSum := 0
    res := 0

    for i := range nums {
        prefixSum += nums[i]
        res += count[prefixSum-k]
        count[prefixSum]++
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We traverse the array of length $n$ exactly once. Map lookups and inserts run in $O(1)$ constant time on average.
- **Space Complexity**: $O(n)$
    - In the worst case (where every prefix sum is unique), the map will store up to $n + 1$ keys.