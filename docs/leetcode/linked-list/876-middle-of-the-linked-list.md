# 876. Middle of the Linked List

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/middle-of-the-linked-list/description/)

## Solution: Fast and Slow Pointers (Tortoise and Hare)

We can find the middle node of a singly linked list in a single pass using the **two-pointer (fast and slow)** approach.

### Thought Process

1.  **Objective**: Find the middle node of the list. If the list has an even number of elements, return the second middle node.
2.  **Algorithm**:
    *   Initialize two pointers, `slow` and `fast`, both pointing to the `head` of the linked list.
    *   Iterate through the list. In each step:
        *   Move `slow` forward by **one** node: `slow = slow.Next`
        *   Move `fast` forward by **two** nodes: `fast = fast.Next.Next`
3.  **Termination Condition**:
    *   **Odd-length list**: If the list has an odd number of nodes (e.g., $1 \rightarrow 2 \rightarrow 3 \rightarrow 4 \rightarrow 5$), the loop terminates when `fast.Next == nil` (i.e. `fast` is on the last node `5`). At this point, `slow` points to `3`, which is the exact middle.
    *   **Even-length list**: If the list has an even number of nodes (e.g., $1 \rightarrow 2 \rightarrow 3 \rightarrow 4 \rightarrow 5 \rightarrow 6$), the loop terminates when `fast == nil` (i.e. `fast` has moved past the end). At this point, `slow` points to `4`, which is the second middle node.
4.  **Result**:
    *   When the loop finishes, `slow` is guaranteed to point to the correct middle node. Return `slow`.

### Go Code

``` go
func middleNode(head *ListNode) *ListNode {
    slow, fast := head, head
    for fast != nil && fast.Next != nil {
        fast = fast.Next.Next
        slow = slow.Next
    }
    return slow
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We traverse the list in a single pass, visiting each node at most once.
- **Space Complexity**: $O(1)$
    - The algorithm runs in-place, requiring only constant auxiliary variables.