# 105. Construct Binary Tree from Preorder and Inorder Traversal

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/description/)

Given two integer arrays `preorder` and `inorder` where `preorder` is the preorder traversal of a binary tree and `inorder` is the inorder traversal of the same tree, construct and return *the binary tree*.

---

## Solution 1: Recursive Slice Slicing

We can reconstruct the binary tree by exploiting the structure of preorder and inorder traversals:
*   **Preorder** traversal is ordered as: `[Root, [Left Subtree], [Right Subtree]]`. Therefore, the first element `preorder[0]` is always the root of the current subtree.
*   **Inorder** traversal is ordered as: `[[Left Subtree], Root, [Right Subtree]]`.

### Thought Process

1.  **Base Case**:
    *   If either slice is empty, return `nil`.
2.  **Locate Root**:
    *   The root value is `preorder[0]`.
    *   Find the index `m` of this root value in the `inorder` array.
3.  **Divide and Conquer**:
    *   The number of nodes in the left subtree is `m` (the count of elements before index `m` in `inorder`).
    *   Slice the arrays into left and right sub-problems:
        *   **Left Subtree**: `preorder[1 : 1+m]` and `inorder[:m]`
        *   **Right Subtree**: `preorder[1+m:]` and `inorder[m+1:]`
4.  **Recurse**:
    *   Build left and right subtrees recursively:
        $$\text{node.Left} = \text{buildTree(preorder[1:1+m], inorder[:m])}$$
        $$\text{node.Right} = \text{buildTree(preorder[1+m:], inorder[m+1:])}$$
5.  **Return**:
    *   Return the constructed `node`.

### Go Code

``` go
func buildTree(preorder []int, inorder []int) *TreeNode {
    if len(preorder) == 0 || len(inorder) == 0 {
        return nil
    }
    
    m := -1
    for i, num := range inorder {
        if num == preorder[0] {
            m = i
            break
        }
    }
    
    node := &TreeNode{Val: preorder[0]}
    node.Left = buildTree(preorder[1:1+m], inorder[:m])
    node.Right = buildTree(preorder[1+m:], inorder[m+1:])

    return node
}
```

### Code Efficiency

- **Time Complexity**: $O(N^2)$
    - In the worst case of a completely skewed tree, locating the root index `m` in `inorder` takes $O(N)$ linear scans at each recursive step, resulting in $O(N^2)$ time. For a balanced tree, the complexity is $O(N \log N)$.
- **Space Complexity**: $O(H)$
    - Corresponding to the recursion stack depth, where $H$ is the height of the tree. In the worst case, $O(N)$ space is required.

---

## Solution 2: Hash Map Lookup & Index Pointers (Linear Time)

We can optimize the search for the root's index to $O(1)$ by precomputing a hash map mapping values to their indices in the `inorder` array. Additionally, we can pass boundary pointers (`left`, `right`) instead of creating array slices, which avoids memory allocation overhead.

### Thought Process

1.  **Precompute Indices**:
    *   Build a hash map `indices` where `indices[val]` returns the index of `val` in the `inorder` array.
2.  **Track Preorder Index**:
    *   Use a pointer `preIdx` starting at `0` to track the current root in the `preorder` array.
3.  **DFS Helper**:
    *   Define `dfs(left, right)` returning the subtree constructed from the inorder range `[left, right]`.
    *   Base Case: If `left > right`, return `nil`.
    *   Node Creation: Get the root value `rootVal := preorder[preIdx]`, increment `preIdx`, and instantiate `root := &TreeNode{Val: rootVal}`.
    *   Lookup: Find the partition point `mid := indices[rootVal]`.
    *   Recurse:
        *   Left subtree: `dfs(left, mid - 1)`.
        *   Right subtree: `dfs(mid + 1, right)`.

### Go Code

``` go
func buildTree(preorder []int, inorder []int) *TreeNode {
    indices := map[int]int{}
    for i, val := range inorder {
        indices[val] = i
    }

    preIdx := 0

    var dfs func(int, int) *TreeNode
    dfs = func(left, right int) *TreeNode {
        if left > right {
            return nil
        }

        rootVal := preorder[preIdx]
        preIdx++

        root := &TreeNode{Val: rootVal}
        mid := indices[rootVal]

        root.Left = dfs(left, mid-1)
        root.Right = dfs(mid+1, right)

        return root
    }
    return dfs(0, len(inorder)-1)
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - We build the hash map in $O(N)$ time. The helper function `dfs` is called exactly once for each of the $N$ nodes, and each call performs $O(1)$ operations.
- **Space Complexity**: $O(N)$
    - Storing the hash map takes $O(N)$ auxiliary space. The recursion call stack takes $O(H)$ space.