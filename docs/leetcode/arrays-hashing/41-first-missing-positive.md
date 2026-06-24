# 41. First Missing Positive

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/first-missing-positive/description/)

## Solution: Index Placement (Cycle Sort)

To find the first missing positive integer in $O(n)$ time and $O(1)$ auxiliary space, we can treat the input array as a custom hash table. We place each valid positive integer `x` (where $1 \le x \le n$) at its correct index `x - 1` (e.g., `1` at index `0`, `2` at index `1`, etc.). In a second pass, the first index `i` that does not contain `i + 1` identifies the first missing positive integer.

### Thought Process

1.  **Search Boundaries**:
    - The first missing positive integer must lie in the range $[1, n + 1]$, where $n$ is the length of `nums`.
2.  **In-Place Cycle Sort**:
    - Iterate through the array using a pointer `i`.
    - For the current value `nums[i]`, its target index is `targetIdx = nums[i] - 1`.
    - If `targetIdx` is within bounds (`0 <= targetIdx < n`) and the element already at the destination is different (`nums[targetIdx] != nums[i]`), swap the two elements.
    - **Note**: After a swap, do not increment `i`. We need to re-evaluate the newly swapped value at index `i`.
    - If `nums[i]` does not satisfy the criteria (e.g., it is negative, larger than $n$, or is a duplicate of a value already in its correct spot), simply increment `i`.
3.  **Linear Verification**:
    - Iterate through the processed array. The first index `i` where `nums[i] != i + 1` is our missing positive number: return `i + 1`.
    - If all elements are correctly positioned, then numbers $1$ to $n$ are present. Return `n + 1`.

### Go Code

``` go
func firstMissingPositive(nums []int) int {
    n := len(nums)
    for i := 0; i < n; {
        targetIdx := nums[i] - 1
        if 0 <= targetIdx && targetIdx < n && nums[targetIdx] != nums[i] {
            nums[i], nums[targetIdx] = nums[targetIdx], nums[i]
        } else {
            i++
        }
    }

    for i, v := range nums {
        if v != i+1 {
            return i+1
        }
    }
    return n+1
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - Although the first loop does not increment `i` on swaps, every swap places at least one element into its correct, permanent position. Since there are $n$ positions, we can perform at most $n - 1$ swaps across the entire execution. Thus, the time complexity of the sorting pass is $O(n)$. The second verification pass is also $O(n)$.
- **Space Complexity**: $O(1)$
    - All swaps are performed in-place using the input array, resulting in constant auxiliary space.