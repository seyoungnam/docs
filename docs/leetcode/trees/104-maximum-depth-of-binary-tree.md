# 104. Maximum Depth of Binary Tree

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/maximum-depth-of-binary-tree/description/)

Given the `root` of a binary tree, return *its maximum depth*.

A binary tree's **maximum depth** is the number of nodes along the longest path from the root node down to the farthest leaf node.

---

## Solution 1: Recursive Depth-First Search (DFS)

We can solve this problem recursively by breaking it down into subproblems: the maximum depth of a node is 1 plus the maximum of the depths of its left and right subtrees.

### Thought Process

1.  **Base Case**:
    *   If the current node `root` is `nil`, the depth is `0`.
2.  **Recurse**:
    *   Recursively calculate the maximum depth of the left subtree: `l := maxDepth(root.Left)`.
    *   Recursively calculate the maximum depth of the right subtree: `r := maxDepth(root.Right)`.
3.  **Combine**:
    *   The depth of the tree rooted at the current node is the maximum of the two subtree depths plus 1 (for the current node itself):
        $$\text{depth} = \max(l, r) + 1$$

### Go Code

``` go
func maxDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }
    l := maxDepth(root.Left)
    r := maxDepth(root.Right)
    return max(l, r) + 1
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Where $N$ is the total number of nodes in the binary tree. We visit each node exactly once.
- **Space Complexity**: $O(H)$
    - Where $H$ is the height of the binary tree, corresponding to the recursion call stack depth. In the worst case of a completely skewed tree, $H = O(N)$. In the best case of a balanced binary tree, $H = O(\log N)$.

---

## Solution 2: Iterative Breadth-First Search (BFS)

We can also traverse the tree level by level iteratively using a queue, tracking each node's depth as we enqueue it.

### Thought Process

1.  **Helper Struct**:
    *   Define an `Entry` struct to package each node alongside its current depth:
        ```go
        type Entry struct {
            Node    *TreeNode
            Depth   int
        }
        ```
2.  **Initialization**:
    *   If `root` is `nil`, return `0`.
    *   Initialize the queue `q` containing `Entry{root, 1}` and set `res := 1` to track the maximum depth encountered.
3.  **BFS Loop**:
    *   While the queue is not empty:
        *   Pop the first entry `curr` from the queue.
        *   Update the maximum depth: `res = max(res, curr.Depth)`.
        *   If `curr.Node.Left` is not `nil`, append `Entry{curr.Node.Left, curr.Depth + 1}` to the queue.
        *   If `curr.Node.Right` is not `nil`, append `Entry{curr.Node.Right, curr.Depth + 1}` to the queue.
4.  **Return**:
    *   Return `res`.

### Go Code

``` go
type Entry struct {
    Node    *TreeNode
    Depth   int
}

func maxDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }
    entry := Entry{root, 1}
    q := []Entry{entry}
    res := 1
    for len(q) > 0 {
        curr := q[0]
        q = q[1:]

        node, depth := curr.Node, curr.Depth
        res = max(res, depth)
        if node.Left != nil {
            q = append(q, Entry{node.Left, depth+1})
        }
        if node.Right != nil {
            q = append(q, Entry{node.Right, depth+1})
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - We visit and process each node exactly once.
- **Space Complexity**: $O(W)$
    - Where $W$ is the maximum width of the tree. In the worst case of a fully balanced binary tree, the queue will store the leaf level, which contains $\lceil N/2 \rceil = O(N)$ nodes.