# 91. Decode Ways

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/decode-ways/description/)

A message containing letters from `A-Z` can be **encoded** into numbers using the following mapping:
*   `'A' -> "1"`
*   `'B' -> "2"`
*   ...
*   `'Z' -> "26"`

To **decode** an encoded message, all the digits must be grouped then mapped back into letters using the reverse of the mapping above (there may be multiple ways). For example, `"11106"` can be mapped into:
*   `"AAJF"` with the grouping `(1 1 10 6)`
*   `"KJF"` with the grouping `(11 10 6)`

Note that the grouping `(1 11 06)` is invalid because `"06"` cannot be mapped into `'F'` since `"6"` is different from `"06"`.

Given a string `s` containing only digits, return *the **number** of ways to decode it*.

The test cases are generated so that the answer fits in a **32-bit** integer.

---

## Solution 1: Bottom-Up DP (with Memo Array)

We can solve this problem using bottom-up 1D Dynamic Programming. By iterating through the string, we build the number of valid decodings at each index based on the results from the previous indices.

### Thought Process

1.  **Edge Case**:
    *   If the first character is `'0'`, it cannot be decoded; return `0` immediately.
2.  **DP Array**:
    *   Let `dp[i]` represent the total number of ways to decode the prefix substring `s[0:i+1]`.
3.  **State Transitions**:
    *   Iterate through the string from index `0` to `n - 1`. At each index `i`:
        *   **Single-Digit Decoding**: If the current character is not `'0'` (`s[i] != '0'`), it is a valid single digit. It inherits all decoding ways from the previous index:
            *   If `i == 0`, `dp[0] = 1`.
            *   Otherwise, `dp[i] = dp[i-1]`.
        *   **Two-Digit Decoding**: If `i > 0`, check if the two-digit substring ending at `i` (i.e., `s[i-1:i+1]`) forms a valid number between `10` and `26` (inclusive).
            *   If it does, we can combine these two digits into a single character. We add the ways from the index before these two digits:
                *   If `i == 1`, increment `dp[1]++`.
                *   Otherwise, `dp[i] += dp[i-2]`.
4.  **Result**:
    *   Return `dp[n-1]`.

### Go Code

``` go
import "strconv"

func numDecodings(s string) int {
    if s[0] == '0' {
        return 0
    }
    
    n := len(s)
    dp := make([]int, n)
    
    for i := range s {
        // Option 1: Decode as a single digit
        if s[i] != '0' {
            if i == 0 {
                dp[0] = 1
            } else {
                dp[i] = dp[i-1]
            }
        }
        
        if i == 0 {
            continue
        }
        
        // Option 2: Decode as a double digit (with the previous character)
        val, _ := strconv.Atoi(s[i-1:i+1])
        if 10 <= val && val <= 26 {
            if i == 1 {
                dp[1]++
            } else {
                dp[i] += dp[i-2]
            }
        }
    }
    
    return dp[n-1]
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Where $N$ is the length of string `s`. We perform a single pass over the string. The `strconv.Atoi` function runs in $O(1)$ time since it is always executed on a substring of length exactly 2.
- **Space Complexity**: $O(N)$
    - We allocate a `dp` slice of size $N$ to store the subproblem results.

---

## Solution 2: Space-Optimized DP

Notice that to compute the number of decodings at index `i`, we only require the values of the two preceding states (`dp[i-1]` and `dp[i-2]`). We can optimize the space to $O(1)$ by tracking these two states in variables.

### Thought Process

1.  **Optimization**:
    *   Let `prev` represent the state two indices back (`dp[i-2]`) and `curr` represent the previous state (`dp[i-1]`).
2.  **State Transitions**:
    *   At each index `i`:
        *   Initialize `next = 0`.
        *   If `s[i] != '0'`, we can decode it singly: `next += curr`.
        *   If `i > 0` and the two-digit substring ending at `i` is valid (between `10` and `26` inclusive): `next += prev`.
        *   Update the state pointers: `prev = curr`, `curr = next`.
3.  **Result**:
    *   Return `curr`.

### Go Code

``` go
import "strconv"

func numDecodings(s string) int {
    if s[0] == '0' {
        return 0
    }
    
    prev, curr := 1, 1
    
    for i := range s {
        var next int
        
        // Option 1: Decode as a single digit
        if s[i] != '0' {
            next += curr
        } 
        
        // Option 2: Decode as a double digit
        if i > 0 {
            digit, _ := strconv.Atoi(s[i-1:i+1])
            if 10 <= digit && digit <= 26 {
                next += prev
            }
        }
        
        prev, curr = curr, next
    }
    
    return curr
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - We traverse the string `s` exactly once, performing constant time operations.
- **Space Complexity**: $O(1)$
    - We only use constant auxiliary variables, achieving optimal space efficiency.