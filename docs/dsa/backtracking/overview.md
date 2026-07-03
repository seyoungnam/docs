# Backtracking

## Overview

**Backtracking** is a systematic algorithmic technique for exploring all possible paths, configurations, or solutions in a search space. It builds candidate solutions incrementally and abandons a candidate path ("backtracks") as soon as it determines that the current path cannot lead to a valid solution.

> [!NOTE]
> **Key Idea**: Try $\rightarrow$ Explore $\rightarrow$ Undo (if needed)

- **Time Complexity**: Typically exponential, such as $O(k^n)$ or $O(n!)$, since it explores multiple decision branches.
- **Space Complexity**: $O(n)$ auxiliary space, determined by the maximum depth of the recursion stack.

---

## Core Template

Backtracking solutions generally follow a recursive depth-first structure. Below is the standard pattern implemented in Go:

```go
func backtrack(state State, choices Choices, result *Result) {
    // BASE CASE: Found a valid solution
    if isGoal(state) {
        result.Add(state)
        return
    }
    
    // RECURSIVE CASE: Try each choice
    for _, choice := range choices {
        if isValid(choice) {
            // 1. MAKE the choice (update state)
            makeChoice(state, choice)
            
            // 2. EXPLORE further recursively
            backtrack(state, remainingChoices, result)
            
            // 3. UNDO the choice (backtrack state restore)
            undoChoice(state, choice)
        }
    }
}
```

---

## Core Patterns

Most backtracking problems fall into one of the following structural patterns:

### 1. Subsets / Power Set

Generate all possible subsets of a set. At each step, we decide whether to **include** or **exclude** the current element.

**Common LeetCode Problems**:

- [78. Subsets (no duplicates)](../../leetcode/backtracking/78-subsets.md)
- [90. Subsets II (with duplicates)](../../leetcode/backtracking/90-subsets-2.md)
- [784. Letter Case Permutation](../../leetcode/backtracking/784-letter-case-permutation.md)

**Go Template**:

```go
func subsets(nums []int, index int, current []int, result *[][]int) {
    // Every subset state is a valid subset, so we record a copy
    temp := make([]int, len(current))
    copy(temp, current)
    *result = append(*result, temp)
    
    for i := index; i < len(nums); i++ {
        // Include nums[i]
        current = append(current, nums[i])
        // Explore
        subsets(nums, i+1, current, result)
        // Exclude (backtrack)
        current = current[:len(current)-1]
    }
}
```

### 2. Permutations

Generate all possible orderings of a collection. In this pattern, every element must be visited exactly once.

**Common LeetCode Problems**:

- [46. Permutations (no duplicates)](../../leetcode/backtracking/46-permutations.md)
- [47. Permutations II (with duplicates)](../../leetcode/backtracking/47-permutations-ii.md)

**Go Template**:

```go
func permute(nums []int, used []bool, current []int, result *[][]int) {
    if len(current) == len(nums) {
        temp := make([]int, len(current))
        copy(temp, current)
        *result = append(*result, temp)
        return
    }
    
    for i := 0; i < len(nums); i++ {
        if used[i] {
            continue
        }
        // Mark element as used
        used[i] = true
        current = append(current, nums[i])
        
        // Explore
        permute(nums, used, current, result)
        
        // Unmark and remove (backtrack)
        current = current[:len(current)-1]
        used[i] = false
    }
}
```

### 3. Combinations

Choose $k$ elements from a set of $n$ elements where order does not matter. The traversal is structurally similar to subsets, but we stop and record the candidate as soon as it reaches size $k$.

**Common LeetCode Problems**:

- [77. Combinations ($n$ choose $k$)](../../leetcode/backtracking/77-combinations.md)
- [39. Combination Sum (elements can be reused)](../../leetcode/backtracking/39-combination-sum.md)
- [40. Combination Sum II (no reuse, with duplicates)](../../leetcode/backtracking/40-combination-sum-ii.md)
- [216. Combination Sum III (1-9 range, $k$ numbers, sum = $n$)](../../leetcode/backtracking/216-combination-sum-iii.md)

*   **Go Template**:

```go
func combine(n int, k int, start int, current []int, result *[][]int) {
    if len(current) == k {
        temp := make([]int, len(current))
        copy(temp, current)
        *result = append(*result, temp)
        return
    }
    
    for i := start; i <= n; i++ {
        current = append(current, i)
        // Explore next elements (i+1 prevents reuse and maintains order)
        combine(n, k, i+1, current, result)
        current = current[:len(current)-1]
    }
}
```

### 4. Partitioning

Divide the input string or collection into segments that satisfy specific conditions.

**Common LeetCode Problems**:

- [131. Palindrome Partitioning](../../leetcode/backtracking/131-palindrome-partitioning.md)
- [698. Partition to K Equal Sum Subsets(super hard)](../../leetcode/backtracking/698-partition-to-k-equal-sum-subsets.md)
- [93. Restore IP Addresses](../../leetcode/backtracking/93-restore-ip-addresses.md)

### 5. Grid / Board Search

Navigate and explore valid paths or placement configurations on a 2D grid or matrix.

**Common LeetCode Problems**:

- [79. Word Search](../../leetcode/backtracking/79-word-search.md)
- 51. N-Queens
- 37. Sudoku Solver

### 6. Generate Valid Sequences

Build strings or sequences that satisfy structural constraints.

**Common LeetCode Problems**:

- 22. Generate Parentheses
- 17. Letter Combinations of a Phone Number
- 282. Expression Add Operators

---

## Key Optimization Techniques

1.  **Pruning**: Stop exploring invalid branches early.
    ```go
    if sum > target { return } // Exit recursion immediately
    ```
    
2.  **Handling Duplicates**: Sort the input and skip identical consecutive elements to avoid duplicate branches.
    ```go
    if i > start && nums[i] == nums[i-1] {
        continue
    }
    ```
    
3.  **Early Termination**: Stop recursion immediately once a goal is found (e.g., in grid pathfinding or constraint solvers).
    ```go
    if found { return }
    ```
    
4.  **State Representation**:

    *   Use a **slice** to track the current path or state. Remember to copy the slice content before storing it in results.
    *   Use a **boolean slice** or **hash map** to keep track of visited/used states.
    *   Use a **bitmask** (integer bitwise representation) for highly optimized state representations (useful when the state space is small, e.g., $\le 30$ elements).
    
5.  **Avoiding Revisits**:

    *   Pass a `start` index to control subset/combination boundaries.
    *   Use a `visited` flag slice for order-dependent permutation problems.
    *   Mark and unmark grid cells temporarily (e.g., setting a visited cell to `#` or using a separate visited matrix) to avoid revisiting cells in grid searches.