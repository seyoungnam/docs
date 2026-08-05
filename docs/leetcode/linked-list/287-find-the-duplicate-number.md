# 287. Find the Duplicate Number

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/find-the-duplicate-number/description/)

## Solution 1: Floyd's Cycle-Finding Algorithm (Tortoise and Hare)

This is the optimal solution because it solves the problem in $O(n)$ time and $O(1)$ auxiliary space without modifying the input array.

### Thought Process

1.  **Linked List Cycle Analogy**:
    *   Consider the array indices as node addresses, and `nums[i]` as the pointer to the next node (`node.next`).
    *   Since the numbers are in the range $[1, n]$ and the array is of size $n+1$, there is at least one duplicate. This duplicate ensures that multiple indices point to the same value, creating a cycle in the list structure.
2.  **Phase 1: Finding the Intersection Point**:
    *   Initialize `slow = nums[0]` and `fast = nums[nums[0]]`.
    *   Move `slow` by 1 step (`slow = nums[slow]`) and `fast` by 2 steps (`fast = nums[nums[fast]]`) until they meet (`slow == fast`).
3.  **Phase 2: Locating the Cycle Entrance (Duplicate)**:
    *   Initialize a second slow pointer `slow2 = 0`.
    *   Move both `slow` and `slow2` at a speed of 1 step per iteration.
    *   The point where they meet is the entry point of the cycle, which is the duplicate number.

### Go Code

``` go
func findDuplicate(nums []int) int {
    slow, fast := nums[0], nums[nums[0]]
    for slow != fast {
        slow = nums[slow]
        fast = nums[fast]
        fast = nums[fast]
    }
    slow2 := 0
    for slow != slow2 {
        slow = nums[slow]
        slow2 = nums[slow2]
    }
    return slow
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - The search loops run in linear time proportional to the length of the cycle and distance to its entrance.
- **Space Complexity**: $O(1)$
    - Only a few pointer variables are used, and the input array is not modified.

---

## Solution 2: Cycle Sort (Array Mutation)

If we are allowed to modify the array, we can use a cycle-sorting approach to place each number at its corresponding index.

### Thought Process

1.  **Cycle Sort Placement**:
    *   Ideally, each number $x$ in `nums` should be placed at index $x - 1$ (since numbers are in the range $[1, n]$).
2.  **In-place Swapping**:
    *   Iterate through the array using a pointer `i`.
    *   At each step, calculate the target index: `targetIdx = nums[i] - 1`.
    *   If `i == targetIdx`, the number is already in the correct place, so we increment `i`.
    *   If `i != targetIdx`:
        *   Check if the target position already contains the correct value (`nums[i] == nums[targetIdx]`). If so, we have found a duplicate. Return `nums[i]` immediately.
        *   Otherwise, swap `nums[i]` and `nums[targetIdx]` to place the element at its correct index, and repeat the check at `i` (without incrementing `i`).

### Go Code

``` go
func findDuplicate(nums []int) int {
    for i := 0; i < len(nums); {
        targetIdx := nums[i]-1
        if i != targetIdx {
            if nums[i] == nums[targetIdx] {
                return nums[i]
            }
            nums[i], nums[targetIdx] = nums[targetIdx], nums[i]
        } else {
            i++
        }
    }
    return -1
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - Although there is a nested loop behavior via swapping, each element is placed in its correct position at most once. The algorithm performs a maximum of $2n$ steps.
- **Space Complexity**: $O(1)$
    - Modifies the array in-place, requiring no extra memory.
- **Note**: This approach mutates the input array, which is forbidden under LeetCode's default problem constraints.