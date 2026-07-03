# 100. Same Tree

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/same-tree/description/)

## Solution 1: Depth-First Search (Recursive DFS)

We can determine if two binary trees are identical by using recursion to traverse them simultaneously. Two trees are the same if their root nodes have the same value, and their left and right subtrees are also identical.

### Thought Process

1.  **Base Cases**:
    *   If both `p` and `q` are `nil`, we have reached the end of both branches simultaneously, meaning they are identical at this position $\rightarrow$ return `true`.
    *   If only one of them is `nil` (meaning one tree has a node and the other does not), they are structurally different $\rightarrow$ return `false`.
2.  **Recursive Step**:
    *   If both nodes are non-nil, we check:
        *   Their values are equal: `p.Val == q.Val`.
        *   Their left subtrees are identical: `isSameTree(p.Left, q.Left)`.
        *   Their right subtrees are identical: `isSameTree(p.Right, q.Right)`.
    *   If all three conditions are met, return `true`; otherwise, return `false`.

### Go Code

``` go
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

- **Time Complexity**: $O(\min(N, M))$
    - We visit each node of both trees at most once, where $N$ and $M$ are the number of nodes in tree `p` and `q` respectively. The algorithm terminates early as soon as any structural or value mismatch is detected.
- **Space Complexity**: $O(\min(H_p, H_q))$
    - The auxiliary space is determined by the recursion call stack, which is bounded by the height of the trees. In the worst case of a skewed tree, this is $O(N)$ or $O(M)$. In the best case of a completely balanced tree, it is $O(\log N)$ or $O(\log M)$.

---

## Solution 2: Breadth-First Search (Iterative BFS)

Alternatively, we can solve this problem iteratively using a queue to traverse the trees level-by-level (Breadth-First Search). We enqueue corresponding nodes from both trees in pairs and compare them as we pop them.

### Thought Process

1.  **Queue Initialization**:
    *   Initialize a queue containing the root nodes of both trees: `queue := []*TreeNode{p, q}`.
2.  **Iterative Comparison**:
    *   While the queue is not empty:
        *   Pop the first two nodes `n1` and `n2` from the front of the queue.
        *   If both `n1` and `n2` are `nil`, they match structurally $\rightarrow$ `continue` to the next pair.
        *   If only one of them is `nil`, or their values differ, they do not match $\rightarrow$ return `false`.
        *   Enqueue their children in pairs: left child of both nodes (`n1.Left`, `n2.Left`), followed by the right child of both nodes (`n1.Right`, `n2.Right`).
3.  **Result**:
    *   If the loop finishes without returning `false`, the trees are identical $\rightarrow$ return `true`.

### Go Code

``` go
func isSameTree(p *TreeNode, q *TreeNode) bool {
    queue := []*TreeNode{p, q}

    for len(queue) > 0 {
        n1 := queue[0]
        n2 := queue[1]
        queue = queue[2:]

        if n1 == nil && n2 == nil {
            continue
        }
        if n1 == nil || n2 == nil || n1.Val != n2.Val {
            return false
        }

        queue = append(queue, n1.Left, n2.Left)
        queue = append(queue, n1.Right, n2.Right)
    }
    return true
}
```

### Code Efficiency

- **Time Complexity**: $O(\min(N, M))$
    - We visit each node in both trees at most once.
- **Space Complexity**: $O(\min(N, M))$
    - In the worst case, the queue holds nodes proportional to the width of the trees, which is bounded by the total number of nodes in the trees.