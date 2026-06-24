# 392. Is Subsequence

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/is-subsequence/description/)

## Solution: Two Pointers (Linear Scan)

To check if `s` is a subsequence of `t`, we use a two-pointer approach to scan both strings. We traverse `t` from left to right, matching characters of `s` in order.

### Thought Process

1.  **Edge Cases**:
    - If `s` is empty, it is always a subsequence of any string `t`. Return `true` immediately.
    - If `t` is empty but `s` is not, `s` cannot be a subsequence. Return `false` immediately.
2.  **Dual Pointer Setup**:
    - Maintain a pointer `pS` for string `s` and a pointer `pT` for string `t`.
3.  **Linear Matching**:
    - Loop while `pT < len(t)`:
        - If `s[pS] == t[pT]`, we found a match for the current character in `s`. Increment `pS`.
        - If `pS == len(s)`, we have successfully matched all characters in `s` in the correct relative order. Return `true` early.
        - Increment `pT` to continue scanning `t`.
4.  **Failure**:
    - If we scan the entire string `t` and `pS < len(s)`, return `false`.

### Go Code

``` go
func isSubsequence(s string, t string) bool {
    if len(s) == 0 {
        return true
    }
    if len(t) == 0 {
        return false
    }

    pS, pT := 0, 0

    for pT < len(t) {
        if s[pS] == t[pT] {
            pS++
            if pS == len(s) {
                return true
            }
        }
        pT++
    }
    return false
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - Where $n$ is the length of `t`. We scan `t` at most once.
- **Space Complexity**: $O(1)$
    - We only use two integer pointer variables, requiring constant auxiliary space.

---

## Solution for Follow-Up: Binary Search (Precomputed Indices)

**Follow-up scenario**: If there are many incoming strings $S_1, S_2, \dots$ to check against a single static string $T$, scanning $T$ linearly for every query is inefficient. Instead, we can preprocess $T$ to store character index lists and use binary search to check queries.

### Thought Process

1.  **Precomputation**:
    - Construct a map `charIndices` mapping each character byte to a sorted slice of its index positions in `t`.
2.  **Query Processing**:
    - For each query string `s`, maintain `currTIdx` representing the lowest valid index in `t` where the next character can be matched. Initially `currTIdx = 0`.
    - Iterate through the characters of `s`:
        - Retrieve the list of indices `posList` for the current character. If the character does not exist in `t`, return `false`.
        - Perform a binary search on `posList` to find the smallest index that is $\ge \text{currTIdx}$.
        - If no such index exists, we cannot match the character in order. Return `false`.
        - Otherwise, update `currTIdx` to `posList[idx] + 1` (since the next character must be located strictly after the current one).
3.  **Result**:
    - If all characters are successfully matched, return `true`.

### Go Code

``` go
import (
    "sort"
)

type Checker struct {
    charIndices map[byte][]int
}

func NewChecker(t string) *Checker {
    indices := make(map[byte][]int)
    for i := 0; i < len(t); i++ {
        indices[t[i]] = append(indices[t[i]], i)
    }
    return &Checker{charIndices: indices}
}

func (m *Checker) isSubsequence(s string, t string) bool {
    if len(s) == 0 {
        return true
    }

    currTIdx := 0

    for i := 0; i < len(s); i++ {
        char := s[i]
        posList, ok := m.charIndices[char]
        
        if !ok {
            return false
        }

        idx := sort.Search(len(posList), func(j int) bool {
            return posList[j] >= currTIdx
        })

        if idx == len(posList) {
            return false
        }

        currTIdx = posList[idx] + 1
    }
    return true
}
```

### Code Efficiency

- **Time Complexity**:
    - **Preprocessing**: $O(N)$ where $N$ is the length of $T$. We scan $T$ once to build the index map.
    - **Per Query**: $O(M \log N)$ where $M$ is the length of $S_i$. For each character of $S_i$, we perform a binary search on a slice of size at most $N$.
- **Space Complexity**: $O(N)$
    - We store $N$ indices of $T$ inside the `charIndices` map.