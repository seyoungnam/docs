# 141. Linked List Cycle

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/linked-list-cycle/description/)

## Solution: Floyd's Tortoise and Hare (Two Pointers)

We can detect a cycle in a linked list in $O(n)$ time and $O(1)$ space using two pointers moving at different speeds: a slow pointer `slow` and a fast pointer `fast`.

### Thought Process

1.  **Initialize Pointers**: Set both `slow` and `fast` to start at `head`.
2.  **Pointer Chase**: Loop while `fast` and `fast.Next` are not `nil`:
    *   Advance the `slow` pointer by 1 step: `slow = slow.Next`.
    *   Advance the `fast` pointer by 2 steps: `fast = fast.Next.Next`.
    *   **Cycle Detection**: If `slow == fast`, they have met! This means the fast pointer has caught up to the slow pointer within a cycle. Return `true`.
3.  **No Cycle**: If the loop terminates (meaning `fast` or `fast.Next` became `nil`), we have reached the end of the list with no cycles. Return `false`.

### Go Code

``` go
func hasCycle(head *ListNode) bool {
    slow, fast := head, head
    for fast != nil && fast.Next != nil {
        slow = slow.Next
        fast = fast.Next.Next
        if slow == fast {
            return true
        }
    }
    return false
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - If there is no cycle, `fast` reaches the end of the list in $n/2$ steps. If there is a cycle, the distance between `slow` and `fast` decreases by `1` at each step, ensuring they will meet in at most $n$ iterations.
- **Space Complexity**: $O(1)$
    - We only use two tracking pointer variables, requiring $O(1)$ auxiliary space.
