# 682. Baseball Game

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/baseball-game/description/)

## Solution: Stack (Iterative Array Simulation)

We can simulate the scoring of the baseball game using a **Stack**. A stack is ideal here because the operations (`+`, `D`, `C`) only interact with the most recently recorded scores.

### Thought Process

1.  **Baseball Game Rules**:
    *   An integer `x`: Record a new score of `x`.
    *   `"+"`: Record a new score that is the sum of the last two valid scores.
    *   `"D"`: Record a new score that is double the last valid score.
    *   `"C"`: Invalidate and remove the last valid score.
2.  **Stack-based Simulation**:
    *   We maintain a slice `stack` of integers to store the active scores.
    *   Iterate through the `operations` slice:
        *   **Sum (`+`)**: Retrieve the last two scores `stack[len(stack)-2]` and `stack[len(stack)-1]`, sum them, and push the result.
        *   **Double (`D`)**: Retrieve the last score `stack[len(stack)-1]`, multiply it by 2, and push the result.
        *   **Cancel (`C`)**: Remove the last score by slicing: `stack = stack[:len(stack)-1]`.
        *   **Numeric Score**: Convert the string operation to an integer using `strconv.Atoi` and push it to the stack.
3.  **Result Aggregation**:
    *   After processing all operations, iterate through `stack` to sum all remaining scores and return the total sum.

### Go Code

``` go
import "strconv"

func calPoints(operations []string) int {
    stack := []int{}
    for _, op := range operations {
        switch op {
        case "+":
            a, b := stack[len(stack)-2], stack[len(stack)-1]
            stack = append(stack, a+b)
        case "D":
            last := stack[len(stack)-1]
            stack = append(stack, 2*last)
        case "C":
            stack = stack[:len(stack)-1]
        default:
            val, _ := strconv.Atoi(op)
            stack = append(stack, val)
        }
    }
    res := 0
    for _, v := range stack {
        res += v
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Where $N$ is the number of operations. We loop through the operations exactly once. Each switch-case operation (including stack pushes, pops, and lookups) runs in $O(1)$ constant time. Calculating the final sum of the stack scores also takes linear $O(N)$ time.
- **Space Complexity**: $O(N)$
    - In the worst case (where all operations are score additions), the `stack` slice will store $N$ elements, requiring linear space.
