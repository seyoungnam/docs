# 78. Subsets

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/subsets/description/)

## Solution 1: Backtracking (DFS Closure)

### Thought Process

- **Decision Tree**: Each element in `nums` presents a binary decision: either include it in the current subset or exclude it.
- **Recursive Branching**:
    - **Include**: Append the current element `nums[i]` to `curr` and recursively call `dfs(i+1, curr)`.
    - **Exclude / Backtrack**: Backtrack by removing the last element (`curr = curr[:len(curr)-1]`) and recursively call `dfs(i+1, curr)` to explore paths that exclude `nums[i]`.
- **Base Case**: When index `i` equals `n` (the length of `nums`), we have processed all elements. We create a deep copy of the `curr` slice and append it to our global result slice `res`.

### Go Code

``` go
func subsets(nums []int) [][]int {
    n := len(nums)
    res := make([][]int, 0)
    var dfs func(i int, curr []int)
    dfs = func(i int, curr []int) {
        // base case
        if i == n {
            copied := make([]int, len(curr))
            copy(copied, curr)
            res = append(res, copied)
            return
        }
        // update state and call next item recursively
        curr = append(curr, nums[i])
        dfs(i+1, curr)
        // backtrack state and call next item recursively
        curr = curr[:len(curr)-1]
        dfs(i+1, curr)
    } 
    
    dfs(0, []int{})
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