# 1603. Design Parking System

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/design-parking-system/description/)

## Solution: Mutex-Protected Array (Thread-Safe)

To design a parking system that operates correctly in a concurrent environment, we must prevent race conditions when multiple threads attempt to park cars simultaneously. We serialize access to the parking slots using a mutual exclusion lock (`sync.Mutex`). The capacities are stored in a fixed-size array where the `carType` maps directly to the corresponding array index.

### Thought Process

1.  **State Management**:
    - We track three types of parking spaces: big (`1`), medium (`2`), and small (`3`).
    - Using a fixed-size array `slots [4]int` allows us to use `carType` as a direct index. Index `0` is left unused.
2.  **Concurrency Safety**:
    - Under heavy concurrent load, multiple threads calling `AddCar` at the same time can cause a data race (e.g. two threads decrementing the last remaining slot, resulting in double booking).
    - To prevent this, we introduce `sync.Mutex`.
3.  **AddCar Execution**:
    - Lock the mutex (`this.mu.Lock()`) immediately upon entry.
    - Use `defer this.mu.Unlock()` to guarantee the lock is released when the function returns.
    - Check if `this.slots[carType] > 0`. If so, decrement the value by 1 and return `true`. Otherwise, return `false`.

### Go Code

``` go
import "sync"

type ParkingSystem struct {
    mu      sync.Mutex
    slots   [4]int
}


func Constructor(big int, medium int, small int) ParkingSystem {
    return ParkingSystem{
        slots: [4]int{0, big, medium, small},
    }
}


func (this *ParkingSystem) AddCar(carType int) bool {
    this.mu.Lock()
    defer this.mu.Unlock()

    if this.slots[carType] <= 0 {
        return false
    }
    this.slots[carType]--
    return true
}
```

### Code Efficiency

- **Time Complexity**:
    - **`Constructor`**: $O(1)$ constant time.
    - **`AddCar`**: $O(1)$ constant time. Array index lookups and mutex locks are performed in $O(1)$ operations.
- **Space Complexity**: $O(1)$
    - We use a fixed-size array of 4 integers and a single mutex, requiring constant auxiliary space.