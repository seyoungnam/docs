# 143. Reorder List

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/reorder-list/description/)

## Solution: Split, Reverse, and Merge (Three-Step Process)

We can reorder the list in-place in $O(n)$ time and $O(1)$ space by breaking the problem down into three fundamental linked list operations:
1.  **Find the Middle**: Split the list into two halves using Floyd's Tortoise and Hare pointers (`slow` and `fast`).
2.  **Reverse the Second Half**: Reverse the second half of the list in-place (starting from `slow.Next`), and sever the link `slow.Next = nil` to separate the two sublists.
3.  **Merge the Two Halves**: Alternately merge the nodes from the `first` and `second` sublists.

### Thought Process

1.  **Find Middle**: Set `slow, fast = head, head`. While `fast != nil` and `fast.Next != nil`, advance `slow` by 1 step and `fast` by 2 steps. The `slow` pointer will stop at the middle of the list.
2.  **Reverse Second Half**: Call the `reverse` helper on `slow.Next` to reverse the second half of the list, saving it in the variable `second`. Set `slow.Next = nil` to sever the connection between the first and second halves.
3.  **Alternate Merge**: Set `first = head` and initialize a `dummy` node to merge both lists. Alternately attach a node from the `first` list, then a node from the `second` list, advancing the `curr` pointer at each step.

### Go Code

``` go
func reorderList(head *ListNode)  {
    if head == nil || head.Next == nil {
        return
    }
    
    // 1. Find the middle of the list
    slow, fast := head, head
    for fast != nil && fast.Next != nil {
        slow = slow.Next
        fast = fast.Next.Next
    }
    
    // 2. Reverse the second half and sever the link
    second := reverse(slow.Next)
    slow.Next = nil
    
    // 3. Alternately merge the two halves
    first := head
    dummy := &ListNode{}
    curr := dummy
    for first != nil && second != nil {
        curr.Next = first
        first = first.Next
        curr = curr.Next

        curr.Next = second
        second = second.Next
        curr = curr.Next
    }
    
    // Append any remaining node
    if first != nil {
        curr.Next = first
    } else {
        curr.Next = second
    }
}

// Helper function to reverse a linked list
func reverse(curr *ListNode) *ListNode {
    var prev, next *ListNode
    for curr != nil {
        next = curr.Next
        curr.Next = prev
        prev, curr = curr, next
    }
    return prev
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - Finding the middle takes $n/2$ steps, reversing the second half takes $n/2$ steps, and merging both halves takes $n$ steps. The overall time complexity is linear.
- **Space Complexity**: $O(1)$
    - The operations are performed entirely in-place by rewriting next pointers, requiring $O(1)$ auxiliary space.
