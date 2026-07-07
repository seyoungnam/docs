# Backtracking

## Overview

**Backtracking** is a systematic algorithmic technique for exploring all possible paths, configurations, or solutions in a search space. It builds candidate solutions incrementally and abandons a candidate path ("backtracks") as soon as it determines that the current path cannot lead to a valid solution.

> [!NOTE]
> **Key Idea**: Choose $\rightarrow$ Explore $\rightarrow$ Backtrack (if needed)

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
- [2597. The Number of Beautiful Subsets](../../leetcode/backtracking/2597-the-number-of-beautiful-subsets.md)
- [1239. Maximum Length of a Concatenated String with Unique Characters](../../leetcode/backtracking/1239-maximum-length-of-a-concatenated-string-with-unique-characters.md)

**Go Template**:

```go
// TC: O(n*2^n), SC: O(n)
func subsets(nums []int) [][]int {
    res := make([][]int, 0)
    n := len(nums)

    var dfs func(i int, curr []int)
    dfs = func(i int, curr []int) {
        if i == n {
            copied := make([]int, len(curr))
            copy(copied, curr)
            res = append(res, copied)
            return
        }
        // decision to include nums[i]
        curr = append(curr, nums[i])
        dfs(i+1, curr)
        curr = curr[:len(curr)-1]

        // // remove duplicated subset
        // for i+1 < n && nums[i] == nums[i+1] {
        //     i++
        // }

        // decision to NOT include nums[i]
        dfs(i+1, curr)
    }

    dfs(0, []int{})
    return res
}
```

### 2. Combinations

Choose $k$ elements from a set of $n$ elements where order does not matter. The traversal is structurally similar to subsets, but we stop and record the candidate as soon as it reaches size $k$.

**Common LeetCode Problems**:

- [77. Combinations ($n$ choose $k$)](../../leetcode/backtracking/77-combinations.md)
- [39. Combination Sum (elements can be reused)](../../leetcode/backtracking/39-combination-sum.md)
- [40. Combination Sum II (no reuse, with duplicates)](../../leetcode/backtracking/40-combination-sum-ii.md)
- [216. Combination Sum III (1-9 range, $k$ numbers, sum = $n$)](../../leetcode/backtracking/216-combination-sum-iii.md)
- [377. Combination Sum IV(DP)](../../leetcode/dp-1/377-combination-sum-iv.md)
- [494. Target Sum](../../leetcode/backtracking/494-target-sum.md)
- [416. Partition Equal Subset Sum(Knapsack)](../../leetcode/dp-1/416-partition-equal-subset-sum.md)
- [2305. Fair Distribution of Cookies](../../leetcode/backtracking/2305-fair-distribution-of-cookies.md)

*   **Go Template**:

```go
// TC: O(k*C(n,k)), SC: O(k)
func combine(n int, k int) [][]int {
    res := make([][]int, 0)
    
    var dfs func(start int, curr []int)
    dfs = func(start int, curr []int) {
        if len(curr) == k {
            copied := make([]int, k)
            copy(copied, curr)
            res = append(res, copied)
            return
        }
        // traverse all available options at the current level
        for i := start; i <= n; i++ {
            curr = append(curr, i)
            dfs(i+1, curr)
            curr = curr[:len(curr)-1]
        }
    }

    dfs(1, []int{})
    return res
}
```

### 3. Permutations

Generate all possible orderings of a collection. In this pattern, every element must be visited exactly once.

**Common LeetCode Problems**:

- [46. Permutations (no duplicates)](../../leetcode/backtracking/46-permutations.md)
- [47. Permutations II (with duplicates)](../../leetcode/backtracking/47-permutations-ii.md)
- 31. Next Permutation
- [60. Permutation Sequence](../../leetcode/backtracking/60-permutation-sequence.md)
- [526. Beautiful Arrangement](../../leetcode/backtracking/526-beautiful-arrangement.md)
- [1220. Count Vowels Permutation(DP)](../../leetcode/backtracking/1220-count-vowels-permutation.md)

**Go Template**:

```go
// TC: O(n*n!), SC: O(n)
func permuteUnique(nums []int) [][]int {
    res := make([][]int, 0)
    n := len(nums)
    sort.Ints(nums)
    used := make([]bool, n)

    var dfs func(curr []int)
    dfs = func(curr []int) {
        if len(curr) == n {
            copied := make([]int, n)
            copy(copied, curr)
            res = append(res, copied)
            return
        }
        // traverse all options except the already used ones
        for i := 0; i < n; i++ {
            if used[i] {
                continue
            }
            // avoid already allocated value at the current level
            if i > 0 && nums[i-1] == nums[i] && !used[i-1] {
                continue
            }
            used[i] = true
            curr = append(curr, nums[i])
            dfs(curr)
            curr = curr[:len(curr)-1]
            used[i] = false
        }
    }
    dfs([]int{})
    return res
}
```

### 4. Partitioning

Divide the input string or collection into segments that satisfy specific conditions.

**Common LeetCode Problems**:

- [131. Palindrome Partitioning](../../leetcode/backtracking/131-palindrome-partitioning.md)
- [698. Partition to K Equal Sum Subsets(super hard)](../../leetcode/backtracking/698-partition-to-k-equal-sum-subsets.md)
- [93. Restore IP Addresses](../../leetcode/backtracking/93-restore-ip-addresses.md)
- [140. Word Break II](../../leetcode/backtracking/140-word-break-ii.md)
- [842. Split Array into Fibonacci Sequence](../../leetcode/backtracking/842-split-array-into-fibonacci-sequence.md)
- [639. Decode Ways II](../../leetcode/backtracking/639-decode-ways-ii.md)

### 5. Grid / Board Search

Navigate and explore valid paths or placement configurations on a 2D grid or matrix.

**Common LeetCode Problems**:

- [79. Word Search](../../leetcode/backtracking/79-word-search.md)
- [51. N-Queens](../../leetcode/backtracking/51-n-queens.md)
- [37. Sudoku Solver](../../leetcode/backtracking/37-sudoku-solver.md)
- [212. Word Search II](../../leetcode/backtracking/212-word-search-ii.md)
- [980. Unique Paths III](../../leetcode/backtracking/980-unique-paths-iii.md)
- [351. Android Unlock Patterns](../../leetcode/backtracking/351-android-unlock-patterns.md)


### 6. Generate Valid Sequences

Build strings or sequences that satisfy structural constraints.

**Common LeetCode Problems**:

- [22. Generate Parentheses](../../leetcode/backtracking/22-generate-parentheses.md)
- [17. Letter Combinations of a Phone Number](../../leetcode/backtracking/17-letter-combinations-of-a-phone-number.md)
- [282. Expression Add Operators](../../leetcode/backtracking/282-expression-add-operators.md)
- [241. Different Ways to Add Parentheses](../../leetcode/backtracking/241-different-ways-to-add-parentheses.md)
- [320. Generalized Abbreviation](../../leetcode/backtracking/320-generalized-abbreviation.md)
- [401. Binary Watch](../../leetcode/backtracking/401-binary-watch.md)
- [357. Count Numbers with Unique Digits](../../leetcode/backtracking/357-count-numbers-with-unique-digits.md)

---

## Key Optimization Techniques

1.  **Pruning**: Stop exploring invalid branches early.

```go
if sum > target { return } // Exit recursion immediately
```
    
2.  **Handling Duplicates**: Sort the input and skip identical consecutive elements to avoid duplicate branches.

```go
if i > start && nums[i-1] == nums[i] {
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