# 359. Logger Rate Limiter

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/logger-rate-limiter/description/)

## Solution: Hash Map Tracking

To rate limit messages so that the same message is not printed more than once every 10 seconds, we can maintain a hash map where the key is the message string and the value is the timestamp when it was last printed.

### Thought Process

1.  **State Fields**:
    - `mp`: A map from message strings to integer timestamps.
2.  **Rate Limiting Check**:
    - For each incoming message at `timestamp`:
        - Look up the last printed timestamp `lastTime` in the map.
        - If the message has never been logged before (`!ok`) OR the elapsed time since it was last logged is $\ge 10$ seconds (`timestamp >= lastTime + 10`):
            - Update the map with the new `timestamp` for this message.
            - Return `true` to allow printing.
        - Otherwise, the message is rate-limited: return `false`.

### Go Code

``` go
type Logger struct {
    mp  map[string]int
}


func Constructor() Logger {
    return Logger{
        mp: map[string]int{},
    }
}


func (this *Logger) ShouldPrintMessage(timestamp int, message string) bool {
    lastTime, ok := this.mp[message]
    if !ok || timestamp >= lastTime + 10 {
        this.mp[message] = timestamp
        return true
    }
    return false
}
```

### Code Efficiency

- **Time Complexity**: $O(1)$
    - Map lookups and insertions run in $O(1)$ average time.
- **Space Complexity**: $O(M)$
    - Where $M$ is the number of unique messages logged. The map grows proportionally to the number of distinct messages.

---

## Optimized Solution: Concurrency-Safe Logger (sync.Mutex)

In concurrent applications, multiple threads (or goroutines) can attempt to log messages simultaneously. Since standard Go maps are not concurrency-safe, concurrent reads and writes will cause a runtime panic. To ensure thread safety, we wrap the map operations with a mutual exclusion lock (`sync.Mutex`).

### Thought Process

1.  **Mutex Lock**:
    - Add `mu sync.Mutex` to the `Logger` struct.
2.  **Serialize Access**:
    - At the start of `ShouldPrintMessage`, acquire the lock: `this.mu.Lock()`.
    - Use `defer this.mu.Unlock()` to ensure the lock is always released when the function returns, preventing deadlocks.
    - Perform the same mapping checks as the basic solution under the protected block.

### Go Code

``` go
import "sync"

type Logger struct {
    mu  sync.Mutex
    mp  map[string]int
}


func Constructor() Logger {
    return Logger{
        mp: make(map[string]int),
    }
}


func (this *Logger) ShouldPrintMessage(timestamp int, message string) bool {
    this.mu.Lock()
    defer this.mu.Unlock()

    lastTime, ok := this.mp[message]
    if !ok || timestamp >= lastTime + 10 {
        this.mp[message] = timestamp
        return true
    }
    return false
}
```

### Code Efficiency

- **Time Complexity**: $O(1)$
    - Mutex lock/unlock operations and map lookups run in $O(1)$ time. Under high contention, threads may block briefly, but the logic remains constant-time.
- **Space Complexity**: $O(M)$
    - Constant auxiliary space is used besides the map of size $O(M)$ storing unique messages.