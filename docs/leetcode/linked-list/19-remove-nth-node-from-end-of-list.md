# 19. Remove Nth Node From End of List

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/remove-nth-node-from-end-of-list/description/)

Given the `head` of a linked list, remove the $n$-th node from the end of the list and return its head.

---

## Solution 1: Stack-Based (Nodes Accumulation)

We can use a stack (implemented as a slice in Go) to store pointers to all the nodes as we traverse the list. This allows us to access the nodes in reverse order.

### Thought Process

1.  **Traverse & Push**:
    *   Iterate through the linked list from the head to the end, appending each node pointer to the `stack`.
2.  **Pop $n$ Nodes**:
    *   Pop $n$ elements off the stack. The last node popped is `top`, which is the node we want to remove.
3.  **Find Predecessor**:
    *   After popping $n$ elements, the node currently at the top of the stack is the predecessor of `top`. Let's call this `last`.
4.  **Edge Case (Removing the Head)**:
    *   If the stack is empty (meaning `last == nil`), the node to remove is the `head` itself. In this case, we simply return `head.Next`.
5.  **Remove Node**:
    *   Otherwise, remove `top` by updating the predecessor's next pointer:
        $$\text{last.Next} = \text{top.Next}$$
6.  **Return**:
    *   Return the original `head`.

### Go Code

``` go
func removeNthFromEnd(head *ListNode, n int) *ListNode {
    stack := []*ListNode{}
    curr := head
    for curr != nil {
        stack = append(stack, curr)
        curr = curr.Next
    }
    var top *ListNode
    for range n {
        top = stack[len(stack)-1]
        stack = stack[:len(stack)-1]
    }
    var last *ListNode
    if len(stack) > 0 {
        last = stack[len(stack)-1]
    }
    if last == nil {
        return head.Next
    }
    last.Next = top.Next
    return head
}
```

### Code Efficiency

- **Time Complexity**: $O(L)$
    - Where $L$ is the number of nodes in the linked list. We traverse the list once to populate the stack, and then perform $n$ constant-time pop operations.
- **Space Complexity**: $O(L)$
    - We store all $L$ node pointers in the slice, requiring $O(L)$ auxiliary space.

---

## Solution 2: Two Pointers (One Pass, Space Optimized)

We can optimize the space complexity to $O(1)$ by using two pointers (`fast` and `slow`) separated by a gap of $n$ nodes.

### Thought Process

1.  **Dummy Node**:
    *   Create a dummy node pointing to `head` (`dummy := &ListNode{Next: head}`). This simplifies handling edge cases, such as removing the head node.
2.  **Establish Gap**:
    *   Initialize both `fast` and `slow` pointers at the dummy node.
    *   Move `fast` forward $n + 1$ steps so that there is a gap of $n$ nodes between `fast` and `slow`.
3.  **Simultaneous Traversal**:
    *   Move both `fast` and `slow` forward one step at a time until `fast` reaches `nil`.
    *   Because of the $n$-node gap, when `fast` reaches the end, `slow` will point to the node **just before** the target node (the predecessor).
4.  **Splicing**:
    *   Skip the target node: `slow.Next = slow.Next.Next`.
5.  **Return**:
    *   Return `dummy.Next`.

### Go Code

``` go
func removeNthFromEnd(head *ListNode, n int) *ListNode {
    dummy := &ListNode{Next: head}
    fast, slow := dummy, dummy
    
    // Move fast pointer n + 1 steps forward
    for i := 0; i <= n; i++ {
        fast = fast.Next
    }
    
    // Move both pointers until fast reaches the end
    for fast != nil {
        fast = fast.Next
        slow = slow.Next
    }
    
    // Remove the target node
    slow.Next = slow.Next.Next
    return dummy.Next
}
```

### Code Efficiency

- **Time Complexity**: $O(L)$
    - We traverse the list of length $L$ in a single pass.
- **Space Complexity**: $O(1)$
    - We only use two pointer variables, achieving optimal constant space complexity.