# 323. Number of Connected Components in an Undirected Graph

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/description/)

## Solution: Disjoint Set Union (DSU / Union-Find)

We can find the number of connected components in an undirected graph in near-linear time using the **Disjoint Set Union (DSU)** data structure.

### Thought Process

1.  **Component Counting Strategy**:
    *   Initially, with $N$ vertices and $0$ edges, every vertex is its own connected component. Therefore, we initialize the component count `res = n`.
    *   As we process each edge $(u, v)$ in the input, we check if $u$ and $v$ are already connected.
        *   If they belong to different components, we merge (union) their components. Since two distinct components are joined into one, the total number of components decreases by 1 (`res--`).
        *   If they are already in the same component, they are already connected. The edge does not change the number of components.
2.  **DSU Optimizations**:
    *   **Path Compression**: In the `Find` method, recursively link nodes directly to their root (`uf.parent[i] = uf.Find(uf.parent[i])`) to flatten the trees.
    *   **Union by Rank**: Attach the root of the tree with a smaller rank (depth) to the root of the larger tree to keep the overall tree height minimized.
3.  **Result**:
    *   After iterating through all edges, the remaining value of `res` is the total number of connected components in the graph.

### Go Code

``` go
type UnionFind struct {
    parent  []int
    rank    []int
}

func NewUnionFind(n int) *UnionFind {
    parent, rank := make([]int, n), make([]int, n)
    for i := 0; i < n; i++ {
        parent[i] = i
        rank[i] = 1
    }
    return &UnionFind{parent, rank}
}

func (uf *UnionFind) Find(i int) int {
    if uf.parent[i] != i {
        uf.parent[i] = uf.Find(uf.parent[i])
    }
    return uf.parent[i]
}

func (uf *UnionFind) Union(a, b int) bool {
    p1, p2 := uf.Find(a), uf.Find(b)
    if p1 == p2 {
        return false
    }
    if uf.rank[p1] > uf.rank[p2] {
        uf.parent[p2] = p1
        uf.rank[p1]++
    } else {
        uf.parent[p1] = p2
        uf.rank[p2]++
    }
    return true
}

func countComponents(n int, edges [][]int) int {
    res := n
    uf := NewUnionFind(n)
    for _, edge := range edges {
        u, v := edge[0], edge[1]
        if uf.Union(u, v) {
            res--
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N + E \cdot \alpha(N))$
    - Let $N$ be the number of vertices and $E$ be the number of edges.
    - Initializing the DSU takes $O(N)$ time.
    - We loop through $E$ edges and perform Union/Find operations, each running in near-constant $O(\alpha(N))$ time (where $\alpha$ is the Inverse Ackermann function).
    - Thus, the total time complexity is $O(N + E \cdot \alpha(N))$, which is functionally linear $O(N + E)$ for all practical input sizes.
- **Space Complexity**: $O(N)$
    - The DSU parent and rank slices use $O(N)$ auxiliary space.