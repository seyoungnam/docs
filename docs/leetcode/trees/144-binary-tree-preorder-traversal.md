# 144. Binary Tree Preorder Traversal

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/binary-tree-preorder-traversal/description/)

Preorder traversal visits binary tree nodes in the following order: **Root $\rightarrow$ Left $\rightarrow$ Right**.

---

## Solution 1: Recursive DFS

This approach implements the standard recursive traversal using the system's call stack.

### Thought Process

1.  **Traversal Order**: Process the current node's value first, then recursively traverse its left subtree, followed by its right subtree.
2.  **Base Case**: If the current node is `nil`, return immediately.

### Go Code

``` go
func preorderTraversal(root *TreeNode) []int {
    res := make([]int, 0)
    
    var dfs func(*TreeNode)
    dfs = func(node *TreeNode) {
        if node == nil {
            return
        }
        res = append(res, node.Val)
        dfs(node.Left)
        dfs(node.Right)
    }
    dfs(root)
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - We visit every node in the binary tree exactly once.
- **Space Complexity**: $O(H)$
    - The space is determined by the recursion call stack, which is equal to the height $H$ of the tree. In the worst case (skewed tree), this is $O(N)$; in the best case (balanced tree), this is $O(\log N)$.

---

## Solution 2: Iterative DFS (Stack Simulation)

This approach simulates the recursive call stack manually using an explicit slice.

### Thought Process

1.  **Simulating Recursion**:
    *   Maintain a pointer `curr` initialized to the root, and a stack slice.
2.  **Iterative Traversal**:
    *   While `curr != nil` or the stack contains nodes:
        *   Keep traveling down the left subtree:
            *   **Visit**: Append the current node's value (`curr.Val`) to the result slice (since root is visited first in preorder).
            *   **Push**: Push the current node onto the stack.
            *   **Move Left**: Update `curr = curr.Left`.
        *   Once we hit a `nil` leaf:
            *   **Pop**: Pop the last visited parent node from the stack.
            *   **Move Right**: Transition to its right child: `curr = curr.Right`.

### Go Code

``` go
func preorderTraversal(root *TreeNode) []int {
    res := make([]int, 0)
    stack := make([]*TreeNode, 0)
    curr := root
    for curr != nil || len(stack) > 0 {
        for curr != nil {
            res = append(res, curr.Val)
            stack = append(stack, curr)
            curr = curr.Left
        }
        curr = stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        curr = curr.Right
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Each node is pushed onto and popped from the stack at most once, resulting in linear execution time.
- **Space Complexity**: $O(H)$
    - The stack stores at most the path from the root to a leaf node, requiring space proportional to the height $H$ of the tree.