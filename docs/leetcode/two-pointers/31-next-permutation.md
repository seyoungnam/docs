# 31. Next Permutation

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/next-permutation/description/)

## Solution: Single Pass (Scan, Swap, and Reverse)

To find the lexicographically next greater permutation of numbers, we can implement an in-place algorithm that runs in $O(n)$ time. The logic is divided into three key steps: finding the pivot where the descending order is broken, swapping the pivot with the next larger element from the right, and reversing the remaining suffix to make it as small as possible.

### Thought Process

1.  **Step 1: Find the Pivot (`i`)**:
    *   Scan the array backwards from the second-to-last element (`i = n - 2`) until we find the first index `i` such that `nums[i] < nums[i+1]`.
    *   The element at `i` is our **pivot**. The suffix starting at `i + 1` is guaranteed to be in non-increasing order.
2.  **Step 2: Find the Swap Target (`j`)**:
    *   If a valid pivot `i` is found (i.e., `i >= 0`):
        *   Scan the suffix from the end of the array (`j = n - 1`) backwards until we find the first element `nums[j]` that is strictly greater than `nums[i]`.
        *   Swap `nums[i]` and `nums[j]` to place the next larger element into the pivot position.
3.  **Step 3: Reverse the Suffix**:
    *   Since the suffix starting at `i + 1` is in descending order, reversing it in-place transforms it into ascending order (the smallest possible sequence for that suffix).
    *   If no pivot `i` was found (meaning `i < 0`), it means the entire array is sorted in descending order (the largest possible permutation). Reversing the entire array starting from index `0` converts it to ascending order, which is the smallest possible permutation.

### Go Code

``` go
func nextPermutation(nums []int) {
    n := len(nums)
    if n <= 1 {
        return
    }

    // Step 1: Find the first decreasing element from the right (the pivot)
    i := n - 2
    for i >= 0 && nums[i] >= nums[i+1] {
        i--
    }

    // Step 2: If we found a valid pivot, find its replacement from the right
    if i >= 0 {
        j := n - 1
        for nums[j] <= nums[i] {
            j--
        }
        // Swap the pivot with the next larger element
        nums[i], nums[j] = nums[j], nums[i]
    }

    // Step 3: Reverse the suffix to make it the smallest possible sequence
    reverse(nums, i+1, n-1)
}

func reverse(nums []int, start, end int) {
    for start < end {
        nums[start], nums[end] = nums[end], nums[start]
        start++
        end--
    }
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We perform at most two backwards linear scans of size $n$, and one in-place reverse operation that takes $O(n)$ time. Thus, the total time complexity is linear.
- **Space Complexity**: $O(1)$
    - The array is rearranged in-place, requiring constant auxiliary space.