# 1046. Last Stone Weight

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/last-stone-weight/description/)

You are given an array of integers `stones` where `stones[i]` is the weight of the $i$-th stone.

We are playing a game with the stones. On each turn, we choose the **heaviest two stones** and smash them together. Suppose the heaviest two stones have weights `x` and `y` with `x <= y`. The result of this smash is:
*   If `x == y`, both stones are destroyed.
*   If `x != y`, the stone of weight `x` is destroyed, and the stone of weight `y` has new weight `y - x`.

At the end of the game, there is **at most one** stone left.

Return *the weight of the last remaining stone*. If there are no stones left, return `0`.

---

## Solution: Max-Heap

To efficiently simulate the process of repeatedly retrieving the two heaviest stones, we can use a **Max-Heap**. In Go, we implement this using the `container/heap` package by reversing the comparison logic in the `Less` method.

### Thought Process

1.  **Max-Heap Setup**:
    *   Implement Go's `heap.Interface` on a slice of integers `maxHeap`. By defining `Less(i, j) bool` as `h[i] > h[j]`, the heap acts as a Max-Heap, placing the largest value at the root (index `0`).
2.  **Simulation Loop**:
    *   Initialize the heap and push all stone weights onto it.
    *   While the heap has more than 1 stone:
        *   Pop the heaviest stone `y := heap.Pop(h).(int)`.
        *   Pop the second heaviest stone `x := heap.Pop(h).(int)`. (Since `y` was popped first, `y >= x` is guaranteed).
        *   If the stones are not of equal weight (`x != y`), push the difference `y - x` back into the heap.
3.  **Result**:
    *   If exactly 1 stone is left, return its weight `(*h)[0]`.
    *   If no stones are left, return `0`.

### Go Code

``` go
import "container/heap"

type maxHeap []int

func (h maxHeap) Len() int           { return len(h) }
func (h maxHeap) Less(i, j int) bool { return h[i] > h[j] }
func (h maxHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *maxHeap) Push(x interface{}) { *h = append(*h, x.(int)) }
func (h *maxHeap) Pop() interface{} {
    last := (*h)[len(*h)-1]
    *h = (*h)[:len(*h)-1]
    return last
}

func lastStoneWeight(stones []int) int {
    h := &maxHeap{}
    heap.Init(h)
    for _, s := range stones {
        heap.Push(h, s)
    }
    
    // Simulate smashing the two heaviest stones
    for h.Len() > 1 {
        y := heap.Pop(h).(int)
        x := heap.Pop(h).(int)
        if x != y {
            heap.Push(h, y-x)
        }
    }
    
    if h.Len() == 1 {
        return (*h)[0]
    }
    return 0
}
```

### Code Efficiency

- **Time Complexity**: $O(N \log N)$
    - Where $N$ is the number of stones. Initializing the heap by pushing $N$ elements takes $O(N \log N)$ time. Each smash step does two `Pop` operations and at most one `Push` operation, which takes $O(\log N)$ time. Since each smash reduces the number of stones by at least 1, the loop runs at most $N-1$ times, taking $O(N \log N)$ total time.
- **Space Complexity**: $O(N)$
    - The heap stores up to $N$ elements, requiring $O(N)$ auxiliary space.