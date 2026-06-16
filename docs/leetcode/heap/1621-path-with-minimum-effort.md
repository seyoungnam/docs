# 1631. Path With Minimum Effort

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/path-with-minimum-effort/description/)

> **Note on Filename**: This file is named `1621-path-with-minimum-effort.md` in the repository, but it refers to **LeetCode 1631**.

## Solution: Dijkstra's Algorithm (Min-Heap)

The "effort" of a path is defined as the maximum absolute difference between consecutive cells. We want to find a path from the top-left to the bottom-right cell that minimizes this maximum effort.

### Thought Process

1.  **Dijkstra Eligibility**:
    *   The path effort is monotonic—as we traverse more cells, the maximum difference along the path can only increase or stay the same:
        $$\text{newEffort} = \max(\text{currentEffort}, \text{stepEffort})$$
    *   This property allows us to use **Dijkstra's Algorithm** to greedily find the path of minimum effort.
    *   We use a **Min-Heap** to always explore the path with the smallest effort first.

2.  **Dijkstra Traversal**:
    *   Start at the top-left cell `(0, 0)` with an effort of `0`.
    *   Use a 2D integer array `efforts` of size `ROWS x COLS` initialized to infinity (`math.MaxInt32`) to track the minimum effort required to reach each cell.
    *   Set `efforts[0][0] = 0`.
    *   **Early Termination**: The first time we pop the target cell `(ROWS-1, COLS-1)` from the min-heap, we return its effort immediately. The min-heap guarantees this is the minimum possible effort path.
    *   **Pruning**: If the popped path effort `d` is larger than the recorded `efforts[r][c]`, we skip it as it's a suboptimal path.
    *   **Relaxation**: From the current cell `(r, c)`, check all 4 adjacent directions:
        *   For each neighbor `(nr, nc)`, calculate the new path effort:
            $$\text{newDiff} = \max(d, | \text{heights}[r][c] - \text{heights}[nr][nc] | )$$
        *   If `newDiff` is smaller than the current `efforts[nr][nc]`, update `efforts[nr][nc] = newDiff` and push `entry{nr, nc, newDiff}` to the heap.

### Go Code

``` go
import (
    "container/heap"
    "math"
)

type entry struct {
    row     int
    col     int
    diff    int
}

type minHeap []entry

func (h minHeap) Len() int { return len(h) }
func (h minHeap) Less(i, j int) bool { return h[i].diff < h[j].diff }
func (h minHeap) Swap(i, j int) { h[i], h[j] = h[j], h[i] }
func (h *minHeap) Push(x interface{}) { *h = append(*h, x.(entry)) }
func (h *minHeap) Pop() interface{} {
    last := (*h)[len(*h)-1]
    *h = (*h)[:len(*h)-1]
    return last
}

func minimumEffortPath(heights [][]int) int {
    ROWS, COLS := len(heights), len(heights[0])
    efforts := make([][]int, ROWS)
    for r := 0; r < ROWS; r++ {
        efforts[r] = make([]int, COLS)
        for c := 0; c < COLS; c++ {
            efforts[r][c] = math.MaxInt32
        }
    }
    
    dr := [4]int{1, 0, -1, 0}
    dc := [4]int{0, 1, 0, -1}
    
    queue := &minHeap{entry{0,0,0}}
    heap.Init(queue)
    efforts[0][0] = 0

    for queue.Len() > 0 {
        curr := heap.Pop(queue).(entry)
        r, c, d := curr.row, curr.col, curr.diff
        
        if r == ROWS-1 && c == COLS-1 {
            return d
        }

        if d > efforts[r][c] {
            continue
        }

        for i := 0; i < 4; i++ {
            nr := r + dr[i]
            nc := c + dc[i]
            
            if nr < 0 || nr >= ROWS || nc < 0 || nc >= COLS {
                continue
            }
            newDiff := max(d, abs(heights[r][c] - heights[nr][nc]))
            if newDiff < efforts[nr][nc] {
                efforts[nr][nc] = newDiff
                heap.Push(queue, entry{nr, nc, newDiff})
            }
            
        }
    }
    return -1
}

func abs(a int) int {
    if a < 0 {
        return -a
    }
    return a
}
```

### Code Efficiency

- **Time Complexity**: $O(R \times C \log (R \times C))$
    - Let $R$ and $C$ be the rows and columns of the grid. The number of grid cells (vertices) is $V = R \times C$, and the number of transitions (edges) is $E \approx 4 \times R \times C$.
    - Pushing and popping from the min-heap takes $O(\log V)$ time. In the worst case, we perform a heap operation for each transition, resulting in $O(E \log V) = O(R \times C \log (R \times C))$ time complexity.
- **Space Complexity**: $O(R \times C)$
    - Required to store the 2D `efforts` array of size $R \times C$ and the min-heap elements.
