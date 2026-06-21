# 341. Flatten Nested List Iterator

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/flatten-nested-list-iterator/description/)

## Solution: Stack-Based Iterator (Lazy Flattening)

To implement the iterator, we can use a stack containing pointers to `NestedInteger` elements. By pushing elements onto the stack in reverse order, we ensure that the next element to process is always at the top of the stack. Flattening is done lazily in `HasNext` to avoid unnecessary work if the iterator is not fully consumed.

### Thought Process

1.  **Stack Initialization**: Push all the elements of the initial `nestedList` onto the stack in reverse order (from right to left) so that the first element of the list is at the top of the stack.
2.  **Lazy Flattening in `HasNext`**:
    - Loop while the stack is not empty.
    - Inspect the top element of the stack.
    - If the top element is an integer, return `true` immediately because the iterator has a next integer ready.
    - If the top element is a nested list, pop it from the stack, retrieve its list contents, and push them onto the stack in reverse order.
    - If the loop finishes and the stack is empty, return `false`.
3.  **Value Retrieval in `Next`**: Since `HasNext` guarantees that the top element of the stack is an integer, `Next` simply pops the top element and returns its integer value using `GetInteger()`.

### Go Code

``` go
type NestedIterator struct {
    stack   []*NestedInteger
}

func Constructor(nestedList []*NestedInteger) *NestedIterator {
    stack := make([]*NestedInteger, 0, len(nestedList))
    for i := len(nestedList)-1; i >= 0; i-- {
        stack = append(stack, nestedList[i])
    }
    return &NestedIterator{stack: stack}
}

func (this *NestedIterator) Next() int {
    topIdx := len(this.stack) - 1
    curr := this.stack[topIdx]
    this.stack = this.stack[:topIdx]
    return curr.GetInteger()
}

func (this *NestedIterator) HasNext() bool {
    for len(this.stack) > 0 {
        topIdx := len(this.stack) - 1
        top := this.stack[topIdx]

        if top.IsInteger() {
            return true
        }

        this.stack = this.stack[:topIdx]
        subList := top.GetList()
        for i := len(subList)-1; i >= 0; i-- {
            this.stack = append(this.stack, subList[i])
        }
    }
    return false
}
```

### Code Efficiency

- **Time Complexity**:
    - `Constructor`: $O(n)$ where $n$ is the number of elements in the initial list.
    - `Next`: $O(1)$ time, as it simply pops the pre-validated integer from the top of the stack.
    - `HasNext`: $O(1)$ amortized time. Although a single call to `HasNext` might unpack multiple nested lists, each element (nested list or integer) is pushed onto and popped from the stack at most once.
- **Space Complexity**: $O(d + k)$ where $d$ is the maximum depth of nesting, and $k$ is the number of elements across all nested lists. In the worst case, the stack holds all elements along the path to the current integer.