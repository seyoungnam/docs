# 399. Evaluate Division

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/evaluate-division/description/)

## Solution: Graph Representation and Depth-First Search (DFS)

We can model the variables and equations as a directed, weighted graph:

- Each variable (e.g., `a`, `b`) is represented as a vertex.
- An equation $a / b = \text{val}$ translates to a directed edge from $a$ to $b$ with weight $\text{val}$, and a reverse edge from $b$ to $a$ with weight $1 / \text{val}$.
- To answer a query $x / y$, we search for a path from vertex $x$ to vertex $y$. If a path exists, the result is the product of the edge weights along that path. If no path exists, the result is $-1.0$.

### Thought Process

1.  **Graph Construction**:
    - Use a map `graph` from node strings to a list of `edge` structures (containing destination `to` and weight `val`).
    - For each equation `u / v = val`, add `edge{to: v, val: val}` to `graph[u]` and `edge{to: u, val: 1.0 / val}` to `graph[v]`.
2.  **Handling Queries**:
    - For each query `start / end`:
        - If `start` or `end` does not exist in the graph, the answer is undefined. Return `-1.0`.
        - If `start == end`, return `1.0`.
        - Otherwise, initialize a `visited` set and execute DFS pathfinding from `start` to `end`.
3.  **DFS Pathfinding**:
    - Mark the current node as visited.
    - Loop through all outgoing edges `next` of the current node:
        - If the neighbor `next.to` is the target, return the accumulated product: `currProduct * next.val`.
        - If the neighbor has not been visited, recursively call DFS with the updated product: `currProduct * next.val`.
        - If a valid path is found (result is not `-1.0`), propagate the result back.
    - If all paths are explored and no route to the target is found, return `-1.0`.

### Go Code

``` go
type edge struct {
    to  string
    val float64
}

func calcEquation(equations [][]string, values []float64, queries [][]string) []float64 {
    graph := make(map[string][]edge)

    for i, eq := range equations {
        u, v := eq[0], eq[1]
        val := values[i]

        graph[u] = append(graph[u], edge{to: v, val: val})
        graph[v] = append(graph[v], edge{to: u, val: 1.0/val})
    }

    res := make([]float64, len(queries))

    for i, q := range queries {
        start, end := q[0], q[1]

        if _, ok := graph[start]; !ok {
            res[i] = -1.0
            continue
        }

        if _, ok := graph[end]; !ok {
            res[i] = -1.0
            continue
        }

        if start == end {
            res[i] = 1.0
            continue
        }

        visited := make(map[string]bool)
        res[i] = dfs(graph, start, end, 1.0, visited)
    }
    return res
}

func dfs(graph map[string][]edge, curr, target string, currProduct float64, visited map[string]bool) float64 {
    visited[curr] = true

    for _, next := range graph[curr] {
        if next.to == target {
            return currProduct * next.val
        }
        if !visited[next.to] {
            pathResult := dfs(graph, next.to, target, currProduct*next.val, visited)

            if pathResult != -1.0 {
                return pathResult
            }
        }
    }
    return -1.0
}
```

### Code Efficiency

- **Time Complexity**:
    - **Graph Construction**: $O(N)$ where $N$ is the number of equations.
    - **Query Processing**: $O(Q \cdot (V + E))$ where $Q$ is the number of queries, $V$ is the number of unique variables (vertices), and $E$ is the number of equations (edges). For each query, we perform a DFS traversal that visits each vertex and edge at most once in the worst case.
- **Space Complexity**: $O(V + E)$
    - We use $O(V + E)$ space to store the adjacency list representation of the graph.
    - The auxiliary space for DFS (recursion stack and `visited` map) is at most $O(V)$ per query.