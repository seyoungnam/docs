# 1010. Pairs of Songs With Total Durations Divisible by 60

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/pairs-of-songs-with-total-durations-divisible-by-60/description/)

## Solution: Two-Pass Hash Map (Frequency Counting)

We can solve this by counting the frequency of each song's duration modulo `60`. Any song with remainder `r` can form a valid pair with a song of remainder `60 - r`. 

### Thought Process

1.  **Count Remainders**: Store the frequency of each remainder `t % 60` in a hash map.
2.  **Special Cases**: 
    *   Songs with remainder `0` can only pair with other songs of remainder `0`.
    *   Songs with remainder `30` can only pair with other songs of remainder `30`.
    *   For these, the number of pairs is given by the combination formula: `cnt * (cnt - 1) / 2`.
3.  **General Case**: For any other remainder `r`, the number of pairs it forms is `count[r] * count[60 - r]`.
4.  **Avoid Double Counting**: Maintain a `visited` set to ensure each remainder pair is counted exactly once.

### Go Code

``` go
func numPairsDivisibleBy60(time []int) int {
    res := 0
    count := make(map[int]int)
    for _, t := range time {
        count[t%60]++
    }
    
    visited := make(map[int]struct{})
    for key1, cnt1 := range count {
        if _, exists := visited[key1]; exists {
            continue
        }
        
        // Songs with 0 or 30 remainders pair among themselves
        if key1 == 0 || key1 == 30 {
            res += cnt1 * (cnt1 - 1) / 2
            visited[key1] = struct{}{}
            continue
        }

        key2 := 60 - key1
        if _, exists := visited[key2]; exists {
            continue
        }
        res += cnt1 * count[key2]
        visited[key1] = struct{}{}
        visited[key2] = struct{}{}
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We perform one pass to count remainders and a second pass bounded by $O(1)$ size (at most 60 keys).
- **Space Complexity**: $O(1)$
    - The number of unique remainders is strictly bounded by 60, so the hash maps take $O(1)$ auxiliary space.

---

## Optimized Solution: One-Pass (Fixed-Size Array)

We can optimize this to a single pass using a stack-allocated fixed-size array of size `60` instead of heap-allocated hash maps, completely eliminating the need for `visited` sets and division combination math.

### Thought Process

1.  **Fixed Array**: Initialize an array `counts` of size `60` to store the remainder frequencies.
2.  **Modulo Complement**: For each song duration `t`:
    *   Calculate the remainder: `rem = t % 60`.
    *   The required remainder to form a multiple of 60 is: `target = (60 - rem) % 60`.
3.  **On-the-Fly Pairing**: Add the frequency of `target` currently stored in our array to `res`, then increment `counts[rem]`. This automatically avoids double-counting since we only pair with previously seen songs.

### Go Code

``` go
func numPairsDivisibleBy60(time []int) int {
    counts := [60]int{}
    res := 0
    
    for _, t := range time {
        rem := t % 60
        target := (60 - rem) % 60
        
        // Add all compatible songs seen so far
        res += counts[target]
        counts[rem]++
    }
    
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We perform a single, extremely fast pass over the song times.
- **Space Complexity**: $O(1)$
    - We use a stack-allocated array of size 60, requiring $O(1)$ auxiliary space with zero heap allocation overhead.
