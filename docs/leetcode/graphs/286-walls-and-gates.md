# 286. Walls and Gates

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/walls-and-gates/description/)

You are given a `m x n` 2D grid `rooms` initialized with these three possible values:
*   `-1`: A wall or an obstacle.
*   `0`: A gate.
*   `INF`: Infinity, represented by the integer `2^31 - 1 = 2147483647` (i.e., `math.MaxInt32` in Go), which represents an empty room.

Fill each empty room with the distance to its nearest gate. If it is impossible to reach a gate, it should be filled with `INF`.

---

## Solution: Multi-Source Breadth-First Search (BFS)

Instead of performing a search starting from each empty room, we can run a **Multi-Source BFS** starting simultaneously from all gates. Since BFS explores nodes level-by-level (by distance), the first time we visit an empty room, we are guaranteed to have found the shortest path to a gate.

### Thought Process

1.  **Queue Initialization**:
    *   Find all gates (`rooms[r][c] == 0`) in the grid.
    *   Push each gate's coordinates and initial distance `0` onto a queue as `[3]int{r, c, 0}`.
2.  **BFS Traversal**:
    *   While the queue is not empty, pop the front element `[r, c, v]`.
    *   Set the value of the room to the minimum of its current value and the distance `v`:
        $$\text{rooms[r][c]} = \min(\text{rooms[r][c]}, v)$$
    *   Check all 4 adjacent neighbors (up, down, left, right):
        *   If a neighbor `[nr, nc]` is within bounds and is an empty room (`rooms[nr][nc] == math.MaxInt32`):
            *   Push it onto the queue with the incremented distance `v + 1`.
3.  **Completion**:
    *   When the queue is empty, all reachable rooms will have been updated in-place with their minimum distances.

> [!TIP]
> **Optimization Note**:
> In the code below, we mark the room's value when it is *popped* from the queue. To avoid duplicate queue entries when multiple paths reach the same cell in the same step, you can assign `rooms[nr][nc] = v + 1` *immediately* when appending it to the queue.

### Go Code

``` go
import "math"

func wallsAndGates(rooms [][]int)  {
    if len(rooms) == 0 {
        return
    }
    
    ROWS, COLS := len(rooms), len(rooms[0])
    q := [][3]int{}
    
    // Find all gates and add them to the queue
    for r := 0; r < ROWS; r++ {
        for c := 0; c < COLS; c++ {
            if rooms[r][c] == 0 {
                q = append(q, [3]int{r, c, 0})
            }
        }
    }
    
    dr := [4]int{0, -1, 0, 1}
    dc := [4]int{1, 0, -1, 0}
    
    for len(q) > 0 {
        curr := q[0]
        q = q[1:]
        r, c, v := curr[0], curr[1], curr[2]
        
        // Update current room distance
        rooms[r][c] = min(rooms[r][c], v)

        // Explore 4 neighbors
        for i := 0; i < 4; i++ {
            nr, nc := r+dr[i], c+dc[i]
            if nr >= 0 && nr < ROWS && nc >= 0 && nc < COLS && rooms[nr][nc] == math.MaxInt32 {
                q = append(q, [3]int{nr, nc, v+1})
            }
        }
    }
}
```

### Code Efficiency

- **Time Complexity**: $O(M \times N)$
    - Where $M$ is the number of rows and $N$ is the number of columns. Each room is added to the queue at most once.
- **Space Complexity**: $O(M \times N)$
    - The queue can store up to $M \times N$ elements in the worst case (e.g., when the entire grid consists of gates).