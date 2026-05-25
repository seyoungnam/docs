# 2. Add Two Numbers

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/add-two-numbers/description/)

## Solution: Elementary Math (Iterative)

We can simulate addition column by column from right to left (least significant digit to most significant digit) using a single loop, carrying over any remainder.

### Thought Process

1.  **Initialize Dummy Node**: Start with a `dummy` node to act as the head of the output linked list, and a pointer `curr` to track the current tail.
2.  **Iterate and Sum**: Loop while `l1` is not nil, `l2` is not nil, or `carry` is not 0:
    *   Extract the current digits: `x` from `l1` and `y` from `l2` (using `0` if the node is `nil`).
    *   Compute the sum of the digits plus the carry: `sum = carry + x + y`.
    *   Calculate the new carry: `carry = sum / 10`.
    *   Create a new node with value `sum % 10` and attach it to `curr.Next`.
3.  **Advance Pointers**: Move `curr` to its next node. Advance `l1` and `l2` to their next nodes if they are not nil.
4.  **Result**: Return `dummy.Next`.

### Go Code

``` go
func addTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {
    var carry int = 0
    dummy := &ListNode{Val: 0}
    curr := dummy
    for l1 != nil || l2 != nil || carry != 0 {
        x, y := 0, 0
        if l1 != nil {
            x = l1.Val
        }
        if l2 != nil {
            y = l2.Val
        }
        sum := carry + x + y
        carry = sum / 10
        curr.Next = &ListNode{Val: sum % 10}

        curr = curr.Next
        if l1 != nil {
            l1 = l1.Next
        }
        if l2 != nil {
            l2 = l2.Next
        }
    }
    return dummy.Next
}
```

### Code Efficiency

- **Time Complexity**: $O(\max(m, n))$
    - We loop at most $\max(m, n) + 1$ times, where $m$ and $n$ are the lengths of `l1` and `l2`.
- **Space Complexity**: $O(1)$ auxiliary space
    - Excluding the memory allocated for the new output list, we only use a few integer variables, requiring $O(1)$ extra space.
