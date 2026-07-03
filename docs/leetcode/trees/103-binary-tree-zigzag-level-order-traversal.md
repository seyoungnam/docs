# 103. Binary Tree Zigzag Level Order Traversal

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/description/)

## Solution: Breadth-First Search (BFS) with Conditional Reversal

This problem is a variation of the standard Binary Tree Level Order Traversal. Instead of traversing every level from left to right, we alternate directions level-by-level. 

Rather than changing the traversal order in the queue itself (which can complicate child node insertions), we perform a standard level-by-level **BFS** to collect the node values, and then reverse the level array if the current level index dictates a right-to-left traversal.

### Thought Process

1.  **Iterative Level Order Traversal (BFS)**:
    *   Initialize a queue containing the root node: `queue := []*TreeNode{root}`.
    *   Iterate while the queue has elements.
2.  **Level Processing**:
    *   Before processing the level, record the current queue size `length = len(queue)`.
    *   Loop `length` times (using Go's integer range loop `for range length`) to process exactly the nodes belonging to the current level:
        *   Pop the node `curr` from the front of the queue.
        *   Append `curr.Val` to the level slice `sub`.
        *   Enqueue non-nil left and right children.
3.  **Zigzag Direction Toggle**:
    *   Since `res` is 0-indexed:
        *   Level 0 (1st level) is left-to-right.
        *   Level 1 (2nd level) is right-to-left $\rightarrow$ reverse `sub`.
        *   Level 2 (3rd level) is left-to-right.
    *   Before appending `sub` to `res`, we check if `len(res) % 2 == 1`. If `true`, we call the in-place helper function `reversed(sub)` to reverse the slice values.
4.  **Reversal Helper (`reversed`)**:
    *   Uses a classic two-pointer approach (`l` and `r`) starting from both ends of the slice and swapping elements moving inwards.

### Go Code

``` go
func zigzagLevelOrder(root *TreeNode) [][]int {
    if root == nil {
        return nil
    }
    res := make([][]int, 0)
    queue := []*TreeNode{root}
    for len(queue) > 0 {
        length := len(queue)
        sub := make([]int, 0)
        for range length {
            curr := queue[0]
            queue = queue[1:]

            sub = append(sub, curr.Val)
            if curr.Left != nil {
                queue = append(queue, curr.Left)
            }
            if curr.Right != nil {
                queue = append(queue, curr.Right)
            }
        }
        if len(res)%2 == 1 {
            reversed(sub)
        }
        res = append(res, sub)
    }
    return res
}

func reversed(arr []int) {
    for l, r := 0, len(arr)-1; l < r; l, r = l+1, r-1 {
        arr[l], arr[r] = arr[r], arr[l]
    }
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - We visit every node in the binary tree exactly once. Reversing lists level-by-level takes time proportional to the number of nodes at each level. The sum of the number of nodes across all levels is $N$, so the total time complexity is $O(N)$.
- **Space Complexity**: $O(N)$
    - In the worst case (a fully balanced tree), the queue holds up to $O(N)$ leaf nodes at the bottom level. The output array `res` also takes $O(N)$ space.