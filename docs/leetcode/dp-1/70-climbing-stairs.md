# 70. Climbing Stairs

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/climbing-stairs/description/)

You are climbing a staircase. It takes `n` steps to reach the top.

Each time you can either climb `1` or `2` steps. In how many distinct ways can you climb to the top?

---

## Solution 1: Dynamic Programming (Bottom-Up with Memo Array)

We can break down this problem into smaller subproblems. To reach step `i`, we can either take a single step from `i+1` or a double step from `i+2`.

### Thought Process

1.  **DP Array Setup**:
    *   Let `count[i]` represent the number of distinct ways to reach the top (`n`) starting from step `i`.
    *   Create a slice `count` of size `n + 1`.
2.  **Base Case**:
    *   `count[n] = 1` (there is exactly 1 way to reach the top when we are already at the top).
3.  **State Transition**:
    *   Iterate backwards from $i = n - 1$ down to $0$.
    *   For each step `i`, we can transition to step `i+1` (1 step) or step `i+2` (2 steps, if `i+2 <= n`):
        $$\text{count}[i] = \text{count}[i+1] + \text{count}[i+2]$$
4.  **Result**:
    *   Return `count[0]`, which represents the total ways to reach the top starting from the ground (step `0`).

### Go Code

``` go
func climbStairs(n int) int {
    count := make([]int, n+1)
    count[n] = 1
    
    for i := n-1; i >= 0; i-- {
        count[i] += count[i+1]
        if i+2 <= n {
            count[i] += count[i+2]
        }
    } 
    return count[0]
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - We compute the number of ways for each step exactly once in a single linear pass from $n-1$ down to $0$.
- **Space Complexity**: $O(N)$
    - We allocate a `count` slice of size $N + 1$ to store the DP states.

---

## Solution 2: Space-Optimized Dynamic Programming

The bottom-up DP relation is identical to generating the Fibonacci sequence. Since the state at index `i` only depends on the two preceding states (`i-1` and `i-2`), we can discard the entire array and keep track of only the last two values.

### Thought Process

1.  **Optimization**:
    *   Instead of maintaining a full slice of size $N$, we track only the previous two state values.
2.  **State Transition**:
    *   Initialize `prev = 1` (representing ways to climb $0$ steps) and `curr = 1` (representing ways to climb $1$ step).
    *   Iterate from step `2` up to `n`:
        *   Compute the ways to reach the current step: `next = prev + curr`.
        *   Update the state pointers: `prev = curr` and `curr = next`.
3.  **Result**:
    *   Return `next` (or `curr`).

### Go Code

``` go
func climbStairs(n int) int {
    if n == 1 {
        return 1
    }
    
    prev, curr, next := 1, 1, 0
    for i := 2; i <= n; i++ {
        next = prev + curr
        prev, curr = curr, next
    }
    return next
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - We iterate from $2$ up to $N$ exactly once.
- **Space Complexity**: $O(1)$
    - We only use constant auxiliary variables, achieving optimal space efficiency.