# 973. K Closest Points to Origin

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/k-closest-points-to-origin/description/)

## Solution: Heap

### Thought Process

1.  **Distance Calculation**: Use the squared Euclidean distance $x^2 + y^2$ to compare points. This avoids unnecessary square root operations.
2.  **Heap Selection**: Implement a Min-Heap using the `container/heap` package in Go. The heap will store the points, with the "smallest" element being the one with the minimum distance.
3.  **Heap Initialization**: Use `heap.Init` to organize the points into a valid heap structure. This is an $O(n)$ operation.
4.  **Extraction**: Pop the top element from the heap $k$ times to collect the $k$ closest points.

### Go Code

``` go
import "container/heap"

type minHeap [][]int

func (h minHeap) Len() int { return len(h) }
func (h minHeap) Less(i, j int) bool { 
    distA := h[i][0]*h[i][0] + h[i][1]*h[i][1]
    distB := h[j][0]*h[j][0] + h[j][1]*h[j][1]
    return distA < distB
}
func (h minHeap) Swap(i, j int) { h[i], h[j] = h[j], h[i] }
func (h *minHeap) Push(x interface{}) { *h = append(*h, x.([]int)) }
func (h *minHeap) Pop() interface{} {
    last := (*h)[len(*h)-1]
    *h = (*h)[:len(*h)-1]
    return last
}

func kClosest(points [][]int, k int) [][]int {
    h := minHeap(points)
    heap.Init(&h)
    res := [][]int{}
    for range k {
        res = append(res, heap.Pop(&h).([]int))
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n + k \log n)$
    - `heap.Init` takes $O(n)$ time to build the heap from the input points.
    - Each of the $k$ `heap.Pop` operations takes $O(\log n)$ time to maintain the heap property.
- **Space Complexity**: $O(1)$ auxiliary space (excluding the result slice).
    - The heap is built directly on the input `points` slice using a type cast, so no extra space proportional to $n$ is used beyond the input itself.
    

---

## Optimized Solution: Quickselect

### Thought Process

1.  **Selection Algorithm**: Instead of sorting or using a heap, use Quickselect to find the $k$-th smallest distance. Quickselect is a selection algorithm that finds the $k$-th smallest element in an unordered list in $O(n)$ average time.
2.  **Partitioning**: Choose a pivot (the last element in the current range) and rearrange the points such that all points with a distance less than or equal to the pivot's distance are on the left.
3.  **Recursive Reduction**:
    - If the resulting pivot index is exactly $k$, then the first $k$ elements in the array are the $k$ closest points.
    - If the pivot index is less than $k$, we only need to search the right side of the partition.
    - If the pivot index is greater than $k$, we search the left side.
4.  **In-place Operations**: Perform the partitioning directly on the input `points` slice to minimize extra space.

### Go Code

``` go
func kClosest(points [][]int, k int) [][]int {
    quickSelect(points, 0, len(points)-1, k)
    return points[:k]
}

func dist(p []int) int {
    return p[0]*p[0] + p[1]*p[1]
}

func partition(points [][]int, l int, r int) int {
    pivotDist := dist(points[r])
    i := l
    for j := l; j < r; j++ {
        if dist(points[j]) <= pivotDist {
            points[i], points[j] = points[j], points[i]
            i++
        }
    }
    points[i], points[r] = points[r], points[i]
    return i
}

func quickSelect(points [][]int, l int, r int, k int) {
    if l >= r {
        return
    }
    pivotIdx:= partition(points, l, r)
    if pivotIdx == k {
        return
    } else if pivotIdx < k {
        quickSelect(points, pivotIdx+1, r, k)
    } else {
        quickSelect(points, l, pivotIdx-1, k)
    }
    return
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$ average
    - In each step of Quickselect, we partition the array and recurse into only one side. On average, this leads to $n + n/2 + n/4 + \dots = 2n$ operations, which is $O(n)$.
    - The worst-case time complexity is $O(n^2)$ if the pivot selection is poor, but the average case is highly efficient.
- **Space Complexity**: $O(\log n)$ average
    - The partitioning is done in-place, but the recursive calls use stack space. On average, the recursion depth is $O(\log n)$.

