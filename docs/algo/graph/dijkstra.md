# Dijkstra's Algorithm

## What is Dijkstra's algorithm?

- It is a Single Source Shortest Path (SSSP) algorithm for graphs with {==non-negative edge weights.==}
    - This property enables Dijkstra's algorithm to act in a greedy manner by always selecting the next most promising node.

## Complexity

- **Time Complexity**: $\mathcal{O}((E + V) \log V)$ (often simplified to $\mathcal{O}(E \log V)$ for connected graphs)
    - Extracting the minimum distance node from the priority queue takes $\mathcal{O}(\log V)$ time, which occurs at most $V$ times.
    - For every edge relaxation, we might insert/update a node in the queue, taking $\mathcal{O}(\log V)$ time. Across all $E$ edges, this takes $\mathcal{O}(E \log V)$ time.
    - Using a Fibonacci heap can optimize this to $\mathcal{O}(E + V \log V)$.
- **Space Complexity**: $\mathcal{O}(V + E)$
    - Storing the graph (e.g., using an adjacency list) requires $\mathcal{O}(V + E)$ space.
    - The priority queue stores up to $\mathcal{O}(E)$ elements in the lazy implementation (or $\mathcal{O}(V)$ in the eager implementation).
    - The distance array requires $\mathcal{O}(V)$ space.

## Algorithm Overview

- Maintain a `dist` array where the distance to every node is positive infinity. Mark the distance to the start node `s` to be 0.
- Maintain a Priority Queue of key-value pairs of (`node index`, `distance`), which tell us which node to visit next based on sorted min value.
- Insert (`s`, `0`) into the Priority Queue and loop while PQ is not empty pulling out the next most promising (`node index`, `distance`) pair.
- Iterate over all outward edges and relax each edge appending a new (`node index`, `distance`) key-value pair to the PQ for every relaxation.

## Sudo Code (Go Style)

Below is the conceptual Go-like pseudocode of Dijkstra's algorithm, assuming a generic min-priority queue structure.

```go
type Edge struct {
    To     int
    Weight int
}

type Item struct {
    Node     int
    Distance int
}

// dijkstra returns the shortest distances from start node to all other nodes.
func dijkstra(graph [][]Edge, start int, numNodes int) []int {
    // 1. Initialize distances to infinity, start node distance to 0
    dist := make([]int, numNodes)
    for i := range dist {
        dist[i] = math.MaxInt
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
                pq.Push(Item{Node: v, Distance: dist[v]})
            }
        }
    }

    return dist
}
```

### Complete, Executable Go Implementation

In standard Go, the priority queue is implemented using the standard library's `container/heap` package. Here is a fully functional implementation:

```go
package main

import (
	"container/heap"
	"math"
)

// Edge represents a directed edge in the graph.
type Edge struct {
	To     int
	Weight int
}

// Item represents a node and its current calculated distance from the start node.
type Item struct {
	Node     int
	Distance int
}

// PriorityQueue implements heap.Interface and holds Items.
type PriorityQueue []Item

func (pq PriorityQueue) Len() int           { return len(pq) }
func (pq PriorityQueue) Less(i, j int) bool { return pq[i].Distance < pq[j].Distance }
func (pq PriorityQueue) Swap(i, j int)      { pq[i], pq[j] = pq[j], pq[i] }
func (pq *PriorityQueue) Push(x any)        { *pq = append(*pq, x.(Item)) }
func (pq *PriorityQueue) Pop() any {
	old := *pq
	n := len(old)
	item := old[n-1]
	*pq = old[0 : n-1]
	return item
}

// Dijkstra calculates the shortest path from start node to all other nodes.
func Dijkstra(graph [][]Edge, start int, numNodes int) []int {
	dist := make([]int, numNodes)
	for i := range dist {
		dist[i] = math.MaxInt32
	}
	dist[start] = 0

	pq := &PriorityQueue{}
	heap.Init(pq)
	heap.Push(pq, Item{Node: start, Distance: 0})

	for pq.Len() > 0 {
		curr := heap.Pop(pq).(Item)
		u := curr.Node
		d := curr.Distance

		// Lazy deletion check
		if d > dist[u] {
			continue
		}

		for _, edge := range graph[u] {
			v := edge.To
			weight := edge.Weight
			if dist[u]+weight < dist[v] {
				dist[v] = dist[u] + weight
				heap.Push(pq, Item{Node: v, Distance: dist[v]})
			}
		}
	}

	return dist
}
```



## References

<iframe width="560" height="315" src="https://www.youtube.com/embed/pSqmAO-m7Lk?si=SfTovylzqXs6Wkk5" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
