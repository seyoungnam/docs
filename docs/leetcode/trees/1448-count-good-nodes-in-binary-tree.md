# 1448. Count Good Nodes in Binary Tree

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/count-good-nodes-in-binary-tree/description/)

Given a binary tree `root`, a node *X* in the tree is named **good** if in the path from root to *X* there are no nodes with a value *greater than* *X*.

Return the number of **good** nodes in the binary tree.

---

## Solution 1: Recursive DFS with Closure Counter

We can traverse the tree using Depth-First Search (DFS) while carrying the maximum value seen so far on the current path from the root.

### Thought Process

1.  **Path Maximum (`maxVal`)**:
    *   As we traverse from the root to any node, the node is "good" if its value is greater than or equal to the maximum value found along the path from the root to its parent.
2.  **State Tracking**:
    *   Initialize a counter `res := 0`.
    *   Define a recursive helper `dfs(node, maxVal)`.
3.  **DFS Steps**:
    *   If `node` is `nil`, return.
    *   If `node.Val >= maxVal`, increment the good node counter: `res++`.
    *   Recurse on the left child with the updated path maximum: `max(maxVal, node.Val)`.
    *   Recurse on the right child with the updated path maximum: `max(maxVal, node.Val)`.
4.  **Start Traversal**:
    *   Invoke `dfs(root, root.Val)`. The root itself is always a good node.

### Go Code

``` go
func goodNodes(root *TreeNode) int {
    res := 0
    var dfs func(node *TreeNode, maxVal int)
    dfs = func(node *TreeNode, maxVal int) {
        if node == nil {
            return
        }
        if node.Val >= maxVal {
            res++
        }
        dfs(node.Left, max(maxVal, node.Val))
        dfs(node.Right, max(maxVal, node.Val))
    }
    dfs(root, root.Val)
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Where $N$ is the number of nodes in the binary tree. We visit and check each node exactly once.
- **Space Complexity**: $O(H)$
    - Where $H$ is the height of the binary tree, corresponding to the recursion call stack depth. In the worst case of a completely skewed tree, $H = O(N)$. In the best case of a balanced binary tree, $H = O(\log N)$.

---

## Solution 2: Pure Functional Recursive DFS

Instead of using a closure with a shared state variable, we can write a pure recursive function where each node returns the count of good nodes within its subtree.

### Thought Process

1.  **Recursive Subproblem**:
    *   The total number of good nodes in a subtree rooted at `node` is:
        $$\text{Total} = \text{IsGood}(node) + \text{GoodNodes}(node.Left) + \text{GoodNodes}(node.Right)$$
2.  **DFS Helper**:
    *   Define `dfs(node, maxVal) int` which returns the count of good nodes in the subtree.
    *   Base Case: If `node` is `nil`, return `0`.
    *   Check Current: If `node.Val >= maxVal`, the current node is good, contributing `1`. Otherwise, it contributes `0`.
    *   Return the sum of the current node's contribution and the recursive calls on its left and right subtrees (with the updated path maximum).

### Go Code

``` go
func goodNodes(root *TreeNode) int {
    return dfs(root, root.Val)
}

func dfs(node *TreeNode, maxVal int) int {
    if node == nil {
        return 0
    }
    
    count := 0
    if node.Val >= maxVal {
        count = 1
    }
    
    newMax := max(maxVal, node.Val)
    return count + dfs(node.Left, newMax) + dfs(node.Right, newMax)
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - We traverse each node exactly once.
- **Space Complexity**: $O(H)$
    - Requires $O(H)$ space for the recursion stack, where $H$ is the height of the tree.