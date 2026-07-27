# Bit Manipulation

## Overview

**Bit Manipulation** involves performing operations at the binary digit (bit) level. By manipulating bits directly, algorithms can achieve $O(1)$ execution time and $O(1)$ constant auxiliary space. It is a critical optimization technique in low-level programming, cryptographic systems, and space-constrained environments.

> [!NOTE]
> **Key Idea**: Use bitwise operations (`&`, `|`, `^`, `~`, `<<`, `>>`) to represent states, perform quick arithmetic, or solve array-based frequency problems.

- **Time Complexity**: $O(1)$ for basic bitwise operations.
- **Space Complexity**: $O(1)$ auxiliary space.

---

## Basic Operations

The table below outlines fundamental bitwise operations and patterns. The examples assume $n = 5$ (binary: $101_2$) and $i$-indexed positions starting from `0` on the right:

| Operation | Expression | Example ($n=5$, binary: $101_2$) |
| :--- | :--- | :--- |
| **Set bit $i$** | `n | (1 << i)` | Set bit 1: `101 | 010 = 111` (7) |
| **Clear bit $i$** | `n & ~(1 << i)` | Clear bit 2: `101 & 011 = 001` (1) |
| **Toggle bit $i$** | `n ^ (1 << i)` | Toggle bit 0: `101 ^ 001 = 100` (4) |
| **Check bit $i$** | `(n >> i) & 1` | Check bit 2: `(101 >> 2) & 1 = 1` |
| **Get lowest set bit** | `n & (-n)` | `101 & 011 = 001` (1) |
| **Clear lowest set bit** | `n & (n - 1)` | `101 & 100 = 100` (4) |
| **Check power of 2** | `n > 0 && (n & (n - 1)) == 0` | `100 & 011 = 0` $\rightarrow$ power of 2 |
| **Count set bits** | `__builtin_popcount` / Custom DP | `popcount(101) = 2` |
| **Swap two numbers** | `a ^= b; b ^= a; a ^= b` | Swaps $a$ and $b$ without temp |

---

## XOR Properties

Exclusive OR (XOR, represented as $\oplus$ or `^`) is one of the most powerful operators in bit manipulation due to its properties:

1.  **Identity**: $a \oplus 0 = a$
2.  **Self-Inverse**: $a \oplus a = 0$ (any number XORed with itself cancels out)
3.  **Commutative**: $a \oplus b = b \oplus a$
4.  **Associative**: $(a \oplus b) \oplus c = a \oplus (b \oplus c)$

### Key Applications

*   **Find the Unique Element**: In an array where every element appears twice except for one, XORing all elements yields the unique element (since pairs cancel out).
*   **Missing Number**: XORing all indices $0 \dots n$ and all array values finds the single missing number in the range.

---

## Core Questions

### 1. Counting Set Bits (Brian Kernighan's Algorithm)

Rather than shifting through every bit, we can use `n & (n-1)` to clear the lowest set bit of $n$ in each iteration. The number of iterations is exactly equal to the number of set bits.

```go
// TC: O(k) where k is the number of set bits, SC: O(1)
func countSetBits(n int) int {
    count := 0
    for n > 0 {
        n = n & (n - 1) // Clears the lowest set bit
        count++
    }
    return count
}
```

### 2. Subset Enumeration (Power Set)

An integer of size $N$ can represent a subset of a set with $N$ elements. Generating all numbers from $0$ to $2^N - 1$ enumerates all possible subsets.

```go
// TC: O(n * 2^n), SC: O(1)
func generateSubsets(set []int) [][]int {
    n := len(set)
    subsets := make([][]int, 0, 1<<n)
    
    for mask := 0; mask < (1 << n); mask++ {
        currSub := []int{}
        for i := 0; i < n; i++ {
            if (mask & (1 << i)) != 0 {
                currSub = append(currSub, set[i])
            }
        }
        subsets = append(subsets, currSub)
    }
    return subsets
}
```

### 3. Enumerating Submasks of a Mask

Given a bitmask $M$, we want to iterate over all submasks (subsets of the set bits in $M$) without scanning all values from $0$ to $M$.

```go
// TC: O(3^n) when iterating submasks of all masks, SC: O(1)
func traverseSubmasks(mask int) {
    for sub := mask; sub > 0; sub = (sub - 1) & mask {
        // Process submask
        _ = sub
    }
    // Process the empty submask (0) separately if needed
}
```

### 4. Splitting Arrays by XOR (Two Unique Numbers)

If an array has two unique numbers (say $x$ and $y$) and all others appear twice, XORing all values gives $xorSum = x \oplus y$. Since $x \neq y$, $xorSum$ must have at least one set bit.
By isolating the lowest set bit `diff := xorSum & -xorSum`, we can partition the array elements into two groups and find both uniques.

```go
// TC: O(n), SC: O(1)
func singleNumberIII(nums []int) []int {
    xorSum := 0
    for _, num := range nums {
        xorSum ^= num
    }
    
    // Get the lowest set bit
    diff := xorSum & (-xorSum)
    
    x, y := 0, 0
    for _, num := range nums {
        if (num & diff) == 0 {
            x ^= num
        } else {
            y ^= num
        }
    }
    return []int{x, y}
}
```

### 5. Check if Power of Four

A number is a power of 4 if it is a power of 2 AND its single set bit resides on an even bit position (i.e. positions 0, 2, 4, etc.).

```go
// TC: O(1), SC: O(1)
func isPowerOfFour(n int) bool {
    // 0x55555555 is 01010101010101010101010101010101 in binary (even bits set)
    return n > 0 && (n & (n - 1)) == 0 && (n & 0x55555555) != 0
}
```

---

## Core Patterns

### 1. Basic Operations

**Common LeetCode Problems**:

- [191. Number of 1 Bits](../leetcode/bit-manipulation/191-number-of-1-bits.md)
- [231. Power of Two](../leetcode/bit-manipulation/231-power-of-two.md)
- [342. Power of Four](../leetcode/bit-manipulation/342-power-of-four.md)
- [190. Reverse Bits](../leetcode/bit-manipulation/190-reverse-bits.md)
- [201. Bitwise AND of Numbers Range](../leetcode/bit-manipulation/201-bitwise-and-of-numbers-range.md)
- [371. Sum of Two Integers](../leetcode/bit-manipulation/371-sum-of-two-integers.md)
- [29. Divide Two Integers](../leetcode/bit-manipulation/29-divide-two-integers.md)
- [693. Binary Number with Alternating Bits](../leetcode/bit-manipulation/693-binary-number-with-alternating-bits.md)
- [868. Binary Gap](../leetcode/bit-manipulation/868-binary-gap.md)
- [1009. Complement of Base 10 Integer](../leetcode/bit-manipulation/1009-complement-of-base-10-integer.md)


### 2. XOR Tricks

**Common LeetCode Problems**:

- 136. Single Number
- 137. Single Number II
- 260. Single Number III
- 268. Missing Number
- 287. Find the Duplicate Number
- 448. Find All Numbers Disappeared in an Array
- 442. Find All Duplicates in an Array
- 1310. XOR Queries of a Subarray
- 421. Maximum XOR of Two Numbers in an Array
- 1720. Decode XORed Array
- 1442. Count Triplets That Can Form Two Arrays of Equal XOR


### 3. Counting Bits & Masking

**Common LeetCode Problems**:

- 338. Counting Bits
- 461. Hamming Distance
- 476. Number Complement
- 477. Total Hamming Distance
- 762. Prime Number of Set Bits in Binary Representation
- 1178. Number of Valid Words for Each Puzzle
- 1318. Minimum Flips to Make a OR b Equal to c
- 1356. Sort Integers by The Number of 1 Bits
- 2044. Count Number of Maximum Bitwise-OR Subsets
- 2220. Minimum Bit Flips to Convert Number


### 4. Subsets & Bitmask DP

**Common LeetCode Problems**:

- 78. Subsets
- 698. Partition to K Equal Sum Subsets
- 847. Shortest Path Visiting All Nodes
- 1255. Maximum Score Words Formed by Letters
- 1494. Parallel Courses II
- 1723. Find Minimum Time to Finish All Jobs
- 1986. Minimum Number of Work Sessions to Finish the Tasks

### 5. Advanced Tricks

**Common LeetCode Problems**:

- 89. Gray Code
- 318. Maximum Product of Word Lengths
- 393. UTF-8 Validation
- 397. Integer Replacement
- 898. Bitwise ORs of Subarrays
- 1238. Circular Permutation in Binary Representation
- 1461. Check If a String Contains All Binary Codes of Size K
- 1611. Minimum One Bit Operations to Make Integer Zero
- 1680. Concatenation of Consecutive Binary Numbers
- 1738. Find Kth Largest XOR Coordinate Value



Below are standard patterns and templates implemented in G

