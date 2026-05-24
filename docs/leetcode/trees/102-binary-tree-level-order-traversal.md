# 102. Binary Tree Level Order Traversal

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/binary-tree-level-order-traversal/description/)

## Solution: Breadth-First Search (BFS)

We can traverse the tree level by level using a queue. By recording the queue size at the start of each level, we can process all nodes belonging to the current level before moving to the next one.

### Thought Process

1.  **Queue Initialization**: Start with a queue containing the `root` node.
2.  **Level Tracking**: Loop while the queue is not empty. At the beginning of each iteration, record the current length of the queue (`curLen`), which represents the number of nodes at this level.
3.  **Process Level**: Run a inner loop `curLen` times. For each node:
    *   Pop it from the front of the queue.
    *   Append its value to the current level's slice.
    *   Push its non-nil left and right children to the queue.
4.  **Append Result**: Add the level's slice to the final results slice.

### Go Code

``` go
func levelOrder(root *TreeNode) [][]int {
    if root == nil {
        return nil
    }
    var res [][]int
    var queue []*TreeNode
    queue = append(queue, root)
    for len(queue) > 0 {
        var sub []int
        curLen := len(queue)
        for i := 0; i < curLen; i++ {
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
        res = append(res, sub)
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We visit every node in the binary tree exactly once.
- **Space Complexity**: $O(n)$
    - The queue stores up to $O(n)$ nodes at the maximum level width. In a perfect binary tree, the last level has $\approx n/2$ nodes.

---

## Alternative Solution: Depth-First Search (DFS)

We can also solve this problem using DFS (Preorder traversal) by keeping track of the current `level` of recursion and appending elements to the corresponding sub-slice in our result.

### Thought Process

1.  **Level Parameter**: Define a recursive DFS helper that accepts a node and its current `level` (starting at `0`).
2.  **Base Case**: If the current node is `nil`, return.
3.  **Initialize Level Slice**: If the `level` matches the length of the result slice `res` (`level == len(res)`), it means we are visiting the first node of a new level. Append a new empty slice to `res`.
4.  **Collect Node**: Append the current node's value to the sub-slice at `res[level]`.
5.  **Recurse**: Recursively traverse the left child with `level + 1`, and then the right child with `level + 1`.

### Go Code

``` go
func levelOrder(root *TreeNode) [][]int {
    var res [][]int
    var dfs func(node *TreeNode, level int)
    dfs = func(node *TreeNode, level int) {
        if node == nil {
            return
        }
        
        // Visiting the first node of this level
        if level == len(res) {
            res = append(res, []int{})
        }
        
        res[level] = append(res[level], node.Val)
        
        dfs(node.Left, level+1)
        dfs(node.Right, level+1)
    }
    
    dfs(root, 0)
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We visit each node in the tree exactly once.
- **Space Complexity**: $O(h)$
    - Auxiliary space is determined by the recursion call stack, which is $O(h)$ where $h$ is the height of the tree (up to $O(n)$ in the worst case of a skewed tree).
