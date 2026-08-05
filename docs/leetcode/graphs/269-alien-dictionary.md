# 269. Alien Dictionary

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/alien-dictionary/description/)

## Solution: Topological Sort via DFS (Post-Order Traversal with Cycle Detection)

We can determine the alphabetical order of the alien language by modeling the character ordering rules as a **Directed Graph** and performing a **Topological Sort** while checking for cycles.

### Thought Process

1.  **Graph Construction**:
    *   **Nodes**: Every unique character in all the words represents a node in the graph. We initialize the adjacency map `adj` for all these characters.
    *   **Edges**: Compare consecutive words $w_1$ and $w_2$:
        *   **Lexicographical Violation (Prefix Check)**: If $w_1$ is longer than $w_2$, and $w_2$ is a prefix of $w_1$ (e.g. $w_1 = \text{"abcde"}$, $w_2 = \text{"abc"}$), this is invalid because shorter prefix words must come first. Return `""` immediately.
        *   Otherwise, locate the first index $j$ where $w_1[j] \neq w_2[j]$. The character $w_1[j]$ must come before $w_2[j]$. Create a directed edge:
            $$w_1[j] \to w_2[j]$$
        *   Break after finding the first mismatch because subsequent characters in the words do not determine their relative sorting.
2.  **Topological Sort and Cycle Detection**:
    *   We use Depth First Search (DFS) with a state tracking map `visited` to find the topological order and detect cycles:
        *   `1`: currently visiting (node is on the active recursion stack).
        *   `2`: fully visited.
    *   **Cycle Detection**: If we encounter a node marked as `1` during our traversal, we have found a cycle (inconsistent ordering rules). Return `""` immediately.
    *   **Backtracking**: When we finish exploring all neighbors of a node, we mark it as `2` (visited) and append it to our result slice `res`.
3.  **Result Generation**:
    *   Because DFS appends nodes during backtracking, the compiled slice `res` will be in reverse order.
    *   Reverse the `res` slice to obtain the correct topological order and return it.

### Go Code

``` go
func alienOrder(words []string) string {
    adj := make(map[rune]map[rune]struct{})
    for _, w := range words {
        for _, c := range w {
            if _, ok := adj[c]; !ok {
                adj[c] = make(map[rune]struct{})
            }
        }
    }

    for i := 0; i < len(words)-1; i++ {
        w1, w2 := words[i], words[i+1]
        minLen := min(len(w1), len(w2))
        if len(w1) > len(w2) && w1[:minLen] == w2[:minLen] {
            return ""
        }
        for j := 0; j < minLen; j++ {
            if w1[j] != w2[j] {
                adj[rune(w1[j])][rune(w2[j])] = struct{}{}
                break
            }
        }
    }

    // 0 = unvisited, 1 = visiting, 2 = visited
    visited := make(map[rune]int)
    var res []rune

    var hasCycle func(curr rune) bool
    hasCycle = func(curr rune) bool {
        if status, ok := visited[curr]; ok {
            return status == 1
        }

        visited[curr] = 1

        for next := range adj[curr] {
            if hasCycle(next) {
                return true
            }
        }

        visited[curr] = 2
        res = append(res, curr)
        return false
    }

    for char := range adj {
        if hasCycle(char) {
            return ""
        }
    }

    for i, j := 0, len(res)-1; i < j; i, j = i+1, j-1 {
        res[i], res[j] = res[j], res[i]
    }
    return string(res)
}
```

### Code Efficiency

- **Time Complexity**: $O(C)$
    - Let $C$ be the total length of all words in the input.
    - Initializing the unique characters and parsing mismatches between consecutive words takes $O(C)$ time.
    - The graph contains at most $V \le 26$ nodes (unique lowercase English letters) and $E \le N - 1$ edges. The DFS traversal runs in $O(V + E)$ time, which is very small.
    - Overall time complexity is dominated by parsing the input, yielding $O(C)$ time.
- **Space Complexity**: $O(1)$ auxiliary space
    - Since the alphabet size is fixed (at most 26 lowercase English letters), the space occupied by the adjacency lists, visited states, and result arrays is bounded by $O(26) = O(1)$ constant space.