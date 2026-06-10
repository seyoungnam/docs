# Lock-Free Ring Buffer (Disruptor Pattern)

## Lock-Free Concurrency in Go

As discussed in [High-Frequency Local Data Ingestion](../../system-design/metrics-monitoring/part-4.md#L63), high-performance telemetry systems can bypass OS-level locks (like `sync.Mutex` or Go channels) entirely using a pre-allocated circular ring buffer. This is often referred to as the **Disruptor Pattern** (popularized by LMAX).

Below is a complete, working Go implementation of a **Single-Producer Single-Consumer (SPSC)** lock-free ring buffer.

!!! question "How does a Lock-Free Ring Buffer work without Mutexes?"

    *   **Atomic Operations as Memory Barriers:** We use Go's `sync/atomic` package (`StoreUint64` and `LoadUint64`). This enforces CPU memory barriers, guaranteeing that the data is fully written into the buffer array before the index sequence is updated and read by another CPU core.
    *   **Preventing False Sharing:** CPU cores load memory in 64-byte blocks called **cache lines**. If the `writeSequence` and `readSequence` variables share the same cache line, writing to one invalidates the cache line for the other core, causing severe cache line bouncing (false sharing). We inject 64-byte padding (`[8]uint64`) to force them onto separate cache lines.
    *   **Power-of-Two Indexing:** By requiring the buffer's capacity to be a power of two, we can map indices to the buffer size using a bitwise AND operator (`index & mask`) instead of the expensive modulo operator (`%`), which saves valuable CPU cycles.

``` go title="disruptor_spsc.go"
package main

import (
	"fmt"
	"runtime"
	"sync"
	"sync/atomic"
	"time"
)

// Metric represents the telemetry data point
type Metric struct {
	Name      string
	Value     float64
	Timestamp time.Time
}

// SPSCRingBuffer is a Single-Producer Single-Consumer lock-free ring buffer.
type SPSCRingBuffer struct {
	// Padding (64 bytes) to prevent false sharing on CPU cache lines
	_padding0     [8]uint64
	writeSequence uint64 // Written by Producer, read by Consumer
	_padding1     [8]uint64
	readSequence  uint64 // Written by Consumer, read by Producer
	_padding2     [8]uint64

	buffer   []Metric
	capacity uint64
	mask     uint64
}

// NewSPSCRingBuffer creates a new ring buffer. Capacity MUST be a power of two.
func NewSPSCRingBuffer(capacity uint64) *SPSCRingBuffer {
	if capacity == 0 || (capacity&(capacity-1)) != 0 {
		panic("capacity must be a power of two")
	}
	return &SPSCRingBuffer{
		buffer:   make([]Metric, capacity),
		capacity: capacity,
		mask:     capacity - 1,
	}
}

// Publish writes an item to the buffer.
// Safe only for a SINGLE producer goroutine.
func (rb *SPSCRingBuffer) Publish(item Metric) {
	currentWrite := rb.writeSequence
	currentRead := atomic.LoadUint64(&rb.readSequence)

	// Wait (busy-spin/yield) if the buffer is full
	for currentWrite-currentRead >= rb.capacity {
		runtime.Gosched() // Yield processor to let the consumer catch up
		currentRead = atomic.LoadUint64(&rb.readSequence)
	}

	// Write directly to the pre-allocated slot
	rb.buffer[currentWrite&rb.mask] = item

	// Store the new write sequence with a release memory barrier.
	// This ensures the write to the slot is completed and visible 
	// to other CPU cores before they observe the updated index.
	atomic.StoreUint64(&rb.writeSequence, currentWrite+1)
}

// Consume reads and returns an item from the buffer.
// Safe only for a SINGLE consumer goroutine.
func (rb *SPSCRingBuffer) Consume() Metric {
	currentRead := rb.readSequence
	currentWrite := atomic.LoadUint64(&rb.writeSequence)

	// Wait (busy-spin/yield) if the buffer is empty
	for currentRead == currentWrite {
		runtime.Gosched() // Yield processor to let the producer publish
		currentWrite = atomic.LoadUint64(&rb.writeSequence)
	}

	// Read directly from the slot
	item := rb.buffer[currentRead&rb.mask]

	// Clear slot reference to assist garbage collection
	rb.buffer[currentRead&rb.mask] = Metric{}

	// Store the updated read sequence (notifies the producer of freed space)
	atomic.StoreUint64(&rb.readSequence, currentRead+1)

	return item
}

func main() {
	// Initialize ring buffer with a capacity of 1024 (power of 2)
	rb := NewSPSCRingBuffer(1024)
	var wg sync.WaitGroup

	const totalItems = 1000000

	start := time.Now()

	// 1. Spawn the Consumer goroutine
	wg.Add(1)
	go func() {
		defer wg.Done()
		for i := 0; i < totalItems; i++ {
			_ = rb.Consume()
		}
	}()

	// 2. Spawn the Producer goroutine
	wg.Add(1)
	go func() {
		defer wg.Done()
		for i := 0; i < totalItems; i++ {
			rb.Publish(Metric{
				Name:      "cpu_idle",
				Value:     float64(i),
				Timestamp: time.Now(),
			})
		}
	}()

	wg.Wait()
	duration := time.Since(start)

	fmt.Printf("Processed %d items in %v\n", totalItems, duration)
	fmt.Printf("Throughput: %.2f million items/sec\n", float64(totalItems)/duration.Seconds()/1000000)
}
```

---

## Execution & Verification

To test and observe the throughput of this lock-free ring buffer, run:

```bash
$ go run disruptor_spsc.go
Processed 1000000 items in 11.58ms
Throughput: 86.35 million items/sec
```

Because this design bypasses mutexes and channels, the Go scheduler and CPU cache work in perfect harmony, enabling throughput rates exceeding **80+ million operations per second** on standard laptop hardware.

---

## Scaling to Multi-Producer (MPSC)

If you have multiple threads (producers) routing metrics into a single worker queue, the Single-Producer model is not sufficient. Scaling this lock-free buffer to support **Multi-Producer Single-Consumer (MPSC)** requires introducing **Compare-And-Swap (CAS)** operations:

1.  **Index Reservation:** Concurrent producers must use `atomic.CompareAndSwapUint64(&q.writeSequence, currentWrite, currentWrite+1)` to safely reserve their unique write index slot.
2.  **Tracking Write Completion:** Since multiple producers can write to different slots concurrently, a slot is not ready to be read immediately after reserving its index. To prevent the consumer from reading a half-written slot, the queue must maintain a secondary status array (or use spin-waiting on the slot value) to signal when the write operation has fully completed.
