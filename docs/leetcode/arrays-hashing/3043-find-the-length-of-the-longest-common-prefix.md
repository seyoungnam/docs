# 3043. Find the Length of the Longest Common Prefix

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/find-the-length-of-the-longest-common-prefix/description/)

## Solution: Hash Set of Prefixes

A common prefix of two integers is a prefix shared by their string representations. Instead of converting numbers to strings (which is slow due to string allocations), we can extract prefixes numerically by repeatedly dividing by $10$. By storing all numerical prefixes of numbers from the first array in a hash set, we can quickly search for matching prefixes from the second array.

### Thought Process

1.  **Generate Prefix Set**:
    *   Initialize a hash set `prefixSet` (map from `int` to `bool`).
    *   For each number `num` in `arr1`, strip digits from right to left by doing `num /= 10` in a loop.
    *   Add each intermediate number (representing a prefix) to `prefixSet`. E.g., for `1234`, we add `1234`, `123`, `12`, and `1`.
2.  **Find Longest Match**:
    *   Maintain `maxLen` to record the length of the longest common prefix.
    *   For each number `num` in `arr2`, strip digits from right to left using division by $10$.
    *   At each step, check if the current value `num` exists in `prefixSet`.
    *   Since we start checking from the original number (the longest possible prefix) and strip digits one by one, the first match we encounter in the loop is guaranteed to be the longest common prefix for that number.
    *   Calculate its length (number of digits) using the `getLength` helper function, update `maxLen = max(maxLen, length)`, and `break` early to move to the next number in `arr2`.
3.  **Digit Length Helper (`getLength`)**:
    *   To count digits of an integer numerically, count how many times we can divide the number by $10$ until it reaches $0$.

### Go Code

``` go
func longestCommonPrefix(arr1 []int, arr2 []int) int {
    prefixSet := make(map[int]bool)

    for _, num := range arr1 {
        for num > 0 {
            prefixSet[num] = true
            num /= 10
        }
    }
    maxLen := 0

    for _, num := range arr2 {
        for num > 0 {
            if prefixSet[num] {
                length := getLength(num)
                maxLen = max(maxLen, length)
                break
            }
            num /= 10
        }
    }
    return maxLen
}

func getLength(num int) int {
    length := 0
    for num > 0 {
        length++
        num /= 10
    }
    return length
}
```

### Code Efficiency

- **Time Complexity**: $O((N + M) \times \log_{10}(D))$
    - Let $N = \text{len(arr1)}$, $M = \text{len(arr2)}$, and $D$ be the maximum value of any number in the arrays. The number of digits in any number is at most $\log_{10}(D)$ (for $D \le 10^8$, it is at most $8$). The hash map lookups/insertions run in $O(1)$ average time, making the runtime linear in the total number of digits.
- **Space Complexity**: $O(N \times \log_{10}(D))$
    - The hash set stores up to $\log_{10}(D)$ numerical prefixes for each element in `arr1`.