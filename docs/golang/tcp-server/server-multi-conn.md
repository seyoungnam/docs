# TCP Server for Multiple Connections

## TCP Server Codebase

The following Go implementation demonstrates a TCP server capable of handling multiple concurrent client connections.

!!! question "How does this server support multiple concurrent connections?"

    *   **Asynchronous Execution:** Instead of executing `processConnection(clientConn)` synchronously within the main thread, the server delegates each new connection to a separate **goroutine** using the `go` keyword (`go processConnection(...)`).
    *   **Non-Blocking Loop:** Because the connection processing runs asynchronously, the main thread does not block. It immediately returns to the top of the `for` loop and waits to accept the next incoming connection on `l.Accept()`.

``` go title="tcp-multi-conn.go" hl_lines="29"
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
		
		// Process connection concurrently in a new goroutine
		go processConnection(clientConn)
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

1.  **Start the Server:** Start the server by executing `go run tcp-multi-conn.go` in a terminal window. The server begins listening on port `8080`.
2.  **Establish the First Connection:** Open a second terminal window and connect to the server using Netcat:
		```bash
		$ nc localhost 8080
		Current Time: 2026-06-09T00:19:00-04:00
		Current Time: 2026-06-09T00:19:01-04:00
		Current Time: 2026-06-09T00:19:02-04:00
		Current Time: 2026-06-09T00:19:03-04:00
		...
		...
		```
3.  **Establish Multiple Parallel Connections:** Open a third terminal window and run `nc localhost 8080` again.
		``` bash
		$ nc localhost 8080
		Current Time: 2026-06-09T00:19:03-04:00
		Current Time: 2026-06-09T00:19:04-04:00
		Current Time: 2026-06-09T00:19:05-04:00
		Current Time: 2026-06-09T00:19:06-04:00
		...
		```
		Unlike the single-connection server, this second client immediately begins receiving timestamps. You can open any number of terminals and run Netcat concurrently; the server will handle all of them in parallel without blocking.

---

## Problem

While this server successfully handles concurrent connections, it does so without bounds. Spawning an unchecked number of goroutines makes the server vulnerable to resource exhaustion; under high traffic or a Denial of Service (DoS) attack, the server will continuously accept new connections and allocate goroutine stacks until it runs out of memory (OOM).

To address this security and stability issue, we need to cap the maximum number of concurrent connections. Let's proceed to the [TCP Server for Restricted Multi Connection](server-multi-conn-restricted.md) page to learn how to implement connection throttling.
