# 374. Guess Number Higher or Lower

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/guess-number-higher-or-lower/description/)

## Solution: Binary Search on Value Range

We are playing a guessing game to find a pre-picked number between $1$ and $n$. We query a helper API `guess(num)` which returns:
- `0`: Your guess is correct (we found the number).
- `-1`: Your guess is higher than the picked number (meaning the target is smaller).
- `1`: Your guess is lower than the picked number (meaning the target is larger).

Since the search space is a contiguous range of sorted integers, we can locate the correct number in $O(\log n)$ time using binary search.

### Thought Process

1.  **Define Search Boundaries**:
    *   The picked number is in the closed interval $[1, n]$.
    *   Set `l = 1` and `r = n`.
2.  **Binary Search Loop (`l <= r`)**:
    *   Calculate the midpoint guess: `m = l + (r - l) / 2`.
    *   Call `guess(m)`:
        *   **If $\text{guess}(m) == 0$**: The guess is correct. Return `m` immediately.
        *   **If $\text{guess}(m) == -1$**: The guess is too high. The picked number is smaller. Search the left half: `r = m - 1`.
        *   **If $\text{guess}(m) == 1$**: The guess is too low. The picked number is larger. Search the right half: `l = m + 1`.
3.  **Termination**:
    *   A picked number is guaranteed to exist, so the loop will always return from the `guess(m) == 0` case.

### Go Code

``` go
/** 
 * Forward declaration of guess API.
 * @param  num   your guess
 * @return 	     -1 if num is higher than the picked number
 *			      1 if num is lower than the picked number
 *               otherwise return 0
 * func guess(num int) int;
 */

func guessNumber(n int) int {
    l, r := 1, n
    for l <= r {
        m := l + (r-l)/2
        res := guess(m)
        if res == 0 {
            return m
        }
        if res < 0 {
            r = m-1
        } else {
            l = m+1
        }
    }
    return -1
}
```

### Code Efficiency

- **Time Complexity**: $O(\log n)$
    - The search interval is halved at each iteration, performing at most $\log_2 n$ queries to the `guess` API.
- **Space Complexity**: $O(1)$
    - The algorithm runs using constant auxiliary memory.