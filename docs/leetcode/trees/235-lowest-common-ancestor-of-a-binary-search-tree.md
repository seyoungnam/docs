# 235. Lowest Common Ancestor of a Binary Search Tree

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/description/)

Given a binary search tree (BST), find the lowest common ancestor (LCA) node of two given nodes in the BST.

According to the definition of LCA on Wikipedia: “The lowest common ancestor is defined between two nodes `p` and `q` as the lowest node in `T` that has both `p` and `q` as descendants (where we allow **a node to be a descendant of itself**).”

---

## Solution 1: Recursive BST Traversal

By utilizing the properties of a **Binary Search Tree (BST)**, we can determine the location of `p` and `q` relative to the current node:
*   For any node, all left descendants have values strictly smaller than the node, and all right descendants have values strictly larger.
*   The **Lowest Common Ancestor (LCA)** will be the **split node** where `p` and `q` diverge into different subtrees, or where one of them matches the current node.

### Thought Process

1.  **Search State**:
    *   Compare the values of `p` and `q` against the current `root.Val`.
2.  **Decisions**:
    *   **Branch Left**: If both `p.Val` and `q.Val` are less than `root.Val` (which can be checked as `max(p.Val, q.Val) < root.Val`), then both nodes reside in the left subtree. Recurse left:
        $$\text{LCA} = \text{lowestCommonAncestor(root.Left, p, q)}$$
    *   **Branch Right**: If both `p.Val` and `q.Val` are greater than `root.Val` (which can be checked as `min(p.Val, q.Val) > root.Val`), then both nodes reside in the right subtree. Recurse right:
        $$\text{LCA} = \text{lowestCommonAncestor(root.Right, p, q)}$$
    *   **Split Point**: If neither condition is met, one node is in the left subtree, and the other is in the right subtree (or one of them is the current `root`). The current `root` is the split point and therefore the LCA. Return `root`.

### Go Code

``` go
func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
    if root == nil || p == nil || q == nil {
        return nil
    }
    if max(p.Val, q.Val) < root.Val {
        return lowestCommonAncestor(root.Left, p, q)
    } else if min(p.Val, q.Val) > root.Val {
        return lowestCommonAncestor(root.Right, p, q)
    } else {
        return root
    }
}
```

### Code Efficiency

- **Time Complexity**: $O(H)$
    - Where $H$ is the height of the BST. We traverse one path from the root down to the split node, visiting at most $H$ nodes. In a balanced tree, $H = O(\log N)$; in a skewed tree, $H = O(N)$.
- **Space Complexity**: $O(H)$
    - Corresponding to the recursion stack depth. In the worst case, $O(N)$ space is required.

---

## Solution 2: Iterative BST Traversal (Space Optimized)

We can optimize the space complexity to $O(1)$ by converting the recursion into an iterative loop, eliminating the call stack overhead.

### Thought Process

1.  **Pointer Initialization**:
    *   Initialize `curr := root`.
2.  **Iterative Loop**:
    *   While `curr` is not `nil`:
        *   If both `p.Val` and `q.Val` are greater than `curr.Val`, move the pointer to the right child: `curr = curr.Right`.
        *   If both `p.Val` and `q.Val` are less than `curr.Val`, move the pointer to the left child: `curr = curr.Left`.
        *   Otherwise, we have found the split node. Return `curr`.
3.  **Result**:
    *   Returns `curr` directly as the LCA.

### Go Code

``` go
func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
    curr := root
    for curr != nil {
        if p.Val > curr.Val && q.Val > curr.Val {
            curr = curr.Right
        } else if p.Val < curr.Val && q.Val < curr.Val {
            curr = curr.Left
        } else {
            return curr
        }
    }
    return nil
}
```

### Code Efficiency

- **Time Complexity**: $O(H)$
    - We traverse a single path from the root to the LCA node, taking at most $O(H)$ iterations.
- **Space Complexity**: $O(1)$
    - We only maintain a single pointer `curr` to traverse the tree, requiring constant auxiliary space.