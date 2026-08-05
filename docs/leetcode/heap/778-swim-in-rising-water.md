# 778. Swim in Rising Water

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/swim-in-rising-water/description/)

## Solution: Dijkstra's Algorithm (Min-Heap)

This problem can be modeled as finding the path from the top-left corner `(0, 0)` to the bottom-right corner `(n-1, n-1)` that minimizes the maximum cell elevation encountered along the path. This is a classic variation of the shortest path problem, which can be solved efficiently using **Dijkstra's algorithm** with a min-heap.

### Thought Process

1.  **Path Cost Definition**:
    *   Let the cost of a path be the maximum elevation among all cells in the path: $\max(grid[r_0][c_0], grid[r_1][c_1], \dots, grid[r_k][c_k])$.
    *   We want to find a path that minimizes this maximum cost.
2.  **Dijkstra's Exploration**:
    *   Maintain a **Min-Heap** containing elements of type `Item`. Each `Item` tracks a cell's coordinates `(row, col)` and the maximum path height (`height`) encountered to reach that cell.
    *   Always extract the cell with the smallest `height` from the min-heap. This guarantees that the first time we pop the destination cell `(n-1, n-1)`, we have found the optimal path.
3.  **State Transitions & Visited Tracking**:
    *   For the popped cell `(cr, cc)` with path cost `height`, we explore its four adjacent neighbors `(nr, nc)`.
    *   The path cost to reach a neighbor `(nr, nc)` becomes:
        $$\text{new\_height} = \max(\text{height}, grid[nr][nc])$$
    *   Mark visited cells in-place by updating `grid[cr][cc] = -1` to prevent redundant processing.

### Go Code

``` go
import (
    "container/heap"
)

type Item struct {
    row     int
    col     int
    height  int
}

type minHeap []Item
func (h minHeap) Len() int           { return len(h) }
func (h minHeap) Less(i, j int) bool { return h[i].height < h[j].height }
func (h minHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *minHeap) Push(x interface{}) { *h = append(*h, x.(Item)) }
func (h *minHeap) Pop() interface{} {
    last := (*h)[len(*h)-1]
    *h = (*h)[:len(*h)-1]
    return last
}

func swimInWater(grid [][]int) int {
    n := len(grid)
    h := minHeap{{0, 0, grid[0][0]}}
    heap.Init(&h)

    dr := [4]int{0, 1, 0, -1}
    dc := [4]int{1, 0, -1, 0}

    for h.Len() > 0 {
        item := heap.Pop(&h).(Item)
        cr, cc, height := item.row, item.col, item.height
        if cr == n-1 && cc == n-1 {
            return height
        } 
        grid[cr][cc] = -1
        for i := 0; i < 4; i++ {
            nr, nc := cr+dr[i], cc+dc[i]
            if nr < 0 || nr >= n || nc < 0 || nc >= n || grid[nr][nc] == -1 {
                continue
            }
            heap.Push(&h, Item{nr, nc, max(height, grid[nr][nc])})
        }
    }
    return -1
}
```

### Code Efficiency

- **Time Complexity**: $O(N^2 \log N)$
    - The grid contains $N^2$ cells (vertices) and up to $4N^2$ edges.
    - Each cell is pushed and popped from the min-heap at most once. Each heap operation takes $O(\log(N^2)) = O(\log N)$ time.
    - Therefore, the overall time complexity is $O(N^2 \log N)$.
- **Space Complexity**: $O(N^2)$
    - The min-heap stores at most $O(N^2)$ items at any given time.