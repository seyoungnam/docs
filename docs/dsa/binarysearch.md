# Binary Search

## Overview

**Binary Search** is an efficient search algorithm that finds the position of a target value within a **sorted or monotonic** search space. It operates by repeatedly dividing the search interval in half. At each step, it compares the middle element of the interval with the target, discarding the half in which the target cannot lie.

> [!NOTE]
> **Key Idea**: Divide search space in half based on a monotonic property.
> **Monotonic Property**: A condition that is false up to a point, then true thereafter (F $\to$ T), or vice versa (T $\to$ F).

- **Time Complexity**: $O(\log n)$ — Since the search space is halved in each iteration.
- **Space Complexity**: 
  - $O(1)$ auxiliary space for iterative implementations.
  - $O(\log n)$ auxiliary space for recursive implementations (due to the call stack).

---

## Core Templates

Binary search implementations usually follow one of three main patterns depending on the search criteria:

### 1. Exact Match (`left <= right`)

Use this template when looking for an exact element in a sorted collection. If the target is found, return its index. If the search space is exhausted without finding the target, return `-1`.

```go
// TC: O(log n), SC: O(1)
func binarySearchExact(arr []int, target int) int {
    left, right := 0, len(arr)-1
    
    for left <= right {
        mid := left + (right-left)/2
        if arr[mid] == target {
            return mid
        } else if arr[mid] < target {
            left = mid + 1 // narrow to right half
        } else {
            right = mid - 1 // narrow to left half
        }
    }
    return -1 // target not found
}
```

### 2. Boundary Search (`left < right`)

Use this template when finding the first or last index that satisfies a specific condition (e.g., lower bound, upper bound, first occurrence). The loop terminates when `left == right`, pointing to the boundary transition.

```go
// Find first position where element >= target
func lowerBound(nums []int, target int) int {
    left, right := 0, len(nums)  // Note: right = len(nums) (can insert at end)
    
    for left < right {
        mid := left + (right-left)/2
        if nums[mid] < target {
            left = mid + 1     // Need larger
        } else {
            right = mid        // Found candidate, but look for earlier
        }
    }
    return left // return the index of the first element >= target
}
```

```go
// Find first position where element > target
func upperBound(nums []int, target int) int {
    left, right := 0, len(nums)
    
    for left < right {
        mid := left + (right-left)/2
        if nums[mid] <= target {
            left = mid + 1     // Need strictly larger
        } else {
            right = mid        // Found candidate
        }
    }
    return left // return the index of the first element > target
}
```

``` go
// Find the first occurrence of the target
func firstOccurrence(nums []int, target int) int {
    left, right := 0, len(nums)-1
    result := -1
    
    for left <= right {
        mid := left + (right-left)/2
        if nums[mid] == target {
            result = mid       // Save this position
            right = mid - 1    // Keep looking left for earlier occurrence
        } else if nums[mid] < target {
            left = mid + 1
        } else {
            right = mid - 1
        }
    }
    return result
}
```

``` go
// Find the last occurrence of the target
func lastOccurrence(nums []int, target int) int {
    left, right := 0, len(nums)-1
    result := -1
    
    for left <= right {
        mid := left + (right-left)/2
        if nums[mid] == target {
            result = mid       // Save this position
            left = mid + 1     // Keep looking right for later occurrence
        } else if nums[mid] < target {
            left = mid + 1
        } else {
            right = mid - 1
        }
    }
    return result
}
```

```go
// Floor: Largest element <= target
func findFloor(nums []int, target int) int {
    left, right := 0, len(nums)-1
    floor := -1
    
    for left <= right {
        mid := left + (right-left)/2
        if nums[mid] <= target {
            floor = nums[mid]  // Candidate for floor
            left = mid + 1     // Try to find larger
        } else {
            right = mid - 1
        }
    }
    return floor
}
// Example: [1,3,5,7,9], target=6 -> returns 5

// Ceil: Smallest element >= target
func findCeil(nums []int, target int) int {
    left, right := 0, len(nums)-1
    ceil := -1
    
    for left <= right {
        mid := left + (right-left)/2
        if nums[mid] >= target {
            ceil = nums[mid]   // Candidate for ceil
            right = mid - 1    // Try to find smaller
        } else {
            left = mid + 1
        }
    }
    return ceil
}
// Example: [1,3,5,7,9], target=6 -> returns 7
```

### 3. Binary Search on Answer

Use this template when the search space is a range of integer values `[minPossible, maxPossible]` representing potential answers. We test the feasibility of the midpoint to determine whether to search higher or lower.

```go
// Find the minimum feasible answer in a range [low, high]
// TC: O(log(high - low) * cost(isFeasible)), SC: O(1)
func binarySearchOnAnswer(low, high int, isFeasible func(int) bool) int {
    left, right := low, high
    result := -1
    
    for left <= right {
        mid := left + (right-left)/2
        if isFeasible(mid) {
            result = mid     // mid works, store it
            right = mid - 1 // try to find a smaller feasible answer (minimize)
            // For maximize: left = mid + 1
        } else {
            left = mid + 1  // need a larger value
            // For maximize: right = mid - 1
        }
    }
    return result
}
```

---

## Core Patterns

Most binary search problems fall into one of the following categories:

### 1. Classic Binary Search

*   Exact element search in a sorted array.
*   Lower bounds / Upper bounds (first/last occurrences).
*   Search insert position.

**Common Problems**: 

- [704. Binary Search](../leetcode/binary-search/704-binary-search.md)
- [35. Search Insert Position](../leetcode/binary-search/35-search-insert-position.md)
- [34. Find First and Last Position of Element in Sorted Array](../leetcode/binary-search/34-find-first-and-last-position-of-element-in-sorted-array.md)
- [540. Single Element in a Sorted Array](../leetcode/binary-search/540-single-element-in-a-sorted-array.md)

### 2. Rotated Sorted Array

*   Finding elements or minimums in shifted arrays.
*   Identifying the pivot or rotation point.
*   Handling arrays with or without duplicates.

**Common Problems**: 

- [33. Search in Rotated Sorted Array](../leetcode/binary-search/33-search-in-rotated-sorted-array.md)
- [81. Search in Rotated Sorted Array II](../leetcode/binary-search/81-search-in-rotated-sorted-array-ii.md)
- [153. Find Minimum in Rotated Sorted Array](../leetcode/binary-search/153-find-minimum-in-rotated-sorted-array.md)
- [154. Find Minimum in Rotated Sorted Array II](../leetcode/binary-search/154-find-minimum-in-rotated-sorted-array-ii.md)

### 3. Binary Search on Answer

*   Determining feasibility on a range of integer results rather than querying an index.
*   Often phrased as "Minimize the maximum" or "Maximize the minimum" capacity, speed, or distance constraints.

**Common Problems**: 

- [875. Koko Eating Bananas](../leetcode/binary-search/875-koko-eating-bananas.md)
- [1011. Capacity To Ship Packages Within D Days](../leetcode/binary-search/1011-capacity-to-ship-packages-within-d-day.md)
- [410. Split Array Largest Sum](../leetcode/binary-search/410-split-array-largest-sum.md)
- [1482. Minimum Number of Days to Make m Bouquets](../leetcode/binary-search/1482-minimum-number-of-days-to-make-m-bouquets.md)
- [1552. Magnetic Force Between Two Balls](../leetcode/binary-search/1552-magnetic-force-between-two-balls.md)
- [774. Minimize Max Distance to Gas Station](../leetcode/binary-search/774-minimize-max-distance-to-gas-station.md)
- [1231. Divide Chocolate](../leetcode/binary-search/1231-divide-chocolate.md)

### 4. 2D Matrix Search

*   Treating a 2D sorted matrix as a flattened 1D array.
*   Searching in row-wise and column-wise sorted grids.

**Common Problems**:

- [74. Search a 2D Matrix](../leetcode/binary-search/74-search-a-2d-matrix.md)
- [240. Search a 2D Matrix II](../leetcode/binary-search/240-search-a-2d-matrix-ii.md) 
- 378. Kth Smallest Element in a Sorted Matrix
- 1351. Count Negative Numbers in a Sorted Matrix
- 1901. Find a Peak Element II

### 5. Advanced Intervals & Bitonic Searches

*   Finding peak elements or boundaries in unsorted or partially sorted arrays using local slopes.
*   Finding medians or statistics across multiple sorted arrays.

**Common Problems**: 

- 162. Find Peak Element
- 1095. Find in Mountain Array
- 4. Median of Two Sorted Arrays
- 69. Sqrt(x)
- 658. Find K Closest Elements
- 528. Random Pick with Weight
- 275. H-Index II
- 374. Guess Number Higher or Lower

---

## Key Optimization & Common Pitfalls

1.  **Integer Overflow**:
    *   Avoid: `mid = (left + right) / 2` (can overflow for large indices).
    *   Use: `mid = left + (right - left) / 2`.
2.  **Infinite Loops**:
    *   When using `left < right` with `left = mid`, integer division rounds down. If `left == right - 1`, `mid` becomes `left`, leading to an infinite loop if the branch takes the `left = mid` path.
    *   Fix: Shift rounding up using `mid = left + (right - left + 1) / 2`.
3.  **Boundary Off-by-One**:
    *   Pay close attention to whether the right bound starts at `n` or `n-1`, and whether you shrink to `right = mid` or `right = mid - 1`.
4.  **Identifying Monotonicity**:
    *   Always verify: *"If answer X is feasible, does it imply all answers > X (or < X) are also feasible?"* If yes, the search space is monotonic and binary search is applicable.

---

## Complexity Reference

| Search Space Size ($n$) | Maximum Iterations ($\log_2 n$) |
| :--- | :--- |
| 10 | 4 |
| 100 | 7 |
| 1,000 | 10 |
| 1,000,000 | 20 |
| 1,000,000,000 | 30 |
| $10^{18}$ | 60 |
