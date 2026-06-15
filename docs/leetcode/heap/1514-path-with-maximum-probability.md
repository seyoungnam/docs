# 1514. Path with Maximum Probability

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/path-with-maximum-probability/description/)

## Solution: Dijkstra's Algorithm (Max-Heap)

This problem is a variation of the single-source shortest path problem. Instead of minimizing the *sum* of edge weights, we want to maximize the *product* of edge probabilities.

### Thought Process

1.  **Dijkstra Eligibility**:
    *   Since all edge probabilities lie in the range $[0.0, 1.0]$, multiplying them along a path monotonically decreases the probability (e.g., $1.0 \times 0.5 \times 0.2 = 0.1$). This decreasing property allows us to greedily find the optimal path using Dijkstra's algorithm.
    *   We use a **Max-Heap** instead of a Min-Heap so we always process the path with the highest probability first.

2.  **Initialize Probability Table**:
    *   Create a `pTable` array of size `n` initialized to `0.0` to track the maximum probability of reaching each node.
    *   Set the start node's probability to `1.0` (`pTable[start_node] = 1.0`), representing a 100% chance of starting at the source.

3.  **Adjacency List**:
    *   Build an undirected adjacency list `adj` from the `edges` and `succProb` slices. Each entry stores the destination node and the success probability of the edge.

4.  **Dijkstra Traversal**:
    *   Push the starting node `{start_node, 1.0}` to the max-heap.
    *   Pop the entry `curr` with the maximum probability.
    *   **Early Termination**: If `curr.node == end_node`, we can return the probability immediately. Because we are using a Max-Heap, the first time we pop the target node, it is guaranteed to have the maximum possible probability.
    *   **Edge Relaxation**: For each neighbor of `curr`, calculate the path probability: `nextProb = curr.prob * next.prob`. If this probability is strictly greater than the previously recorded `pTable[next.node]`, update the table and push the neighbor to the heap.

5.  **Unreachable Path**:
    *   If the queue becomes empty without reaching `end_node`, return `0.0`.

### Go Code

``` go
import "container/heap"

type entry struct {
    node    int
    prob    float64
}

type maxHeap []entry

func (h maxHeap) Len() int           { return len(h) }
func (h maxHeap) Less(i, j int) bool { return h[i].prob > h[j].prob }
func (h maxHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *maxHeap) Push(x interface{}) { *h = append(*h, x.(entry)) }
func (h *maxHeap) Pop() interface{} {
    last := (*h)[len(*h)-1]
    *h = (*h)[:len(*h)-1]
    return last
}

func maxProbability(n int, edges [][]int, succProb []float64, start_node int, end_node int) float64 {
    adj := make([][]entry, n)
    for i, e := range edges {
        u, v, p := e[0], e[1], succProb[i]
        adj[u] = append(adj[u], entry{v, p})
        adj[v] = append(adj[v], entry{u, p})
    }
    pTable := make([]float64, n)
    pTable[start_node] = 1.0

    queue := &maxHeap{entry{start_node, 1.0}}
    heap.Init(queue)
    for queue.Len() > 0 {
        curr := heap.Pop(queue).(entry)

        if curr.node == end_node {
            return curr.prob
        }

        if curr.prob < pTable[curr.node] {
            continue
        }
        
        for _, next := range adj[curr.node] {
            nextProb := curr.prob * next.prob
            if nextProb > pTable[next.node] {
                pTable[next.node] = nextProb
                heap.Push(queue, entry{next.node, nextProb})
            }
        }
    }
    return pTable[end_node]
}
```

### Code Efficiency

- **Time Complexity**: $O(E \log V)$
    - $E$ is the number of edges (length of `edges`), and $V$ is the number of vertices (`n`).
    - Just like standard Dijkstra, each edge relaxation performs a heap operation taking $O(\log V)$ time, resulting in $O(E \log V)$ worst-case time complexity.
- **Space Complexity**: $O(E + V)$
    - The undirected adjacency list requires $O(E + V)$ space.
    - The max-heap and the `pTable` array require $O(E)$ and $O(V)$ space respectively.