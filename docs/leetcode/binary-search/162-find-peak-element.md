# 162. Find Peak Element

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/find-peak-element/description/)

## Solution: Binary Search on Slopes (Peak Finder)

A peak element is an element that is strictly greater than its neighbors. To find a peak in logarithmic $O(\log n)$ time, we can binary search by checking the local slope at the midpoint. Since $\text{nums}[-1] = \text{nums}[n] = -\infty$, a peak is mathematically guaranteed to exist in any partition that is rising.

### Thought Process

1.  **Binary Search on Slopes**:
    *   Set `l = 0` and `r = len(nums) - 1`.
    *   While `l < r`:
        *   Calculate the midpoint: `m = l + (r - l) / 2`.
        *   Compare the middle element `nums[m]` with its right neighbor `nums[m+1]`:
            *   **Descending Slope ($\text{nums}[m] > \text{nums}[m+1]$)**: The values are decreasing to the right. A peak is guaranteed to exist on the left side (and `m` itself could be the peak). We narrow our search space to the left half: `r = m`.
            *   **Ascending Slope ($\text{nums}[m] < \text{nums}[m+1]$)**: The values are increasing to the right. A peak is guaranteed to exist on the right side (starting at `m+1` or beyond). We narrow our search space to the right half: `l = m + 1`.
2.  **Termination**:
    *   The loop terminates when `l == r`. The index `r` (or `l`) will point to a local peak.

### Go Code

``` go
func findPeakElement(nums []int) int {
    l, r := 0, len(nums)-1
    for l < r {
        m := l + (r-l)/2
        if nums[m] > nums[m+1] {
            r = m
        } else {
            l = m+1
        }
    }
    return l
}
```

### Code Efficiency

- **Time Complexity**: $O(\log n)$
    - The search interval is halved at each iteration, running in logarithmic time.
- **Space Complexity**: $O(1)$
    - Only a constant number of helper variables are tracked.