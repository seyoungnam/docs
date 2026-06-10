# In-Memory Ring Buffer for Metrics Collection

## Context: Edge Telemetry Buffering

As described in [Metrics Monitoring Part 4](../system-design/metrics-monitoring/part-4.md#L15), edge telemetry collection agents need to buffer incoming metrics in RAM before batch-shipping them to the cloud or spilling them to disk during network blips. 

A standard Go slice or queue grows dynamically, leading to memory fragmentation and garbage collection (GC) latency spikes on constrained edge hardware. An **In-Memory Ring Buffer** solves this by:

1.  **Pre-allocating** a fixed-size buffer to eliminate dynamic allocation overhead.
2.  **Overwriting** the oldest telemetry data automatically when the buffer reaches capacity, enforcing a strict storage boundary.

---

## Ring Buffer Codebase

The following Go implementation demonstrates a thread-safe circular ring buffer designed to store telemetry metrics, featuring an overwrite policy when full.

``` go title="ring_buffer.go"
package main

import (
	"errors"
	"fmt"
	"sync"
	"time"
)

// Metric represents a telemetry data point
type Metric struct {
	Name      string
	Value     float64
	Timestamp time.Time
}

// RingBuffer implements a thread-safe circular buffer for Metric items.
// When the buffer is full, new writes overwrite the oldest unread data.
type RingBuffer struct {
	mu       sync.Mutex
	data     []Metric
	capacity int
	writeIdx int // Index where the next item will be written
	readIdx  int // Index where the next item will be read
	size     int // Current number of unread items in the buffer
}

// NewRingBuffer initializes a RingBuffer with a fixed capacity.
func NewRingBuffer(capacity int) *RingBuffer {
	return &RingBuffer{
		data:     make([]Metric, capacity),
		capacity: capacity,
	}
}

// Push adds a metric to the buffer. If the buffer is full,
// it overwrites the oldest unread item and returns true.
func (rb *RingBuffer) Push(item Metric) (overwritten bool) {
	rb.mu.Lock()
	defer rb.mu.Unlock()

	rb.data[rb.writeIdx] = item
	rb.writeIdx = (rb.writeIdx + 1) % rb.capacity

	if rb.size == rb.capacity {
		// Buffer is full; we just overwrote the oldest item at readIdx.
		// We must advance readIdx to discard that item.
		rb.readIdx = (rb.readIdx + 1) % rb.capacity
		overwritten = true
	} else {
		rb.size++
	}

	return overwritten
}

// Pop removes and returns the oldest unread metric from the buffer.
// Returns an error if the buffer is empty.
func (rb *RingBuffer) Pop() (Metric, error) {
	rb.mu.Lock()
	defer rb.mu.Unlock()

	if rb.size == 0 {
		return Metric{}, errors.New("buffer is empty")
	}

	item := rb.data[rb.readIdx]
	// Clear the reference to allow garbage collection if storing pointers
	rb.data[rb.readIdx] = Metric{} 
	rb.readIdx = (rb.readIdx + 1) % rb.capacity
	rb.size--

	return item, nil
}

// Size returns the current number of unread items.
func (rb *RingBuffer) Size() int {
	rb.mu.Lock()
	defer rb.mu.Unlock()
	return rb.size
}

func main() {
	// Initialize a ring buffer with a capacity of 3
	rb := NewRingBuffer(3)

	// 1. Push 3 metrics to fill the buffer
	rb.Push(Metric{Name: "cpu_usage", Value: 42.5, Timestamp: time.Now()})
	rb.Push(Metric{Name: "mem_usage", Value: 68.2, Timestamp: time.Now()})
	rb.Push(Metric{Name: "disk_io",   Value: 12.1, Timestamp: time.Now()})

	fmt.Printf("Buffer size after 3 pushes: %d\n", rb.Size())

	// 2. Push a 4th metric (should overwrite "cpu_usage")
	overwritten := rb.Push(Metric{Name: "net_tx", Value: 104.8, Timestamp: time.Now()})
	fmt.Printf("Was data overwritten on 4th push? %v\n", overwritten)
	fmt.Printf("Buffer size after overwrite: %d\n", rb.Size())

	// 3. Pop remaining items (expecting mem_usage, disk_io, and net_tx)
	for rb.Size() > 0 {
		m, _ := rb.Pop()
		fmt.Printf("Popped Metric: %s = %.1f\n", m.Name, m.Value)
	}

	// 4. Verify buffer is empty
	_, err := rb.Pop()
	if err != nil {
		fmt.Printf("Pop error: %v\n", err)
	}
}
```

---

## Execution & Verification

Run the code to verify the circular buffer's overwrite behavior:

```bash
$ go run ring_buffer.go
Buffer size after 3 pushes: 3
Was data overwritten on 4th push? true
Buffer size after overwrite: 3
Popped Metric: mem_usage = 68.2
Popped Metric: disk_io = 12.1
Popped Metric: net_tx = 104.8
Pop error: buffer is empty
```

Notice that the first pushed metric (`cpu_usage`) was overwritten and discarded when the fourth metric (`net_tx`) was pushed, demonstrating the bounded circular memory footprint.

---

## Advanced: Lock-Free Ring Buffers

While the implementation above uses a `sync.Mutex` for thread safety, extremely high-frequency metrics collections (e.g., hundreds of concurrent worker threads writing millions of data points per second) can experience lock contention.

To eliminate lock contention, performance-critical edge collectors use **Lock-Free Ring Buffers**. These are implemented using Go's `sync/atomic` primitives and constraints:

*   **Power-of-Two Size:** Sizing the buffer's capacity to a power of two (e.g., 1024, 4096) allows wrapping the indices using a fast bitwise AND operation (`index & (capacity - 1)`) instead of the slow modulo (`%`) operator.
*   **Atomic CAS (Compare-And-Swap):** The write and read indices are updated atomically, allowing concurrent producers and consumers to access different slots in the underlying array without acquiring a mutex.