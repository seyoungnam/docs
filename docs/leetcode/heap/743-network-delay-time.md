# 743. Network Delay Time

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/network-delay-time/description/)

## Solution: Dijkstra's Algorithm (Min-Heap)

Since all edge weights (transmission delays) are non-negative, we can find the shortest path from the source node `k` to all other nodes using **Dijkstra's Algorithm**.

### Thought Process

1.  **Build Adjacency List**:
    *   Construct an adjacency list `adj` from the given `times` array, mapping each source node to a list of destination nodes and their respective delays.

2.  **Initialize Distance Array**:
    *   Create a `minTime` array of size `n + 1` initialized to infinity (`math.MaxInt32`) to track the minimum time required to reach each node.
    *   Set the start node's time to `0` (`minTime[k] = 0`).

3.  **Min-Heap (Priority Queue) Optimization**:
    *   Implement Go's `container/heap` interface to maintain a priority queue of nodes sorted by their accumulated cost/time.
    *   Start by pushing the source node `{k, 0}` to the heap.

4.  **Edge Relaxation**:
    *   Pop the node `curr` with the smallest travel cost from the heap.
    *   If `curr.cost` is larger than the already recorded `minTime[curr.num]`, skip it (prune suboptimal paths).
    *   Iterate through all neighbors of `curr`. If the path through `curr` is shorter than the neighbor's current recorded `minTime`, update it and push the neighbor with the new time to the heap.

5.  **Determine Final Result**:
    *   After the queue is exhausted, scan the `minTime` array from node `1` to `n`.
    *   If any node is still at `math.MaxInt32`, it means not all nodes are reachable; return `-1`.
    *   Otherwise, return the maximum time in the array. This value represents the time taken for the signal to reach the furthest node.

### Go Code

``` go
import (
    "container/heap"
    "math"
)

type node struct {
    num     int
    cost    int
}

type minHeap []node

func (h minHeap) Len() int           { return len(h) }
func (h minHeap) Less(i, j int) bool { return h[i].cost < h[j].cost }
func (h minHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *minHeap) Push(x interface{}) { *h = append(*h, x.(node)) }
func (h *minHeap) Pop() interface{} {
    last := (*h)[len(*h)-1]
    *h = (*h)[:len(*h)-1]
    return last
}

func networkDelayTime(times [][]int, n int, k int) int {
    adj := make([][]node, n+1)
    for _, t := range times {
        src, dst, dur := t[0], t[1], t[2]
        adj[src] = append(adj[src], node{dst, dur})
    }

    minTime := make([]int, n+1)
    for i := 1; i < n+1; i++ {
        minTime[i] = math.MaxInt32
    }
    
    queue := &minHeap{node{k, 0}}
    heap.Init(queue)
    minTime[k] = 0
    for queue.Len() > 0 {
        curr := heap.Pop(queue).(node)
        dst, time := curr.num, curr.cost

        if time > minTime[dst] {
            continue
        }

        for _, nextNode := range adj[dst] {
            newPathTime := time + nextNode.cost
            if newPathTime < minTime[nextNode.num] {
                minTime[nextNode.num] = newPathTime
                heap.Push(queue, node{nextNode.num, newPathTime})
            }
        }
    }
    res := -1
    for i := 1; i < n+1; i++ {
        if minTime[i] == math.MaxInt32 {
            return -1
        }
        res = max(res, minTime[i])
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(E \log V)$
    - $E$ is the number of edges (length of `times`), and $V$ is the number of vertices (`n`).
    - Standard Dijkstra using a binary heap takes $O(E \log V)$ time since each edge relaxation performs a heap push/pop operation taking $O(\log V)$ time.
- **Space Complexity**: $O(E + V)$
    - The adjacency list requires $O(E + V)$ space.
    - The heap can store up to $O(E)$ nodes in the worst case, and the `minTime` array requires $O(V)$ space.