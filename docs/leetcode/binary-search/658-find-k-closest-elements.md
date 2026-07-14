# 658. Find K Closest Elements

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/find-k-closest-elements/description/)

## Solution: Binary Search on Start Index of the Window

Instead of searching for the closest element and expanding outwards using two pointers, we can binary search to find the **starting index** of our result window. Since the result is a contiguous subarray of size $k$, the starting index of this window is bounded in the range $[0, \text{len}(arr) - k]$.

### Thought Process

1.  **Representing the Window**:
    *   Let the window start at index `m`. The window of size $k$ spans from index `m` to `m + k - 1`.
    *   To decide if we should slide our window left or right, we compare the boundaries: the element `arr[m]` at the start of our current window, and the element `arr[m+k]` immediately to the right of the window.
2.  **Binary Search Criterion**:
    *   Compare the distance of the target `x` to both boundary elements:
        *   Distance to `arr[m]` is $x - \text{arr}[m]$ (since $arr$ is sorted, we can use simple difference checks).
        *   Distance to `arr[m+k]` is $\text{arr}[m+k] - x$.
    *   **Decision**:
        *   **If $x - \text{arr}[m] > \text{arr}[m+k] - x$**: The element `arr[m+k]` is strictly closer to `x` than `arr[m]`. This means the optimal window must start further to the right. Adjust left boundary: `l = m + 1`.
        *   **Otherwise ($x - \text{arr}[m] \le \text{arr}[m+k] - x$)**: The element `arr[m]` is closer or equally close to `x` than `arr[m+k]`. This means the window starting at `m` is better than or equal to a window starting further to the right. Adjust right boundary: `r = m`.
3.  **Termination**:
    *   The loop terminates when `l == r`. The index `l` represents the optimal starting position. We return the slice `arr[l : l+k]`.

### Go Code

``` go
func findClosestElements(arr []int, k int, x int) []int {
    l, r := 0, len(arr)-k
    for l < r {
        m := l + (r-l)/2
        if x-arr[m] > arr[m+k]-x {
            l = m+1
        } else {
            r = m
        }
    }
    return arr[l:l+k]
}
```

### Code Efficiency

- **Time Complexity**: $O(\log(N - K) + K)$
    - Where $N$ is the number of elements in `arr`. The binary search runs on a range of size $N - K$, taking $O(\log(N - K))$ time. Slicing and returning the result window of size $K$ takes $O(K)$ time.
- **Space Complexity**: $O(1)$ auxiliary space
    - The algorithm operates directly on the input array indices.