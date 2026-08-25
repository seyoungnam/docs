# 226. Invert Binary Tree

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/invert-binary-tree/description/)

Given the `root` of a binary tree, invert the tree, and return *its root*.

---

## Solution 1: Recursive Depth-First Search (DFS)

We can solve this problem recursively by traversing the tree and swapping the left and right child pointers of each node.

### Thought Process

1.  **Base Case**:
    *   If the current node `root` is `nil`, there is nothing to invert; return `nil`.
2.  **Pointer Swap**:
    *   Swap the left and right child pointers of the current node:
        $$\text{root.Left, root.Right} = \text{root.Right, root.Left}$$
3.  **Recursion**:
    *   Recursively call `invertTree` on the new left child.
    *   Recursively call `invertTree` on the new right child.
4.  **Return**:
    *   Return the `root` of the inverted tree.

### Go Code

``` go
func invertTree(root *TreeNode) *TreeNode {
    if root == nil {
        return nil
    }
    root.Left, root.Right = root.Right, root.Left
    invertTree(root.Left)
    invertTree(root.Right)
    return root
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Where $N$ is the number of nodes in the binary tree. We visit and swap the children of each node exactly once.
- **Space Complexity**: $O(H)$
    - Where $H$ is the height of the binary tree, corresponding to the recursion call stack depth. In the worst case of a completely skewed tree, $H = O(N)$. In the best case of a balanced binary tree, $H = O(\log N)$.

---

## Solution 2: Iterative Breadth-First Search (BFS)

Alternatively, we can invert the tree iteratively by using a queue to traverse the tree level by level.

### Thought Process

1.  **Edge Case**:
    *   If `root` is `nil`, return `nil`.
2.  **Queue Initialization**:
    *   Initialize a queue `q := []*TreeNode{root}` to keep track of nodes to process.
3.  **BFS Loop**:
    *   While the queue is not empty:
        *   Pop the first node `curr` from the queue.
        *   Swap `curr.Left` and `curr.Right`.
        *   If `curr.Left` is not `nil`, append it to the queue.
        *   If `curr.Right` is not `nil`, append it to the queue.
4.  **Return**:
    *   Return the `root`.

### Go Code

``` go
func invertTree(root *TreeNode) *TreeNode {
    if root == nil {
        return nil
    }

    q := []*TreeNode{root}
    for len(q) > 0 {
        curr := q[0]
        q = q[1:]

        curr.Left, curr.Right = curr.Right, curr.Left
        if curr.Left != nil {
            q = append(q, curr.Left)
        }
        if curr.Right != nil {
            q = append(q, curr.Right)
        }
    }
    return root
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - We process each node exactly once as we enqueue and dequeue it.
- **Space Complexity**: $O(W)$
    - Where $W$ is the maximum width of the binary tree. In the worst case of a fully balanced binary tree, the queue will store the leaf level, which contains $\lceil N/2 \rceil = O(N)$ nodes.