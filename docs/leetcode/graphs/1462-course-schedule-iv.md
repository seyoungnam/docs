# 1462. Course Schedule IV

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/course-schedule-iv/description/)

This problem asks us to answer queries of the form "is course $u$ a prerequisite of course $v$?". The relationships are represented as a Directed Acyclic Graph (DAG) where an edge $u \to v$ means course $u$ is a direct prerequisite of course $v$. 

We can solve this problem by computing the **transitive closure** of the graph (reachability map) using **DFS** or **Topological Sort**.

---

## Approach 1: Depth First Search (Precomputed Reachability Map)

We can precompute the set of reachable nodes for each course using recursive DFS with a memoized map.

### Thought Process

1.  **Memoized Reachability Map**:
    *   Maintain a map `crsMap` where `crsMap[curr]` is a set (implemented as a map `map[int]bool`) containing all descendant courses reachable from `curr`.
2.  **DFS and Merge Descendants**:
    *   For a course `curr`, if its reachable set has not been computed yet:
        *   Recursively compute the reachable sets for all its outgoing neighbors in `adj[curr]`.
        *   Merge all the descendant sets of neighbors into the set `crsMap[curr]`.
        *   Add `curr` to its own reachable set: `crsMap[curr][curr] = true`.
3.  **Lookup**:
    *   After precomputing reachability for all nodes, we can answer each query `[u, v]` in $O(1)$ time by checking if `v` exists in `crsMap[u]`.

### Go Code

``` go
func checkIfPrerequisite(numCourses int, prerequisites [][]int, queries [][]int) []bool {
    adj := make([][]int, numCourses)
    for _, p := range prerequisites {
        pre, crs := p[0], p[1]
        adj[pre] = append(adj[pre], crs)
    }

    crsMap := make(map[int]map[int]bool)

    var dfs func(curr int) map[int]bool
    dfs = func(curr int) map[int]bool {
        if _, ok := crsMap[curr]; !ok {
            crsMap[curr] = make(map[int]bool)
            for _, next := range adj[curr] {
                for n := range dfs(next) {
                    crsMap[curr][n] = true
                }
            }
            crsMap[curr][curr] = true
        }
        return crsMap[curr]
    }

    for crs := 0; crs < numCourses; crs++ {
        dfs(crs)
    }
    res := make([]bool, len(queries))
    for i, q := range queries {
        res[i] = crsMap[q[0]][q[1]]
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(V \cdot (V + E) + Q)$
    - The DFS visits each vertex and edge. However, at each step, we merge the child sets into the parent set, which takes up to $O(V)$ time. Thus, the precomputation takes $O(V \cdot E + V^2)$ time.
    - Answering queries takes $O(Q)$ time.
- **Space Complexity**: $O(V^2)$
    - The `crsMap` can store up to $V$ sets of size $V$, taking $O(V^2)$ auxiliary space.

---

## Approach 2: DFS with Memoization (On-Demand Query)

Instead of precomputing the entire reachability graph up front, we can query on-demand and cache the prerequisite relationships between pairs of nodes.

### Thought Process

1.  **Prerequisite Table**:
    *   Maintain a 2D matrix `isPrereq[pre][crs]` with three states:
        *   `0`: unvisited relationship.
        *   `1`: `pre` is a prerequisite of `crs`.
        *   `2`: `pre` is NOT a prerequisite of `crs`.
2.  **DFS with Memoization**:
    *   When queried `dfs(pre, crs)`:
        *   If `isPrereq[pre][crs]` is already calculated (`!= 0`), return `isPrereq[pre][crs] == 1`.
        *   Check each direct outgoing neighbor `next` of `pre`. If `next == crs` or `dfs(next, crs)` is true, cache the state as `1` and return `true`.
        *   If all paths fail, cache the state as `2` and return `false`.

### Go Code

``` go
func checkIfPrerequisite(numCourses int, prerequisites [][]int, queries [][]int) []bool {
    adj := make([][]int, numCourses)
    // 0 = unvisited, 1 = prereq, 2 = X prereq
    isPrereq := make([][]int, numCourses)
    for i := range isPrereq {
        isPrereq[i] = make([]int, numCourses)
    }
    
    for _, p := range prerequisites {
        pre, crs := p[0], p[1]
        adj[pre] = append(adj[pre], crs)
        isPrereq[pre][crs] = 1
    }

    var dfs func(pre, crs int) bool
    dfs = func(pre, crs int) bool {
        if isPrereq[pre][crs] != 0 {
            return isPrereq[pre][crs] == 1
        }
        for _, next := range adj[pre] {
            if next == crs || dfs(next, crs) {
                isPrereq[pre][crs] = 1
                return true
            }
        }
        isPrereq[pre][crs] = 2 
        return false
    }

    res := make([]bool, len(queries))
    for i, q := range queries {
        res[i] = dfs(q[0], q[1])
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(V \cdot (V + E) + Q)$
    - Thanks to memoization, the relationship between any pair of nodes `[pre][crs]` is computed at most once. Over the course of all queries, the algorithm visits at most $O(V^2)$ states and performs $O(V \cdot E)$ edge traversals.
- **Space Complexity**: $O(V^2)$
    - Storing the 2D memoization array takes $O(V^2)$ space, plus $O(V)$ stack space for DFS recursion.

---

## Approach 3: Topological Sort (Kahn's Algorithm)

We can propagate prerequisite information in topological order using Kahn's algorithm.

### Thought Process

1.  **Topological Traversal**:
    *   Maintain the indegree for all nodes. Queue nodes with indegree $0$.
2.  **Prerequisite Propagation**:
    *   When we pop a node `curr` and process its neighbor `next`:
        *   Record that `curr` is a prerequisite of `next`: `isPrereq[curr][next] = true`.
        *   **Transitive Closure**: For any course `p` that is a prerequisite of `curr` (`isPrereq[p][curr] == true`), propagate that `p` is also a prerequisite of `next` (`isPrereq[p][next] = true`).
        *   Decrement the indegree of `next`, and queue it if it reaches $0$.

### Go Code

``` go
func checkIfPrerequisite(numCourses int, prerequisites [][]int, queries [][]int) []bool {
    adj := make([]map[int]bool, numCourses)
    isPrereq := make([]map[int]bool, numCourses)
    indegree := make([]int, numCourses)

    for i := 0; i < numCourses; i++ {
        adj[i] = make(map[int]bool)
        isPrereq[i] = make(map[int]bool)
    }

    for _, p := range prerequisites {
        pre, crs := p[0], p[1]
        adj[pre][crs] = true
        indegree[crs]++
    }

    q := make([]int, 0)
    for i := 0; i < numCourses; i++ {
        if indegree[i] == 0 {
            q = append(q, i)
        }
    }

    for len(q) > 0 {
        curr := q[0]
        q = q[1:]
        for next := range adj[curr] {
            isPrereq[curr][next] = true
            for p := 0; p < numCourses; p++ {
                if isPrereq[p][curr] {
                    isPrereq[p][next] = true
                }
            }
            indegree[next]--
            if indegree[next] == 0 {
                q = append(q, next)
            }
        }
    }

    res := make([]bool, len(queries))
    for i, q := range queries {
        res[i] = isPrereq[q[0]][q[1]]
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(V \cdot E + V^2 + Q)$
    - We process each edge $(curr, next)$ in the topological sort exactly once. For each edge, we run a loop of size $V$ to propagate the transitive prerequisites, taking $O(V \cdot E)$ total propagation time.
    - Initializing and scanning nodes takes $O(V^2)$ time. Answering queries takes $O(Q)$ time.
- **Space Complexity**: $O(V^2)$
    - Storing the DSU-like transitive maps takes $O(V^2)$ auxiliary space.