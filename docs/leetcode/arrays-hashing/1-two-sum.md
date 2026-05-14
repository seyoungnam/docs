# 1. Two Sum

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/two-sum/description/)

## Solution: One-Pass Hash Map

The goal is to find two indices in an array such that their values sum up to a specific target. A brute-force approach would take $O(n^2)$ time, but we can optimize this to $O(n)$ using a hash map to trade space for speed.

### Thought Process

1.  **Objective**: Find two numbers `a` and `b` such that `a + b = target`.
2.  **Logic**: For any number `num` in the array, we are looking for its complement `diff = target - num`.
3.  **Data Structure**: Use a `map` to store the numbers we have already seen and their corresponding indices (`value -> index`).
4.  **Single Pass**:
    - Iterate through the array `nums`.
    - At each step, calculate the required complement: `diff = target - num`.
    - Check if `diff` exists in our `visited` map.
    - **Match Found**: If it exists, we have found the two numbers. Return the stored index and the current index.
    - **No Match**: If it doesn't exist, store the current number and its index in the map so it can be used as a complement for future elements.
5.  **Completion**: Return an empty slice if no such pair is found (though the problem usually guarantees exactly one solution).

### Go Code

``` go
func twoSum(nums []int, target int) []int {
    visited := make(map[int]int)
    for i, num := range nums {
        diff := target - num
        if idx, exists := visited[diff]; exists {
            return []int{idx, i}
        } else {
            visited[num] = i
        }
    }
    return []int{}
}
```


### Code Efficiency

- **Time Complexity**: $O(n)$
    - We traverse the list containing $n$ elements exactly once. Each lookup in the hash map takes $O(1)$ time on average.
- **Space Complexity**: $O(n)$
    - In the worst case, we store $n$ elements in the hash map.


