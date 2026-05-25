# 1151. Minimum Swaps to Group All 1's Together

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/minimum-swaps-to-group-all-1s-together/description/)

## Solution: Fixed-Size Sliding Window

To group all `1`s together in the minimum number of swaps, they must occupy a contiguous block of size `totalOnes` (where `totalOnes` is the total number of `1`s in the array). The problem then reduces to finding a window of size `totalOnes` that already contains the **maximum number of `1`s**.

### Thought Process

1.  **Count Total 1s**: Scan the array to find the total count of `1`s. Let this count be `totalOnes`.
    *   *Edge Case*: If `totalOnes <= 1`, no swaps are needed; return `0`.
2.  **Initialize Window**: Calculate the number of `1`s in the first window of size `totalOnes` (indices `0` to `totalOnes-1`). Let this be `currOnes`.
3.  **Slide the Window**: Iterate from index `totalOnes` to the end of the array:
    *   Add the new element entering the window: `data[i]`.
    *   Remove the old element leaving the window: `data[i-totalOnes]`.
    *   Update `currOnes` and track the maximum `1`s seen in any window (`maxOnes`).
4.  **Result**: The minimum swaps required is the number of `0`s in the best window: `totalOnes - maxOnes`.

### Go Code

``` go
func minSwaps(data []int) int {
    n := len(data)
    
    // 1. Count total 1's in the array (this will be our window size)
    totalOnes := 0
    for _, val := range data {
        if val == 1 {
            totalOnes++
        }
    }
    
    // If there are no 1's or only one 1, no swaps are needed
    if totalOnes <= 1 {
        return 0
    }
    
    // 2. Count 1's in the first window of size totalOnes
    currOnes := 0
    for i := 0; i < totalOnes; i++ {
        if data[i] == 1 {
            currOnes++
        }
    }
    
    maxOnes := currOnes
    
    // 3. Slide the window of size totalOnes across the array
    for i := totalOnes; i < n; i++ {
        // Add new element entering the window
        if data[i] == 1 {
            currOnes++
        }
        // Remove old element leaving the window
        if data[i-totalOnes] == 1 {
            currOnes--
        }
        
        maxOnes = max(maxOnes, currOnes)
    }
    
    // 4. Minimum swaps = totalOnes minus the max 1's already grouped in any window
    return totalOnes - maxOnes
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We perform a single pass to count the `1`s and another pass to slide the window across the array.
- **Space Complexity**: $O(1)$
    - We only use a few integer variables, requiring $O(1)$ auxiliary space.
