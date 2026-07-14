# 1095. Find in Mountain Array

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/find-in-mountain-array/description/)

## Solution: Triple Binary Search

We are given a mountain array and want to find the minimum index containing `target` using a limited number of API queries (at most 100 calls to `MountainArray.get(index)`). To achieve this in $O(\log n)$ time, we use a three-step binary search approach.

### Thought Process

1.  **Step 1: Locate the Peak**:
    *   A mountain array rises to a peak and then falls. Finding this peak allows us to split the array into two monotonic halves: one strictly ascending, and one strictly descending.
    *   Perform binary search on the slope:
        *   If `mountainArr.get(m) >= mountainArr.get(m+1)`, we are on a descending slope; the peak lies at `m` or to the left (`r = m`).
        *   Otherwise, we are on an ascending slope; the peak lies to the right (`l = m + 1`).
    *   Store the peak index: `peak = l`.
2.  **Step 2: Binary Search the Left (Ascending) Slope**:
    *   Since we want to find the *minimum* index where target exists, we search the ascending left side (`[0, peak]`) first.
    *   Perform a standard binary search. If found, return the index immediately.
3.  **Step 3: Binary Search the Right (Descending) Slope**:
    *   If the target is not on the left slope, search the descending right side (`[peak, n-1]`).
    *   Since this section is sorted in descending order, modify the boundary updates:
        *   If `val > target`, search to the right: `l = m + 1`.
        *   If `val < target`, search to the left: `r = m - 1`.
    *   If found, return the index. If not found in either half, return `-1`.

### Go Code

``` go
func findInMountainArray(target int, mountainArr *MountainArray) int {
    n := mountainArr.length()
    l, r := 0, n-1
    for l < r {
        m := l + (r-l)/2
        if mountainArr.get(m) >= mountainArr.get(m+1) {
            r = m
        } else {
            l = m+1
        }
    }
    peak := l
    l, r = 0, peak
    for l <= r {
        m := l + (r-l)/2
        val := mountainArr.get(m)
        if val == target {
            return m
        }
        if val > target {
            r = m-1
        } else {
            l = m+1
        }
    }
    l, r = peak, n-1
    for l <= r {
        m := l + (r-l)/2
        val := mountainArr.get(m)
        if val == target {
            return m
        }
        if val > target {
            l = m+1
        } else {
            r = m-1
        }
    }
    return -1
}
```

### Code Efficiency

- **Time Complexity**: $O(\log n)$
    - We perform three binary searches. Each binary search takes $O(\log n)$ time. For $N = 10,000$, we make approximately $3 \times \log_2(10000) \approx 42$ queries to `get()`, staying well within the 100-call constraint.
- **Space Complexity**: $O(1)$
    - We only allocate a few variables (`n`, `l`, `r`, `m`, `peak`, `val`), running in $O(1)$ auxiliary space.