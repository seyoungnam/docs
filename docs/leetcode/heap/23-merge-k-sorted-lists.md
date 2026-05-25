# 23. Merge k Sorted Lists

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/merge-k-sorted-lists/description/)

## Solution: Divide and Conquer (Merge with Pairwise Reduction)

The problem asks to merge $k$ sorted linked lists into one sorted linked list. While a Min-Heap is a common approach, this implementation uses a **Divide and Conquer** strategy (similar to Merge Sort) to merge lists in pairs until only one remains.

### Thought Process

1.  **Base Case**: If the input `lists` is empty, return `nil`.
2.  **Pairwise Merging**: Instead of merging lists one by one (which would be $O(N \cdot k)$), merge them in pairs. This reduces the number of lists by half in each iteration.
3.  **Iteration Strategy**: 
    - Use a loop that continues as long as there is more than one list in the collection.
    - Inside the loop, iterate through the current `lists` with a step of 2.
    - Merge `lists[i]` and `lists[i+1]` (if `i+1` exists) using a standard two-list merge function.
    - Store the results in a new temporary slice and replace the original `lists` with it.
4.  **Two-List Merge**: Implement a helper function `merge(nodeA, nodeB)` that uses a dummy head to simplify the construction of the merged list, comparing values of the heads of two lists and appending the smaller one to the result.

### Go Code

``` go
func mergeKLists(lists []*ListNode) *ListNode {
    if len(lists) == 0 {
        return nil
    }
    for len(lists) > 1 {
        n := len(lists)
        newLists := make([]*ListNode, 0)
        for i := 0; i < n; i += 2 {
            var nodeA, nodeB *ListNode
            nodeA = lists[i]
            if i+1 < n {
                nodeB = lists[i+1]
            }
            mergedNode := merge(nodeA, nodeB)
            newLists = append(newLists, mergedNode)
        }
        lists = newLists
    }
    return lists[0]
}

func merge(nodeA, nodeB *ListNode) *ListNode {
    dummyHead := &ListNode{}
    curr := dummyHead
    for nodeA != nil && nodeB != nil {
        if nodeA.Val <= nodeB.Val {
            curr.Next = nodeA
            nodeA = nodeA.Next
        } else {
            curr.Next = nodeB
            nodeB = nodeB.Next
        }
        curr = curr.Next
    }
    if nodeA != nil {
       curr.Next = nodeA
    } 
    if nodeB != nil {
        curr.Next = nodeB
    }
    return dummyHead.Next
}
```

### Code Efficiency

- **Time Complexity**: $O(N \log k)$
    - Let $k$ be the number of lists and $N$ be the total number of nodes across all lists.
    - There are $\log k$ levels of merging (since we halve the number of lists each time).
    - At each level, every node is processed exactly once during the merge operations, taking $O(N)$ time.
- **Space Complexity**: $O(k)$
    - We use a temporary slice `newLists` to store the merged lists at each level. The size of this slice starts at $k/2$ and decreases, leading to $O(k)$ auxiliary space. (The iterative approach for merging two lists uses $O(1)$ extra space).

---

## Optimized Solution: In-Place Merging ($O(1)$ Auxiliary Space)

### Thought Process

1.  **Iterative In-Place Merge**: This approach refines the Divide and Conquer strategy by performing the merge directly within the original `lists` slice, avoiding the creation of new slices in each iteration.
2.  **Step-based Reduction**: 
    - Use a `step` variable (starting at 1) to define the distance between two lists to be merged.
    - In each "round", iterate through the slice and merge `lists[i]` with `lists[i + step]`.
    - Store the merged result back into `lists[i]`.
    - Double the `step` size after each round until it exceeds the length of the slice.
3.  **Space Optimization**: By reusing the `lists` slice, we eliminate the $O(k)$ extra space required by the previous implementation to store temporary results.
4.  **Two-List Merge**: Uses the same efficient `merge` helper function to combine two sorted lists.

### Go Code

``` go
func mergeKLists(lists []*ListNode) *ListNode {
    if len(lists) == 0 {
        return nil
    }
    step := 1
    for step < len(lists) {
        for i := 0; i+step < len(lists); i += step * 2 {
            lists[i] = merge(lists[i], lists[i+step])
        }
        step *= 2
    }
    return lists[0]
}

func merge(nodeA, nodeB *ListNode) *ListNode {
    dummyHead := &ListNode{}
    curr := dummyHead
    for nodeA != nil && nodeB != nil {
        if nodeA.Val <= nodeB.Val {
            curr.Next = nodeA
            nodeA = nodeA.Next
        } else {
            curr.Next = nodeB
            nodeB = nodeB.Next
        }
        curr = curr.Next
    }
    if nodeA != nil {
       curr.Next = nodeA
    } 
    if nodeB != nil {
        curr.Next = nodeB
    }
    return dummyHead.Next
}
```

### Code Efficiency

- **Time Complexity**: $O(n \log k)$
    - $k$ is the number of lists, $n$ is the total number of nodes.
    - There are $\log k$ merge rounds. In each round, every node is processed at most once, leading to $O(n)$ work per round.
- **Space Complexity**: $O(1)$
    - Unlike the previous version, this uses the input slice `lists` for storage. No extra data structures proportional to $k$ or $N$ are used. (Note: The recursion depth is not an issue here as the merge is iterative).

---

## Solution 2: Min Heap

### Thought Process

1.  **Min-Heap Advantage**: A Min-Heap is perfect for this problem because it allows us to efficiently find the smallest element among the heads of $k$ sorted lists in $O(\log k)$ time.
2.  **Heap Setup**: Implement the `heap.Interface` from Go's `container/heap` package to create a Min-Heap of `*ListNode`. The priority is based on the node's `Val`.
3.  **Initialization**: 
    - Initialize an empty heap.
    - Traverse the input `lists` slice and push the head node of every non-empty list into the heap.
4.  **Merge Loop**:
    - Use a `dummy` node and a `curr` pointer to build the result list.
    - While the heap is not empty:
        - `heap.Pop` the smallest node (`minNode`).
        - Attach `minNode` to `curr.Next`.
        - Advance `curr`.
        - If `minNode.Next` is not `nil`, push the next node from the same list into the heap to keep the comparison pool complete.
5.  **Return**: Return `dummy.Next`.

### Go Code

``` go
import "container/heap"

type minHeap []*ListNode

func (h minHeap) Len() int { return len(h) }
func (h minHeap) Less(i, j int) bool { return h[i].Val < h[j].Val }
func (h minHeap) Swap(i, j int) { h[i], h[j] = h[j], h[i] }
func (h *minHeap) Push(x interface{}) { *h = append(*h, x.(*ListNode)) }
func (h *minHeap) Pop() interface{} {
    last := (*h)[len(*h)-1]
    *h = (*h)[:len(*h)-1]
    return last
}

func mergeKLists(lists []*ListNode) *ListNode {
    h := &minHeap{}
    heap.Init(h)

    for _, node := range lists {
        if node != nil {
            heap.Push(h, node)
        }
    }

    dummy := &ListNode{}
    curr := dummy
    for h.Len() > 0 {
        minNode := heap.Pop(h).(*ListNode)
        curr.Next = minNode
        curr = curr.Next

        if minNode.Next != nil {
            heap.Push(h, minNode.Next)
        }
    }
    return dummy.Next
}
```

### Code Efficiency

- **Time Complexity**: $O(N \log k)$
    - $N$ is the total number of nodes across all lists, and $k$ is the number of lists.
    - We perform $N$ `Push` and $N$ `Pop` operations.
    - Each heap operation takes $O(\log k)$ since the heap contains at most $k$ elements (one from each list).
- **Space Complexity**: $O(k)$
    - The heap stores at most $k$ nodes at any given time.
