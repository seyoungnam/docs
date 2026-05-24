# 543. Diameter of Binary Tree

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/diameter-of-binary-tree/)

## Solution: Depth-First Search

The diameter of a binary tree is the longest path between any two nodes. We can find this by calculating the maximum height of the left and right subtrees for every node, where the diameter passing through any node is the sum of its left and right heights.

### Thought Process

1.  **Calculate Subtree Heights**: Define a DFS helper that returns the height of a subtree (number of nodes on the longest path, where `nil` returns `0`).
2.  **Determine Diameter**: At each node, the longest path passing through it is `leftHeight + rightHeight`. Update the global maximum `res` with this value.
3.  **Return Height**: Return the height of the current node to its parent: `1 + max(leftHeight, rightHeight)`.

### Go Code

``` go
func diameterOfBinaryTree(root *TreeNode) int {
    var res int
    var dfs func(node *TreeNode) int
    dfs = func(node *TreeNode) int {
        if node == nil {
            return 0
        }
        left := dfs(node.Left)
        right := dfs(node.Right)
        
        // Update diameter (left nodes + right nodes)
        res = max(res, left+right)
        
        return 1 + max(left, right)
    }
    dfs(root)
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We visit each node in the tree exactly once.
- **Space Complexity**: $O(h)$
    - Auxiliary space is determined by the recursion call stack, where $h$ is the height of the tree (up to $O(n)$ for a skewed tree).
