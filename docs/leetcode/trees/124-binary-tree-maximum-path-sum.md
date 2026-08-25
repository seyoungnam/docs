# 124. Binary Tree Maximum Path Sum

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/binary-tree-maximum-path-sum/description/)

A **path** in a binary tree is a sequence of nodes where each pair of adjacent nodes in the sequence has an edge connecting them. A node can only appear in the sequence **at most once**. Note that the path does not need to pass through the root.

The **path sum** of a path is the sum of the node's values in the path.

Given the `root` of a binary tree, return *the maximum path sum of any non-empty path*.

---

## Solution: Recursive DFS (Bottom-Up Path Calculation)

We can solve this problem using a post-order traversal (DFS). For each node, we calculate the maximum path sum that can go through it, while returning the maximum single-branch path sum to its parent.

### Thought Process

1.  **Define Subproblem**:
    *   For any given node, we want to compute the **maximum single-branch path sum** extending down from it. This value is returned to the parent node.
2.  **Pruning Negative Contributions**:
    *   If a subtree contributes a negative path sum, it is always better to ignore it (representing a path sum contribution of `0`).
    *   Compute the left branch contribution: `l := max(dfs(node.Left), 0)`.
    *   Compute the right branch contribution: `r := max(dfs(node.Right), 0)`.
3.  **Update Global Maximum**:
    *   At the current node, a complete path can be formed spanning from the left child, through the current node, and down into the right child.
    *   The sum of this path is:
        $$\text{currentPathSum} = \text{node.Val} + l + r$$
    *   Update our global tracker `maxSum` with this value.
4.  **Return Value**:
    *   A parent node can only connect to the current node and extend down into **one** of its children. Therefore, the function returns:
        $$\text{returnVal} = \text{node.Val} + \max(l, r)$$
5.  **Initialization**:
    *   Initialize `maxSum := math.MinInt32`. Start the traversal by calling `dfs(root)`.

### Go Code

``` go
import "math"

func maxPathSum(root *TreeNode) int {
    maxSum := math.MinInt32
    var dfs func(node *TreeNode) int
    dfs = func(node *TreeNode) int {
        if node == nil {
            return 0
        }
        
        // Compute maximum path sum from left and right children; prune if negative
        l := max(dfs(node.Left), 0)
        r := max(dfs(node.Right), 0)
        
        // Update global maximum path sum (spanning left, current, and right)
        maxSum = max(maxSum, node.Val + l + r)

        // Return the maximum single-branch path sum to the parent node
        return node.Val + max(l, r)
    }
    dfs(root)
    return maxSum
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Where $N$ is the number of nodes in the binary tree. We visit and compute the path sums for each node exactly once.
- **Space Complexity**: $O(H)$
    - Where $H$ is the height of the binary tree, corresponding to the recursion call stack depth. In the worst case of a completely skewed tree, $H = O(N)$. In the best case of a balanced binary tree, $H = O(\log N)$.