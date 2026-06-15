# 2069. Walking Robot Simulation II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/walking-robot-simulation-ii/description/)

## Solution: Simulation with Modular Math

The robot only moves along the boundary of the grid in a counter-clockwise direction. Since the movement path forms a closed loop, we can use modular arithmetic to optimize the simulation.

### Thought Process

1.  **Calculate Perimeter**:
    *   The total number of boundary cells (the perimeter) is the total distance traveled in one complete loop:
        $$\text{Perimeter} = 2 \times (\text{width} - 1) + 2 \times (\text{height} - 1) = 2 \times (\text{width} + \text{height} - 2)$$
    *   Any movement of size equal to the perimeter returns the robot to its exact starting position.

2.  **Modular Reduction**:
    *   We can reduce large values of `num` by taking `num %= perimeter`. This bounds the remaining steps to be strictly less than one full loop ($O(\text{width} + \text{height})$).

3.  **Handling the Origin Edge Case**:
    *   If `num %= perimeter` becomes `0` and the robot is at the origin `(0, 0)`, it means the robot completed one or more full loops. 
    *   Although the robot is back at `(0, 0)`, its direction has changed. Since it arrived at `(0, 0)` by moving downwards along the left boundary, its direction must be set to **South** (`dir = 3` or `"South"`), rather than the initial **East** direction.

4.  **Simulation of Remaining Steps**:
    *   Since the remaining `num` is small ($\text{num} < \text{perimeter}$), we simulate the movement step-by-step.
    *   Calculate the next position `(nx, ny)` in the current direction.
    *   If it goes out of bounds, rotate left (counter-clockwise): `dir = (dir + 1) % 4`.
    *   Otherwise, move the robot and decrement `num`.

### Go Code

``` go
type Robot struct {
    x         int
    y         int
    dir         int
    dirString   [4]string
    dx          [4]int
    dy          [4]int
    width       int
    height      int
    perimeter   int
}


func Constructor(width int, height int) Robot {
    perimeter := 2*(width-1) + 2*(height-1)
    return Robot{
        x: 0,
        y: 0,
        dir: 0,
        dirString: [4]string{"East", "North", "West", "South"},
        dx: [4]int{1, 0, -1, 0},
        dy: [4]int{0, 1, 0, -1},
        width: width,
        height: height,
        perimeter: perimeter,
    }
}


func (this *Robot) Step(num int)  {
    num %= this.perimeter
    if num == 0 && this.x == 0 && this.y == 0 {
        this.dir = 3 // South
        return
    }

    for num > 0 {
        nx := this.x + this.dx[this.dir]
        ny := this.y + this.dy[this.dir]

        if nx < 0 || nx >= this.width || ny < 0 || ny >= this.height {
            this.dir = (this.dir + 1) % 4
            continue
        }

        this.x = nx
        this.y = ny
        num--
    }
}


func (this *Robot) GetPos() []int {
    return []int{this.x, this.y}
}


func (this *Robot) GetDir() string {
    return this.dirString[this.dir]
}
```

### Code Efficiency

- **Time Complexity**:
    - `Constructor`: $O(1)$
    - `Step`: $O(\text{width} + \text{height})$
        - Taking `num %= perimeter` limits `num` to be less than the perimeter, which is $2 \times (\text{width} + \text{height} - 2)$. Simulating the remaining steps takes time proportional to the dimensions of the grid.
    - `GetPos` / `GetDir`: $O(1)$
- **Space Complexity**: $O(1)$
    - The `Robot` struct only stores a few primitive state variables, requiring constant auxiliary space.