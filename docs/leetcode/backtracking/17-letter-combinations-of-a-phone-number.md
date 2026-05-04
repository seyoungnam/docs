# 17. Letter Combinations of a Phone Number

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/letter-combinations-of-a-phone-number/description/)

## Solution : Backtracking

### Thought Process

1.  **Digit Mapping**: Use an array or map to store the letters associated with each digit (2-9), corresponding to a standard phone keypad.
2.  **Backtracking (DFS)**:
    -   **Base Case**: When the current index `i` reaches the length of `digits`, we have built a complete combination. Convert the current byte slice to a string and add it to the result list.
    -   **Recursive Step**:
        -   Identify the letters corresponding to the digit at `digits[i]`.
        -   Iterate through each letter:
            -   Place the letter into the current position of our combination buffer (`curr[i]`).
            -   Recursively call DFS for the next digit (`i + 1`).
3.  **Optimization**: By using a pre-allocated byte slice (`curr`) and overwriting values at each index, we avoid the overhead of repeatedly creating and destroying string objects during the recursion.

### Go Code

``` go
var letters = []string{
    "", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz",
}

func letterCombinations(digits string) []string {
    if len(digits) == 0 {
        return []string{}
    }
    res := []string{}
    curr := make([]byte, len(digits))
    dfs(digits, 0, curr, &res)
    return res
}

func dfs(digits string, i int, curr []byte, res *[]string) {
    if i == len(digits) {
        *res = append(*res, string(curr))
        return
    }
    chars := letters[digits[i]-'0']
    
    for j := 0; j < len(chars); j++ {
        curr[i] = chars[j]
        dfs(digits, i+1, curr, res)
    }
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
