# 787. Cheapest Flights Within K Stops

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/cheapest-flights-within-k-stops/description/)

## Solution: Bellman-Ford Algorithm (K + 1 Iterations)

The problem asks for the cheapest price from `src` to `dst` with at most `k` stops. A path with at most `k` stops contains at most `k + 1` flights (edges). This bound makes **Bellman-Ford** a natural fit.

### Thought Process

1.  **Iterative Edge Relaxation**:
    *   Standard Bellman-Ford relaxes all edges $V-1$ times to find the absolute shortest paths.
    *   Here, we only relax all edges exactly $K+1$ times. The $i$-th iteration computes the cheapest prices to reach all nodes using at most $i$ edges.

2.  **Double Buffering (Preventing Chain Relaxation)**:
    *   If we update our costs array in-place, a relaxed edge $A \rightarrow B$ could immediately be used to relax $B \rightarrow C$ in the same iteration, representing a 2-edge path in a single step.
    *   To prevent this, we copy `costs` to a temporary array `temp` at the start of each iteration. We read path costs from `costs` (representing paths of length $\le i-1$) and write the relaxed updates to `temp` (representing paths of length $\le i$).
    *   At the end of the iteration, we apply the updates: `costs = temp`.

3.  **Result**:
    *   If `costs[dst]` is still `math.MaxInt32` after $K+1$ iterations, the destination is unreachable; return `-1`. Otherwise, return `costs[dst]`.

### Go Code

``` go
import "math"

func findCheapestPrice(n int, flights [][]int, src int, dst int, k int) int {
    costs := make([]int, n)
    for i := range costs {
        costs[i] = math.MaxInt32
    }
    costs[src] = 0

    for i := 0; i <= k; i++ {
        temp := make([]int, n)
        copy(temp, costs)

        for _, flight := range flights {
            from, to, price := flight[0], flight[1], flight[2]
            if costs[from] != math.MaxInt32 {
                if costs[from]+price < temp[to] {
                    temp[to] = costs[from] + price
                }
            }
        }
        costs = temp
    }
    if costs[dst] == math.MaxInt32 {
        return -1
    }
    return costs[dst]
}
```

### Code Efficiency

- **Time Complexity**: $O(K \times E)$
    - We run the outer loop $K+1$ times. In each loop, we iterate through all $E$ flights (edges).
- **Space Complexity**: $O(V)$
    - We allocate the `costs` and `temp` arrays of size $V$ ($n$), requiring linear auxiliary space.

---

## Alternative Solution: Dijkstra's Algorithm (Min-Heap)

Since this file is placed under the `heap` directory, we can also solve this using a modified version of **Dijkstra's Algorithm**.

### Thought Process

1.  **State Representation**:
    *   We track states using the `Info` struct: `Info{node, price, stops}`.
    *   The `Info` struct is reused for both graph edges (where `stops` is unused/0) and priority queue states (where `price` is the accumulated cost and `stops` is the number of flights/edges taken).
2.  **Pruning Suboptimal Paths**:
    *   We maintain a `minStops` array where `minStops[u]` stores the minimum number of flights used to reach node `u` so far.
    *   If we pop a state with `stops >= minStops[node]`, we discard it, as we have already found a path to the same node with fewer or equal flights at a cheaper or equal price (since the min-heap pops by minimum price first).
3.  **Early Stop-Limit Pruning**:
    *   Before pushing a neighbor state to the min-heap, we check if the new flight count satisfies `nextStops <= k+1` (which restricts the path to at most $k$ intermediate stops).
4.  **Greedy Order**:
    *   Since the min-heap always pops the state with the minimum accumulated price first, the first time we pop `dst` from the queue, we are guaranteed to have found the cheapest price within the stop limit.

### Go Code

``` go
import (
    "container/heap"
    "math"
)

type Info struct {
    node    int
    price   int
    stops   int
}

type minHeap []Info

func (h minHeap) Len() int { return len(h) }
func (h minHeap) Less(i, j int) bool {return h[i].price < h[j].price }
func (h minHeap) Swap(i, j int) { h[i], h[j] = h[j], h[i] }
func (h *minHeap) Push(x interface{}) { *h = append(*h, x.(Info)) }
func (h *minHeap) Pop() interface{} {
    last := (*h)[len(*h)-1]
    *h = (*h)[:len(*h)-1]
    return last
}

func findCheapestPrice(n int, flights [][]int, src int, dst int, k int) int {
    graph := make([][]Info, n)
    for _, f := range flights {
        from, to, price := f[0], f[1], f[2]
        graph[from] = append(graph[from], Info{to, price, 0})
    }

    minStops := make([]int, n)
    for i := range minStops {
        minStops[i] = math.MaxInt32
    }

    q := &minHeap{Info{src, 0, 0}}
    heap.Init(q)
    
    for q.Len() > 0 {
        curr := heap.Pop(q).(Info)
        node, price, stops := curr.node, curr.price, curr.stops
        if node == dst {
            return price
        }
        if stops >= minStops[node] {
            continue
        }
        minStops[node] = stops
        for _, next := range graph[node] {
            nextNode := next.node
            nextPrice := price + next.price
            nextStops := stops + 1
            if nextStops <= k+1 {
                heap.Push(q, Info{nextNode, nextPrice, nextStops})
            } 
        }
    }
    return -1
}
```

### Code Efficiency

- **Time Complexity**: $O(E \log V)$
    - In the worst case, each transition performs a heap operation. The `minStops` pruning ensures we visit each state at most once.
- **Space Complexity**: $O(E + V)$
    - Adjacency list requires $O(E + V)$ space, the `minStops` array requires $O(V)$ space, and the heap holds up to $O(E)$ elements in the worst case.