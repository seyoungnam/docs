# 46. Permutations

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/permutations/description/)

## Solution 1: Backtracking

### Thought Process

- **Swapping Strategy**: Instead of using extra space for a `used` array, we can generate permutations by swapping elements within the original `nums` array.
- **Decision Tree**: {==At each level of the recursion (represented by index `i`), we decide which element should occupy the `i`-th position.==}
- **Recursive Branching**:
    - Iterate from index `j = i` to `len(nums) - 1`.
    - Swap `nums[i]` with `nums[j]` to place the `j`-th element in the current position.
    - Recurse to the next position (`i + 1`).
    - **Backtrack**: Swap `nums[i]` and `nums[j]` back to restore the array's original state for the next iteration of the loop.
- **Base Case**: When index `i` reaches the end of the array, a complete permutation has been formed. Add a deep copy of the current `nums` to the result.

### Go Code

``` go
func permute(nums []int) [][]int {
    res := make([][]int, 0)
    backtrack(nums, 0, &res)
    return res
}

func backtrack(nums []int, i int, res *[][]int) {
    if i == len(nums) {
        copied := make([]int, len(nums))
        copy(copied, nums)
        *res = append(*res, copied)
        return
    }
    for j := i; j < len(nums); j++ {
        nums[i], nums[j] = nums[j], nums[i]
        backtrack(nums, i+1, res)
        nums[i], nums[j] = nums[j], nums[i]
    }
    return
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

