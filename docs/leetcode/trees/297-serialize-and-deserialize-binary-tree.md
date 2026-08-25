# 297. Serialize and Deserialize Binary Tree

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/description/)

Serialization is the process of converting a data structure or object into a sequence of bits so that it can be stored in a file or memory buffer, or transmitted across a network connection link to be reconstructed later in the same or another computer environment.

Design an algorithm to serialize and deserialize a binary tree. There is no restriction on how your serialization/deserialization algorithm should work. You just need to ensure that a binary tree can be serialized to a string and this string can be deserialized to the original tree structure.

---

## Solution: Preorder DFS Traversal

We can serialize and deserialize the binary tree using a **Preorder DFS** traversal (Root $\rightarrow$ Left $\rightarrow$ Right). Null nodes are explicitly serialized as a placeholder `"N"`, which allows us to uniquely reconstruct the tree structure without needing both preorder and inorder traversals.

### Thought Process

#### 1. Serialization (`serialize`)
*   **Traversal**: Recursively visit nodes in preorder.
*   **Null Nodes**: If the current node is `nil`, append `"N"` to the result slice and return.
*   **Value Nodes**: If the node is not `nil`, append its string representation (using `strconv.Itoa`) to the slice, then recursively serialize the left and right subtrees.
*   **Format**: Join the slice with a comma delimiter: `strings.Join(res, ",")`.

#### 2. Deserialization (`deserialize`)
*   **Split**: Split the serialized data string by commas into a slice of values: `vals := strings.Split(data, ",")`.
*   **State Tracker**: Maintain a pointer index `i := 0` to track our progress through the slice.
*   **Reconstruct**: Recursively build the tree using a preorder DFS helper:
    *   If `vals[i] == "N"`, increment `i` and return `nil`.
    *   Otherwise, parse the integer value, construct a new `TreeNode`, increment `i`, and recursively assign its left and right children.

### Go Code

``` go
import (
    "strconv"
    "strings"
)

type Codec struct{}

func Constructor() Codec {
    return Codec{}
}

// Encodes a tree to a single string.
func (this *Codec) serialize(root *TreeNode) string {
    var res []string

    var dfs func(node *TreeNode)
    dfs = func(node *TreeNode) {
        if node == nil {
            res = append(res, "N")
            return
        }
        res = append(res, strconv.Itoa(node.Val))
        dfs(node.Left)
        dfs(node.Right)
    }

    dfs(root)
    return strings.Join(res, ",")
}

// Decodes your encoded data to tree.
func (this *Codec) deserialize(data string) *TreeNode {
    if len(data) == 0 {
        return nil
    }
    
    vals := strings.Split(data, ",")
    i := 0

    var dfs func() *TreeNode
    dfs = func() *TreeNode {
        if vals[i] == "N" {
            i++
            return nil
        }
        val, _ := strconv.Atoi(vals[i])
        node := &TreeNode{Val: val}
        i++
        node.Left = dfs()
        node.Right = dfs()
        return node
    }

    return dfs()
}
```

### Code Efficiency

- **Time Complexity**:
    - **Serialization**: $O(N)$ where $N$ is the number of nodes in the binary tree. We visit each node and null node once.
    - **Deserialization**: $O(N)$ since we split the string into a slice of size $O(N)$ and process each element in the slice exactly once.
- **Space Complexity**:
    - **Serialization**: $O(N)$ to store the array of string elements, plus $O(H)$ for the recursion stack where $H$ is the height of the tree.
    - **Deserialization**: $O(N)$ to store the split slice, plus $O(H)$ for the recursion stack.