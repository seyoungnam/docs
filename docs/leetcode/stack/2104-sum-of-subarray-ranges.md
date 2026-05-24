# 2104. Sum of Subarray Ranges

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/sum-of-subarray-ranges/description/)

## Solution: Subarray Min/Max Simulation

For moderate array sizes ($n \le 1000$), we can iterate through all possible subarrays using two nested loops, keeping track of the minimum and maximum elements on the fly.

### Thought Process

1.  **Nested Loops**: Let `i` be the start of the subarray and `j` be the end.
2.  **Maintain Extremes**: For each start position `i`, initialize `minVal` and `maxVal` to `nums[i]`.
3.  **Expand Subarray**: As we increment `j`, update the extremes:
    *   `minVal = min(minVal, nums[j])`
    *   `maxVal = max(maxVal, nums[j])`
4.  **Accumulate**: Add the range `maxVal - minVal` to the running sum.

### Go Code

``` go
func subArrayRanges(nums []int) int64 {
    var res int64
    n := len(nums)
    
    for i := 0; i < n; i++ {
        minVal, maxVal := nums[i], nums[i]
        for j := i; j < n; j++ {
            minVal = min(minVal, nums[j])
            maxVal = max(maxVal, nums[j])
            res += int64(maxVal - minVal)
        }
    }
    
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n^2)$
    - We check all $n(n+1)/2$ subarrays, doing $O(1)$ work for each.
- **Space Complexity**: $O(1)$
    - We only use a few tracking variables.

---

## Optimized Solution: Monotonic Stack (Linear Time)

For larger arrays ($n \le 10^5$), we must optimize this to $O(n)$ time. We can use the mathematical identity:
$\sum \text{Range} = \sum \text{Max} - \sum \text{Min}$
This allows us to find the sum of all subarray maximums and minimums independently in $O(n)$ time using a **monotonic stack**.

### Thought Process

1.  **Subarray Contribution**: For each element `nums[i]`, we want to find how many subarrays it acts as the minimum (or maximum) element.
2.  **Find Boundaries**:
    *   For the minimum: find the index of the first smaller element to the left (`left`) and the first smaller/equal element to the right (`right`).
    *   For the maximum: find the index of the first larger element to the left and the first larger/equal element to the right.
3.  **Calculate Subarrays**: The number of subarrays where `nums[i]` is the minimum (or maximum) is `(i - left) * (right - i)`.
4.  **One-Side Strict Inequality**: We use strict inequality on one side and non-strict on the other (e.g., `<` on left, `<=` on right) to handle duplicate values correctly without double-counting.

### Go Code

``` go
func subArrayRanges(nums []int) int64 {
    return sumSubarrayMax(nums) - sumSubarrayMin(nums)
}

func sumSubarrayMin(nums []int) int64 {
    n := len(nums)
    left := make([]int, n)
    right := make([]int, n)
    
    // Initialize defaults
    for i := range right {
        right[i] = n
        left[i] = -1
    }
    
    // Monotonic increasing stack to find first smaller element
    stack := []int{}
    for i := 0; i < n; i++ {
        for len(stack) > 0 && nums[stack[len(stack)-1]] > nums[i] {
            popIdx := stack[len(stack)-1]
            stack = stack[:len(stack)-1]
            right[popIdx] = i
        }
        if len(stack) > 0 {
            left[i] = stack[len(stack)-1]
        }
        stack = append(stack, i)
    }
    
    var sum int64
    for i := 0; i < n; i++ {
        count := int64(i - left[i]) * int64(right[i] - i)
        sum += count * int64(nums[i])
    }
    return sum
}

func sumSubarrayMax(nums []int) int64 {
    n := len(nums)
    left := make([]int, n)
    right := make([]int, n)
    
    // Initialize defaults
    for i := range right {
        right[i] = n
        left[i] = -1
    }
    
    // Monotonic decreasing stack to find first larger element
    stack := []int{}
    for i := 0; i < n; i++ {
        for len(stack) > 0 && nums[stack[len(stack)-1]] < nums[i] {
            popIdx := stack[len(stack)-1]
            stack = stack[:len(stack)-1]
            right[popIdx] = i
        }
        if len(stack) > 0 {
            left[i] = stack[len(stack)-1]
        }
        stack = append(stack, i)
    }
    
    var sum int64
    for i := 0; i < n; i++ {
        count := int64(i - left[i]) * int64(right[i] - i)
        sum += count * int64(nums[i])
    }
    return sum
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - Each element is pushed and popped from the stack at most once in both helper functions.
- **Space Complexity**: $O(n)$
    - We allocate the `left`, `right`, and `stack` slices of size up to $n$.
