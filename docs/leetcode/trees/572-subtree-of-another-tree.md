# 572. Subtree of Another Tree

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/subtree-of-another-tree/description/)

Given the roots of two binary trees `root` and `subRoot`, return `true` if there is a subtree of `root` with the same structure and node values of `subRoot` and `false` otherwise.

A subtree of a binary tree `tree` is a tree that consists of a node in `tree` and all of this node's descendants. The tree `tree` could also be considered as a subtree of itself.

---

## Solution: Recursive DFS (Same Tree Check)

We can solve this problem recursively by traversing the main tree and checking if the subtree rooted at any node is identical to the target `subRoot`.

### Thought Process

1.  **Base Cases**:
    *   If `subRoot` is `nil`, it is a subtree of any tree (even an empty one); return `true`.
    *   If `root` is `nil` (and `subRoot` is not `nil`), it is impossible for `subRoot` to be a subtree; return `false`.
2.  **Identical Check**:
    *   Compare the tree rooted at the current `root` node with `subRoot` using a helper function `isSameTree(root, subRoot)`. If they match, return `true`.
3.  **Recursive Subtree Search**:
    *   If they do not match, check if `subRoot` is a subtree of the left child or the right child of `root`:
        $$\text{result} = \text{isSubtree(root.Left, subRoot)} \lor \text{isSubtree(root.Right, subRoot)}$$
4.  **Same Tree Helper (`isSameTree`)**:
    *   Two trees are identical if:
        *   Both are `nil`.
        *   Their current values are equal (`p.Val == q.Val`).
        *   Their left subtrees are identical.
        *   Their right subtrees are identical.

### Go Code

``` go
func isSubtree(root *TreeNode, subRoot *TreeNode) bool {
    if subRoot == nil {
        return true
    }
    if root == nil {
        return false
    }
    if isSameTree(root, subRoot) {
        return true
    }
    return isSubtree(root.Left, subRoot) || isSubtree(root.Right, subRoot)
}

func isSameTree(p *TreeNode, q *TreeNode) bool {
    if p == nil && q == nil {
        return true
    }
    if p == nil || q == nil {
        return false
    }
    return p.Val == q.Val && isSameTree(p.Left, q.Left) && isSameTree(p.Right, q.Right)
}
```

### Code Efficiency

- **Time Complexity**: $O(N \cdot M)$
    - Where $N$ is the number of nodes in the `root` tree, and $M$ is the number of nodes in the `subRoot` tree. In the worst case (e.g., all nodes have the same value), we perform a full comparison of size $O(M)$ at each of the $N$ nodes.
- **Space Complexity**: $O(H_{root})$
    - Where $H_{root}$ is the height of the `root` tree, representing the recursion stack depth. In the worst case of a skewed tree, this is $O(N)$.