# 78. Subsets

[LeetCode Problem Description](https://leetcode.com/problems/subsets/description/)

## Solution 1: Bactracking


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
        // fmt.Println(i, subset)
        res = append(res, subset)
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n \cdot 2^n)$
    - We iterate through $2^n$ bitmasks, performing $O(n)$ work for each to construct the subset.
- **Space Complexity**: $O(n \cdot 2^n)$
    - This reflects the total space required to store all $2^n$ subsets.