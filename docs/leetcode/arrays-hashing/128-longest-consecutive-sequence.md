# 128. Longest Consecutive Sequence

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/longest-consecutive-sequence/description/)

## Solution: Hash Set (Sequence Start Detection)

Finding the longest consecutive sequence in $O(n)$ time requires a way to quickly check for the existence of elements. A Hash Set (implemented as a `map` in Go) allows for $O(1)$ lookups, which we can leverage to identify the start of each sequence and count its length.

### Thought Process

1.  **Requirement**: Find the length of the longest consecutive elements sequence in an unsorted array.
2.  **Data Structure**: Populate a Hash Set (`seen`) with all numbers from the input array. This removes duplicates and provides $O(1)$ lookup time.
3.  **Identifying Sequence Starts**: Iterate through the keys in the map. A number `key` is the **start** of a sequence if `key - 1` is not present in the map. This is a crucial optimization that prevents redundant counting of the same sequence multiple times.
4.  **Counting Length**: 
    - If a number is identified as a sequence start:
        - Use a loop to check for the presence of `key + 1`, `key + 2`, etc., in the map.
        - Keep track of the current sequence length (`curLen`).
5.  **Tracking Maximum**: Update the global maximum length (`res`) whenever a longer sequence is found.

### Go Code

``` go
func longestConsecutive(nums []int) int {
    seen := make(map[int]bool)
    for _, num := range nums {
        seen[num] = true
    }
    res := 0
    for key := range seen {
        // Only start counting if 'key' is the beginning of a sequence
        if !seen[key-1] {
            curLen := 0
            curr := key
            for seen[curr] {
                curLen++
                curr++
            }
            res = max(res, curLen)
        }
    }
    return res
}
```


### Code Efficiency

- **Time Complexity**: $O(n)$
    - We perform a single pass to build the hash set ($O(n)$).
    - We then iterate through the map. Although there is a nested loop, each number in the array is visited at most twice (once in the outer loop and at most once during a sequence count). This results in a linear $O(n)$ time complexity.
- **Space Complexity**: $O(n)$
    - We store all $n$ unique elements of the array in a hash set.




