# 1584. Min Cost to Connect All Points

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/min-cost-to-connect-all-points/description/)

The problem asks us to find the minimum cost to connect all points such that there is exactly one simple path between any two points. The cost of connecting two points $(x_1, y_1)$ and $(x_2, y_2)$ is their Manhattan distance: $|x_1 - x_2| + |y_1 - y_2|$.

This is a classic problem of finding the **Minimum Spanning Tree (MST)** in a complete (fully connected) undirected graph. We can solve it using **Kruskal's Algorithm** or **Prim's Algorithm**.

---

## Approach 1: Kruskal's Algorithm

Kruskal's algorithm finds the MST by sorting all edges and adding them one by one, using a **Union-Find (DSU)** data structure to detect cycles.

### Thought Process

1.  **Generate Edges**:
    *   Since every point is connected to every other point, we have $O(N^2)$ possible edges.
    *   Compute the Manhattan distance between all pairs of points and store them as edges of the form `[distance, point_i, point_j]`.
2.  **Sort Edges**:
    *   Sort all edges in ascending order of their distance.
3.  **Union-Find Processing**:
    *   Initialize a DSU structure for $N$ points.
    *   Iterate through the sorted edges. If the endpoints of the edge belong to different components, union them (`Union(u, v) == true`) and add the distance to the total result.
    *   Stop early if we have successfully added $N-1$ edges.

### Go Code

``` go
import "sort"

type UnionFind struct {
    parent  []int
    size    []int
}

func NewUnionFind(n int) *UnionFind {
    parent, size := make([]int, n), make([]int, n)
    for i := range parent {
        parent[i] = i
        size[i] = 1
    }
    return &UnionFind{parent, size}
}

func (uf *UnionFind) Find(node int) int {
    if uf.parent[node] != node {
        uf.parent[node] = uf.Find(uf.parent[node])
    }
    return uf.parent[node]
}

func (uf *UnionFind) Union(u, v int) bool {
    pu, pv := uf.Find(u), uf.Find(v)
    if pu == pv {
        return false
    }
    if uf.size[pu] < uf.size[pv] {
        pu, pv = pv, pu
    }
    uf.size[pu] += uf.size[pv]
    uf.parent[pv] = pu
    return true
}

func minCostConnectPoints(points [][]int) int {
    n := len(points)
    uf := NewUnionFind(n)
    var edges [][]int
    for i := 0; i < n; i++ {
        x1, y1 := points[i][0], points[i][1]
        for j := i+1; j < n; j++ {
            x2, y2 := points[j][0], points[j][1]
            dist := abs(x1-x2) + abs(y1-y2)
            edges = append(edges, []int{dist, i, j})
        }
    }

    sort.Slice(edges, func(a, b int) bool {
        return edges[a][0] < edges[b][0]
    })

    res := 0
    for _, edge := range edges {
        dist, u, v := edge[0], edge[1], edge[2]
        if uf.Union(u, v) {
            res += dist
        }
    }
    return res
}

func abs(a int) int {
    if a < 0 {
        return -a
    }
    return a
}
```

### Code Efficiency

- **Time Complexity**: $O(N^2 \log N)$
    - We generate $O(N^2)$ edges. Sorting these edges takes $O(N^2 \log (N^2)) = O(N^2 \log N)$ time.
    - DSU operations take $O(N^2 \cdot \alpha(N))$ time.
- **Space Complexity**: $O(N^2)$
    - The `edges` slice stores $O(N^2)$ elements.

---

## Approach 2: Prim's Algorithm (Min-Heap)

Prim's algorithm grows the MST node-by-node starting from a source node, using a min-heap to fetch the cheapest edge connecting an MST node to a non-MST node.

### Thought Process

1.  **Adjacency List**:
    *   Construct an adjacency map of the complete graph to track all edge weights.
2.  **Min-Heap Selection**:
    *   Initialize a min-heap containing candidates, starting with `[0, 0]` representing path weight 0 to reach point 0.
3.  **Exploration**:
    *   While the count of visited points is less than $N$:
        *   Pop the edge with the lowest cost.
        *   If the destination node has already been visited, skip.
        *   Otherwise, add its cost to the result, mark it as visited, and push all edges connecting to its unvisited neighbors onto the heap.

### Go Code

``` go
import "container/heap"

type minHeap [][]int
func (h minHeap) Len() int           { return len(h) }
func (h minHeap) Less(i, j int) bool { return h[i][0] < h[j][0] }
func (h minHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *minHeap) Push(x interface{}) { *h = append(*h, x.([]int)) }
func (h *minHeap) Pop() interface{} {
    last := (*h)[len(*h)-1]
    *h = (*h)[:len(*h)-1]
    return last
}

func minCostConnectPoints(points [][]int) int {
    n := len(points)
    adj := make(map[int][][]int)
    for i := 0; i < n; i++ {
        x1, y1 := points[i][0], points[i][1]
        for j := i+1; j < n; j++ {
            x2, y2 := points[j][0], points[j][1]
            dist := abs(x1-x2) + abs(y1-y2)
            adj[i] = append(adj[i], []int{dist, j})
            adj[j] = append(adj[j], []int{dist, i})
        }
    }

    res := 0
    visit := make(map[int]bool)
    q := minHeap{{0, 0}}
    heap.Init(&q)
    for len(visit) < n {
        item := heap.Pop(&q).([]int)
        cost, point := item[0], item[1]
        if visit[point] {
            continue
        }
        res += cost
        visit[point] = true
        for _, edge := range adj[point] {
            nextCost, nextPoint := edge[0], edge[1]
            if !visit[nextPoint] {
                heap.Push(&q, []int{nextCost, nextPoint})
            }
        }
    }
    return res
}

func abs(a int) int {
    if a < 0 {
        return -a
    }
    return a
}
```

### Code Efficiency

- **Time Complexity**: $O(N^2 \log N)$
    - Constructing the adjacency list takes $O(N^2)$ time.
    - We push and pop up to $O(N^2)$ edges in the min-heap, which takes $O(N^2 \log N)$ time.
- **Space Complexity**: $O(N^2)$
    - Storing the complete graph representation in the adjacency map takes $O(N^2)$ space.

---

## Approach 3: Prim's Algorithm (Optimized Dense Graph - No Heap)

For a complete (dense) graph where $E = O(N^2)$, we can optimize Prim's algorithm to run in $O(N^2)$ time and $O(N)$ space by scanning for the minimum distance node in an array instead of using a binary heap.

### Thought Process

1.  **Distance Tracking**:
    *   Maintain a `dist` array where `dist[i]` represents the minimum Manhattan distance to point `i` from any point already included in the MST. Initialize all values to a large number.
    *   Maintain a boolean array `visit` of size $N$ to track nodes currently in the MST.
2.  **Point Inclusion**:
    *   Start at point `0`. Perform $N-1$ iterations.
    *   In each iteration:
        *   Mark the current point `curr` as visited.
        *   For each unvisited node `i`, calculate the Manhattan distance to it from `curr`. Update `dist[i] = min(dist[i], currDist)`.
        *   Track the unvisited node `next` that has the smallest value in `dist[next]`.
        *   Add `dist[next]` to our total cost.
        *   Set `curr = next` and repeat.

### Go Code

``` go
func minCostConnectPoints(points [][]int) int {
    n := len(points)
    dist, visit := make([]int, n), make([]bool, n)
    for i := range dist {
        dist[i] = 100000000 // Represent infinity
    }
    curr, count, res := 0, 0, 0
    for count < n-1 {
        visit[curr] = true
        next := -1
        for i := 0; i < n; i++ {
            if visit[i] {
                continue
            }
            currDist := abs(points[curr][0]-points[i][0]) + abs(points[curr][1]-points[i][1])
            dist[i] = min(dist[i], currDist)
            if next == -1 || dist[i] < dist[next] {
                next = i
            }
        }
        res += dist[next]
        curr = next
        count++
    }
    return res
}

func abs(a int) int {
    if a < 0 {
        return -a
    }
    return a
}
```

### Code Efficiency

- **Time Complexity**: $O(N^2)$
    - We run a loop $N-1$ times. In each loop, we iterate through all $N$ points to update their distances and find the minimum next node. This yields a strict $O(N^2)$ time complexity, which is much faster than the heap-based $O(N^2 \log N)$ approaches on dense graphs.
- **Space Complexity**: $O(N)$
    - Uses only simple arrays of size $N$ (`dist` and `visit`), achieving optimal $O(N)$ space complexity.