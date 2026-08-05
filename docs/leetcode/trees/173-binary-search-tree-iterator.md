# 173. Binary Search Tree Iterator

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/binary-search-tree-iterator/description/)

This problem requires implementing an iterator over the in-order traversal of a Binary Search Tree (BST). The optimal implementation must satisfy:
*   `Next()` and `HasNext()` run in average $O(1)$ time.
*   The auxiliary space complexity is bounded by $O(H)$, where $H$ is the height of the tree.

---

## Approach 1: Flattening the BST (Precomputed DFS)

This approach precomputes the entire in-order traversal of the BST and stores it in an array during initialization.

### Thought Process

1.  **Flattening**: 
    *   Since an in-order traversal of a BST yields elements in sorted ascending order, we traverse the BST using Depth-First Search (DFS) and save all node values in a slice `arr`.
    *   We can perform this traversal recursively or iteratively during construction.
2.  **Pointer Tracking**:
    *   Maintain a pointer/index `idx` (initialized to `0`) to track the current element position.
3.  **Operations**:
    *   `Next()`: Returns `arr[idx]` and increments `idx`.
    *   `HasNext()`: Returns true if `idx < len(arr)`.

### Go Code (Recursive DFS)

``` go
type BSTIterator struct {
    arr []int
    idx int
}

func Constructor(root *TreeNode) BSTIterator {
    arr := make([]int, 0)
    
    var dfs func(*TreeNode)
    dfs = func(node *TreeNode) {
        if node == nil {
            return
        }
        dfs(node.Left)
        arr = append(arr, node.Val)
        dfs(node.Right)
    }
    dfs(root)
    return BSTIterator{
        arr: arr, idx: 0,
    }
}

func (this *BSTIterator) Next() int {
    res := this.arr[this.idx]
    this.idx++
    return res
}

func (this *BSTIterator) HasNext() bool {
    return this.idx < len(this.arr)
}
```

### Go Code (Iterative DFS)

``` go
type BSTIterator struct {
    arr []int
    idx int
}

func Constructor(root *TreeNode) BSTIterator {
    arr := make([]int, 0)
    stack := make([]*TreeNode, 0)
    curr := root
    for curr != nil || len(stack) > 0 {
        for curr != nil {
            stack = append(stack, curr)
            curr = curr.Left
        }
        curr = stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        arr = append(arr, curr.Val)
        curr = curr.Right        
    }
    return BSTIterator{arr: arr, idx: 0}
}

func (this *BSTIterator) Next() int {
    res := this.arr[this.idx]
    this.idx++
    return res
}

func (this *BSTIterator) HasNext() bool {
    return this.idx < len(this.arr)
}
```

### Code Efficiency
*   **Time Complexity**:
    *   **Constructor**: $O(N)$ to visit and flatten all $N$ nodes of the tree.
    *   **Next() / HasNext()**: $O(1)$ constant time operations.
*   **Space Complexity**: $O(N)$
    *   Requires storing all $N$ elements in the array, violating the $O(H)$ memory constraint.

---

## Approach 2: Controlled Iterative DFS (Optimal)

This approach simulates the in-order traversal stack dynamically on-demand, caching only the path from the root to the current node.

### Thought Process

1.  **Controlled In-Order Traversal**:
    *   Instead of flattening the entire tree at once, we use a `stack` to store parent nodes. 
    *   We push all the left-most nodes starting from the root onto the stack. The top of the stack will contain the smallest node in the tree.
2.  **Next() Operation**:
    *   Pop the top node `node` from the stack. This is the next smallest element to return.
    *   **Traverse Right**: If the popped node has a right child, we must process its elements. Move to the right child (`curr = node.Right`) and push it, along with all of its left-most descendants, onto the stack.
    *   Return `node.Val`.
3.  **HasNext() Operation**:
    *   Simply check if the stack is not empty.

### Go Code (Eager Left-Path Traversal)

``` go
type BSTIterator struct {
    stack []*TreeNode
}

func Constructor(root *TreeNode) BSTIterator {
    stack := make([]*TreeNode, 0)
    curr := root
    for curr != nil {
        stack = append(stack, curr)
        curr = curr.Left
    }
    return BSTIterator{stack: stack}
}

func (this *BSTIterator) Next() int {
    node := this.stack[len(this.stack)-1]
    this.stack = this.stack[:len(this.stack)-1]
    curr := node.Right
    for curr != nil {
        this.stack = append(this.stack, curr)
        curr = curr.Left
    }
    return node.Val
}

func (this *BSTIterator) HasNext() bool {
    return len(this.stack) > 0 
}
```

### Go Code (Lazy Left-Path Traversal)

``` go
type BSTIterator struct {
    curr    *TreeNode
    stack   []*TreeNode
}

func Constructor(root *TreeNode) BSTIterator {
    return BSTIterator{curr: root, stack: make([]*TreeNode, 0)}
}

func (this *BSTIterator) Next() int {
    for this.curr != nil {
        this.stack = append(this.stack, this.curr)
        this.curr = this.curr.Left
    }
    node := this.stack[len(this.stack)-1]
    this.stack = this.stack[:len(this.stack)-1]
    this.curr = node.Right
    return node.Val
}

func (this *BSTIterator) HasNext() bool {
    return this.curr != nil || len(this.stack) > 0
}
```

### Code Efficiency

*   **Time Complexity**:
    *   **Constructor**: $O(H)$ where $H$ is the height of the tree.
    *   **Next()**: Amortized $O(1)$ time. Although the loop inside `Next()` can take $O(H)$ time in the worst case (e.g. traveling down to a deep left leaf node), each node in the tree is pushed onto and popped from the stack exactly once. Across $N$ total calls, the average cost per call is $O(1)$.
    *   **HasNext()**: $O(1)$ time.
*   **Space Complexity**: $O(H)$
    *   At any given time, the stack stores at most the path from the root to a leaf node, requiring space proportional to the height of the tree. This is $O(\log N)$ for a balanced tree and $O(N)$ for a skewed tree.