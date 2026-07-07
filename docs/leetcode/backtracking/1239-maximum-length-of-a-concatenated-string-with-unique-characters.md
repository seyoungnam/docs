# 1239. Maximum Length of a Concatenated String with Unique Characters

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/maximum-length-of-a-concatenated-string-with-unique-characters/description/)

## Solution: Backtracking with Bitmask (DFS Closure)

To find the maximum length of a concatenated string formed by a subset of `arr` with unique characters, we can use recursive depth-first backtracking. We represent the set of characters in our path using a 32-bit integer **bitmask**, which allows us to check for character conflicts and duplicates in $O(1)$ constant time.

### Thought Process

1.  **Character Set Representation (Bitmask)**:
    *   Since the input strings contain only lowercase English letters (`'a'` to `'z'`), we can represent the presence of letters in a string using a 32-bit integer.
    *   Each letter `ch` corresponds to a bit position:
        $$\text{bit} = 1 \ll (ch - 'a')$$
    *   A bit value of `1` indicates the letter is present, and `0` indicates it is absent.
2.  **Recursive DFS Closure**:
    *   Define a helper closure `dfs(i, mask, currLen)`:
        *   `i`: Current index of the string in `arr` under consideration.
        *   `mask`: Running bitmask representing all unique letters in the active concatenated path.
        *   `currLen`: Total length of the active concatenated path.
3.  **Recursive Decisions**:
    *   At each index `i`, we make a binary choice:
        *   **Branch 1 (Include `arr[i]`)**:
            *   We check if the string `arr[i]` has unique characters internally and does not conflict with our running `mask`.
            *   Iterate through characters of `arr[i]`, checking:
                $$\text{mask} \ \& \ \text{bit} \neq 0 \quad \text{or} \quad \text{currMask} \ \& \ \text{bit} \neq 0$$
            *   If a conflict is detected, set `isUnique = false` and `break` the loop.
            *   If `isUnique` is `true`, recurse by combining the masks and adding the length: `dfs(i+1, mask | currMask, currLen + len(arr[i]))`.
        *   **Branch 2 (Exclude `arr[i]`)**:
            *   We can always skip `arr[i]`, passing the current state forward: `dfs(i+1, mask, currLen)`.
4.  **Base Case**:
    *   When the index `i` reaches `len(arr)`, we have finished checking all candidates. Update the global `maxLen` with `currLen` and return.

### Go Code

``` go
func maxLength(arr []string) int {
    maxLen := 0
    n := len(arr)

    var dfs func(i int, mask int, currLen int)
    dfs = func(i int, mask, currLen int) {
        if i == n {
            maxLen = max(maxLen, currLen)
            return
        }

        isUnique := true
        currMask := 0
        for j := 0; j < len(arr[i]); j++ {
            bit := 1 << (arr[i][j] - 'a')
            if (mask & bit) != 0 || (currMask & bit) != 0 {
                isUnique = false
                break
            }
            currMask |= bit
        }
        if isUnique {
            dfs(i+1, mask|currMask, currLen+len(arr[i]))
        }
        dfs(i+1, mask, currLen)
    }

    dfs(0, 0, 0)
    return maxLen
}
```

### Code Efficiency

- **Time Complexity**: $O(2^N + L)$
    - Where $N$ is the number of strings in `arr` ($N \le 16$) and $L$ is the total sum of the lengths of all strings in `arr`. The recursion tree has at most $2^N$ states, and computing the bitmask of strings takes linear time proportional to their character counts. With $N \le 16$, the maximum number of recursive branches is $2^{16} = 65,536$, which runs in less than a millisecond.
- **Space Complexity**: $O(N)$ auxiliary space. The recursion call stack depth goes up to a maximum of $N$. The bitmasks use $O(1)$ constant space.