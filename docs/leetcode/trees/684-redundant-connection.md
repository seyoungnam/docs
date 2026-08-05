# 684. Redundant Connection

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/redundant-connection/description/)

## Solution: Disjoint Set Union (DSU / Union-Find)

We can identify the redundant edge in nearly linear time by processing edges sequentially using the **Disjoint Set Union (DSU)** data structure to detect cycles.

### Thought Process

1.  **Problem Analysis**:
    *   We start with $N$ vertices and $N$ edges. A tree with $N$ vertices always has exactly $N-1$ edges. The presence of $N$ edges means there is exactly one cycle in the graph.
    *   We need to find and return the last edge in the input that, when added, completes the cycle.
2.  **Union-Find / Cycle Detection Strategy**:
    *   Initialize $N$ nodes in disjoint sets, where each node is its own parent.
    *   Iterate through the list of `edges` $(u, v)$ one by one:
        *   Find the root representative of the sets containing $u$ and $v$.
        *   **If their roots are different**: Union the two sets. This represents adding the edge to our spanning tree structure.
        *   **If their roots are the same**: A path already exists between $u$ and $v$. Adding the current edge $(u, v)$ would complete a cycle. Therefore, this edge is redundant.
3.  **DSU Optimizations**:
    *   **Path Compression**: In the `Find` method, recursively link all visited nodes directly to the root node (`uf.parent[i] = uf.Find(uf.parent[i])`). This flattens the tree.
    *   **Union by Rank**: Attach the root of the tree with smaller depth (rank) to the root of the tree with larger depth. This keeps the overall tree height minimized.

### Go Code

``` go
type UnionFind struct {
    parent  []int
    rank    []int
}

func NewUnionFind(n int) *UnionFind {
    parent, rank := make([]int, n+1), make([]int, n+1)
    for i := 0; i <= n; i++ {
        parent[i] = i
        rank[i] = 1
    }
    return &UnionFind{parent: parent, rank: rank}
}

func (uf *UnionFind) Find(i int) int {
    if uf.parent[i] != i {
        uf.parent[i] = uf.Find(uf.parent[i])
    }
    return uf.parent[i]
}

func (uf *UnionFind) Union(i, j int) bool {
    rootI := uf.Find(i)
    rootJ := uf.Find(j)
    if rootI == rootJ {
        return false
    }

    if uf.rank[rootI] < uf.rank[rootJ] {
        uf.parent[rootI] = rootJ
    } else if uf.rank[rootI] > uf.rank[rootJ] {
        uf.parent[rootJ] = rootI
    } else {
        uf.parent[rootJ] = rootI
        uf.rank[rootI]++
    }
    return true
}

func findRedundantConnection(edges [][]int) []int {
    n := len(edges)
    uf := NewUnionFind(n)

    for _, edge := range edges {
        u, v := edge[0], edge[1]
        if !uf.Union(u, v) {
            return edge
        }
    }
    return nil
}
```

### Code Efficiency

- **Time Complexity**: $O(N \cdot \alpha(N))$ $\approx$ $O(N)$
    - **Amortized Near-Constant Cost**: A naive Union-Find implementation without optimizations can degrade to a linear chain, leading to $O(N)$ per operation ($O(N^2)$ total). However, by combining **Path Compression** (flattening the tree during `Find`) and **Union by Rank** (attaching the shallower tree under the deeper tree), the amortized time complexity per operation is reduced to $O(\alpha(N))$.
    - **Inverse Ackermann Function ($\alpha(N)$)**: This function grows extremely slowly. For any practical input size $N$ (even up to the total number of atoms in the observable universe), $\alpha(N)$ is less than or equal to $4$. Thus, for all practical purposes, each DSU operation runs in $O(1)$ constant time.
    - **Overall Execution**: We iterate through the $N$ edges exactly once, performing two `Find` operations and one `Union` operation per edge. Consequently, the total time complexity is $O(N \cdot \alpha(N))$, which is functionally indistinguishable from linear $O(N)$ time.
- **Space Complexity**: $O(N)$
    - The `parent` and `rank` slices inside the `UnionFind` struct take $O(N)$ auxiliary space to track subsets for $N$ nodes.