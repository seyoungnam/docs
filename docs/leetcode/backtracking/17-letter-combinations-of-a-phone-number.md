# 17. Letter Combinations of a Phone Number

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/letter-combinations-of-a-phone-number/description/)

## Solution: Backtracking (DFS Closure with In-Place Overwriting)

### Thought Process

1.  **Digit Mapping**: Use a local array `letters` to map each digit character (indices 2–9) to its corresponding string of characters on a phone keypad.
2.  **In-Place Buffer Overwriting**:
    *   Pre-allocate a single byte slice `curr` of size `len(digits)` at the beginning.
    *   At each recursion index `i`, we directly overwrite `curr[i]` with the candidate characters for the current digit.
    *   Overwriting values directly in place avoids slice resizing, dynamic appending, or backtrack pop operations.
3.  **Recursive Decisions**:
    *   Find the characters corresponding to the current digit `digits[i]`.
    *   Loop through each mapped character:
        1. Write the character to the buffer: `curr[i] = char`.
        2. Explore recursively: `dfs(i+1)`.
4.  **Base Case**:
    *   When the index `i` reaches `len(digits)`, the combination buffer `curr` is fully populated. We convert `curr` to a string and append it to our results slice `res`.
5.  **Empty Input Handling**:
    *   If `digits` is empty, we return an empty slice immediately to avoid generating a list containing a single empty string `[""]`.

### Go Code

``` go
func letterCombinations(digits string) []string {
    letters := []string{
        "", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz",
    }

    res := make([]string, 0)
    if len(digits) == 0 {
        return res
    }
    
    curr := make([]byte, len(digits))

    var dfs func(int)
    dfs = func(i int) {
        if i == len(digits) {
            res = append(res, string(curr))
            return
        }
        idx := digits[i]-'0'
        for j := 0; j < len(letters[idx]); j++ {
            char := letters[idx][j]
            curr[i] = char
            dfs(i+1)
        }
    }
    dfs(0)
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(4^N \cdot N)$
    - $N$ is the number of digits in the input. 
    - In the worst case (digits '7' and '9'), there are 4 letters to choose from. The total number of combinations is at most $4^N$.
    - Creating a string from the byte slice at the base case takes $O(N)$ time.
- **Space Complexity**: $O(N)$
    - The recursion stack depth is equal to $N$.
    - The auxiliary space for the combination buffer (`curr`) is $O(N)$.
    - This excludes the space required to store the $O(4^N)$ combinations in the output.
