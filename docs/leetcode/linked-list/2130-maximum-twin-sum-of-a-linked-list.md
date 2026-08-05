# 2130. Maximum Twin Sum of a Linked List

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/maximum-twin-sum-of-a-linked-list/description/)

## Solution: Finding Middle, Reversing Second Half, and Twin Pointers

We can solve this problem in $O(n)$ time and $O(1)$ auxiliary space by finding the middle of the list, reversing the second half, and using two pointers to calculate the twin sums.

### Thought Process

1.  **Understand Twins**:
    *   For a linked list of even length $n$, the twin of the $i$-th node is the $(n-1-i)$-th node.
    *   For example, in a list of size 4:
        *   Index `0` and index `3` are twins.
        *   Index `1` and index `2` are twins.
2.  **Find the Middle**:
    *   Use the fast and slow pointer approach to split the list in half.
    *   Initialize `slow, fast := head, head`.
    *   Advance `fast` by two steps and `slow` by one step. When the loop terminates, `slow` will point to the start of the second half of the list (index $n/2$).
3.  **Reverse the Second Half**:
    *   To pair elements from the beginning of the list with elements from the end, we reverse the second half of the list starting from `slow`.
    *   We perform an in-place reversal using a dummy node (`&ListNode{}`) as the sentinel base (`prev`).
4.  **Compute Max Twin Sum**:
    *   Initialize pointer `p1` at the beginning (`head`) and `p2` at the head of the reversed second half (`prev`).
    *   Traverse both halves in parallel. Since the second half is reversed, `p1` and `p2` will point to twin nodes at each step.
    *   Compute their sum, update the running maximum (`res`), and advance both pointers.

### Go Code

``` go
func pairSum(head *ListNode) int {
    slow, fast := head, head
    for fast != nil && fast.Next != nil {
        fast = fast.Next.Next
        slow = slow.Next
    }
    prev := &ListNode{}
    curr := slow
    for curr != nil {
        next := curr.Next
        curr.Next = prev

        prev = curr
        curr = next
    }

    res := 0
    p1, p2 := head, prev
    for p2 != nil {
        res = max(res, p1.Val + p2.Val)
        p1 = p1.Next
        p2 = p2.Next
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - Finding the middle takes $O(n)$ time.
    - Reversing the second half takes $O(n)$ time.
    - Calculating the twin sums takes $O(n)$ time.
    - Overall time complexity is linear.
- **Space Complexity**: $O(1)$
    - We modify the list pointers in-place, requiring only a constant number of auxiliary pointer variables and a single dummy node.