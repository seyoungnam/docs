# 874. Walking Robot Simulation

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/walking-robot-simulation/description/)

## Solution: Hash Map Simulation

### Thought Process

1.  **Represent Direction Vector**:
    *   Directions are ordered clockwise: `North (0)`, `East (1)`, `South (2)`, `West (3)`.
    *   We can map these to coordinate offsets:
        *   `North`: `(0, 1)`
        *   `East`: `(1, 0)`
        *   `South`: `(0, -1)`
        *   `West`: `(-1, 0)`
    *   A left turn (`-2`) changes the direction index by `(i + 3) % 4`.
    *   A right turn (`-1`) changes the direction index by `(i + 1) % 4`.

2.  **Optimize Obstacle Lookups**:
    *   Looking up obstacles in the input slice directly takes $O(O)$ time per step, which is too slow.
    *   Instead, we insert all obstacle coordinates into a hash map. In Go, struct types are comparable, so we can define a `Point` struct and use it as a key for `map[Point]bool`.
    *   This reduces the lookup time for any coordinate to $O(1)$ on average.

3.  **Simulate the Path**:
    *   For each step command, move one unit forward at a time.
    *   If the next coordinate `(x + dx, y + dy)` is found in the obstacle hash map, terminate the current step command immediately.
    *   Otherwise, move to the next coordinate, and update the maximum squared Euclidean distance from the origin ($x^2 + y^2$).

### Go Code

``` go
type Point struct {
    x, y int
}

func robotSim(commands []int, obstacles [][]int) int {
    dir := [4][2]int{{0, 1}, {1, 0}, {0, -1}, {-1, 0}}
    x, y, i := 0, 0, 0
    res := 0
    
    block := make(map[Point]bool)
    for _, obs := range obstacles {
        x, y := obs[0], obs[1]
        block[Point{x: x, y: y}] = true
    }

    for _, command := range commands {
        switch command {
        case -2:
            i = (i+3)%4
        case -1:
            i = (i+1)%4
        default:
            for range command {
                dx, dy := dir[i][0], dir[i][1]
                nx, ny := x+dx, y+dy
                
                if block[Point{x:nx, y:ny}] {
                    break
                }

                x, y = nx, ny
                res = max(res, x*x + y*y)
            }
        }
    }
    return res
}
```


### Code Efficiency

- **Time Complexity**: $O(O + C \times M)$
    - $O(O)$ to populate the obstacle hash map, where $O$ is the number of obstacles.
    - $O(C \times M)$ to simulate the steps, where $C$ is the number of commands and $M$ is the maximum number of steps per command ($M \le 9$). Because $M$ is bounded by a small constant, this runs in linear time with respect to the input size.
- **Space Complexity**: $O(O)$
    - We allocate a hash map to store the coordinates of the $O$ obstacles.

