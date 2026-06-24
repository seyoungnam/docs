# 909. Snakes and Ladders

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/snakes-and-ladders/description/)

## Solution: Breadth-First Search (BFS) with Boustrophedon Coordinate Mapping

To find the minimum number of die rolls required to reach the last square ($n^2$), we model the board as an unweighted directed graph and use Breadth-First Search (BFS). The primary challenges are mapping the 1D board labels (from $1$ to $n^2$) to the 2D boustrophedon (zigzag) matrix coordinates, and handling the automatic slide transitions when landing on a snake or ladder.

### Thought Process

1.  **Boustrophedon Coordinate Mapping**:
    - The board starts at the bottom-left corner (row $n-1$, col $0$) and zigzags upwards.
    - Given a 1D square index `k` (from $1$ to $n^2$):
        - Compute the quotient and remainder: `q = (k - 1) / n` (number of rows from the bottom) and `r = (k - 1) % n` (horizontal offset).
        - The matrix row is `row = n - 1 - q` (since index 0 is at the top).
        - The column direction alternates:
            - If `q` is even, we count from left to right: `col = r`.
            - If `q` is odd, we count from right to left: `col = n - 1 - r`.
2.  **Shortest Path BFS**:
    - BFS is ideal for finding the shortest path in an unweighted graph.
    - Maintain a `visited` array of size $n^2 + 1$.
    - Start the queue at square `1` and mark it visited.
3.  **Simulate Die Rolls**:
    - For the current square `curr`, explore all reachable squares from `curr + 1` to `curr + 6` (without exceeding $n^2$).
    - For each unvisited square `next`:
        - Mark `next` as visited.
        - Retrieve the 2D coordinates `(r, c)` of `next` using our mapping helper.
        - Check if `next` has a snake or ladder (`board[r][c] != -1`). If so, the destination becomes `board[r][c]`. Otherwise, it remains `next`.
        - Push the final destination onto the queue.
4.  **Result**:
    - Once we dequeue the final square $n^2$, return the number of `steps` taken.
    - If the queue is exhausted and the target is unreachable, return `-1`.

### Go Code

``` go
func snakesAndLadders(board [][]int) int {
    n := len(board)
    var getRowCol func(k int) (int, int) 
    getRowCol = func(k int) (int, int) {
        q, r := (k-1)/n, (k-1)%n
        row := n-1-q
        col := r
        if q%2 == 1 {
            col = n-1-r
        }
        return row, col
    }
    
    visited := make([]bool, n*n+1)
    queue := make([]int, 0)
    queue = append(queue, 1)
    visited[1] = true
    steps := 0
    for len(queue) > 0 {
        size := len(queue)
        for range size {
            curr := queue[0]
            queue = queue[1:]

            if curr == n*n {
                return steps
            }

            for i := 1; i <= 6 && curr+i <= n*n; i++ {
                next := curr+i
                if !visited[next] {
                    visited[next] = true
                    
                    r, c := getRowCol(next)
                    destination := next
                    if board[r][c] != -1 {
                        destination = board[r][c]
                    }
                    queue = append(queue, destination)
                }
            }
        }
        steps++
    }
    return -1
}
```

### Code Efficiency

- **Time Complexity**: $O(n^2)$
    - Where $n \times n$ is the size of the board. The BFS visits each square at most once and simulates up to 6 moves per square. Thus, the total time complexity scales linearly with the number of cells on the board, $O(n^2)$.
- **Space Complexity**: $O(n^2)$
    - The `visited` array and the BFS queue store at most $n^2$ elements.