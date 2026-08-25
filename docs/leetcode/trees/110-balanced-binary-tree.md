# 110. Balanced Binary Tree

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/balanced-binary-tree/description/)

Given a binary tree, determine if it is **height-balanced**.

A height-balanced binary tree is defined as:
> A binary tree in which the left and right subtrees of every node differ in height by no more than 1.

---

## Solution 1: Depth-First Search with Global Flag

We can perform a bottom-up post-order traversal to calculate the height of each subtree, updating a global boolean flag if any node violates the balance condition.

### Thought Process

1.  **Global Flag**:
    *   Maintain a boolean variable `res := true` to track the overall balance state.
2.  **DFS Helper**:
    *   Define `dfs(node)` which returns the height of `node` (where the height of `nil` is `-1`).
    *   Recursively calculate the left subtree height: `l := 1 + dfs(node.Left)`.
    *   Recursively calculate the right subtree height: `r := 1 + dfs(node.Right)`.
    *   **Balance Check**: If the absolute difference $|l - r| > 1$, an imbalance is detected; set `res = false`.
    *   Return the current node's height: `max(l, r)`.
3.  **Custom Abs Helper**:
    *   Since Go does not provide a built-in integer absolute value function in the standard library, we implement a simple `abs` helper.

### Go Code

``` go
func isBalanced(root *TreeNode) bool {
    res := true
    var dfs func(node *TreeNode) int
    dfs = func(node *TreeNode) int {
        if node == nil {
            return -1
        }
        l := 1 + dfs(node.Left)
        r := 1 + dfs(node.Right)
        if abs(l-r) > 1 {
            res = false
        }
        return max(l, r)
    }
    dfs(root)
    return res
}

func abs(a int) int {
    if a < 0 {
        return -a
    }
    return a
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Where $N$ is the number of nodes in the binary tree. We visit each node exactly once.
- **Space Complexity**: $O(H)$
    - Where $H$ is the height of the binary tree, corresponding to the recursion call stack depth. In the worst case of a completely skewed tree, $H = O(N)$. In the best case of a balanced binary tree, $H = O(\log N)$.

---

## Solution 2: Fail-Fast Depth-First Search (Bottom-Up Height Check)

We can optimize Solution 1 by immediately propagating an imbalance up the recursion stack, avoiding unnecessary traversal of the rest of the tree once an imbalance is found.

### Thought Process

1.  **Imbalance Flag Value**:
    *   Define a helper function `checkHeight(node)` that returns the height of the subtree if it is balanced, or `-1` if it is unbalanced.
2.  **Early Termination**:
    *   If `checkHeight(node.Left)` returns `-1`, we immediately return `-1` to the parent caller, bypassing the right subtree traversal entirely.
    *   Do the same if the right subtree returns `-1`.
3.  **Balance Check**:
    *   If both subtrees are balanced, check the height difference. If $|left - right| > 1$, return `-1`.
    *   Otherwise, return the height of the current node: `max(left, right) + 1`.

### Go Code

``` go
func isBalanced(root *TreeNode) bool {
    return checkHeight(root) != -1
}

func checkHeight(node *TreeNode) int {
    if node == nil {
        return 0
    }
    
    left := checkHeight(node.Left)
    if left == -1 {
        return -1 // Left subtree is unbalanced; fail-fast
    }
    
    right := checkHeight(node.Right)
    if right == -1 {
        return -1 // Right subtree is unbalanced; fail-fast
    }
    
    if abs(left-right) > 1 {
        return -1 // Current node is unbalanced
    }
    
    return max(left, right) + 1
}

func abs(a int) int {
    if a < 0 {
        return -a
    }
    return a
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - In the worst case (when the tree is balanced), we visit every node once. However, in unbalanced trees, this approach terminates much faster on average due to the fail-fast optimization.
- **Space Complexity**: $O(H)$
    - Requires $O(H)$ space for the recursion stack, where $H$ is the height of the tree.