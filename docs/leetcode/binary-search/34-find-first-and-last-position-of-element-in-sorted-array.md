# 34. Find First and Last Position of Element in Sorted Array

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/description/)

## Solution: Dual Binary Search (First and Last Occurrence)

To locate the starting and ending position of a target value in a sorted array, we perform two independent binary searches: one to find the first occurrence of the target and another to find the last occurrence.

### Thought Process

1.  **First Occurrence Search (`firstOccurrance`)**:
    *   Initialize `l = 0` and `r = len(nums) - 1`.
    *   Perform a standard binary search loop (`l <= r`).
    *   If `nums[m] == target`:
        *   We record the index (`res = m`).
        *   Since we want the *first* occurrence, we continue searching the left half by updating `r = m - 1`.
    *   If `nums[m] < target`, shift left bound: `l = m + 1`.
    *   If `nums[m] > target`, shift right bound: `r = m - 1`.
2.  **Last Occurrence Search (`lastOccurrance`)**:
    *   Initialize `l = 0` and `r = len(nums) - 1`.
    *   Perform a standard binary search loop (`l <= r`).
    *   If `nums[m] == target`:
        *   We record the index (`res = m`).
        *   Since we want the *last* occurrence, we continue searching the right half by updating `l = m + 1`.
    *   Same boundary updates for `<` and `>` comparisons.
3.  **Synthesis**:
    *   If the input array is empty, return `[-1, -1]`.
    *   Otherwise, return the results of both searches: `[]int{firstOccurrance(nums, target), lastOccurrance(nums, target)}`.

### Go Code

``` go
func searchRange(nums []int, target int) []int {
    if len(nums) == 0 {
        return []int{-1, -1}
    }
    return []int{firstOccurrance(nums, target), lastOccurrance(nums, target)}
}

func firstOccurrance(nums []int, target int) int {
    l, r := 0, len(nums)-1
    res := -1

    for l <= r {
        m := l + (r-l)/2
        if nums[m] > target {
            r = m-1
        } else if nums[m] < target {
            l = m+1
        } else {
            res = m
            r = m-1
        }
    }
    return res
}

func lastOccurrance(nums []int, target int) int {
    l, r := 0, len(nums)-1
    res := -1

    for l <= r {
        m := l + (r-l)/2
        if nums[m] > target {
            r = m-1
        } else if nums[m] < target {
            l = m+1
        } else {
            res = m
            l = m+1
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(\log n)$
    - We execute two binary searches sequentially. Each search halves the search space at each step, running in $O(\log n)$ time.
- **Space Complexity**: $O(1)$
    - The algorithm runs in-place, requiring only a constant amount of auxiliary memory.