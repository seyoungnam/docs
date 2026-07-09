# 81. Search in Rotated Sorted Array II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/search-in-rotated-sorted-array-ii/description/)

## Solution: Binary Search with Duplicate Handling

This problem is an extension of **Search in Rotated Sorted Array**. The key difference is the presence of duplicate elements, which can make it impossible to determine which half of the array is sorted using standard comparison.

### Thought Process

1.  **Challenge of Duplicates**:
    *   If $\text{nums}[l] == \text{nums}[m] == \text{nums}[r]$, we cannot tell whether the rotation pivot lies in the left half or the right half.
    *   *Example*: $\text{nums} = [1, 0, 1, 1, 1]$, target = $0$. Here, $l=0, r=4, m=2$. $\text{nums}[0] == \text{nums}[2] == \text{nums}[4] == 1$.
2.  **Handling the Ambiguity**:
    *   When we encounter $\text{nums}[l] == \text{nums}[m] == \text{nums}[r]$, we shrink the search space from both ends by doing `l++` and `r--`, then skip to the next iteration. This slowly eliminates the duplicate boundaries until we find distinct values to decide our search direction.
3.  **Standard Rotated Binary Search**:
    *   Once we resolve the duplicate edge case, we check which half of the active range is sorted:
        *   **Left half is sorted** ($\text{nums}[l] \le \text{nums}[m]$):
            *   If target is within the sorted left region ($\text{nums}[l] \le \text{target} < \text{nums}[m]$), search left: `r = m - 1`.
            *   Otherwise, search right: `l = m + 1`.
        *   **Right half is sorted** ($\text{nums}[l] > \text{nums}[m]$):
            *   If target is within the sorted right region ($\text{nums}[m] < \text{target} \le \text{nums}[r]$), search right: `l = m + 1`.
            *   Otherwise, search left: `r = m - 1`.
4.  **Termination**:
    *   If the target is found, return `true`. If the loop finishes without finding the target, return `false`.

### Go Code

``` go
func search(nums []int, target int) bool {
    l, r := 0, len(nums)-1
    for l <= r {
        m := l + (r-l)/2
        if nums[m] == target {
            return true
        }
        if nums[l] == nums[m] && nums[m] == nums[r] {
            l++
            r--
            continue
        }
        if nums[l] <= nums[m] {
            if nums[l] <= target && target < nums[m] {
                r = m-1
            } else {
                l = m+1
            }
        } else {
            if nums[m] < target && target <= nums[r] {
                l = m+1
            } else {
                r = m-1
            }
        }
    }
    return false
}
```

### Code Efficiency

- **Time Complexity**:
    - **Average Case**: $O(\log n)$ — Since the search space is halved in most iterations.
    - **Worst Case**: $O(n)$ — If the array is filled with duplicates (e.g. $[1, 1, 1, 1, 1]$), we must constantly shrink the bounds linearly.
- **Space Complexity**: $O(1)$
    - Only constant auxiliary variables are used for index tracking.