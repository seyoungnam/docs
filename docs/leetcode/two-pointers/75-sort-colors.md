# 75. Sort Colors

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/sort-colors/description/)

## Solution: Dutch National Flag Algorithm (Three-Way Partitioning)

This problem can be solved in a single pass with $O(1)$ auxiliary space using Edsger Dijkstra's **Dutch National Flag Algorithm**. The algorithm partitions the array into three logical sections using three pointers: `low`, `mid`, and `high`.

### Thought Process

1.  **Logical Partitioning**:
    *   `nums[0 ... low-1]` contains all `0`s (red).
    *   `nums[low ... mid-1]` contains all `1`s (white).
    *   `nums[mid ... high]` contains unsorted elements yet to be examined.
    *   `nums[high+1 ... len(nums)-1]` contains all `2`s (blue).
2.  **Pointers Setup**:
    *   `low = 0`: The boundary index where the next `0` should be placed.
    *   `mid = 0`: The current scanning index.
    *   `high = len(nums) - 1`: The boundary index where the next `2` should be placed.
3.  **Iteration Steps (while `mid <= high`)**:
    *   We inspect `nums[mid]`:
        *   **Case 0**: The element is `0`.
            *   Swap `nums[low]` and `nums[mid]`.
            *   Since the element originally at `low` must have been a `1` (or same index if `low == mid`), we can safely advance both pointers: `low++` and `mid++`.
        *   **Case 1**: The element is `1`.
            *   It is already in the correct relative section. Simply advance `mid++`.
        *   **Case 2**: The element is `2`.
            *   Swap `nums[high]` and `nums[mid]`.
            *   We know the `2` is now in its correct place, so decrement `high--`.
            *   *Note: We do **not** increment `mid` here, because the element swapped from the end into `mid` is unexamined and could be a `0`, `1`, or `2`. We must evaluate it in the next loop iteration.*

### Go Code

``` go
func sortColors(nums []int)  {
    low, mid, high := 0, 0, len(nums)-1

    for mid <= high {
        switch nums[mid] {
        case 0:
            nums[low], nums[mid] = nums[mid], nums[low]
            low++
            mid++
        case 1:
            mid++
        case 2:
            nums[high], nums[mid] = nums[mid], nums[high]
            high--
        }
    }
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We visit and partition each element of the array of length $n$ at most once in a single pass.
- **Space Complexity**: $O(1)$
    - We sort the array in-place, requiring no extra memory allocations.