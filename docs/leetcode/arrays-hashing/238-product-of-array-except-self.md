# 238. Product of Array Except Self

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/product-of-array-except-self/description/)

## Solution: Prefix and Suffix Products

The problem requires calculating the product of all elements except `nums[i]` without using division. The product for any index `i` can be viewed as the product of all elements to the **left** of `i` multiplied by the product of all elements to the **right** of `i`.

### Thought Process

1.  **Objective**: Find `res[i]` such that it equals the product of every element in `nums` except `nums[i]`.
2.  **Constraint**: No division allowed, and the algorithm must run in $O(n)$ time.
3.  **Core Logic**: 
    - For an element at index `i`, the result is `(product of nums[0...i-1]) * (product of nums[i+1...n-1])`.
4.  **Prefix Array**: Create an array `prefix` where `prefix[i]` stores the product of all elements from the beginning up to index `i`.
5.  **Suffix Array**: Create an array `suffix` where `suffix[i]` stores the product of all elements from index `i` to the end.
6.  **Final Construction**:
    - For the first element ($i=0$), the result is just the suffix product starting at index 1.
    - For the last element ($i=n-1$), the result is just the prefix product ending at index $n-2$.
    - For any other element, the result is `prefix[i-1] * suffix[i+1]`.

### Go Code

``` go
func productExceptSelf(nums []int) []int {
    n := len(nums)
    prefix, suffix := make([]int, n), make([]int, n)
    // make prefix
    for i, v := range nums {
        if i == 0 {
            prefix[i] = v
        } else {
            prefix[i] = prefix[i-1]*v
        }
    }
    // make suffix
    for i := n-1; i >= 0; i-- {
        if i == n-1 {
            suffix[i] = nums[n-1]
        } else {
            suffix[i] = suffix[i+1]*nums[i]
        }
    }

    // traverse nums
    res := make([]int, n)
    for i := range nums {
        if i == 0 {
            res[i] = suffix[i+1]
        } else if i == n-1 {
            res[i] = prefix[i-1]
        } else {
            res[i] = prefix[i-1]*suffix[i+1]
        }
    }
    return res
}
```


### Code Efficiency

- **Time Complexity**: $O(n)$
    - The algorithm performs three separate linear passes: one to build the prefix array, one to build the suffix array, and one to construct the result.
- **Space Complexity**: $O(n)$
    - We use two auxiliary arrays (`prefix` and `suffix`) each of size $n$ to store intermediate products. (Note: This can be optimized to $O(1)$ auxiliary space by calculating products on the fly).





