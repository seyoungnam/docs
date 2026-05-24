# 236. Lowest Common Ancestor of a Binary Tree

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/description/)

## Solution: DFS with State Tracking

We can perform a post-order traversal (DFS) where each node returns a boolean indicating whether it contains either `p` or `q` in its subtree (including itself). The Lowest Common Ancestor (LCA) is the unique node where the target nodes are first split or joined.

### Thought Process

1.  **Boolean State**: For each node, determine if either `p` or `q` is found in:
    *   The left subtree: `left := postOrder(node.Left)`
    *   The right subtree: `right := postOrder(node.Right)`
    *   The current node itself: `curr := (node == p) || (node == q)`
2.  **LCA Condition**: The LCA `res` is found when any two of these three flags (`left`, `right`, `curr`) are `true`:
    *   `curr && (left || right)` (the current node is one target, and the other target is in a subtree)
    *   `left && right` (one target is in the left subtree, and the other is in the right subtree)
3.  **No Reassignment**: Once `res` is assigned, parent nodes up the tree will only receive a `true` flag from *one* subtree (while the other subtree and `curr` remain `false`). Thus, the LCA condition evaluates to `false` for all higher ancestors, preventing `res` from ever being reassigned.

### Go Code

``` go
func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
    var res *TreeNode
    var postOrder func(node *TreeNode) bool
    postOrder = func(node *TreeNode) bool {
        if node == nil {
            return false
        }
        left := postOrder(node.Left)
        right := postOrder(node.Right)
        curr := (node == p) || (node == q)
        
        // LCA is the node where p and q are first united/split
        if (curr && (left || right)) || (left && right) {
            res = node
        }
        return curr || left || right
    }
    postOrder(root)
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We visit every node in the binary tree at most once.
- **Space Complexity**: $O(h)$
    - Auxiliary space is determined by the recursion call stack, where $h$ is the height of the tree (up to $O(n)$ for a skewed tree).

---

## Alternative Solution: Direct Recursive Propagation

Instead of using a helper function and a separate tracking variable, we can write a direct recursive function that returns the LCA node (or the target node if found, or `nil` if neither is found).

### Thought Process

1.  **Base Cases**: If the current node is `nil`, or if it matches `p` or `q`, return the current node directly.
2.  **Recurse**: Search for `p` and `q` in the left and right subtrees.
3.  **Union / LCA**:
    *   If both `left` and `right` searches return non-nil nodes, the current node is the LCA, so we return it.
    *   If only one search returns non-nil, propagate that non-nil result upwards (since both targets must be under that single branch).
    *   If both return `nil`, return `nil`.

### Go Code

``` go
func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
    if root == nil || root == p || root == q {
        return root
    }
    
    left := lowestCommonAncestor(root.Left, p, q)
    right := lowestCommonAncestor(root.Right, p, q)
    
    // If targets are found in both left and right subtrees, root is the LCA
    if left != nil && right != nil {
        return root
    }
    
    // Otherwise, return the non-nil child
    if left != nil {
        return left
    }
    return right
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We traverse the tree, visiting each node at most once.
- **Space Complexity**: $O(h)$
    - The recursion stack takes $O(h)$ space where $h$ is the tree's height.
