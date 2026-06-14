# 1041. Robot Bounded In Circle

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/robot-bounded-in-circle/description/)

## Solution: Simulation (Vector Rotation)


### Thought Process

1.  **Analyze the Movement**:
    *   The robot starts at `(0, 0)` facing North.
    *   We can represent the four directions in clockwise order: `North (0)`, `East (1)`, `South (2)`, `West (3)`.
    *   Left turns (`L`) decrement direction (equivalent to `+3 % 4`).
    *   Right turns (`R`) increment direction (equivalent to `+1 % 4`).
    *   Step moves (`G`) increment coordinates based on current direction vectors:
        *   `dx = [0, 1, 0, -1]`
        *   `dy = [1, 0, -1, 0]`

2.  **Determine Boundedness (Mathematical Proof)**:
    *   After running the instructions once, let the final coordinate be `(x, y)` and the direction be `dir`.
    *   **Case 1**: The robot returns to the origin `(x == 0 && y == 0)`. It will repeat the same path in every loop, remaining bounded.
    *   **Case 2**: The robot is NOT at the origin, but is **not facing North** (`dir != 0`). 
        *   If it faces East or West (90-degree turn), after 4 cycles of the instructions, it will have turned $360^\circ$ and returned to the origin `(0, 0)`.
        *   If it faces South (180-degree turn), after 2 cycles of the instructions, it will have turned $360^\circ$ and returned to the origin `(0, 0)`.
        *   Thus, it is guaranteed to form a closed circle/loop and remain bounded.
    *   **Case 3**: The robot is NOT at the origin and **still faces North** (`dir == 0`). In each cycle, it will move by the same displacement vector `(x, y)`, escaping to infinity (unbounded).

3.  **Result**: The robot is bounded if it returns to the origin `x == 0 && y == 0` **OR** if its final direction is not North (`dir != 0`).

### Go Code

``` go
func isRobotBounded(instructions string) bool {
  dx := []int{0, 1, 0, -1}
	dy := []int{1, 0, -1, 0}

	x, y := 0, 0
	dir := 0 

	for i := 0; i < len(instructions); i++ {
		r := instructions[i]
		switch r {
		case 'G':
			x += dx[dir]
			y += dy[dir]
		case 'L':
			dir = (dir + 3) % 4 
		case 'R':
			dir = (dir + 1) % 4
		}
	}
	return (x == 0 && y == 0) || dir != 0
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We loop through the list of instructions of length $n$ exactly once.
- **Space Complexity**: $O(1)$
    - We only store a few primitive variables (`x`, `y`, `dir`) for state tracking.
