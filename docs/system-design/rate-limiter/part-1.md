# Rate Limiter

**What is a Rate Limiter?**

A rate limiter controls how many requests a client can make within a specific timeframe. It returns HTTP 429 "Too Many Requests" response if the request hits the threshold. Rate limiters prevent abuse, protect your servers from being overwhelmed by bursts of traffic.

---

## Functional Requirements

1. Identify clients by user ID, IP address, or API key to apply the limits
2. Limit HTTP requests based on configurable rules
3. Reject requests with HTTP 429 when limits are exceeded

**Out of Scope:**

- Complex querying or analytics on rate limit data
- Long-term persistence of rate limiting data

---

## Non-Functional Requirements

- low latency check (< 10 ms)
- availability >> consistency(eventual consistency)
- scale to 1M RPS

---

## Core Entities

*   **Rules:** Each rule specifies parameters like requests per time window, which clients it applies to, and what endpoints it covers. (e.g., `authenticated users get 1000 requests/hour` or `the search API allows 10 requests/minute per IP`).
*   **Clients:** The entities being rate limited (e.g., `user ID`, `IP addresses`, `API keys` or combination).
*   **Requests:** The incoming API requests that need to be evaluated against rate limiting rules.


### System Interface

`isRequestAllowed(clientId, ruleId) -> { passes: boolean, remaining: number, resetTime: timestamp }`

---

## API Design

### 1. Identify clients by user ID, IP address, or API key

#### Where to place the rate limiter?

??? warning "Dedicated Service"

    The rate limiter becomes its own microservice. When a request arrives at an application server, the server first makes an API call to the rate limiting service.

    **Pros**

    - App servers can provide more client options like user subscription tier, account status.
    - Can also have different rate limiting services for different parts of your system.
    - The rate limiting service maintains global state. 

    **Cons**

    - Requires an additional network round trip (+ 10ms).
    - Introduce another point of failture. Decide fail open or fail closed.
    - handle operational complexity and network issues.

??? success "API Gateway"

    The rate limiter runs at the very edge of your system, integrated into your API gateway or load balancer.

    **Pros**

    - Conceptually simple and provides strong failure protection.

    **Cons**

    - The rate limiter only has access to information available in the HTTP request - headers, URL, IP address, and authentication tokens.
    - Require external dependencies like Redis to store the rate limiting state.

#### How do we identify clients(keys/dimensions)?

The key we use determines how limits get applied. We have three main options:

- **User ID:** This is typically present in the `Authorization` header as a JWT token.
- **IP Address:** The IP address is typically present in the `X-Forwarded-For` header.
- **API Key:** Common for developer APIs. Each key holder gets their own limits. Most typically, denoted in the `X-API-Key` header.



### 2. Limit requests based on configurable rules

Four main algorithms used in production systems:

#### Fixed Window Counter

- divides time into fixed windows (1-minute buckets) and counts requests in each window.
- maintain a counter that resets to zero at the start of each new window.
- really **simple to implement**. It's just a hash table mapping client IDs to (counter, window_start_time) pairs.
- The main challenges are **boundary effects**: a user could make 100 requests at 12:00:59, then immediately make another 100 requests at 12:01:00, effectively getting 200 requests in 2 seconds.

``` json
{
  "alice:12:00:00": 100,
  "alice:12:01:00": 5,
  "bob:12:00:00": 20,
  "charlie:12:00:00": 0,
  "dave:12:00:00": 54,
  "eve:12:00:00": 0,
  "frank:12:00:00": 12,
}
```

#### Sliding Window Log

- keeps a log of individual request timestamps for each user.
- When a new request arrives, you remove all timestamps older than your window (e.g., older than 1 minute ago), then check if the remaining count exceeds your limit.
- This gives you perfect accuracy.
- The downside is memory usage and computational overhead(scanning through timestamp logs for each request)

#### Sliding Window Counter

- a clever hybrid that approximates sliding windows using fixed windows with some math.
- estimate how many requests the user "should have" made in a true sliding window by weighing the previous and current windows based on how far you are into the current window.
- For example, if you're 30% through the current minute, you count 70% of the previous minute's requests plus 100% of the current minute's requests.
- much **better accuracy than fixed windows** while using **minimal memory**.
- The trade-off is that it's an approximation that assumes traffic is evenly distributed within windows.

#### Token Bucket

a bucket that can hold a certain number of tokens (the burst capacity). Tokens are added to the bucket at a steady rate (the refill rate). Each request consumes one token. If there are no tokens available, the request is rejected.

- This handles both sustained load (the refill rate) and temporary bursts (the bucket capacity).
- simple to implement.

##### How do we actually store each user's bucket? Redis

Redis is a fast, in-memory data store that all our gateway instances can access. Redis can become our central source of truth for all token bucket state.

How the token bucket algorithm works with Redis:

1. The gateway calls Redis to fetch Alice's current bucket state using `HMGET alice:bucket tokens last_refill`.
2. The gateway calculates how many tokens to add to Alice's bucket based on the time elapsed since her last refill.
3. The gateway then updates Alice's bucket state atomically using a Redis transaction to prevent race conditions:

``` lua
MULTI
HSET alice:bucket tokens <new_token_count>
HSET alice:bucket last_refill <current_timestamp>
EXPIRE alice:bucket 3600
EXEC
```
The `MULTI/EXEC` block ensures all commands execute as a single **atomic operation**. 
**Move the entire read-calculate-update logic into a single atomic operation**. With Redis, this can be achieved using something called Lua scripting. Lua scripts are atomic, so the entire rate limiting decision becomes race-condition free.

**Why Redis works perfectly for this:**

- **Speed** - Sub-millisecond responses for simple operations
- **Automatic cleanup** - `EXPIRE` removes inactive user buckets after 1 hour of no activity
- **High availability** - Can be replicated across multiple Redis instances
- **Atomic operations** - The `MULTI/EXEC` transaction ensures no race conditions between gateways


### 3. Reject requests with HTTP 429

**Two response strategies:**

- immediately return an HTTP 429 status code when limits are exceeded
  - `X-RateLimit-Limit`: The rate limit ceiling for that request (e.g., "100")
  - `X-RateLimit-Remaining`: Number of requests left in the current window (e.g., "0")
  - `X-RateLimit-Reset`: When the rate limit resets, as a Unix timestamp (e.g., "1640995200")
  - `Retry-After`: tells the client how many seconds to wait before trying again.
- queue them for later processing
  - While this sounds user-friendly, it creates more problems than it solves.

``` json
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1640995200
Retry-After: 60
Content-Type: application/json

{
  "error": "Rate limit exceeded",
  "message": "You have exceeded the rate limit of 100 requests per minute. Try again in 60 seconds."
} 
```