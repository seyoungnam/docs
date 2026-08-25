# 621. Task Scheduler

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/task-scheduler/description/)

Given a characters array `tasks`, representing the tasks a CPU needs to do, where each character represents a different task. Tasks could be done in any order. Each task is done in one unit of time. For each unit of time, the CPU could complete either one task or just be idle.

However, there is a non-negative integer `n` that represents the cooldown period between two of the **same tasks** (the same letter in the array), that is that there must be at least `n` units of time between any two same tasks.

Return *the minimum number of units of time that the CPU will take to finish all the given tasks*.

---

## Solution: Max-Heap with Cooldown Queue

To minimize idle time, we should prioritize executing the task with the **highest remaining frequency** at any given moment. We can implement this greedy strategy using a **Max-Heap** to store active task frequencies and a **Queue** to track tasks that are cooling down.

### Thought Process

1.  **Count Frequencies**:
    *   Count the occurrences of each task using a hash map `count := map[byte]int{}`.
2.  **Max-Heap Setup**:
    *   Push the frequencies of each unique task onto a max-heap. Since tasks are uppercase English letters, the heap size is bounded by a maximum of 26 elements.
3.  **Cooldown Queue**:
    *   Use a queue `q` to store tasks undergoing cooldown. Each entry in the queue is a pair `[remainingCount, readyTime]`, where `readyTime` is the timestamp at which the task can be scheduled again.
4.  **CPU Simulation Loop**:
    *   Initialize `time := 0`.
    *   While the heap is not empty or the queue contains cooling tasks:
        *   Increment `time`.
        *   **Fast-Forward Idle Time**: If the heap is empty, the CPU must idle. To avoid stepping through time incrementally, we fast-forward `time` directly to the `readyTime` of the first task in the queue: `time = q[0][1]`.
        *   **Execute Task**: If the heap has active tasks:
            *   Pop the task with the highest remaining frequency: `cnt := heap.Pop(h).(int) - 1`.
            *   If the task still has runs remaining (`cnt > 0`), append it to the cooldown queue with a ready time of `time + n`.
        *   **Re-queue Ready Tasks**: If the task at the front of the queue has finished cooling down (`q[0][1] == time`), pop it from the queue and push its remaining count back onto the heap.
5.  **Return**:
    *   Return `time`.

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

func leastInterval(tasks []byte, n int) int {
    // Count frequencies of each task
    count := map[byte]int{}
    for _, t := range tasks {
        count[t]++
    }

    // Populate max-heap
    h := &maxHeap{}
    for _, cnt := range count {
        *h = append(*h, cnt)
    }
    heap.Init(h)
    
    time := 0
    // Cooldown queue storing pairs of [remainingCount, readyTime]
    q := make([][2]int, 0)

    for h.Len() > 0 || len(q) > 0 {
        time++
        
        if h.Len() == 0 {
            // Fast-forward time to the next ready task's cooldown time
            time = q[0][1]
        } else {
            cnt := heap.Pop(h).(int) - 1
            if cnt > 0 {
                q = append(q, [2]int{cnt, time + n})
            }
        }

        // Check if the front task of the queue has finished its cooldown
        if len(q) > 0 && q[0][1] == time {
            heap.Push(h, q[0][0])
            q = q[1:]
        }
    }
    
    return time
}
```

### Code Efficiency

- **Time Complexity**: $O(T)$
    - Where $T$ is the number of task items in `tasks`. Counting the task frequencies takes $O(T)$ time. Since the alphabet size is fixed (at most 26 unique tasks), the heap and queue operations take $O(\log 26) = O(1)$ constant time. Thus, the simulation loop processes the task elements in linear time.
- **Space Complexity**: $O(1)$
    - The space required for the frequency map, max-heap, and cooldown queue is at most $O(26) = O(1)$ auxiliary space, since there are at most 26 unique English uppercase letters.