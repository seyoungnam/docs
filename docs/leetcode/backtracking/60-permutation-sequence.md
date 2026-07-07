# 60. Permutation Sequence

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/permutation-sequence/description/)

## Solution: Backtracking (DFS Closure with Early Termination)

To find the $k$-th lexicographical permutation of numbers from $1$ to $n$, we can use recursive backtracking. By iterating the choices in ascending order, we generate permutations in lexicographical sequence. We also use early termination to stop the search immediately once the $k$-th permutation is found.

### Thought Process

1.  **Lexicographical Ordering**:
    *   By looping through candidate digits `i` from $1$ to $n$ in ascending order, we guarantee that the recursive path naturally generates permutations in their correct lexicographical sequence.
2.  **Visited Tracker & Path Buffer**:
    *   Maintain a boolean slice `used` of size $n+1$ to keep track of which numbers are currently inside our active path.
    *   Use a pre-allocated byte slice `currPath` of capacity $n$ to build the permutation efficiently without string concatenation overhead.
3.  **Early Termination**:
    *   We maintain a counter `count` of how many full permutations have been constructed.
    *   When a path reaches length $n$, we increment `count`. If `count == k`, we store the permutation, and return `true`.
    *   If the recursive call `dfs()` returns `true`, we bubble the `true` value back up immediately through the recursion stack, stopping any further iteration or branch exploration.
4.  **Base Case**:
    *   When `len(currPath) == n`, check if `count == k` after incrementing the counter. Return `true` if matched, `false` otherwise.

### Go Code

``` go
func getPermutation(n int, k int) string {
    res := ""
    count := 0
    used := make([]bool, n+1)

    currPath := make([]byte, 0, n)

    var dfs func() bool
    dfs = func() bool {
        if len(currPath) == n {
            count++
            if count == k {
                 res = string(currPath)
                 return true
            }
            return false
        }

        for i := 1; i <= n; i++ {
            if used[i] {
                continue
            }

            used[i] = true
            currPath = append(currPath, byte('0'+i))
            if dfs() {
                return true
            }
            currPath = currPath[:len(currPath)-1]
            used[i] = false
        }
        return false
    }

    dfs()
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n \cdot k)$
    - We generate and explore exactly $k$ permutations before terminating. At each node in the search tree, we iterate up to $n$ times. When the $k$-th permutation is found, converting the byte slice of length $n$ to a string takes $O(n)$ time.
    - *Note*: While a mathematical factorial-based approach can solve this problem in $O(n^2)$ time, this backtracking method is a direct, search-oriented implementation that highlights recursion pruning and early stack termination.
- **Space Complexity**: $O(n)$ auxiliary space. The recursion call stack depth goes up to a maximum of $n$, and the tracking slice `used` and byte buffer `currPath` use $O(n)$ space.