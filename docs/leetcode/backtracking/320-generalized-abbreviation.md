# 320. Generalized Abbreviation

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/generalized-abbreviation/description/)

## Solution: Backtracking (DFS Closure)

To generate all possible generalized abbreviations of a string `word`, we can use recursive depth-first backtracking. At each character, we make a binary decision: either abbreviate the current character (collapsing consecutive abbreviated letters into a count) or keep it in its original form.

### Thought Process

1.  **Binary Decisions**:
    *   For each character `word[i]`, we have two paths:
        *   **Branch 1 (Abbreviate)**: Increment the running count of consecutive abbreviated characters:
            $$\text{count} \to \text{count} + 1$$
            Call `dfs(i+1, currentPath, count+1)`.
        *   **Branch 2 (Keep Character)**: Record the current character:
            1. If there is a pending abbreviation count (`count > 0`), append it to our path string using `strconv.Itoa(count)`.
            2. Append the current character `word[i]`.
            3. Recurse with a reset count of 0: `dfs(i+1, nextPath, 0)`.
2.  **Base Case**:
    *   When the index `i` reaches `len(word)` (we have processed all characters):
        *   If there is a trailing abbreviation count (`count > 0`), append it to the path.
        *   Add the final abbreviation string `currentPath` to the result list `res` and return.

### Go Code

``` go
import "strconv"

func generateAbbreviations(word string) []string {
    var res []string

    var dfs func(i int, currentPath string, count int)
    dfs = func(i int, currentPath string, count int) {
        if i == len(word) {
            if count > 0 {
                currentPath += strconv.Itoa(count)
            }
            res = append(res, currentPath)
            return
        }
        // Branch 1: Abbreviate the current character
        dfs(i+1, currentPath, count+1)

        // Branch 2: Keep the current character
        nextPath := currentPath
        if count > 0 {
            nextPath += strconv.Itoa(count)
        }
        nextPath += string(word[i])
        dfs(i+1, nextPath, 0)
    }

    dfs(0, "", 0)
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N \cdot 2^N)$
    - Where $N$ is the length of `word`. Since we make a binary choice at each character index, the recursion tree has exactly $2^N$ leaf nodes. Building and copying the string path along each branch takes $O(N)$ time.
- **Space Complexity**: $O(N)$ auxiliary space. The recursion call stack depth goes up to a maximum of $N$. (The output slice `res` holds all $2^N$ combinations, taking $O(2^N)$ space).