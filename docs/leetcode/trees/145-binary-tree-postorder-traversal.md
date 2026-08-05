# 145. Binary Tree Postorder Traversal

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/binary-tree-postorder-traversal/description/)

Postorder traversal visits binary tree nodes in the following order: **Left $\rightarrow$ Right $\rightarrow$ Root**.

---

## Solution 1: Recursive DFS

This approach implements the standard recursive traversal using the system's call stack.

### Thought Process

1.  **Traversal Order**: Process the left subtree recursively, then the right subtree recursively, and finally append the current node's value.
2.  **Base Case**: If the current node is `nil`, return immediately.

### Go Code

``` go
func postorderTraversal(root *TreeNode) []int {
    res := make([]int, 0)

    var dfs func(*TreeNode)
    dfs = func(node *TreeNode) {
        if node == nil {
            return
        }
        dfs(node.Left)
        dfs(node.Right)
        res = append(res, node.Val)
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

## Solution 2: Iterative DFS (Stack with Visit Flag)

To perform postorder traversal iteratively, we must visit the parent node *after* both its left and right subtrees have been processed. We can achieve this by storing nodes in a stack with a boolean flag tracking whether the node's children have been pushed.

### Thought Process

1.  **Node State Wrapping**:
    *   Create a helper struct `stackItem` containing `node *TreeNode` and a boolean `visit`.
2.  **First vs. Second Encounter**:
    *   When we pop an item from the stack:
        *   **If `visit == true`**: This is our second encounter with the node (meaning its left and right subtrees have already been processed). We visit it: append `node.Val` to the result.
        *   **If `visit == false`**: This is our first encounter. We must push the node back with `visit = true` and push its children.
3.  **Reverse Order Push**:
    *   Because a stack is First-In-Last-Out (FILO) and we want a traversal order of Left $\rightarrow$ Right $\rightarrow$ Root, we push the elements onto the stack in the **reverse** order:
        1.  Push the parent node: `stackItem{node, true}`
        2.  Push the right child: `stackItem{node.Right, false}`
        3.  Push the left child: `stackItem{node.Left, false}`
    *   This guarantees that the left child is popped and processed first, then the right child, and finally the parent node.

### Go Code

``` go
type stackItem struct {
    node    *TreeNode
    visit   bool
}

func postorderTraversal(root *TreeNode) []int {
    res := make([]int, 0)
    stack := []stackItem{{root, false}}

    for len(stack) > 0 {
        item := stack[len(stack)-1]
        stack = stack[:len(stack)-1]

        if item.node != nil {
            if item.visit {
                res = append(res, item.node.Val)
            } else {
                stack = append(stack, stackItem{item.node, true})
                stack = append(stack, stackItem{item.node.Right, false})
                stack = append(stack, stackItem{item.node.Left, false})
            }
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Each node is pushed onto and popped from the stack at most twice, resulting in linear execution time.
- **Space Complexity**: $O(N)$
    - In the worst case, the stack can store up to $O(N)$ elements representing the nodes and their visit states.