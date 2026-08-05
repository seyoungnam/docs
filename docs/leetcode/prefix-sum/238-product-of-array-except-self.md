# 238. Product of Array Except Self

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/product-of-array-except-self/description/)

## Solution 1: Prefix and Suffix Products (Two Auxiliary Arrays)

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

---

## Solution 2: Suffix Precomputation and Prefix-on-the-Fly

We can optimize the space complexity by avoiding one of the auxiliary arrays. Instead of storing both prefix and suffix products in separate arrays, we only precompute the suffix products and accumulate the prefix products dynamically using a single variable as we construct the result.

### Thought Process

1.  **Reduce Auxiliary Space**:
    *   Initialize an array `right` to store the suffix products from the right side of the array:
        `right[i] = nums[i] * right[i+1]` (processed backwards).
2.  **Accumulate Prefix on the Fly**:
    *   Create the result array `res`.
    *   Set `res[0] = right[1]` (since there are no elements to the left of index `0`).
    *   Initialize a running variable `left = nums[0]` to track the prefix product.
    *   Loop forward starting from index `i = 1`:
        *   The left product up to `i` is stored in `left`. So we assign: `res[i] = left`.
        *   If `i` is not the last index (`i != n-1`), multiply the value by the suffix product of the remaining elements: `res[i] *= right[i+1]`.
        *   Update the running `left` product for the next iteration: `left *= nums[i]`.
3.  **Result**:
    *   Return `res`.

### Go Code

``` go
func productExceptSelf(nums []int) []int {
    n := len(nums)
    right := make([]int, n)
    right[n-1] = nums[n-1]
    for i := n-2; i >= 0; i-- {
        right[i] = nums[i]*right[i+1]
    }
    res := make([]int, n)
    res[0] = right[1]
    left := nums[0]
    for i := 1; i < n; i++ {
        res[i] = left
        if i != n-1 {
            res[i] *= right[i+1]
        }
        left *= nums[i]
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We perform two linear passes over the array: one to build the `right` suffix array, and one to populate the result array while accumulating the prefix product.
- **Space Complexity**: $O(n)$
    - We use one auxiliary array (`right`) of size $n$, which uses half the auxiliary memory compared to the first solution.


