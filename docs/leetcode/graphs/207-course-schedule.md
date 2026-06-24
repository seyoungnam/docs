# 207. Course Schedule

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/course-schedule/description/)

## Solution: BFS Topological Sort (Kahn's Algorithm)

To determine if all courses can be finished, we can model the courses and their prerequisites as a Directed Graph, where an edge exists from course `A` to course `B` if `A` is a prerequisite for `B`. The problem then boils down to detecting whether this directed graph contains a cycle. Kahn's Algorithm performs cycle detection by iteratively resolving nodes with an in-degree (number of incoming dependencies) of `0`.

### Thought Process

1.  **Build Dependency Graph**:
    - Build an adjacency list `adj` representing outgoing edges from prerequisites to courses.
    - Build an `inbound` array to store the in-degree of each course.
2.  **Initialize Queue**:
    - Identify all courses that have no prerequisites (`inbound[crs] == 0`) and push them into a queue.
3.  **Process Dependencies**:
    - While the queue is not empty, pop the front course `curr` and increment a `visited` counter.
    - For each dependent course `next` of `curr`, decrement its in-degree count: `inbound[next]--`.
    - If `inbound[next]` reaches `0`, it means all its prerequisites are satisfied. Add it to the queue.
4.  **Detect Cycle**:
    - If `visited == numCourses`, we successfully processed all courses without getting stuck in a cycle. Return `true`.
    - Otherwise, a cyclic dependency exists, making it impossible to complete all courses. Return `false`.

### Go Code

``` go
func canFinish(numCourses int, prerequisites [][]int) bool {
    adj := make([][]int, numCourses)
    inbound := make([]int, numCourses)
    for _, p := range prerequisites {
        crs, pre := p[0], p[1]
        adj[pre] = append(adj[pre], crs)
        inbound[crs]++
    }

    queue := make([]int, 0)
    for crs, val := range inbound {
        if val == 0 {
            queue = append(queue, crs)
        }
    }

    visited := 0
    for len(queue) > 0 {
        curr := queue[0]
        queue = queue[1:]
        
        visited++

        for _, next := range adj[curr] {
            inbound[next]--
            if inbound[next] == 0 {
                queue = append(queue, next)
            }
        }
    }
    return numCourses == visited
}
```

### Code Efficiency

- **Time Complexity**: $O(V + E)$
    - Where $V = \text{numCourses}$ and $E = \text{len(prerequisites)}$. Building the graph takes $O(E)$ time. During the BFS, each course is pushed/popped from the queue once, and each edge is traversed once, leading to $O(V + E)$ execution time.
- **Space Complexity**: $O(V + E)$
    - We store the adjacency list of size $O(V + E)$ and the in-degree array of size $O(V)$.

---

## Alternative Solution: DFS Cycle Detection (Graph Coloring)

We can also detect cycles using Depth-First Search (DFS) by keeping track of the visitation states of each node using a status array.

### Thought Process

1.  **Define States**:
    - Keep a `status` array where:
        - `0` = Unvisited.
        - `1` = Visiting (currently in the active recursion stack of DFS).
        - `-1` = Visited (fully processed and confirmed to not belong to a cycle).
2.  **DFS Traversal**:
    - For each course from `0` to `numCourses - 1`, run a DFS to find cycles.
3.  **Recursion Logic**:
    - If `status[curr] == 1`, we hit a node that is currently being visited. This indicates a back-edge (cycle detected). Return `true`.
    - If `status[curr] == -1`, we already processed this branch and found no cycles. Return `false`.
    - Set `status[curr] = 1`.
    - Recursively check all neighbors (`next`) of `curr`. If any neighbor returns `true` (cycle), propagate `true` up.
    - After visiting all neighbors, backtrack by marking `status[curr] = -1` and return `false`.

### Go Code

``` go
func canFinish(numCourses int, prerequisites [][]int) bool {
    adj := make([][]int, numCourses)
    for _, p := range prerequisites {
        crs, pre := p[0], p[1]
        adj[pre] = append(adj[pre], crs)
    }
    // 0 = not visited, 1 = visiting, -1 = visited
    status := make([]int, numCourses)
    for i := 0; i < numCourses; i++ {
        if hasCycle(adj, i, status) {
            return false
        }
    }
    return true
}

func hasCycle(adj [][]int, curr int, status []int) bool {
    if status[curr] == -1 {
        return false
    }
    if status[curr] == 1 {
        return true
    }
    status[curr] = 1
    for _, next := range adj[curr] {
        if hasCycle(adj, next, status) {
            return true
        }
    }
    status[curr] = -1
    return false
}
```

### Code Efficiency

- **Time Complexity**: $O(V + E)$
    - Where $V = \text{numCourses}$ and $E = \text{len(prerequisites)}$. The DFS guarantees that we visit each node and edge at most once.
- **Space Complexity**: $O(V + E)$
    - We use $O(V + E)$ space to store the adjacency list, $O(V)$ space for the status tracker, and up to $O(V)$ recursion stack depth.