# TCP Server for A Single Connnection

## TCP Server Codebase

The following Go implementation demonstrates a basic TCP server that can only handle a single client connection at a time.

!!! question "Why is this server restricted to a single connection?"

    *   **Synchronous Execution:** In the server's main event loop, once an incoming connection is accepted via `l.Accept()`, the program immediately invokes `processConnection(clientConn)` synchronously within the main execution thread.
    *   **Blocking Loop:** Because `processConnection` contains an active infinite loop (`for { time.Sleep(...) }`) to stream data, the main thread remains blocked inside this function. It cannot return to the top of the `for` loop to accept subsequent connections.

``` go title="tcp-single-conn.go" hl_lines="29"
package main

import (
	"fmt"
	"log"
	"net"
	"time"
)

func main() {
	// Start TCP listener on port 8080
	l, err := net.Listen("tcp", "127.0.0.1:8080")
	if err != nil {
		log.Fatalf("failed to start server: %v", err)
	}
	defer l.Close()

	log.Printf("Server listening on 127.0.0.1:8080...")

	for {
		// Accept incoming client connections
		clientConn, err := l.Accept()
		if err != nil {
			log.Printf("connection accept error: %v", err)
			continue
		}
		
		// Process connection synchronously (blocking other connections)
		processConnection(clientConn)
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

1.  **Start the Server:** Execute `go run tcp-single-conn.go` in a terminal window. The server begins listening on port `8080`.
2.  **Establish the First Connection:** Open a second terminal window and connect to the server using Netcat:
    ```bash
    $ nc localhost 8080
    Current Time: 2026-06-08T23:13:05-04:00
    Current Time: 2026-06-08T23:13:06-04:00
    Current Time: 2026-06-08T23:13:07-04:00
    Current Time: 2026-06-08T23:13:08-04:00
    Current Time: 2026-06-08T23:13:09-04:00
    ...
    ```
3.  **Attempt a Second Connection:** Open a third terminal window and run `nc localhost 8080` again. You will observe that the server prints no output. Because the server is blocked by the first client connection, this second request is queued in the OS TCP backlog. The server will only process and begin streaming to the second client once the first client disconnects.

---

## Problem

This implementation processes incoming connections strictly in a synchronous, First-In-First-Out (FIFO) manner. While an active client is being served, all subsequent client connections are blocked and must wait until the current connection is closed. This synchronous blocking behavior is unsuitable for modern network services that require handling multiple clients concurrently.

To resolve this limitation, proceed to the [TCP Server for Multiple Connections](server-multi-conn.md) page to learn how to transition this server to handle concurrent connections using goroutines.