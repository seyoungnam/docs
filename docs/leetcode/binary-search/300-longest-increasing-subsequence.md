# 300. Longest Increasing Subsequence

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/longest-increasing-subsequence/description/)

## Solution: Binary Search (Patience Sorting)

While the classic bottom-up Dynamic Programming approach takes $O(n^2)$ time, we can optimize the search to $O(n \log n)$ time using a variation of **Patience Sorting**. We maintain an active list `subs` where `subs[i]` stores the smallest tail of all increasing subsequences of length $i+1$ found so far.

### Thought Process

1.  **Maintain Increasing Subsequence Tails**:
    *   Create a slice `subs` and initialize it with the first element of `nums`.
    *   The `subs` array will always remain sorted.
2.  **Iterate and Update**:
    *   For each element `nums[i]` starting from index $1$:
        *   If `nums[i]` is greater than the last element of `subs`, it can extend our longest increasing subsequence $\rightarrow$ append `nums[i]` to `subs`.
        *   Otherwise, `nums[i]` cannot extend the subsequence. Instead, it can improve our existing subsequences by replacing the first element in `subs` that is greater than or equal to `nums[i]`. This substitution is done greedily to make the tail of that subsequence as small as possible, maximizing the potential for future extensions.
3.  **Binary Search**:
    *   Since `subs` is always sorted, we can find the substitution index `p` in $O(\log n)$ time using binary search (finding the leftmost element $\ge$ `target`).
4.  **Result**:
    *   Note that `subs` does not necessarily store the actual LIS itself, but its length `len(subs)` is guaranteed to be the length of the longest increasing subsequence.

### Go Code

``` go
func lengthOfLIS(nums []int) int {
    subs := []int{nums[0]}
    for i := 1; i < len(nums); i++ {
        if nums[i] > subs[len(subs)-1] {
            subs = append(subs, nums[i])
        } else {
            p := binarySearch(subs, nums[i])
            subs[p] = nums[i]
        }
    }
    return len(subs)
}

func binarySearch(arr []int, target int) int {
    l, r := 0, len(arr)
    for l < r {
        m := l + (r-l)/2
        if arr[m] >= target {
            r = m
        } else {
            l = m+1
        }
    }
    return l
}
```

### Code Efficiency

- **Time Complexity**: $O(n \log n)$
    - We iterate through `nums` of length $n$ exactly once. For each element, we perform a binary search on the `subs` slice, which has a length at most $n$, taking $O(\log n)$ time.
- **Space Complexity**: $O(n)$
    - In the worst case (when the input array is strictly increasing), the `subs` slice will grow up to size $n$.