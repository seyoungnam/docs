# 981. Time Based Key-Value Store

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/time-based-key-value-store/description/)

## Solution: Hash Map and Custom Binary Search

To implement a time-based key-value store, we map each key to a slice of value-timestamp pairs. Since the problem guarantees that `timestamp` values in `set` calls are strictly increasing, the slice for any key is **already sorted**. This allows us to perform a custom **binary search** (`Get`) in logarithmic time to find the largest timestamp that is less than or equal to the requested target.

### Thought Process

1.  **Data Structure**: Map each `key` to a slice of structs: `pair{val string, time int}`.
2.  **`Set` Operation**: Append the new `pair` to the slice corresponding to `key`. Since timestamps are strictly increasing, this preserves sorting.
3.  **`Get` Operation**:
    *   **Default Check**: If the key does not exist, return `""`.
    *   **Boundary Optimizations**:
        *   If the target `timestamp` is strictly smaller than the first element's timestamp, return `""` immediately.
        *   If the target `timestamp` is greater than or equal to the last element's timestamp, return the last element's value immediately in $O(1)$ time.
    *   **Binary Search**: Run a binary search on the slice using `l <= r` to find the largest index `m` where `slice[m].time <= timestamp`. Keep track of the best match `res = slice[m].val` and shift `l = m + 1`.

### Go Code

``` go
type pair struct {
    val     string
    time    int
}

type TimeMap struct {
    keyToPairs    map[string][]pair
}

func Constructor() TimeMap {
    return TimeMap{
        keyToPairs: make(map[string][]pair),
    }
}

func (this *TimeMap) Set(key string, value string, timestamp int)  {
    if _, exists := this.keyToPairs[key]; !exists {
        this.keyToPairs[key] = make([]pair, 0)
    }
    p := pair{val: value, time: timestamp}
    this.keyToPairs[key] = append(this.keyToPairs[key], p)
}

func (this *TimeMap) Get(key string, timestamp int) string {
    if _, exists := this.keyToPairs[key]; !exists {
        return ""
    }
    
    slice := this.keyToPairs[key]
    l, r := 0, len(slice)-1
    
    // Boundary optimizations
    if timestamp < slice[0].time {
        return ""
    } else if timestamp >= slice[r].time {
        return slice[r].val
    }
    
    res := ""
    for l <= r {
        m := l + (r-l)/2
        // We want the largest timestamp <= target timestamp
        if slice[m].time <= timestamp {
            res = slice[m].val
            l = m + 1
        } else {
            r = m - 1
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**:
    - **`Set`**: $O(1)$ amortized time for slice append.
    - **`Get`**: $O(\log n)$ due to the binary search (where $n$ is the number of values stored for the given `key`). Boundary cases complete in $O(1)$ time.
- **Space Complexity**: $O(N)$
    - Where $N$ is the total number of key-value pairs stored in the data store.

---

## Optimized Solution: Structure of Arrays (SoA) + `sort.Search`

In performance-critical environments (frequent at Google/Amazon), we can optimize this further by restructuring our data from an **Array-of-Structures (AoS)** to a **Structure-of-Arrays (SoA)** and using Go's optimized standard library `sort.Search`.

### Thought Process

1.  **Structure of Arrays (SoA) for Cache Locality**:
    *   Instead of storing custom `pair` structs, we map each key to **two separate parallel slices**: `times []int` and `values []string`.
    *   **Why?** During binary search, the CPU only needs to inspect `times`. Because `times` is a flat slice of raw integers, it has **excellent cache locality** and fits perfectly in CPU cache lines. We avoid loading the heavy `string` descriptors into the cache lines during search loops.
2.  **Standard Library Search**:
    *   Use `sort.Search(n, func(i int) bool { return times[i] > timestamp })` which is bug-free and highly optimized.
    *   `sort.Search` finds the first index `idx` where `times[idx] > timestamp`.
    *   The largest timestamp $\le$ `timestamp` is simply at `idx - 1`.

### Go Code

``` go
import "sort"

type TimeMap struct {
    times   map[string][]int
    values  map[string][]string
}

func Constructor() TimeMap {
    return TimeMap{
        times:  make(map[string][]int),
        values: make(map[string][]string),
    }
}

func (this *TimeMap) Set(key string, value string, timestamp int) {
    this.times[key] = append(this.times[key], timestamp)
    this.values[key] = append(this.values[key], value)
}

func (this *TimeMap) Get(key string, timestamp int) string {
    times := this.times[key]
    n := len(times)
    if n == 0 || timestamp < times[0] {
        return ""
    }
    
    // sort.Search finds the first index where times[i] > timestamp
    idx := sort.Search(n, func(i int) bool {
        return times[i] > timestamp
    })
    
    // The largest timestamp <= target is at idx - 1
    return this.values[key][idx-1]
}
```

### Code Efficiency

- **Time Complexity**:
    - **`Set`**: $O(1)$ amortized time.
    - **`Get`**: $O(\log n)$ using Go's standard library search, with **significantly better average latency** due to improved CPU cache line utilization.
- **Space Complexity**: $O(N)$
    - Storing two slices of size $N$ takes the same space asymptotically but has much lower overhead than allocating struct envelopes.
