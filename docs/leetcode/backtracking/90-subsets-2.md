# 90. Subsets II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/subsets-ii/description/)

## Solution 1: Backtracking

### Thought Process

- **Sorting for Duplicate Handling**: Sorting is the first critical step. It groups identical elements together, allowing us to easily identify and skip duplicates.
- **Decision Tree with Pruning**: At each element, we make a binary choice:
    - **Include**: Add the current element to the subset and move to the next index.
    - **Exclude**: To avoid duplicate subsets, we must skip all subsequent occurrences of that same value before moving to the next unique candidate.
- **Recursive Pattern**:
    - The "Include" path is explored first.
    - After returning from the "Include" branch, we "pop" the element (backtrack) and skip all identical elements to ensure the "Exclude" branch starts with a different value.
- **Result Accumulation**: Subsets are added to the result list as they are built, starting with the empty subset.

### Go Code

``` go
func subsetsWithDup(nums []int) [][]int {
    res := [][]int{{}}
    sort.Ints(nums)
    dfs(nums, 0, []int{}, &res)
    return res
}

func dfs(nums []int, i int, curr []int, res *[][]int) {
    if i == len(nums) {
        return
    }
    
    // Include branch
    curr = append(curr, nums[i])
    copied := append([]int{}, curr...)
    *res = append(*res, copied)
    dfs(nums, i+1, curr, res)
    
    // Backtrack and skip duplicates for Exclude branch
    curr = curr[:len(curr)-1]
    for i+1 < len(nums) && nums[i] == nums[i+1] {
        i++
    }
    dfs(nums, i+1, curr, res)
}
```

### Code Efficiency

- **Time Complexity**: $\mathcal{O}(n \cdot 2^n)$
    - In the worst case (no duplicates), there are $2^n$ possible subsets.
    - Sorting takes $\mathcal{O}(n \log n)$.
    - Each subset takes $\mathcal{O}(n)$ to copy into the result list.
    - Thus, the overall complexity is dominated by subset generation: $\mathcal{O}(n \cdot 2^n)$.
- **Space Complexity**: $\mathcal{O}(n \cdot 2^n)$
    - **Output Space**: We store $2^n$ subsets, each with an average length of $n/2$, requiring $\mathcal{O}(n \cdot 2^n)$ space.
    - **Auxiliary Space**: The recursion stack depth is $\mathcal{O}(n)$. The temporary `curr` slice also uses $\mathcal{O}(n)$ space.

---
## Solution 2: Bitmask

### Thought Process

- **Binary Representation**: For an array of size $n$, there are $2^n$ possible subsets, which can be represented by bitmasks from $0$ to $2^n - 1$.
- **Handling Duplicates with Bitmasks**: To avoid duplicate subsets, we follow two rules:
    1. **Sort the Input**: Groups identical elements together.
    2. **Pruning Condition**: While iterating through the bits of a mask, if we find that the current element `nums[i]` is the same as the previous element `nums[i-1]`, we only include it if the previous element was also included in this mask (i.e., bit `i-1` is set).
    - If `nums[i] == nums[i-1]` and bit `i-1` is **not** set but bit `i` **is** set, we skip this mask. This ensures that for a sequence of $k$ identical elements, we only accept masks that pick them in a prefix-like fashion (first one, first two, etc.), effectively picking only one unique combination for each possible count.

### Go Code
``` go
func subsetsWithDup(nums []int) [][]int {
    sort.Ints(nums)
    n := len(nums)
    res := [][]int{}
    for mask := 0; mask < (1<<n); mask++ {
        subset := []int{}
        valid := true

        for i := 0; i < n; i++ {
            if mask & (1<<i) != 0 {
                if i > 0 && nums[i-1] == nums[i] && mask & (1<<(i-1)) == 0 {
                    valid = false
                    break
                }
                subset = append(subset, nums[i])
            }
        }
        if valid {
            res = append(res, subset)
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $\mathcal{O}(n \cdot 2^n)$
    - We iterate through all $2^n$ possible bitmasks.
    - For each mask, we iterate through $n$ bits to construct the subset and check the pruning condition.
- **Space Complexity**: $\mathcal{O}(n \cdot 2^n)$
    - **Output Space**: In the worst case (no duplicates), we store $2^n$ subsets with an average length of $n/2$.
    - **Auxiliary Space**: $\mathcal{O}(n)$ to store the temporary `subset` slice for each mask.