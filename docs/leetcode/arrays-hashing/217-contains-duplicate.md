# 217. Contains Duplicate

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/contains-duplicate/description/)

## Solution: Hash Set (Hash Map with Empty Struct)

The most efficient way to check for duplicates is to track the elements we've already encountered as we iterate through the array. A Hash Set (implemented as a `map` in Go) provides $O(1)$ average-time complexity for both insertions and lookups.

### Thought Process

1.  **Requirement**: Determine if any value appears at least twice in the array.
2.  **Data Structure**: Use a `map` to store the numbers we've seen. 
    - In Go, using `map[int]struct{}` is a memory-efficient way to implement a set, as an empty `struct{}` occupies zero additional bytes.
3.  **Iteration**:
    - Iterate through the input array `nums`.
    - For each number, check if it already exists in the `seen` map.
    - **Found Duplicate**: If the number is in the map, return `true` immediately (early exit).
    - **New Element**: If the number is not in the map, add it and continue.
4.  **Completion**: If the loop finishes without finding a duplicate, return `false`.
5.  **Optimization**: Pre-allocating the map's capacity with `make(map[int]struct{}, len(nums))` can reduce reallocations if the array contains no duplicates.

### Go Code

``` go
func containsDuplicate(nums []int) bool {
    seen := make(map[int]struct{}, len(nums))
    for _, num := range nums {
        if _, exists := seen[num]; exists {
            return true
        }
        seen[num] = struct{}{}
    }
    return false
}
```


### Code Efficiency

- **Time Complexity**: $O(n)$
    - We iterate through the array once. Each map lookup and insertion takes $O(1)$ on average.
- **Space Complexity**: $O(n)$
    - In the worst case (no duplicates), we store all $n$ elements in the map.

