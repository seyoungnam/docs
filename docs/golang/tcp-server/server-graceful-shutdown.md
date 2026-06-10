# TCP Server with Graceful Shutdown

## TCP Server Codebase

The following Go implementation demonstrates how to cleanly shut down a concurrent TCP server. It builds upon the worker pool pattern, utilizing `sync.WaitGroup` to track running workers and Go's `signal.NotifyContext` to handle termination signals (`SIGINT`, `SIGTERM`).

!!! question "How does this server handle graceful shutdown?"

    *   **OS Signal Handling:** We use Go's modern `signal.NotifyContext` to listen for signals like `SIGINT` (e.g., `Ctrl+C`) or `SIGTERM`. When a signal is received, the returned context (`ctx`) is canceled.
    *   **Closing the Listener:** Once the context is canceled, a separate goroutine shuts down the listener using `l.Close()`. This unblocks the accept loop, making `l.Accept()` immediately return a `net.ErrClosed` error so the server stops accepting new connections.
    *   **Draining the Job Queue:** The accept loop exits, and the server closes the `jobs` channel. Closing the channel signals the workers that no more work is coming. Workers continue processing any remaining queued connections before exiting their loop (`for conn := range jobs`).
    *   **WaitGroup Synchronization:** We use `sync.WaitGroup` to keep track of running workers. The main execution thread blocks on `wg.Wait()` until all worker goroutines complete their remaining tasks.
    *   **Shutdown Timeout:** A fallback timeout (`time.After`) ensures that even if a client connection hangs indefinitely, the shutdown process will exit after a reasonable limit (e.g., 5 seconds) instead of hanging the process forever.

``` go title="tcp-graceful-shutdown.go" hl_lines="34 35"
package main

import (
	"context"
	"errors"
	"fmt"
	"log"
	"net"
	"os"
	"os/signal"
	"sync"
	"syscall"
	"time"
)

const (
	numWorkers = 2
	queueSize  = 10
)

func main() {
	// Set up signal context to capture SIGINT (Ctrl+C) and SIGTERM
	ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
	defer stop()

	// Start TCP listener on port 8080
	l, err := net.Listen("tcp", "127.0.0.1:8080")
	if err != nil {
		log.Fatalf("failed to start server: %v", err)
	}

	log.Printf("Server listening on 127.0.0.1:8080...")

	jobs := make(chan net.Conn, queueSize)
	var wg sync.WaitGroup

	// Spawn the worker pool
	for i := 1; i <= numWorkers; i++ {
		wg.Add(1)
		go func(id int) {
			defer wg.Done()
			worker(id, ctx, jobs)
		}(i)
	}

	// Goroutine to monitor the context and close the listener
	go func() {
		<-ctx.Done()
		log.Println("Shutdown signal received. Closing TCP listener...")
		l.Close()
	}()

	// Accept loop
	for {
		clientConn, err := l.Accept()
		if err != nil {
			// Check if accept failed because the listener was closed
			if errors.Is(err, net.ErrClosed) {
				log.Println("Accept loop stopped (listener closed).")
				break
			}
			log.Printf("connection accept error: %v", err)
			continue
		}

		// Dispatch to worker pool, or reject if shutdown has already begun
		select {
		case <-ctx.Done():
			log.Println("Incoming connection rejected: server is shutting down.")
			clientConn.Close()
		case jobs <- clientConn:
		}
	}

	// Signal to workers that no new connections are coming
	close(jobs)

	log.Println("Waiting for active workers to drain and finish connection queues...")

	// Wait for workers to finish, or force shutdown if they take too long
	done := make(chan struct{})
	go func() {
		wg.Wait()
		close(done)
	}()

	select {
	case <-done:
		log.Println("All workers finished. Graceful shutdown complete.")
	case <-time.After(5 * time.Second):
		log.Println("Shutdown timeout reached. Forcing process exit.")
	}
}

func worker(id int, ctx context.Context, jobs <-chan net.Conn) {
	for conn := range jobs {
		log.Printf("[Worker %d] processing connection", id)
		processConnection(ctx, conn)
		log.Printf("[Worker %d] closed connection", id)
	}
}

func processConnection(ctx context.Context, conn net.Conn) {
	defer conn.Close()

	for {
		select {
		case <-ctx.Done():
			// Server is shutting down: tell the client and disconnect
			_, _ = conn.Write([]byte("Server is shutting down. Disconnecting...\n"))
			return
		default:
			// Stream connection metrics/updates
			timestamp := fmt.Sprintf("Current Time: %s\n", time.Now().Format(time.RFC3339))
			_, err := conn.Write([]byte(timestamp))
			if err != nil {
				return
			}
			time.Sleep(1 * time.Second)
		}
	}
}
```

---

## Execution & Verification

1.  **Start the Server:** Execute `go run tcp-graceful-shutdown.go`.
2.  **Establish Active Connections:** Open two terminal windows and connect to the server:
    *   **Terminal 2:** `nc localhost 8080` (Worker 1 starts streaming)
    *   **Terminal 3:** `nc localhost 8080` (Worker 2 starts streaming)
3.  **Trigger Graceful Shutdown:** Go back to the server terminal (Terminal 1) and press `Ctrl+C`.
4.  **Observe Server Logs:**
		```text
		2026/06/09 02:02:43 Server listening on 127.0.0.1:8080...
		2026/06/09 02:02:55 [Worker 1] processing connection
		2026/06/09 02:03:02 [Worker 2] processing connection
		^C2026/06/09 02:03:15 Shutdown signal received. Closing TCP listener...
		2026/06/09 02:03:15 Accept loop stopped (listener closed).
		2026/06/09 02:03:15 Waiting for active workers to drain and finish connection queues...
		2026/06/09 02:03:15 [Worker 1] closed connection
		2026/06/09 02:03:16 [Worker 2] closed connection
		2026/06/09 02:03:16 All workers finished. Graceful shutdown complete.
		```
5.  **Observe Client Terminals:** In Terminals 2 & 3, the clients receive `"Server is shutting down. Disconnecting..."` and the Netcat sessions terminate cleanly.

---

## Conclusion & Design Review

Throughout this step-by-step tutorial series, we built a modern TCP server in Go by progressively addressing architectural challenges:

1.  **[Single Connection Server](server-single-conn.md):** Built a basic synchronous TCP server, exposing how a single active stream blocks all subsequent clients in a FIFO queue.
2.  **[Multiple Connections](server-multi-conn.md):** Introduced goroutines (`go processConnection(...)`) to enable non-blocking concurrency, highlighting the risk of out-of-memory (OOM) crashes under heavy load due to unbounded resource consumption.
3.  **[Restricted Connections](server-multi-conn-restricted.md):** Applied a counting semaphore via a **buffered channel** to throttle concurrency, demonstrating how to limit memory usage and apply backpressure to the OS TCP backlog.
4.  **[Worker Pool Pattern](server-worker-pool.md):** Pre-spawned a set of worker goroutines and a job queue to minimize goroutine allocation churn and absorb short-term connection spikes.
5.  **[Graceful Shutdown](server-graceful-shutdown.md):** Coupled Go's context cancellation, channel closing, and `sync.WaitGroup` synchronization to cleanly terminate workers and close active client sessions without abruptly dropping them.
