# 199. Binary Tree Right Side View

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/binary-tree-right-side-view/description/)

Given the `root` of a binary tree, imagine yourself standing on the **right side** of it, return *the values of the nodes you can see ordered from top to bottom*.

---

## Solution 1: Iterative BFS (Level-Order Traversal)

We can traverse the tree level by level. At each level, the last node we process from left to right is the rightmost node of that level, which we collect into the result.

### Thought Process

1.  **Edge Case**:
    *   If `root` is `nil`, return an empty slice.
2.  **Queue Initialization**:
    *   Initialize a queue `q := []*TreeNode{root}`.
3.  **Level Traversal**:
    *   While the queue is not empty:
        *   Determine the number of nodes at the current level: `length := len(q)`.
        *   Maintain a variable `rightSide` to store the value of the last node in the current level.
        *   Loop `length` times:
            *   Pop the first node `curr` from the queue.
            *   Update `rightSide = curr.Val`.
            *   If `curr.Left` is not `nil`, enqueue it.
            *   If `curr.Right` is not `nil`, enqueue it.
        *   After processing the entire level, append the final `rightSide` value to the result slice `res`.
4.  **Result**:
    *   Return `res`.

### Go Code

``` go
func rightSideView(root *TreeNode) []int {
    res := []int{}
    if root == nil {
        return res
    }
    q := []*TreeNode{root}
    for len(q) > 0 {
        rightSide := 0
        length := len(q)
        for range length {
            curr := q[0]
            q = q[1:]

            rightSide = curr.Val
            if curr.Left != nil {
                q = append(q, curr.Left)
            }
            if curr.Right != nil {
                q = append(q, curr.Right)
            }
        }
        res = append(res, rightSide)
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Where $N$ is the number of nodes in the binary tree. We visit and process each node exactly once.
- **Space Complexity**: $O(W)$
    - Where $W$ is the maximum width of the binary tree, which represents the maximum number of nodes stored in the queue at any time. In a balanced tree, $W = O(N)$ at the leaf level.

---

## Solution 2: Recursive DFS (Root-Right-Left order)

We can traverse the tree using a customized Depth-First Search (DFS) in the order: **Root $\rightarrow$ Right $\rightarrow$ Left**.

### Thought Process

1.  **State Tracking**:
    *   Pass the current `depth` (0-indexed) down the recursion.
2.  **Rightmost Element Selection**:
    *   Because we visit the right child before the left child at every step, the first node we encounter at any given `depth` is guaranteed to be the rightmost node at that level.
    *   We can check if this is the first visit to the current depth by checking if the length of our result slice matches the depth:
        $$\text{len(res)} == \text{depth}$$
    *   If they match, we append the node's value to `res`.
3.  **Recursive Branching**:
    *   Recurse on the right child first: `dfs(node.Right, depth + 1)`.
    *   Recurse on the left child second: `dfs(node.Left, depth + 1)`.

### Go Code

``` go
func rightSideView(root *TreeNode) []int {
    res := []int{}
    var dfs func(node *TreeNode, depth int)
    dfs = func(node *TreeNode, depth int) {
        if node == nil {
            return
        }

        if len(res) == depth {
            res = append(res, node.Val)
        }
        dfs(node.Right, depth+1)
        dfs(node.Left, depth+1)
        return
    }
    dfs(root, 0)
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - We visit every node in the binary tree exactly once.
- **Space Complexity**: $O(H)$
    - Where $H$ is the height of the binary tree, corresponding to the recursion call stack depth. In the worst case (completely skewed tree), $H = O(N)$. In the best case (balanced binary tree), $H = O(\log N)$.