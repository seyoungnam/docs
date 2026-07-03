# 169. Majority Element

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/majority-element/description/)

## Solution: Boyer-Moore Voting Algorithm

The **Boyer-Moore Voting Algorithm** is an elegant algorithm designed to find the majority element in a sequence. A majority element is defined as an element that appears more than $\lfloor n / 2 \rfloor$ times. Boyer-Moore works in linear time and requires only constant auxiliary space, making it the optimal approach.

### Thought Process

1.  **Core Intuition**:
    *   Think of this algorithm as a voting match where the majority element matches against all other elements combined. 
    *   Since the majority element appears more than half of the time, even if all other elements team up to cancel out the majority element's votes on a 1-to-1 basis, the majority element will still have at least one vote remaining.
2.  **Algorithm Variables**:
    *   `candidate`: The element currently tracking as the potential majority element.
    *   `count`: The net score of the current `candidate`.
3.  **Iteration Steps**:
    *   For each element `n` in `nums`:
        *   If `count == 0`, the previous candidate was completely cancelled out. We set `candidate = n` and reset `count = 1`.
        *   If `n == candidate`, we increment `count++` (we found another vote for the candidate).
        *   If `n != candidate`, we decrement `count--` (this element cancels out one vote for the candidate).
4.  **Result**:
    *   Because a majority element is guaranteed to exist in this problem, the element left in `candidate` at the end of the single pass is guaranteed to be the majority element.

### Go Code

``` go
func majorityElement(nums []int) int {
    count := 0
    var candidate int
    for _, n := range nums {
        if count == 0 {
            candidate = n
            count++
            continue
        }
        if n == candidate {
            count++
        } else {
            count--
        }
    } 
    return candidate
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We iterate through the array of length $n$ exactly once.
- **Space Complexity**: $O(1)$
    - We only track two scalar variables (`count` and `candidate`), requiring no extra memory relative to the input size.