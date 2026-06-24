# 202. Happy Number

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/happy-number/description/)

## Solution: Hash Set Cycle Detection (DFS)

A number is a happy number if the sum of the squares of its digits eventually equals 1. If the sequence loops endlessly in a cycle that does not include 1, it is not a happy number. We can detect if a cycle exists by using a hash set to store the numbers we have already visited.

### Thought Process

1.  **Digit Squaring**: For any current number `curr`, we calculate the sum of the squares of its digits by iteratively extracting the last digit (`curr % 10`), squaring it, and dividing `curr` by 10.
2.  **Cycle Tracking**:
    - Maintain a map `visited` acting as a hash set.
    - If `curr` is already in `visited`, we have encountered a cycle. Return `false` to prevent infinite recursion.
    - Otherwise, mark `curr` as visited.
3.  **Recursive Transition**:
    - Compute the next sum. If the next sum equals `1`, return `true`.
    - Otherwise, recursively evaluate the next sum.

### Go Code

``` go
func isHappy(n int) bool {
    visited := make(map[int]bool)
    var isHappyNumber func(n int) bool
    isHappyNumber = func(curr int) bool {
        if visited[curr] {
            return false
        }
        visited[curr] = true
        next := 0
        for curr > 0 {
            digit := curr % 10
            curr = curr / 10
            next += digit * digit
        }
        if next == 1 {
            return true
        }
        return isHappyNumber(next)
    }
    return isHappyNumber(n)
}
```

### Code Efficiency

- **Time Complexity**: $O(\log n)$
    - Extracting the digits of a number $n$ takes $O(\log_{10} n)$ operations. The length of the sequence is bounded (any number above 243 will strictly decrease, and numbers below 243 will either reach 1 or fall into a cycle of very small length). Thus, the number of steps is bounded, yielding $O(\log n)$ overall time complexity.
- **Space Complexity**: $O(\log n)$
    - In the worst case, we store the visited numbers in a hash set and consume space on the call stack proportional to the length of the sequence.

---

## Solution: Floyd's Cycle Finding Algorithm (Slow and Fast Pointers)

Instead of using a hash set that consumes auxiliary space, we can model the sequence of numbers as a linked list where each number's "next" pointer is the sum of the squares of its digits. We can then apply Floyd's Cycle Detection Algorithm (Tortoise and Hare) to determine if a cycle exists in $O(1)$ space.

### Thought Process

1.  **Two Pointers**:
    - Initialize `slow` to `n`.
    - Initialize `fast` to the next state of `n` (`getNext(n)`).
2.  **Cycle Walk**:
    - In each iteration, move `slow` forward by 1 step (`getNext(slow)`) and `fast` forward by 2 steps (`getNext(getNext(fast))`).
    - If there is a cycle, the `fast` pointer will eventually catch up to the `slow` pointer, and `slow == fast`.
    - If the number is happy, the `fast` pointer will reach `1` and stop.
3.  **Termination**:
    - The loop terminates when `fast == 1` or `slow == fast`.
    - If `fast == 1`, return `true`; otherwise, return `false`.

### Go Code

``` go
func isHappy(n int) bool {
    slow := n
    fast := getNext(n)

    for fast != 1 && slow != fast {
        slow = getNext(slow)
        fast = getNext(getNext(fast))
    }
    return fast == 1
}

func getNext(num int) int {
    sum := 0
    for num > 0 {
        digit := num % 10
        num /= 10
        sum += digit * digit
    }
    return sum
}
```

### Code Efficiency

- **Time Complexity**: $O(\log n)$
    - Similar to the hash set solution, the number of steps taken by the pointers to either meet or reach 1 is bounded, and each step takes $O(\log n)$ time to sum the squared digits.
- **Space Complexity**: $O(1)$
    - We only track states using two integer variables (`slow` and `fast`), requiring constant auxiliary space.