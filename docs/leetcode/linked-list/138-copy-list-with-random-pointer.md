# 138. Copy List with Random Pointer

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/copy-list-with-random-pointer/description/)

## Solution: Hash Map (Recursive DFS)

Your original recursive approach results in **infinite recursion (stack overflow)** if there are cycles in the `Random` pointers (e.g., when a node's random pointer points to a previous node). Even without cycles, it will create duplicate copies of nodes that are pointed to multiple times.

To fix this, we must use a **Hash Map** to keep track of already copied nodes.

### Thought Process

1.  **Visited Map**: Use a hash map mapping `*Node` (original) to `*Node` (copied).
2.  **Base Case**: If the node is `nil`, return `nil`.
3.  **Check Map**: If the node is already in the map, return its existing copy.
4.  **Create Copy**: Otherwise, create a new node, register it in the map first to prevent cycles, and then recursively copy its `Next` and `Random` pointers.

### Go Code

``` go
func copyRandomList(head *Node) *Node {
    visited := make(map[*Node]*Node)
    
    var dfs func(node *Node) *Node
    dfs = func(node *Node) *Node {
        if node == nil {
            return nil
        }
        if copyNode, exists := visited[node]; exists {
            return copyNode
        }
        
        newNode := &Node{Val: node.Val}
        visited[node] = newNode // Register before recursion to prevent infinite loops
        
        newNode.Next = dfs(node.Next)
        newNode.Random = dfs(node.Random)
        return newNode
    }
    
    return dfs(head)
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We visit each node in the list exactly once.
- **Space Complexity**: $O(n)$
    - The hash map takes $O(n)$ space, and the recursion stack takes up to $O(n)$ space.

---

## Optimized Solution: In-Place Interweaving

We can eliminate the $O(n)$ auxiliary space by interleaving the copied nodes directly into the original list, setting the random pointers, and then separating the lists.

### Thought Process

1.  **Interweave**: Traverse the list and insert a copy node directly after each original node (`A -> A' -> B -> B'`).
2.  **Link Randoms**: Traverse the interweaved list. The copy node `curr.Next`'s random pointer points to `curr.Random.Next`.
3.  **Separate Lists**: Restore the original list and extract the copied list by separating the interwoven next links.

### Go Code

``` go
func copyRandomList(head *Node) *Node {
    if head == nil {
        return nil
    }
    
    // 1. Interweave original and copy nodes
    curr := head
    for curr != nil {
        next := curr.Next
        copyNode := &Node{Val: curr.Val}
        curr.Next = copyNode
        copyNode.Next = next
        curr = next
    }
    
    // 2. Link copy nodes' random pointers
    curr = head
    for curr != nil {
        if curr.Random != nil {
            curr.Next.Random = curr.Random.Next
        }
        curr = curr.Next.Next
    }
    
    // 3. Separate the interwoven list
    curr = head
    newHead := head.Next
    copyCurr := newHead
    for curr != nil {
        curr.Next = curr.Next.Next
        if copyCurr.Next != nil {
            copyCurr.Next = copyCurr.Next.Next
        }
        curr = curr.Next
        copyCurr = copyCurr.Next
    }
    
    return newHead
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We perform three linear passes over the list.
- **Space Complexity**: $O(1)$
    - We do not use any extra maps or recursion, achieving $O(1)$ auxiliary space.
