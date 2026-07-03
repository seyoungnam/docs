# 46. Permutations

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/permutations/description/)

## Solution 1: Backtracking (DFS Closure with Visited Slice)

### Thought Process

- **Decision Tree**: At each recursive step, we decide which element from `nums` to place next into our current path `curr`.
- **Visited Slice for Tracking**: To prevent choosing elements that are already part of the current path, we maintain a `used` boolean slice (`[]bool`) of size $n$ to keep track of the chosen elements' indices.
- **Recursive Branching**:
    - Iterate through all element indices in `nums`:
        - If `used[i]` is `true`, skip the element.
        - Otherwise, include the element:
            1. Append `nums[i]` to `curr` and mark index `i` as used: `used[i] = true`.
            2. Explore recursively: `dfs(curr)`.
            3. Backtrack: Remove the last element (`curr = curr[:len(curr)-1]`) and mark index `i` as unused: `used[i] = false`.
- **Base Case**: When `len(curr)` equals `len(nums)`, we have built a complete permutation. We create a deep copy of `curr` and append it to our results slice `res`.

### Go Code

``` go
func permute(nums []int) [][]int {
    n := len(nums)
    res := make([][]int, 0)
    used := make([]bool, n)
    
    var dfs func([]int)
    dfs = func(curr []int) {
        if len(curr) == n {
            copied := make([]int, n)
            copy(copied, curr)
            res = append(res, copied)
            return
        }

        for i := 0; i < n; i++ {
            if used[i] {
                continue
            }
            
            curr = append(curr, nums[i])
            used[i] = true
            dfs(curr)
            curr = curr[:len(curr)-1]
            used[i] = false
        }
    }

    dfs([]int{})
    return res
}
```

### Code Efficiency

- **Time Complexity**: $\mathcal{O}(n \cdot n!)$
    - There are $n!$ leaves in the recursion tree (the total number of permutations).
    - At each leaf, we perform an $\mathcal{O}(n)$ operation to copy the current array into the result.
- **Space Complexity**: $\mathcal{O}(n \cdot n!)$
    - The output list stores $n!$ permutations, each of length $n$.
    - The recursion stack depth is $\mathcal{O}(n)$, which is negligible compared to the output space.


---

## Solution 2: Iterative Building

### Thought Process

- **Iterative Insertion**: The algorithm builds permutations incrementally by processing one number from `nums` at a time.
- **Dynamic Growth**: Start with an empty list containing an empty permutation: `[[]]`. For each new number, we take all currently existing permutations and insert the new number into every possible position (gap).
- **Layer-by-Layer Building**:
    - **Initial State**: `[[]]`
    - **Step 1 (First Number `n1`)**: Insert `n1` into index 0 of `[]` $\rightarrow$ `[[n1]]`.
    - **Step 2 (Second Number `n2`)**: Insert `n2` into indices 0 and 1 of `[n1]` $\rightarrow$ `[[n2, n1], [n1, n2]]`.
    - **Step 3 (Third Number `n3`)**: For each permutation from Step 2, insert `n3` into indices 0, 1, and 2.
- **Completeness**: By inserting each number into all possible gaps of the previously generated smaller permutations, we guarantee that all $n!$ unique orderings are generated without duplicates (assuming all input numbers are unique).

### Go Code

``` go
func permute(nums []int) [][]int {
    res := [][]int{{}}
    for _, n := range nums {
        newRes := [][]int{}
        for _, arr := range res {
            for i := 0; i <= len(arr); i++ {
                newArr := append([]int{}, arr...)
                newArr = append(newArr[:i], append([]int{n}, newArr[i:]...)...)
                newRes = append(newRes, newArr)
            }
        }
        res = newRes
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $\mathcal{O}(n \cdot n!)$
    - There are $n!$ possible permutations for an array of length $n$.
    - During each iteration, the algorithm constructs new permutations by copying elements from existing ones. Creating each of the final $n!$ arrays of length $n$ takes $\mathcal{O}(n)$ time.
    - Therefore, the overall time complexity is dominated by generating all these combinations: $\mathcal{O}(n \cdot n!)$.
- **Space Complexity**: $\mathcal{O}(n \cdot n!)$
    - The result array `res` holds all $n!$ permutations, each of length $n$, which inherently requires $\mathcal{O}(n \cdot n!)$ space.
    - Additionally, the intermediate array `newRes` requires up to $\mathcal{O}(n \cdot n!)$ auxiliary space during the final iteration of the outer loop.

