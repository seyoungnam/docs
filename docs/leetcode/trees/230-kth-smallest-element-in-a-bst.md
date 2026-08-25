# 230. Kth Smallest Element in a BST

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/kth-smallest-element-in-a-bst/description/)

Given the `root` of a binary search tree, and an integer `k`, return *the* $k$-*th smallest value (**1-indexed**) of all the values of the nodes in the tree*.

---

## Solution 1: Recursive Inorder Traversal (Full Array)

An **inorder traversal** (Left $\rightarrow$ Root $\rightarrow$ Right) of a Binary Search Tree (BST) processes the nodes in strictly sorted, ascending order. We can perform an inorder traversal, collect all values into an array, and then index the array to find the $k$-th smallest element.

### Thought Process

1.  **Traverse & Store**:
    *   Initialize an empty integer slice `res`.
    *   Define a recursive function `dfs(node)` that performs an inorder traversal:
        *   If `node` is `nil`, return.
        *   Recurse left: `dfs(node.Left)`.
        *   Record current value: `res = append(res, node.Val)`.
        *   Recurse right: `dfs(node.Right)`.
2.  **Return**:
    *   Because $k$ is 1-indexed, the $k$-th smallest element resides at index `k - 1` in our sorted slice:
        $$\text{kthElement} = \text{res}[k-1]$$

### Go Code

``` go
func kthSmallest(root *TreeNode, k int) int {
    res := []int{}

    var dfs func(node *TreeNode)
    dfs = func(node *TreeNode) {
        if node == nil {
            return
        }
        dfs(node.Left)
        res = append(res, node.Val)
        dfs(node.Right)
    }
    dfs(root)
    return res[k-1]
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Where $N$ is the number of nodes in the BST. We visit and record every node in the tree.
- **Space Complexity**: $O(N)$
    - We allocate a slice of size $N$ to store all node values. Additionally, the recursion call stack takes $O(H)$ space, where $H$ is the height of the tree.

---

## Solution 2: Recursive Inorder Traversal (Optimized Count)

We can optimize Solution 1 by avoiding the collection of all node values. Instead, we maintain a count of nodes visited during the inorder traversal, stopping as soon as the count reaches $k$.

### Thought Process

1.  **Variables**:
    *   Maintain a counter `cnt := 0` to track the number of nodes processed.
    *   Maintain a variable `res := 0` to store the target value once found.
2.  **DFS Count Check**:
    *   During the inorder traversal, increment `cnt` after returning from the left child.
    *   If `cnt == k`, we record the current node's value (`res = node.Val`) and return.

### Go Code

``` go
func kthSmallest(root *TreeNode, k int) int {
    cnt, res := 0, 0

    var dfs func(node *TreeNode)
    dfs = func(node *TreeNode) {
        if node == nil {
            return
        }

        dfs(node.Left)
        cnt++
        if cnt == k {
            res = node.Val
            return
        }
        dfs(node.Right)
    }
    dfs(root)
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(H + k)$
    - We traverse down the left spine to the smallest node ($O(H)$ steps) and then visit $k$ nodes.
- **Space Complexity**: $O(H)$
    - Requires $O(H)$ space for the recursion stack, where $H$ is the height of the tree.

---

## Solution 3: Iterative Inorder Traversal (Optimal Fail-Fast)

We can implement the inorder traversal iteratively using an explicit stack. This allows us to stop the traversal immediately once the $k$-th smallest element is found, saving both time and stack frame memory.

### Thought Process

1.  **Iterative Stack**:
    *   Initialize an empty stack `stack := []*TreeNode{}`.
    *   Maintain a pointer `curr := root`.
2.  **Loop**:
    *   While `curr` is not `nil` or the stack is not empty:
        *   **Go Left**: Push all left descendants of `curr` onto the stack: `curr = curr.Left`. This navigates to the smallest unprocessed element.
        *   **Process Node**: Pop the top node from the stack. Decrement `k`.
        *   **Check Target**: If `k == 0`, we have reached the $k$-th smallest element; return `curr.Val`.
        *   **Go Right**: Move to the right subtree: `curr = curr.Right`.

### Go Code

``` go
func kthSmallest(root *TreeNode, k int) int {
    stack := []*TreeNode{}
    curr := root
    
    for curr != nil || len(stack) > 0 {
        // Navigate to the leftmost node
        for curr != nil {
            stack = append(stack, curr)
            curr = curr.Left
        }
        
        // Process current node
        curr = stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        
        k--
        if k == 0 {
            return curr.Val
        }
        
        // Move to the right child
        curr = curr.Right
    }
    return -1
}
```

### Code Efficiency

- **Time Complexity**: $O(H + k)$
    - We only traverse the path to the $k$-th element. In the worst case where $k = N$, this takes $O(N)$ time.
- **Space Complexity**: $O(H)$
    - The stack holds at most $H$ nodes (the height of the tree) at any given time.