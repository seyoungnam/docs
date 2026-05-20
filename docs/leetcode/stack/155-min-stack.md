# 155. Min Stack

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/min-stack/description/)

## Solution: Dual Stacks

To support `getMin` in constant time, we maintain a secondary stack that tracks the minimum value corresponding to each state of the primary stack.

### Thought Process

1.  **Primary Stack**: A standard stack (`stack`) to store all elements in the order they are pushed.
2.  **Minimum Tracking**: A secondary stack (`minStack`) to store the current minimum.
3.  **Push Logic**: Always push to `stack`. Only push to `minStack` if the value is $\le$ the current minimum (or if `minStack` is empty).
4.  **Pop Logic**: Always pop from `stack`. If the popped value matches the top of `minStack`, pop from `minStack` as well to restore the previous minimum.
5.  **Retrieval**: `Top()` and `GetMin()` simply return the last elements of their respective stacks.

### Go Code

``` go
type MinStack struct {
    stack    []int
    minStack []int // Only tracks when a new minimum (or duplicate minimum) is introduced
}

func Constructor() MinStack {
    return MinStack{
        stack:    make([]int, 0),
        minStack: make([]int, 0),
    }
}

func (this *MinStack) Push(val int) {
    this.stack = append(this.stack, val)
    
    // Only record the value if it's the first element OR smaller/equal to the current minimum
    if len(this.minStack) == 0 || val <= this.minStack[len(this.minStack)-1] {
        this.minStack = append(this.minStack, val)
    }
}

func (this *MinStack) Pop() {
    if len(this.stack) == 0 {
        return
    }
    
    topVal := this.stack[len(this.stack)-1]
    this.stack = this.stack[:len(this.stack)-1]
    
    // If the value we just removed was our minimum, pop it from the minStack too
    if topVal == this.minStack[len(this.minStack)-1] {
        this.minStack = this.minStack[:len(this.minStack)-1]
    }
}

func (this *MinStack) Top() int {
    return this.stack[len(this.stack)-1]
}

func (this *MinStack) GetMin() int {
    return this.minStack[len(this.minStack)-1]
}
```

### Code Efficiency

- **Time Complexity**: $O(1)$ for all operations
    - `Push`, `Pop`, `Top`, and `GetMin` all perform constant-time slice operations.
- **Space Complexity**: $O(n)$
    - We use two stacks. In the worst case (a strictly decreasing sequence), both stacks will store all $n$ elements.
