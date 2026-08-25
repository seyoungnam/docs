# 2192. All Ancestors of a Node in a Directed Acyclic Graph

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/all-ancestors-of-a-node-in-a-directed-acyclic-graph/description/)

## Solution: Memoized DFS on Reversed Graph

Given a Directed Acyclic Graph (DAG) with $n$ nodes and a list of directed edges, we need to find all ancestors for each node. A node $u$ is an ancestor of $v$ if there exists a directed path from $u$ to $v$. The list of ancestors for each node must be returned in sorted ascending order.

### Thought Process

1. **Graph Reversal**:
   - Instead of traversing forward from parents to children, we build a **reversed adjacency list** `adj`.
   - For an edge `[from, to]`, we append `from` to `adj[to]`. Now `adj[curr]` directly lists all immediate predecessors (parents) of `curr`.

2. **DFS with Memoization**:
   - The set of ancestors of node `curr` is the union of its immediate parents and all ancestors of those parents.
   - Since the graph is a DAG, we can use **Depth-First Search (DFS) with Memoization** (`memo`) to avoid redundant traversals:
     - `memo[curr]` stores the set (`map[int]bool`) of all ancestors for node `curr`.
     - For a node `curr`:
       - If `memo[curr]` is already computed, return it immediately.
       - Otherwise, iterate through each direct parent `next` in `adj[curr]`:
         - Add `next` to the ancestor set.
         - Recursively invoke `dfs(next)` and add all returned ancestors to the set.
       - Memoize `memo[curr]` and return.

3. **Result Construction & Sorting**:
   - For each node `i` from `0` to `n - 1`:
     - Retrieve its ancestor set via `dfs(i)`.
     - Extract the set keys into a slice `list`.
     - Sort `list` in ascending order using `sort.Ints(list)`.
     - Assign `res[i] = list`.

### Go Code

``` go
import "sort"

func getAncestors(n int, edges [][]int) [][]int {
    // Build reversed adjacency list (to -> from)
    adj := make([][]int, n)
    for _, e := range edges {
        from, to := e[0], e[1]
        adj[to] = append(adj[to], from)
    }

    memo := make([]map[int]bool, n)

    var dfs func(curr int) map[int]bool
    dfs = func(curr int) map[int]bool {
        if memo[curr] != nil {
            return memo[curr]
        }
        
        ancestors := map[int]bool{}
        for _, next := range adj[curr] {
            ancestors[next] = true
            for anc := range dfs(next) {
                ancestors[anc] = true
            }
        }

        memo[curr] = ancestors
        return ancestors
    }

    res := make([][]int, n)
    for i := 0; i < n; i++ {
        ancMap := dfs(i)
        list := make([]int, 0, len(ancMap))
        for anc := range ancMap {
            list = append(list, anc)
        }
        sort.Ints(list)
        res[i] = list
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(V \cdot (V + E))$
    - Each node is visited once per DFS search to populate `memo`. Set unions combine up to $V$ elements per edge. Extracting and sorting ancestors for each node takes $O(V \log V)$ time. Overall time complexity is bounded by $O(V^2 + V \cdot E)$.
- **Space Complexity**: $O(V^2 + E)$
    - The reversed adjacency list requires $O(V + E)$ space. Memoization table `memo` stores ancestor sets of size up to $V$ for each node, occupying $O(V^2)$ auxiliary space. The recursion call stack is bounded by $O(V)$.