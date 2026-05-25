# 1192. Critical Connections in a Network

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/critical-connections-in-a-network/description/)

## Solution: Tarjan's Bridge-Finding Algorithm (DFS)

A **critical connection** (or **bridge**) in a connected undirected graph is an edge whose removal disconnects the graph. We can find all bridges in $O(V + E)$ time using **Tarjan's DFS-based bridge algorithm**.

### Thought Process

1.  **Definitions**:
    *   `ids[i]`: The discovery time (order of visitation) of node `i` during DFS.
    *   `low[i]`: The lowest discovery time reachable from node `i` using at most one back-edge.
2.  **DFS Traversal**:
    *   Start DFS from any node (e.g. `0`) and assign incremental `ids` and initial `low` values to each node as we visit it.
    *   For each neighbor `next` of the current node `curr` (ignoring the immediate `parent` we just came from):
        *   **If `next` is unvisited**: recursively DFS it. After it returns, update `low[curr] = min(low[curr], low[next])`.
            *   **Bridge Condition**: If `low[next] > ids[curr]`, it means there is no path from `next` to `curr` or any ancestor of `curr` other than the edge `(curr, next)`. Therefore, `(curr, next)` is a **critical connection**!
        *   **If `next` is already visited** (back-edge): update `low[curr] = min(low[curr], ids[next])`.

### Go Code

``` go
func criticalConnections(n int, connections [][]int) [][]int {
    // 1. Build the undirected adjacency list
    adj := make([][]int, n)
    for _, edge := range connections {
        u, v := edge[0], edge[1]
        adj[u] = append(adj[u], v)
        adj[v] = append(adj[v], u)
    }
    
    ids := make([]int, n)
    low := make([]int, n)
    for i := range ids {
        ids[i] = -1 // Mark as unvisited
    }
    
    var bridges [][]int
    time := 0
    
    var dfs func(curr, parent int)
    dfs = func(curr, parent int) {
        time++
        ids[curr] = time
        low[curr] = time
        
        for _, next := range adj[curr] {
            if next == parent {
                continue
            }
            if ids[next] == -1 {
                dfs(next, curr)
                low[curr] = min(low[curr], low[next])
                
                // If the next node cannot reach curr or its ancestors
                if low[next] > ids[curr] {
                    bridges = append(bridges, []int{curr, next})
                }
            } else {
                // Back-edge found: update low using the neighbor's discovery time
                low[curr] = min(low[curr], ids[next])
            }
        }
    }
    
    dfs(0, -1)
    return bridges
}
```

### Code Efficiency

- **Time Complexity**: $O(V + E)$
    - We build the adjacency list in $O(E)$ time. The DFS visits each vertex once and traverses each undirected edge twice (once in each direction), taking $O(V + E)$ time.
- **Space Complexity**: $O(V + E)$
    - We use $O(V + E)$ space to store the adjacency list, and $O(V)$ space for the `ids` and `low` slices and the recursion call stack.
