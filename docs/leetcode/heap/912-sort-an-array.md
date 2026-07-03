# 912. Sort an Array

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/sort-an-array/description/)

## Solution: Heap Sort (In-Place Max-Heap)

**Heap Sort** is an in-place, comparison-based sorting algorithm. It utilizes a binary heap data structure to sort elements. The algorithm is divided into two main phases:
1.  **Build Max-Heap**: Rearrange the array into a max-heap so that the largest element is at the root (`nums[0]`).
2.  **Sort**: Repeatedly swap the root element (maximum value) with the last element of the active heap, shrink the heap size, and restore the max-heap property by sifting the new root down.

### Thought Process

1.  **Phase 1: Build Max-Heap (Heapify)**:
    *   To build the heap in-place, we start from the last non-leaf node and sift it down.
    *   For an array of size $n$, the last leaf node is at index $n - 1$. Its parent (the last non-leaf node) is at index $\lfloor n / 2 - 1 \rfloor$.
    *   Iterate from index $i = n/2 - 1$ down to $0$, calling `siftDown(nums, i, n)`.
    *   This bottom-up construction takes $O(n)$ time.
2.  **Phase 2: Sort**:
    *   Iterate from the last index $i = n - 1$ down to $1$:
        *   Swap the maximum element (`nums[0]`) with the last unsorted element (`nums[i]`). The element at $i$ is now in its correct sorted position.
        *   Reduce the active heap size to $i$.
        *   Restore the heap property by sifting the new root element down: `siftDown(nums, 0, i)`.
3.  **Sift Down Operation (`siftDown`)**:
    *   For a node at index `root` within a heap of size `heapSize`:
        *   Locate children: `leftChild = 2*root + 1` and `rightChild = 2*root + 2`.
        *   Find the largest value among `nums[root]`, `nums[leftChild]`, and `nums[rightChild]`.
        *   If the largest is the `root` itself, the heap property holds $\rightarrow$ break.
        *   Otherwise, swap the values at `root` and `largest`, set `root = largest`, and repeat the process down the subtree.

### Go Code

``` go
func sortArray(nums []int) []int {
    n := len(nums)
    for i := n/2-1; i >= 0; i-- {
        siftDown(nums, i, n)
    }

    for i := n-1; i > 0; i-- {
        nums[0], nums[i] = nums[i], nums[0]
        siftDown(nums, 0, i)
    }
    return nums
}

func siftDown(nums []int, root int, heapSize int) {
    for {
        leftChild := 2*root + 1
        rightChild := 2*root + 2
        largest := root

        if leftChild < heapSize && nums[leftChild] > nums[largest] {
            largest = leftChild
        }

        if rightChild < heapSize && nums[rightChild] > nums[largest] {
            largest = rightChild
        }
        if largest == root {
            break
        }

        nums[largest], nums[root] = nums[root], nums[largest]
        root = largest
    }
}
```

### Code Efficiency

- **Time Complexity**: $O(n \log n)$
    - Building the heap takes $O(n)$ time. The sorting loop performs $n - 1$ iterations, each containing a sift-down operation that takes at most $O(\log n)$ time. Therefore, the overall runtime is $O(n \log n)$ in the best, average, and worst cases.
- **Space Complexity**: $O(1)$
    - The sorting is performed entirely in-place inside the input array, requiring constant auxiliary space.