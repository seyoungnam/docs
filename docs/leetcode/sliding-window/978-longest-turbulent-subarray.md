# 978. Longest Turbulent Subarray

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/longest-turbulent-subarray/description/)

## Solution 1: Sliding Window with Sign Comparison

A subarray is turbulent if the comparison sign between adjacent elements alternates (e.g. $< \ , \ > \ , \ < \ , \ >$). We can track this efficiently using a sliding window and a custom comparison function.

### Thought Process

1.  **Turbulence Comparison**:
    *   Let's define a helper function `compare(x, y)` which returns:
        *   `-1` if $x < y$
        *   `1` if $x > y$
        *   `0` if $x == y$
2.  **Window Boundaries**:
    *   Let the sliding window be defined by `[l, r]`. We initialize `l = 0` and the max size `res = 1`.
    *   Iterate through the array using a right pointer `r` starting from index `1`.
3.  **State Transitions**:
    *   At each step, calculate the comparison sign of the current transition: `c = compare(arr[r-1], arr[r])`.
    *   **Case 1 (Equal Elements)**: If `c == 0` (i.e., `arr[r-1] == arr[r]`), the turbulence condition is completely broken. We must reset the window to start at `r`:
        `l = r`
    *   **Case 2 (Valid Alternating Sign)**: If `r == 1` (first transition) or `c == -compare(arr[r-2], arr[r-1])` (the comparison sign toggled from the previous step), the window continues to grow.
    *   **Case 3 (Non-Alternating Sign)**: If the sign did not toggle (e.g., $arr[r-2] < arr[r-1] < arr[r]$), the turbulence is broken. However, the last transition $arr[r-1] \rightarrow arr[r]$ is still valid. Thus, we shrink the window from the left to start at the previous element:
        `l = r - 1`
4.  **Update Max Size**:
    *   At each position `r`, update the maximum size found: `res = max(res, r - l + 1)`.

### Go Code

``` go
func maxTurbulenceSize(arr []int) int {
    if len(arr) == 1 {
        return 1
    }
    l, res := 0, 1
    for r := 1; r < len(arr); r++ {
        c := compare(arr[r-1], arr[r])
        if c == 0 {
            l = r
        } else if r == 1 || c == -compare(arr[r-2], arr[r-1]) {
            // valid
        } else {
            l = r-1
        }
        res = max(res, r-l+1)
    }
    return res
}

func compare(x, y int) int {
    if x < y {
        return -1
    } else if x > y {
        return 1
    }
    return 0
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We make a single pass through the array.
- **Space Complexity**: $O(1)$
    - The algorithm runs in-place, using only constant auxiliary variables.

---

## Solution 2: Transition Counting (State Tracking)

Instead of maintaining window index pointers, we can count the number of consecutive alternating transitions (increasing or decreasing) while tracking the direction of the last valid transition.

### Thought Process

1.  **State Tracking Variables**:
    *   `cnt`: Tracks the count of consecutive valid alternating transitions.
    *   `sign`: Tracks the direction of the last transition:
        *   `-1`: Unset / no previous transition.
        *   `0`: The last transition was an **increase** ($arr[i-1] < arr[i]$).
        *   `1`: The last transition was a **decrease** ($arr[i-1] > arr[i]$).
2.  **State Transitions**:
    *   **Decreasing**: If $arr[i] > arr[i+1]$:
        *   If the previous transition was increasing (`sign == 0`), we extend the alternating run: `cnt++`.
        *   Otherwise (the previous transition was also decreasing, or unset), we start a new transition run: `cnt = 1`.
        *   Update `sign = 1`.
    *   **Increasing**: If $arr[i] < arr[i+1]$:
        *   If the previous transition was decreasing (`sign == 1`), we extend the alternating run: `cnt++`.
        *   Otherwise (the previous transition was also increasing, or unset), we start a new transition run: `cnt = 1`.
        *   Update `sign = 0`.
    *   **Equal Elements**: If $arr[i] == arr[i+1]$, the turbulence is broken. We reset both indicators: `cnt = 0` and `sign = -1`.
3.  **Global Maximum**:
    *   At each step, update the maximum count of valid transitions: `res = max(res, cnt)`.
4.  **Reconstruction**:
    *   The number of elements in a subarray is always the number of transitions $+ 1$. Thus, return `res + 1`.

### Go Code

``` go
func maxTurbulenceSize(arr []int) int {
    n := len(arr)
    // -1= unset, 0 = increase, 1 = decrease
    res, cnt, sign := 0, 0, -1
    for i := 0; i < n-1; i++ {
        if arr[i] > arr[i+1] {
            if sign == 0 {
                cnt++
            } else {
                cnt = 1
            }
            sign = 1
        } else if arr[i] < arr[i+1] {
            if sign == 1 {
                cnt++
            } else {
                cnt = 1
            }
            sign = 0
        } else {
            cnt = 0 
            sign = -1
        }
        res = max(res, cnt)
    }
    return res+1
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We perform a single pass through the array.
- **Space Complexity**: $O(1)$
    - The algorithm runs in-place, using only constant auxiliary variables.