# 201. Bitwise AND of Numbers Range

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/bitwise-and-of-numbers-range/description/)

The bitwise AND of all numbers in the range $[left, right]$ has a key property: any bit position that changes (flips between 0 and 1) at least once within the range will result in a `0` in the final AND product. Therefore, the problem is equivalent to finding the **longest common binary prefix** of `left` and `right`, with all remaining bits set to `0`.

Below are two distinct ways to implement this in Go.

---

## Solution 1: Brian Kernighan's Bit Clearing (Recommended)

Instead of shifts, we can iteratively clear the lowest set bit of `right` using `right = right & (right - 1)` until `right` is less than or equal to `left`.

### Thought Process

1.  **Bit Clearing Property**:
    *   The expression `right & (right - 1)` clears the rightmost set bit of `right`.
    *   Since all numbers in the range $[left, right]$ are present, any bit positions to the right of the common prefix will flip and be cleared to `0`.
2.  **Algorithm**:
    *   While `left < right`, clear the lowest set bit of `right`.
    *   As soon as `right <= left`, `right` holds the common prefix with all lower bits cleared. Return `right`.

### Go Code

``` go
func rangeBitwiseAnd(left int, right int) int {
    for left < right {
        right = right & (right - 1)
    }
    return right
}
```

---

## Solution 2: Common Prefix Shifting

We shift both `left` and `right` to the right until they become equal, tracking the number of shifts. Then, we shift the common prefix back to its original position.

### Thought Process

1.  **Identify the Common Prefix**:
    *   While `left < right`, shift both numbers right by 1 bit: `left >>= 1`, `right >>= 1`.
    *   Increment a `shift` counter.
2.  **Reconstruct the Answer**:
    *   Once `left == right`, we have isolated the common binary prefix.
    *   Shift the prefix back to the left: `left << shift` to pad the lower bits with `0`s.

### Go Code

``` go
func rangeBitwiseAnd(left int, right int) int {
    shift := 0
    // Shift both numbers right until they share the same prefix
    for left < right {
        left >>= 1
        right >>= 1
        shift++
    }
    // Shift the common prefix back to its original position
    return left << shift
}
```

---

## Code Efficiency

For both solutions:

- **Time Complexity**: $O(1)$
    - Since we are dealing with 32-bit integers, the loops will execute at most 32 times.
- **Space Complexity**: $O(1)$
    - The algorithm runs in-place with constant memory allocation.