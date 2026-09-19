# 1743. Restore the Array From Adjacent Pairs

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/restore-the-array-from-adjacent-pairs/description/)

There is an integer array `nums` that consists of `n` unique elements, but you have forgotten it. However, you do remember every pair of adjacent elements in `nums`.

You are given a 2D integer array `adjacentPairs` of size `n - 1` where each `adjacentPairs[i] = [u_i, v_i]` indicates that the elements `u_i` and `v_i` are adjacent in `nums`.

It is guaranteed that every adjacent pair of elements `nums[i]` and `nums[i+1]` will exist in `adjacentPairs`, either as `[nums[i], nums[i+1]]` or `[nums[i+1], nums[i]]`. The pairs can appear in any order.

Return *the original array `nums`*. If there are multiple solutions, return *any of them*.

---

## Solution 1: DFS (Recursive Traversal)

### Thought Process

1. **Graph Representation**:
    * The original array can be modeled as an undirected path graph where numbers are nodes and adjacent pairs are undirected edges.
2. **Find Starting Node (Degree 1)**:
    * Internal elements of the array have exactly 2 adjacent neighbors (degree = 2).
    * The two endpoints (the first and last elements) have exactly 1 neighbor (degree = 1).
    * Build an adjacency map `adj := make(map[int][]int)` and find any node with `len(adj[node]) == 1` to serve as our starting point `start`.
3. **DFS Traversal**:
    * Traverse the linear graph starting from `start`.
    * To avoid stepping backward without needing an extra visited hash map, pass `prev` along with `curr`.
    * For each neighbor `next` of `curr`, if `next != prev`, recurse into `dfs(next, curr)`.

### Go Code

```go
func restoreArray(adjacentPairs [][]int) []int {
    adj := make(map[int][]int)
    for _, pair := range adjacentPairs {
        u, v := pair[0], pair[1]
        adj[u] = append(adj[u], v)
        adj[v] = append(adj[v], u)
    }

    // Find an endpoint with degree 1
    var start int
    for node, neighbors := range adj {
        if len(neighbors) == 1 {
            start = node
            break
        }
    }

    n := len(adjacentPairs) + 1
    res := make([]int, 0, n)

    var dfs func(curr, prev int)
    dfs = func(curr, prev int) {
        res = append(res, curr)
        for _, next := range adj[curr] {
            if next != prev {
                dfs(next, curr)
            }
        }
    }

    dfs(start, start)
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Where $N$ is the number of elements in the restored array ($N = \text{len}(adjacentPairs) + 1$).
    - Building the adjacency map takes $O(N)$ time.
    - Finding the start node and performing DFS visits each node and edge exactly once, taking $O(N)$ time overall.
- **Space Complexity**: $O(N)$
    - The adjacency map stores $N$ nodes and $2(N-1)$ directed edges, taking $O(N)$ space.
    - The recursive DFS call stack consumes $O(N)$ auxiliary space in the worst case.

---

## Solution 2: Iterative Traversal / Array Reconstruction

### Thought Process

1. **Endpoint Identification**:
    * Just like in Solution 1, build the adjacency map and identify an endpoint `start` with degree 1.
2. **Direct Sequential Filling**:
    * Preallocate the result array `res := make([]int, n)` where $n = \text{len}(adjacentPairs) + 1$.
    * Initialize `res[0] = start` and `res[1] = adj[start][0]`.
    * For each index $i$ from $2$ to $n - 1$:
        * Look at the two neighbors of the previous element `res[i-1]`.
        * One neighbor is `res[i-2]` (the predecessor already placed).
        * The other neighbor must be the next element `res[i]`.
        * If `neighbors[0] == res[i-2]`, then `res[i] = neighbors[1]`; otherwise `res[i] = neighbors[0]`.
3. **Efficiency**:
    * This completely eliminates recursion stack overhead while keeping the code clean and idiomatic.

### Go Code

```go
func restoreArray(adjacentPairs [][]int) []int {
    adj := make(map[int][]int)
    for _, pair := range adjacentPairs {
        u, v := pair[0], pair[1]
        adj[u] = append(adj[u], v)
        adj[v] = append(adj[v], u)
    }

    // Find an endpoint with degree 1
    var start int
    for node, neighbors := range adj {
        if len(neighbors) == 1 {
            start = node
            break
        }
    }

    n := len(adjacentPairs) + 1
    res := make([]int, n)
    res[0] = start
    res[1] = adj[start][0]

    for i := 2; i < n; i++ {
        neighbors := adj[res[i-1]]
        if neighbors[0] == res[i-2] {
            res[i] = neighbors[1]
        } else {
            res[i] = neighbors[0]
        }
    }

    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Building the adjacency map takes $O(N)$ time.
    - The iterative loop runs $N - 2$ iterations, each performing $O(1)$ lookups and assignments.
    - Total time complexity is $O(N)$.
- **Space Complexity**: $O(N)$
    - Requires $O(N)$ auxiliary space for the adjacency map (excluding the return slice `res`).
    - Requires $O(1)$ additional call stack space compared to the recursive DFS.