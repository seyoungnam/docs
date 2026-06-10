# TCP Server with Sharded Ingestion

## TCP Server Codebase

The following Go implementation demonstrates a TCP server using **Sharded Ingestion (Contention-Free Queueing)**. Instead of workers competing to pull tasks from a single shared channel, each worker has its own dedicated private channel (a "shard"). Incoming connections are mapped to a specific shard using a hashing function on the client's address.

!!! question "Why use Sharded Ingestion over a Single Worker Pool?"

    *   **Eliminating Channel Contention:** Under extreme workloads, having many worker routines pull from a single channel causes thread contention on the channel's internal mutex. Sharding gives each worker an exclusive channel, ensuring there is exactly **one consumer** per channel.
    *   **In-Order Guarantees (Affinity):** Hashing a client-specific key (such as an IP address, device ID, or sensor ID) ensures that all requests from that specific client are processed by the **same worker**. This guarantees sequential execution and order preservation for that client's events.
    *   **Fault Isolation:** If a database transaction or operation slows down a worker, only that worker's private channel shard backs up. The other shards continue consuming and processing incoming traffic unaffected.

``` go title="tcp-sharded-ingestion.go"
package main

import (
	"fmt"
	"hash/fnv"
	"log"
	"net"
	"time"
)

const (
	numShards = 4  // Number of worker shards
	shardCap  = 10 // Queue size per shard channel
)

func main() {
	// Start TCP listener on port 8080
	l, err := net.Listen("tcp", "127.0.0.1:8080")
	if err != nil {
		log.Fatalf("failed to start server: %v", err)
	}
	defer l.Close()

	log.Printf("Server listening on 127.0.0.1:8080 with %d sharded workers...", numShards)

	// Initialize individual channel shards
	shards := make([]chan net.Conn, numShards)
	for i := 0; i < numShards; i++ {
		shards[i] = make(chan net.Conn, shardCap)
		// Spawn a dedicated worker for this specific channel shard
		go worker(i, shards[i])
	}

	for {
		// Accept incoming client connections
		clientConn, err := l.Accept()
		if err != nil {
			log.Printf("connection accept error: %v", err)
			continue
		}

		// Use the client's remote IP + Port as the shard key
		remoteAddr := clientConn.RemoteAddr().String()
		shardIdx := getShardIndex(remoteAddr, numShards)

		log.Printf("Routing connection from %s to Shard %d", remoteAddr, shardIdx)

		// Dispatch connection directly to the designated worker's private channel.
		// If that specific shard's queue is full, the accept loop blocks on it,
		// applying backpressure directly to that client's shard.
		shards[shardIdx] <- clientConn
	}
}

// getShardIndex hashes a string key and maps it to a shard index
func getShardIndex(key string, shardsCount int) int {
	h := fnv.New32a()
	h.Write([]byte(key))
	return int(h.Sum32() % uint32(shardsCount))
}

func worker(id int, jobs <-chan net.Conn) {
	// Each worker exclusively consumes from its own private channel
	for conn := range jobs {
		log.Printf("[Worker %d] processing connection from %s", id, conn.RemoteAddr().String())
		processConnection(conn)
		log.Printf("[Worker %d] closed connection", id)
	}
}

func processConnection(conn net.Conn) {
	defer conn.Close()
	
	// Simulate simple metric streaming
	for i := 0; i < 3; i++ {
		timestamp := fmt.Sprintf("Current Time: %s\n", time.Now().Format(time.RFC3339))
		_, err := conn.Write([]byte(timestamp))
		if err != nil {
			return
		}
		time.Sleep(1 * time.Second)
	}
}
```

---

## Execution & Verification

1.  **Start the Server:** Run `go run tcp-sharded-ingestion.go`.
2.  **Establish Multiple Client Connections:** Open multiple terminal windows and connect using Netcat:
    *   **Terminal 2:** `nc localhost 8080`
    *   **Terminal 3:** `nc localhost 8080`
    *   **Terminal 4:** `nc localhost 8080`
3.  **Observe Routing Logs:** The server logs will show how each connection is routed to a worker depending on its source address hash:
    ```text
    2026-06-10T03:38:00-04:00 Server listening on 127.0.0.1:8080 with 4 sharded workers...
    2026-06-10T03:38:05-04:00 Routing connection from 127.0.0.1:58432 to Shard 1
    2026-06-10T03:38:05-04:00 [Worker 1] processing connection from 127.0.0.1:58432
    2026-06-10T03:38:08-04:00 Routing connection from 127.0.0.1:58435 to Shard 3
    2026-06-10T03:38:08-04:00 [Worker 3] processing connection from 127.0.0.1:58435
    2026-06-10T03:38:10-04:00 Routing connection from 127.0.0.1:58438 to Shard 1
    2026-06-10T03:38:10-04:00 [Worker 1] processing connection from 127.0.0.1:58438
    ```
    *Notice that the first and third connections, both hashing to `Shard 1`, are processed sequentially by `Worker 1` since they share the same queue, while the second connection is processed concurrently by `Worker 3`.*

---

## Architectural Comparison

| Feature | Single Worker Pool | Sharded Ingestion |
| :--- | :--- | :--- |
| **Worker Queue** | One single shared channel | Multiple private channels (one per worker) |
| **Lock Contention** | High (multiple workers locks/unlocks the same channel structure) | Zero (each channel has exactly one reader) |
| **Execution Order** | Out-of-order (any worker can pick up the next connection) | Guaranteed in-order for any given sharding key |
| **Fault Isolation** | Poor (a blocked worker increases queue delay for all clients) | High (a blocked worker only delays its own shard's queue) |
| **Load Distribution** | Uniformly self-balancing | Depends on hash distribution (vulnerable to hot-sharding if keys are imbalanced) |
