# 223. Rectangle Area

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/rectangle-area/description/)

## Solution: Inclusion-Exclusion Principle

To calculate the total area covered by two rectilinear rectangles, we can apply the **Inclusion-Exclusion Principle**:
$$\text{Total Area} = \text{Area}(A) + \text{Area}(B) - \text{Area}(A \cap B)$$
Where $\text{Area}(A \cap B)$ is the overlapping area of the two rectangles. If they do not overlap, this value is $0$.

### Thought Process

1.  **Individual Area Calculations**:
    - The area of Rectangle A is: $(ax_2 - ax_1) \times (ay_2 - ay_1)$.
    - The area of Rectangle B is: $(bx_2 - bx_1) \times (by_2 - by_1)$.
2.  **Overlap Boundaries**:
    - Determine the boundaries of the intersection:
        - $\text{left} = \max(ax_1, bx_1)$
        - $\text{right} = \min(ax_2, bx_2)$
        - $\text{down} = \max(ay_1, by_1)$
        - $\text{up} = \min(ay_2, by_2)$
3.  **Overlap Area**:
    - The width of the intersection is $\text{right} - \text{left}$ and the height is $\text{up} - \text{down}$.
    - If the width or height is less than or equal to $0$, the rectangles do not overlap ($\text{Overlap Area} = 0$).
    - Otherwise, $\text{Overlap Area} = (\text{right} - \text{left}) \times (\text{up} - \text{down})$.
4.  **Combine**:
    - The total area is $\text{Area}(A) + \text{Area}(B) - \text{Overlap Area}$.

### Go Code

``` go
func computeArea(ax1 int, ay1 int, ax2 int, ay2 int, bx1 int, by1 int, bx2 int, by2 int) int {
    left, right := max(ax1, bx1), min(ax2, bx2)
    up, down    := min(ay2, by2), max(ay1, by1)
    both := (right - left) * (up - down)
    if right - left <= 0 || up - down <= 0 {
        both = 0
    }
    return (ax2 - ax1)*(ay2 - ay1) + (bx2 - bx1)*(by2 - by1) - both
}
```

### Code Efficiency

- **Time Complexity**: $O(1)$
    - The coordinates are compared and arithmetic calculations are computed in constant time.
- **Space Complexity**: $O(1)$
    - We only allocate a few primitive integer variables, using constant auxiliary space.