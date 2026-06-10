# TCP Server with a Worker Pool

## TCP Server Codebase

The following Go implementation demonstrates a TCP server that uses a pre-spawned worker pool and a job queue (buffered channel) to process incoming connections.

!!! question "How does a Worker Pool compare to the Semaphore approach?"

    *   **Pre-allocation vs. Spawn-on-Demand:** Unlike the semaphore approach which spawns a new goroutine for every connection and limits them dynamically, a worker pool pre-spawns a fixed number of worker goroutines (`numWorkers`) at startup.
    *   **Goroutine Reuse:** Workers run continuously in the background, pulling connections from a shared `jobs` channel. This eliminates the minor overhead of allocating and deallocating goroutines for every connection.
    *   **Queue Buffering:** The buffered channel acts as a task queue of size `queueSize`. This provides a cushion for sudden traffic bursts, allowing the server to immediately accept new connections and queue them even if all workers are currently busy.
    *   **Backpressure:** If all workers are busy and the job queue is full, the accept loop blocks on `jobs <- clientConn`, pausing the acceptance of new connections and utilizing the operating system's TCP socket backlog to handle the overflow.

``` go title="tcp-worker-pool.go" hl_lines="26 29-31 44"
package main

import (
	"fmt"
	"log"
	"net"
	"time"
)

const (
	numWorkers = 2
	queueSize  = 1
)

func main() {
	// Start TCP listener on port 8080
	l, err := net.Listen("tcp", "127.0.0.1:8080")
	if err != nil {
		log.Fatalf("failed to start server: %v", err)
	}
	defer l.Close()

	log.Printf("Server listening on 127.0.0.1:8080 with %d workers...", numWorkers)

	// Channel to distribute connections to workers
	jobs := make(chan net.Conn, queueSize)

	// Spawn the worker pool
	for i := 1; i <= numWorkers; i++ {
		go worker(i, jobs)
	}

	for {
		// Accept incoming client connections
		clientConn, err := l.Accept()
		if err != nil {
			log.Printf("connection accept error: %v", err)
			continue
		}

		// Dispatch connection to the worker pool.
		// If all workers are busy and the job queue (channel buffer) is full,
		// this blocks the main accept loop, applying TCP backpressure.
		jobs <- clientConn
	}
}

func worker(id int, jobs <-chan net.Conn) {
	for conn := range jobs {
		log.Printf("[Worker %d] accepted a new connection", id)
		processConnection(conn)
		log.Printf("[Worker %d] finished connection", id)
	}
}

func processConnection(conn net.Conn) {
	defer conn.Close()
	
	for {
		// Write the current timestamp in RFC3339 format to the client
		timestamp := fmt.Sprintf("Current Time: %s\n", time.Now().Format(time.RFC3339))
		_, err := conn.Write([]byte(timestamp))
		if err != nil {
			log.Printf("client disconnected or write error: %v", err)
			return
		}

		// Wait for 1 second before sending the next update
		time.Sleep(1 * time.Second)
	}
}
```

---

## Execution & Verification

To understand the queuing and backpressure mechanics, configure `numWorkers = 2` and `queueSize = 1` as shown in the code.

1.  **Start the Server:** Execute `go run tcp-worker-pool.go`.
2.  **Connect Client 1 & 2:** Open two terminal windows and connect to the server:
    *   **Terminal 2:** `nc localhost 8080` (Worker 1 immediately processes this client)
    *   **Terminal 3:** `nc localhost 8080` (Worker 2 immediately processes this client)
3.  **Connect Client 3 (Queued):** Open a fourth terminal window and connect:
    *   **Terminal 4:** `nc localhost 8080`
    *   **Observation:** The client connects successfully (the OS TCP handshake completes and `Accept()` returns), but the client receives no data. This is because both workers are busy; the connection is successfully written to the `jobs` channel queue where it waits.
4.  **Connect Client 4 (Blocked Accept Loop):** Open a fifth terminal window and connect:
    *   **Terminal 5:** `nc localhost 8080`
    *   **Observation:** The client connects but receives nothing. On the server side, because the `jobs` queue is full, the accept loop blocks on `jobs <- clientConn`. The server cannot accept any more connections, and any further clients will wait in the OS TCP backlog.
5.  **Disconnect Client 1:** Terminate the connection in Terminal 2 (press `Ctrl+C`).
    *   **Observation:** Worker 1 becomes free, reads the next connection (Client 3) from the `jobs` queue, and starts serving it. Terminal 4 immediately begins receiving timestamp updates. 
    *   Since a slot in `jobs` was vacated, the main accept loop unblocks, accepts Client 4, and pushes it into the now-vacant slot in the `jobs` channel queue.

---

## Problem

Although the worker pool pattern efficiently controls goroutine overhead and provides queue buffering, shutting down the server remains problematic. If we abruptly terminate the process, all worker goroutines are killed instantly, cutting off active client sessions.

To shut down cleanly, we need to signal the workers to stop accepting new tasks, finish their currently active connections, and let the main routine know when they are done. This is where **WaitGroups** (`sync.WaitGroup`) and **OS signals** come in. Let's move over to the [TCP Server with Graceful Shutdown](server-graceful-shutdown.md) page to complete the design.
