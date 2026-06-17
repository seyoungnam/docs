# 1167. Minimum Cost to Connect Sticks


[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/minimum-cost-to-connect-sticks/)

## Solution : Heap

### Thought Process

This problem can be modeled as a greedy optimization problem. 

#### 1. Greedy Strategy
To minimize the total cost of connecting all sticks, we should always **connect the two shortest sticks currently available**. 
- When we connect two sticks of lengths $A$ and $B$, the cost incurred is $A + B$. The new combined stick will then be merged with other sticks later.
- If we merge smaller sticks first, their lengths will contribute to the cumulative cost multiple times, while larger sticks will be merged later and contribute fewer times to the total sum.

#### 2. Min-Heap (Priority Queue) Implementation
To efficiently get and remove the two smallest elements at each step:
1. Initialize a **Min-Heap** with all the stick lengths.
2. In a loop, as long as there is more than one stick in the heap:
   - Pop the two smallest sticks, `a` and `b`.
   - Connect them together: `sum = a + b`.
   - Add the connection cost (`sum`) to the total accumulated cost.
   - Push the new combined stick (`sum`) back into the min-heap.
3. Stop when only one stick remains in the heap, and return the accumulated cost.

### Go Code

``` go
import "container/heap"

type minHeap []int

func (h minHeap) Len() int { return len(h) }
func (h minHeap) Less(i, j int) bool { return h[i] < h[j] }
func (h minHeap) Swap(i, j int) { h[i], h[j] = h[j], h[i] }
func (h *minHeap) Push(x interface{}) { *h = append(*h, x.(int)) }
func (h *minHeap) Pop() interface{} {
    last := (*h)[len(*h)-1]
    *h = (*h)[:len(*h)-1]
    return last
}

func connectSticks(sticks []int) int {
    res := 0
    if len(sticks) <= 1 {
        return res
    }
    h := minHeap(sticks)
    heap.Init(&h)
    for h.Len() > 1 {
        a := heap.Pop(&h).(int)
        b := heap.Pop(&h).(int)
        sum := a+b
        res += sum
        heap.Push(&h, sum)
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $\mathcal{O}(N \log N)$, where $N$ is the number of sticks.
    - Initializing the min-heap takes $\mathcal{O}(N)$ time.
    - We perform $N - 1$ merge iterations. In each iteration, we perform two `Pop` operations and one `Push` operation, each taking $\mathcal{O}(\log N)$ time.
- **Space Complexity**: $\mathcal{O}(N)$
    - The heap requires $\mathcal{O}(N)$ auxiliary space to store the sticks.
