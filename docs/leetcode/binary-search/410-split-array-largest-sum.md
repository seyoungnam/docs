# 410. Split Array Largest Sum

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/split-array-largest-sum/description/)

## Solution: Binary Search on Answer

The problem asks us to divide an array into $k$ continuous subarrays such that the maximum sum among these subarrays is minimized. This is a classic optimization problem that can be solved using **Binary Search on the Answer** combined with a greedy verification helper.

### Thought Process

1.  **Define the Search Space**:
    *   **Lower Bound ($l$)**: The smallest possible value of the minimized largest sum. This must be at least the maximum element in `nums` ($\max(\text{nums})$), because each element must belong to at least one subarray.
    *   **Upper Bound ($r$)**: The largest possible value of the minimized largest sum. This is the sum of all elements in `nums` ($\sum \text{nums}$), which represents the case where $k = 1$.
2.  **Binary Search**:
    *   Calculate the midpoint $m = l + (r - l) / 2$ as a candidate for the largest subarray sum.
    *   Use a helper function `canSplit(nums, m, k)` to check if we can partition `nums` into at most $k$ subarrays such that no subarray sum exceeds $m$.
    *   If `canSplit` is `true`, then $m$ is feasible. We attempt to find a smaller maximum sum by searching the lower half: `r = m`.
    *   If `canSplit` is `false`, then $m$ is too small. We must search the upper half: `l = m + 1`.
3.  **Greedy Feasibility Check (`canSplit`)**:
    *   Iterate through `nums` and keep a running sum of the current subarray.
    *   If adding the current element causes the running sum to exceed the limit $m$, we start a new subarray starting with the current element and increment our subarray count.
    *   If the subarray count exceeds $k$, we return `false`.
    *   If we successfully iterate through the entire array within the $k$-subarray budget, return `true`.

### Go Code

``` go
func splitArray(nums []int, k int) int {
    var l, r int // l = max(nums), r = sum(nums)
    for _, num := range nums {
        l = max(l, num)
        r += num
    }

    for l < r {
        m := l + (r-l)/2
        if canSplit(nums, m, k) {
            r = m
        } else {
            l = m+1
        }
    }
    return l
}

func canSplit(nums []int, m int, k int) bool {
    sum, cnt := 0, 1
    for _, num := range nums {
        sum += num
        if sum > m {
            sum = num
            cnt++
            if cnt > k {
                return false
            }
        }
    }
    return true
}
```

### Code Efficiency

- **Time Complexity**: $O(n \log(\sum \text{nums} - \max(\text{nums})))$
    - The binary search takes $O(\log(\text{sum} - \text{max}))$ steps. In each step, we iterate through the array of length $n$ to run the `canSplit` check, which takes $O(n)$ time.
- **Space Complexity**: $O(1)$
    - We only allocate a few primitive variables, using constant auxiliary space.