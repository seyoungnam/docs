# TCP Server for Restricted Multi Connection

## TCP Server Codebase

The following Go implementation demonstrates a TCP server that restricts the number of concurrent connections using a buffered channel as a counting semaphore.

!!! question "How does this server restrict concurrent connections?"

    *   **Buffered Channel as a Semaphore:** We initialize a buffered channel of empty structs (`sem := make(chan struct{}, maxConnections)`). Writing to this channel (`sem <- struct{}{}`) acquires a slot, and reading from it (`<-sem`) releases it.
    *   **Natural Backpressure via Accept Block:** By attempting to send to `sem` inside the main accept loop *before* spawning the processing goroutine, the main loop blocks once `maxConnections` is reached. This prevents the server from calling `l.Accept()` again, forcing additional incoming connections to wait in the operating system's TCP backlog.

``` go title="tcp-multi-conn-restrict.go" hl_lines="10 23 36 42"
package main

import (
	"fmt"
	"log"
	"net"
	"time"
)

const maxConnections = 2

func main() {
	// Start TCP listener on port 8080
	l, err := net.Listen("tcp", "127.0.0.1:8080")
	if err != nil {
		log.Fatalf("failed to start server: %v", err)
	}
	defer l.Close()

	log.Printf("Server listening on 127.0.0.1:8080 (limit: %d)...", maxConnections)

	// Semaphore to limit concurrent connections
	sem := make(chan struct{}, maxConnections)

	for {
		// Accept incoming client connections
		clientConn, err := l.Accept()
		if err != nil {
			log.Printf("connection accept error: %v", err)
			continue
		}

		// Acquire a slot in the semaphore.
		// If all slots are full, this blocks the main accept loop,
		// preventing the server from spawning new goroutines.
		sem <- struct{}{}

		// Process connection concurrently in a new goroutine
		go func(conn net.Conn) {
			defer func() {
				// Release slot and close connection
				<-sem
				conn.Close()
			}()
			processConnection(conn)
		}(clientConn)
	}
}

func processConnection(conn net.Conn) {
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

1.  **Start the Server:** Execute `go run tcp-multi-conn-restrict.go` in a terminal window. The server starts listening with a limit of 2 concurrent connections.
2.  **Establish First Two Connections:** Open two new terminal windows and connect to the server using Netcat:
    *   **Terminal 2:** `nc localhost 8080` (receives timestamps immediately)
    *   **Terminal 3:** `nc localhost 8080` (receives timestamps immediately)
3.  **Attempt a Third Connection:** Open a fourth terminal window and run `nc localhost 8080`.
    *   **Observation:** You will notice that the third terminal blocks and receives no output. Because both slots in the `sem` channel are occupied, the server's main loop is blocked on `sem <- struct{}{}` and does not resume accepting new connections. The third connection remains queued in the OS TCP socket backlog.
4.  **Release a Connection:** Terminate the Netcat connection in Terminal 2 (press `Ctrl+C`).
    *   **Observation:** Terminal 4 immediately begins receiving timestamp updates. This is because releasing the second connection frees a slot in `sem`, unblocking the main loop to process the next queued connection.

---

## Problem

While this server successfully caps resource usage by limiting concurrent connections, it does so by spawning and destroying goroutines on-demand for every connection. If the server expects a high volume of short-lived connections, {==this continuous creation and destruction of goroutines can introduce garbage collection and memory allocation overhead==}.

To optimize goroutine allocation and support connection buffering, we can pre-spawn a fixed set of workers. Let's proceed to the [TCP Server with a Worker Pool](server-worker-pool.md) page to learn how to implement the worker pool pattern.