# Dijkstra's Algorithm 2

## Shortest Path

If you wish to not only find the optimal distance to a particular node but also {++what sequence of nodes were taken++} to get there, you need to track some additional information.

### Sudo Code

The way to do this is to maintain `prev` array that tracks the index of the node you took to get to the node `i`. 

```go hl_lines="12 15 18 46 52"
type Edge struct {
    To     int
    Weight int
}

type Item struct {
    Node     int
    Distance int
}

// dijkstra returns the shortest distances and predecessor nodes.
func dijkstra(graph [][]Edge, start int, numNodes int) ([]int, []int) {
    // 1. Initialize distances to infinity, start node distance to 0, and prev to -1
    dist := make([]int, numNodes)
    prev := make([]int, numNodes)
    for i := range dist {
        dist[i] = math.MaxInt
        prev[i] = -1
    }
    dist[start] = 0

    // 2. Initialize Priority Queue (Min-Heap) sorted by distance
    pq := NewMinPriorityQueue() // Conceptual Priority Queue
    pq.Push(Item{Node: start, Distance: 0})

    // 3. Process the nodes in the priority queue
    for !pq.IsEmpty() {
        curr := pq.Pop() // Get the node with the minimum distance
        u := curr.Node
        d := curr.Distance

        // Lazy deletion check: if we already found a shorter path to u
        // since this item was queued, skip it.
        if d > dist[u] {
            continue
        }

        // 4. Relax neighboring edges
        for _, edge := range graph[u] {
            v := edge.To
            weight := edge.Weight
            
            // If we found a shorter path to v through u
            if dist[u]+weight < dist[v] {
                dist[v] = dist[u] + weight
                prev[v] = u
                pq.Push(Item{Node: v, Distance: dist[v]})
            }
        }
    }

    return dist, prev
}
```

To reconstruct and return the shortest path from `start` to `end` using the `prev` array, we can trace the predecessors backwards from `end` to `start` and then reverse the resulting path:

```go
func findShortestPath(graph [][]Edge, start int, end int, numNodes int) []int {
    dist, prev := dijkstra(graph, start, numNodes)
    path := []int{}
    
    // If the destination is unreachable, return an empty path
    if dist[end] == math.MaxInt {
        return path
    }
    
    // Trace backward from end to start using the prev array
    for i := end; i >= 0; i = prev[i] {
        path = append(path, i)
    }
    
    // Reverse the path to get the correct start-to-end order
    for i, j := 0, len(path)-1; i < j; i, j = i+1, j-1 {
        path[i], path[j] = path[j], path[i]
    }
    
    return path
}
```
---
## Stopping Early

The main idea for stopping early is that Dijkstra's algorithm processes each next most promising node in order. So if the destination node has been visited, its shortest distance will not change as more future nodes are visited.

### Sudo Code

> [!IMPORTANT]
> We can only guarantee the shortest path is found once the `end` node is **popped** (dequeued) from the priority queue. Do **not** stop early when the `end` node is first **pushed** (enqueued), as a shorter path through another vertex might still be discovered later.

```go hl_lines="22-24"
// dijkstraEarlyStop returns the shortest path distances and predecessor array, 
// stopping the search early once the 'end' node's shortest path is finalized.
func dijkstraEarlyStop(graph [][]Edge, start int, end int, numNodes int) ([]int, []int) {
    dist := make([]int, numNodes)
    prev := make([]int, numNodes)
    for i := range dist {
        dist[i] = math.MaxInt
        prev[i] = -1
    }
    dist[start] = 0

    pq := NewMinPriorityQueue()
    pq.Push(Item{Node: start, Distance: 0})

    for !pq.IsEmpty() {
        curr := pq.Pop()
        u := curr.Node
        d := curr.Distance

        // Stopping early: Once the end node is popped, we are guaranteed 
        // that its shortest path distance is finalized.
        if u == end {
            break
        }

        if d > dist[u] {
            continue
        }

        for _, edge := range graph[u] {
            v := edge.To
            weight := edge.Weight
            
            if dist[u]+weight < dist[v] {
                dist[v] = dist[u] + weight
                prev[v] = u
                pq.Push(Item{Node: v, Distance: dist[v]})
            }
        }
    }

    return dist, prev
}
```

---
## Eager Dijkstra's using an Indexed Priority Queue

Our current lazy implementation of Dijkstra's inserts **duplicate key-value pairs** in our PQ because it's more efficient to insert a new key-value pair in $\mathcal{O}(log V)$ than it is to update an existing key's value in $\mathcal{O}(V)$.

This approach is inefficient for dense graphs because we end up with **several stale outdated key-value pairs** in our PQ. The eager version of Dijkstra's avoids duplicate key-value pairs and supports efficient value updates in $\mathcal{O}(log V)$ by using an {++Indexed Priority Queue (IPQ)++}.

### Sudo Code

Below is the Go-style pseudocode for Eager Dijkstra's algorithm. It assumes a conceptual `IndexedMinPQ` interface that enables checking if a node is in the queue and decreasing its key value in $\mathcal{O}(\log V)$ time.

```go hl_lines="39-45"
// Conceptual interface for an Indexed Min-Priority Queue (IPQ)
type IndexedMinPQ interface {
    Push(node int, dist int)
    DecreaseKey(node int, dist int)
    Contains(node int) bool
    Pop() (node int, dist int)
    IsEmpty() bool
}

// dijkstraEager calculates shortest paths using an Indexed Priority Queue to avoid duplicate keys.
func dijkstraEager(graph [][]Edge, start int, numNodes int) ([]int, []int) {
    dist := make([]int, numNodes)
    prev := make([]int, numNodes)
    for i := range dist {
        dist[i] = math.MaxInt
        prev[i] = -1
    }
    dist[start] = 0

    // Initialize the Indexed Min-Priority Queue
    ipq := NewIndexedMinPQ()
    ipq.Push(start, 0)

    for !ipq.IsEmpty() {
        // Pop the node with the minimum distance
        u, d := ipq.Pop()

        // Notice that we do not need the lazy check "if d > dist[u]" here
        // because the IPQ guarantees there are no duplicate entries.

        for _, edge := range graph[u] {
            v := edge.To
            weight := edge.Weight
            
            if dist[u]+weight < dist[v] {
                dist[v] = dist[u] + weight
                prev[v] = u
                
                if ipq.Contains(v) {
                    // Eagerly update (decrease) the distance value for v in the PQ
                    ipq.DecreaseKey(v, dist[v])
                } else {
                    // Insert v into the PQ for the first time
                    ipq.Push(v, dist[v])
                }
            }
        }
    }

    return dist, prev
}
```

---
## References

<iframe width="560" height="315" src="https://www.youtube.com/embed/pSqmAO-m7Lk?si=7WSJV8nj568w5J4n&amp;start=732" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
