# 78. Subsets

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/subsets/description/)

## Solution 1: Backtracking

### Thought Process

- **Decision Tree**: Each element in `nums` presents a binary choice: either include it in the current subset or exclude it.
- **Recursive Branching**:
    - **Exclude**: Recurse to the next index without adding the current element.
    - **Include**: Add the current element to the path and recurse to the next index.
- **Base Case**: When the index reaches the length of `nums`, a complete subset has been formed. A deep copy of the current path is then added to the result.

### Go Code

``` go
func subsets(nums []int) [][]int {
    return backtrack(nums, []int{}, 0)
}

func backtrack(nums []int, curr []int, i int) [][]int {
    if i == len(nums) {
        copied := make([]int, len(curr))
        copy(copied, curr)
        return [][]int{copied}
    }
    res := make([][]int, 0)
    res = append(res, backtrack(nums, curr, i+1)...)
    res = append(res, backtrack(nums, append(curr, nums[i]), i+1)...)
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n \cdot 2^n)$
    - At each step `i`, make exactly 2 recursive calls. Therefore, the total number of calls is $2^n$.
    - Performing a deep copy of the `curr` array at the base case(`i == len(nums)`) takes `n` at max.
    - Thus, `backtrack` function takes $O(n \cdot 2^n)$ time complexity.
- **Space Complexity**: 
    - $O(n \cdot 2^n)$ if including output: generating $2^n$ subsets. On average, each subset contains $n/2$ elements.
    - $O(n)$ if excluding output: the auxiliary space is determined by the **call stack height**. The recursion tree goes exactly $n$ levels deep.

## Solution 2: Bitmask

### Thought Process

- **Binary Representation of Subsets**: Since each element in `nums` is either included or excluded, there are $2^n$ possible subsets. These can be represented by integers from $0$ to $2^n - 1$.
    - *Example*: For `nums = [1, 2, 3]`, `i` can be `000`, `001`, `010`, `011`, `100`, `101`, `110`, `111`. Total 8 cases($2^3$).
- **Bitmasking**: Each integer `i` serves as a bitmask where the $j$-th bit corresponds to the inclusion status of `nums[j]`.
- **Subsets Construction**: For every bitmask `i`, we iterate through its bits. If the $j$-th bit is set (`i & (1 << j) != 0`), we append `nums[j]` to the current subset.

### Go Code
``` go
func subsets(nums []int) [][]int {
    res := [][]int{}
    n := len(nums)

    for i := 0; i < (1<<n); i++ {
        subset := []int{}
        for j := 0; j < n; j++ {
            if i & (1<<j) != 0 {
                subset = append(subset, nums[j])
            }
        }
        res = append(res, subset)
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n \cdot 2^n)$
    - We iterate through $2^n$ bitmasks, performing $O(n)$ work for each to construct the subset.
- **Space Complexity**: 
    - $O(n \cdot 2^n)$ if including output: You are generating $2^n$ subsets. On average, each subset contains $n/2$ elements.
    - $O(n)$ if excluding output: the temporary `subset` slice you create inside the loop will hold at most $n$ elements.