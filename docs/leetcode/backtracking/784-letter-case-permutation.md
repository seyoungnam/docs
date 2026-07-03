# 784. Letter Case Permutation

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/letter-case-permutation/description/)

## Solution: Backtracking (DFS Closure)

We can generate all possible letter-case permutations using a recursive depth-first backtracking approach. At each character of the string, we decide how to transition to the next state: digits offer only a single choice, while letters present a binary choice (either keep the original case or toggle the case).

### Thought Process

1.  **Binary Choices for Letters**:
    *   For each character in the string at index `i`:
        *   **Digit (`'0' <= char <= '9'`)**: Append it directly to the path and recurse to the next index `i + 1`. There is no alternative branch to explore.
        *   **Letter (`'a' <= char <= 'z'` or `'A' <= char <= 'Z'`)**:
            1.  **Original Case**: Append the original character to `curr` and recursively call `dfs(i+1, curr)`.
            2.  **Toggled Case**: Modify the last element of `curr` to its toggled case (using byte arithmetic) and recursively call `dfs(i+1, curr)`.
            3.  **Backtrack**: Restore the state by removing the last character from `curr` (`curr = curr[:len(curr)-1]`).
2.  **Base Case**:
    *   When the index `i` reaches $n$ (the length of the string), we have processed all characters. Convert the byte slice `curr` to a string and append it to our results slice `res`.
3.  **Optimization**:
    *   We convert the input string to a byte slice `arr` once at the beginning to avoid string indexing and allocation overhead.

### Go Code

``` go
func letterCasePermutation(s string) []string {
    arr := []byte(s)
    n := len(arr)
    res := make([]string, 0)
    var dfs func(i int, curr []byte)
    dfs = func(i int, curr []byte) {
        if i == n {
            res = append(res, string(curr))
            return
        }

        if '0' <= arr[i] && arr[i] <= '9' {
            curr = append(curr, arr[i])
            dfs(i+1, curr)
            return
        }

        if 'a' <= arr[i] && arr[i] <= 'z' {
            curr = append(curr, arr[i])
            dfs(i+1, curr)
            curr[len(curr)-1] = arr[i]-'a'+'A'
            dfs(i+1, curr)
            curr = curr[:len(curr)-1]
        } else {
            curr = append(curr, arr[i])
            dfs(i+1, curr)
            curr[len(curr)-1] = arr[i]-'A'+'a'
            dfs(i+1, curr)
            curr = curr[:len(curr)-1]
        }
    }
    dfs(0, []byte{})
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(L \cdot 2^L)$
    - Let $L$ be the length of the string, and let $U$ be the number of letters in $s$ ($U \le L$).
    - The recursion tree has $2^U$ leaf nodes representing all generated permutations. At each leaf node, we convert the byte slice of length $L$ into a string, which takes $O(L)$ time.
    - Thus, the total time complexity is $O(L \cdot 2^U)$.
- **Space Complexity**:
    - **With Output**: $O(L \cdot 2^L)$ to store the list of all $2^U$ generated strings of length $L$.
    - **Auxiliary Space**: $O(L)$ for the recursion stack depth (at most $L$) and the temporary `curr` slice of size $L$.


---

## Solution 2: Backtracking (In-Place Bitwise XOR)

Instead of maintaining a separate path slice and dynamically appending/slicing characters, we can perform the backtracking in-place directly on the mutable byte slice `arr`. We can also use a bitwise XOR operation to toggle character cases efficiently.

### Thought Process

1.  **In-Place Modification**:
    *   Initialize `arr = []byte(s)`.
    *   We use the same `arr` slice to represent the current state of our string path throughout the recursion.
2.  **Bitwise Case Toggling**:
    *   In the ASCII table:
        *   `'A'` is $65$ (binary `01000001`) and `'a'` is $97$ (binary `01100001`).
        *   The difference between uppercase and lowercase English characters is exactly $32$ (which corresponds to the 6th bit, `0x20` or binary `00100000`).
        *   Performing a bitwise XOR with $32$ (`arr[i] ^= 32`) toggles the case of any letter (upper to lower, or lower to upper) in a single instruction.
3.  **Recursive Decisions**:
    *   For each index `i`:
        *   **Branch 1 (Keep As-Is)**: Call `dfs(i+1)` directly. This covers the digit case, and serves as the "keep original case" branch for letters.
        *   **Branch 2 (Toggle Case)**: If `arr[i]` is a letter, toggle its case (`arr[i] ^= 32`), recursively call `dfs(i+1)`, and then toggle it back to the original case (`arr[i] ^= 32`) to backtrack.
4.  **Base Case**:
    *   When the index `i` reaches `len(arr)`, we convert the current state of the mutable `arr` slice to a string and append it to our results slice `res`.

### Go Code

``` go
func letterCasePermutation(s string) []string {
    res := make([]string, 0)
    arr := []byte(s)
    
    var dfs func(int)
    dfs = func(i int) {
        if i == len(arr) {
            res = append(res, string(arr))
            return
        }
        // leave the character as-is and move to the next index
        dfs(i+1)
        if isLetter(arr[i]) {
            arr[i] ^= 32    // Bitwise XOR trick toggles upper/lower case
            dfs(i+1)        // Explore this path
            arr[i] ^= 32    // Backtrack: toggle it back to original
        }
    }
    dfs(0)
    return res
}

func isLetter(b byte) bool {
    return ('a' <= b && b <= 'z') || ('A' <= b && b <= 'Z')
}
```

### Code Efficiency

- **Time Complexity**: $O(L \cdot 2^L)$
    - Same as Solution 1: We generate $2^U$ leaf states (where $U$ is the number of letters). Converting the byte slice `arr` to a string at each leaf node takes $O(L)$ time.
- **Space Complexity**:
    - **With Output**: $O(L \cdot 2^L)$ to store all $2^U$ generated string permutations.
    - **Auxiliary Space**: $O(L)$ for the recursion stack depth. This solution is more space-efficient in practice than Solution 1 because it does not allocate or manipulate a dynamic path slice.