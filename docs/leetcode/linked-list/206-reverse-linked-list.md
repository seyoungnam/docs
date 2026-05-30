# 206. Reverse Linked List

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/reverse-linked-list/description/)

## Solution: Iterative (Pointer Manipulation)

We can reverse the linked list in-place by iterating through the list and reversing the `Next` pointer of each node to point to its predecessor.

### Thought Process

1.  **Initialize Pointers**: 
    *   *Edge Case*: If `head` is `nil`, return it immediately.
    *   Set `prev = head` and `curr = head.Next`.
    *   Set `prev.Next = nil` (as the original head becomes the tail of the reversed list).
2.  **Pointer Splicing Loop**: While `curr` is not `nil`:
    *   Save the next node: `next = curr.Next`.
    *   Reverse the link: `curr.Next = prev`.
    *   Advance pointers: `prev = curr` and `curr = next`.
3.  **Result**: Return `prev` (the new head of the reversed list).

### Go Code

``` go
func reverseList(head *ListNode) *ListNode {
    if head == nil {
        return head
    }
    prev, curr := head, head.Next
    prev.Next = nil
    for curr != nil {
        next := curr.Next
        curr.Next = prev
        prev, curr = curr, next
    }
    return prev
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We traverse the list of length $n$ exactly once.
- **Space Complexity**: $O(1)$
    - We modify the list in-place, requiring $O(1)$ auxiliary space.

---

## Alternative Solution: Recursive

Alternatively, we can reverse the list recursively. The head of the reversed list is resolved at the base case, and as the recursion unwinds, we reverse the links from right to left.

### Thought Process

1.  **Base Cases**: If `head` is `nil` or `head.Next` is `nil` (only one node left), it is already reversed. Return `head`.
2.  **Recurse**: Recursively reverse the rest of the list: `newHead = reverseList(head.Next)`.
3.  **Reverse Link**: Point the next node's next pointer back to the current node: `head.Next.Next = head`.
4.  **Sever Original Link**: Set the current node's next pointer to `nil` to prevent cycles: `head.Next = nil`.
5.  **Result**: Return `newHead`.

### Go Code

``` go
func reverseList(head *ListNode) *ListNode {
    // Base cases
    if head == nil || head.Next == nil {
        return head
    }
    
    // Reverse the sublist
    newHead := reverseList(head.Next)
    
    // Reverse the current node's link
    head.Next.Next = head
    head.Next = nil
    
    return newHead
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We visit each node exactly once.
- **Space Complexity**: $O(n)$
    - The recursion call stack can reach a depth of $n$, requiring $O(n)$ auxiliary space.
