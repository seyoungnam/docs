# 210. Course Schedule II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/course-schedule-ii/description/)

## Solution 1: BFS (Kahn's Algorithm)

We can solve this problem using **Kahn's Algorithm** for topological sorting. By tracking the in-degree (number of prerequisites) for each course, we can process courses that have zero in-degrees first and gradually decrement the in-degrees of their dependent courses.

### Thought Process

1.  **Build Graph**: Create an adjacency list `adj` representing directed edges from a prerequisite `pre` to its dependent course `crs`. Also maintain an `in` array to store the in-degree (number of prerequisites) of each course.
2.  **Initialize Queue**: Queue all courses that have an in-degree of `0` (courses with no prerequisites).
3.  **Process Queue**: Pop a course `curr` from the queue, append it to the result `res`, and iterate through all its dependent courses:
    *   Decrement the dependent course's in-degree by `1`.
    *   If a dependent course's in-degree drops to `0`, append it to the queue.
4.  **Detect Cycle**: If the final result `res` does not contain all `numCourses`, it means a cycle exists (we cannot complete all courses), so return `nil`.

### Go Code

``` go
func findOrder(numCourses int, prerequisites [][]int) []int {
    in := make([]int, numCourses)
    adj := make([][]int, numCourses)
    for _, pair := range prerequisites {
        crs, pre := pair[0], pair[1]
        in[crs]++
        adj[pre] = append(adj[pre], crs)
    }
    var queue []int
    for crs, cnt := range in {
        if cnt == 0 {
            queue = append(queue, crs)
        }
    } 
    var res []int
    for len(queue) > 0 {
        curr := queue[0]
        queue = queue[1:]
        
        res = append(res, curr)
        
        for _, next := range adj[curr] {
            in[next]--
            if in[next] == 0 {
                queue = append(queue, next)
            }
        }
    }
    if len(res) != numCourses {
        return nil
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(V + E)$
    - Where $V$ is `numCourses` and $E$ is the number of `prerequisites`. We visit each vertex and traverse each edge exactly once.
- **Space Complexity**: $O(V + E)$
    - We use $O(V + E)$ space to store the adjacency list `adj`, in-degree array `in`, and the queue.

---

## Solution 2: DFS (Cycle Detection & Post-Order)

Alternatively, we can solve this using **DFS Topological Sort** with **3-state cycle detection**. We visit each course and recursively visit its prerequisites.

### Thought Process

1.  **Build Graph**: Build an adjacency list `adj` where the directed edge points from a course `crs` to its prerequisite `pre`.
2.  **3-State Visited Array**: Use a `status` slice to represent the state of each node:
    *   `0`: Unvisited.
    *   `1`: Visiting (currently in the active recursion stack).
    *   `-1`: Fully visited (successfully processed with no cycles).
3.  **Detect Cycles**: For each course, recursively run DFS (`hasCycle`). If we encounter a node with `status == 1`, we have found a back-edge (a cycle), so return `true`.
4.  **Post-Order Accumulation**: Once all prerequisites of a course `crs` have been successfully traversed (returning `false` for cycle), we append the course `crs` to the result slice `res` and mark its status as `-1`.

### Go Code

``` go
func findOrder(numCourses int, prerequisites [][]int) []int {
    adj := make([][]int, numCourses)
    for _, pair := range prerequisites {
        crs, pre := pair[0], pair[1]
        adj[crs] = append(adj[crs], pre)
    }
    // 0 = not visited, 1 = visiting, -1 = visited
    status := make([]int, numCourses)
    var res []int
    for crs := 0; crs < numCourses; crs++ {
        if hasCycle(adj, crs, status, &res) {
            return nil
        }
    }
    return res
}

func hasCycle(adj [][]int, crs int, status []int, res *[]int) bool {
    if status[crs] == -1 {
        return false
    }
    if status[crs] == 1 {
        return true
    }
    status[crs] = 1
    for _, pre := range adj[crs] {
        if hasCycle(adj, pre, status, res) {
            return true
        }
    }
    *res = append(*res, crs)
    status[crs] = -1
    return false
}
```

### Code Efficiency

- **Time Complexity**: $O(V + E)$
    - Where $V$ is `numCourses` and $E$ is the number of `prerequisites`. We traverse each vertex and edge exactly once.
- **Space Complexity**: $O(V + E)$
    - The adjacency list takes $O(V + E)$ space, and the recursion stack takes up to $O(V)$ space.
