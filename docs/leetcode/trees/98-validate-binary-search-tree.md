# 98. Validate Binary Search Tree

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/validate-binary-search-tree/description/)

Given the `root` of a binary tree, determine if it is a valid binary search tree (BST).

A **valid BST** is defined as follows:
*   The left subtree of a node contains only nodes with keys **less than** the node's key.
*   The right subtree of a node contains only nodes with keys **greater than** the node's key.
*   Both the left and right subtrees must also be binary search trees.

---

## Solution 1: Recursive DFS with Range Bounds

A common mistake is checking only if a parent's value is greater than its left child and less than its right child. Instead, the constraint must apply to all descendants. We can enforce this by propagating a valid open interval range `(left, right)` down the recursive path.

### Thought Process

1.  **DFS Helper**:
    *   Define a helper function `dfs(node, left, right)` that returns `true` if all nodes in the subtree are strictly within the open interval `(left, right)`.
2.  **Base Case**:
    *   An empty node (`node == nil`) is a valid BST; return `true`.
3.  **Boundary Validation**:
    *   If the current node's value is outside the valid range:
        $$\text{node.Val} \le \text{left} \quad \text{or} \quad \text{node.Val} \ge \text{right}$$
        return `false`.
4.  **Subtree Recursion**:
    *   For the left child, the upper boundary becomes the current node's value: `dfs(node.Left, left, node.Val)`.
    *   For the right child, the lower boundary becomes the current node's value: `dfs(node.Right, node.Val, right)`.
5.  **Initialization**:
    *   Start the traversal from the root with the range boundaries set to the minimum and maximum possible integer values: `math.MinInt64` and `math.MaxInt64`.

### Go Code

``` go
import "math"

func isValidBST(root *TreeNode) bool {
    var dfs func(node *TreeNode, left int, right int) bool
    dfs = func(node *TreeNode, left int, right int) bool {
        if node == nil {
            return true
        }
        if node.Val <= left || node.Val >= right {
            return false
        }
        return dfs(node.Left, left, node.Val) && dfs(node.Right, node.Val, right)
    }
    return dfs(root, math.MinInt64, math.MaxInt64)
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Where $N$ is the number of nodes in the binary tree. We visit and check each node exactly once.
- **Space Complexity**: $O(H)$
    - Where $H$ is the height of the binary tree, corresponding to the recursion call stack depth. In the worst case of a completely skewed tree, $H = O(N)$. In the best case of a balanced binary tree, $H = O(\log N)$.

---

## Solution 2: Inorder Traversal (Strictly Increasing)

An **inorder traversal** (Left $\rightarrow$ Root $\rightarrow$ Right) of a valid BST must produce nodes in strictly sorted, ascending order. We can validate the BST by performing an inorder traversal and verifying that each visited node has a value strictly greater than the previously visited node.

### Thought Process

1.  **State Tracking**:
    *   Maintain a pointer `prev *TreeNode` to track the node visited immediately before the current node.
2.  **Inorder Steps**:
    *   If `node` is `nil`, return `true`.
    *   Recursively validate the left subtree: `if !inorder(node.Left) { return false }`.
    *   Validate the current node: If `prev != nil` and `node.Val <= prev.Val`, the ascending order constraint is violated; return `false`.
    *   Update the previously visited node: `prev = node`.
    *   Recursively validate the right subtree: `return inorder(node.Right)`.

### Go Code

``` go
func isValidBST(root *TreeNode) bool {
    var prev *TreeNode
    var inorder func(node *TreeNode) bool
    inorder = func(node *TreeNode) bool {
        if node == nil {
            return true
        }
        
        // Recurse left
        if !inorder(node.Left) {
            return false
        }
        
        // Evaluate current node
        if prev != nil && node.Val <= prev.Val {
            return false
        }
        prev = node
        
        // Recurse right
        return inorder(node.Right)
    }
    return inorder(root)
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - We visit each node in the tree exactly once.
- **Space Complexity**: $O(H)$
    - Requires $O(H)$ space for the recursion stack, where $H$ is the height of the tree.