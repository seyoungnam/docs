# 21. Merge Two Sorted Lists

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/merge-two-sorted-lists/description/)

## Solution: Iterative (Dummy Node)

We can merge the two sorted lists in-place by using a dummy head node. We iterate through both lists, compare the values of the current nodes, and attach the smaller one to our merged list.

### Thought Process

1.  **Use Dummy Node**: Create a dummy node to act as the head of the new merged list, and a pointer `curr` to track the tail.
2.  **Iterate and Compare**: While both lists are non-empty, append the node with the smaller value to `curr.Next`, then advance that list's pointer.
3.  **Advance Tail**: Move `curr` to `curr.Next` after each addition.
4.  **Append Remaining**: Once one list becomes empty, point `curr.Next` directly to the remaining non-empty list.
5.  **Result**: Return `dummy.Next`.

### Go Code

``` go
func mergeTwoLists(list1 *ListNode, list2 *ListNode) *ListNode {
    dummy := &ListNode{}
    curr := dummy
    for list1 != nil && list2 != nil {
        if list1.Val <= list2.Val {
            curr.Next = list1
            list1 = list1.Next
        } else {
            curr.Next = list2
            list2 = list2.Next
        }
        curr = curr.Next
    }
    if list1 != nil {
        curr.Next = list1
    } else {
        curr.Next = list2
    }
    return dummy.Next
}
```

### Code Efficiency

- **Time Complexity**: $O(m + n)$
    - We traverse each node in both lists at most once, where $m$ and $n$ are the lengths of `list1` and `list2`.
- **Space Complexity**: $O(1)$
    - We only splice existing nodes and use a few pointers, using $O(1)$ auxiliary space.

---

## Alternative Solution: Recursive

We can define the merge recursively. The head of the merged list is the smaller of the two heads, and its next pointer points to the result of merging the remaining nodes.

### Thought Process

1.  **Base Cases**: If either list is empty, return the other list.
2.  **Recursive Step**: Compare the values of the heads of both lists:
    *   If `list1.Val < list2.Val`, set `list1.Next` to the result of merging the rest of `list1` and `list2`, then return `list1`.
    *   Otherwise, set `list2.Next` to the result of merging `list1` and the rest of `list2`, then return `list2`.

### Go Code

``` go
func mergeTwoLists(list1 *ListNode, list2 *ListNode) *ListNode {
    if list1 == nil {
        return list2
    }
    if list2 == nil {
        return list1
    }
    if list1.Val < list2.Val {
        list1.Next = mergeTwoLists(list1.Next, list2)
        return list1
    } else {
        list2.Next = mergeTwoLists(list1, list2.Next)
        return list2
    }
}
```

### Code Efficiency

- **Time Complexity**: $O(m + n)$
    - We perform one comparison step per node in the worst case.
- **Space Complexity**: $O(m + n)$
    - The recursion stack takes $O(m + n)$ auxiliary space due to the function call stack.
