# 1801. Number of Orders in the Backlog

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/number-of-orders-in-the-backlog/description/)

## Solution: Dual Heaps (Min-Heap for Sells, Max-Heap for Buys)

We can simulate an order book matching engine using two priority queues (heaps):
- A **Min-Heap** for backlog **Sell Orders** (to greedily match with the lowest available sell price).
- A **Max-Heap** for backlog **Buy Orders** (to greedily match with the highest available buy price).

---

### Thought Process

1. **Order Matching Rules**:
    * **Incoming Buy Order (`orderType == 0`)**:
        * A buyer wants to pay as little as possible ($\le \text{price}$).
        * Check the **Sell backlog**: While the lowest sell price in the backlog is $\le \text{buyPrice}$ and the incoming buy order still has `amount > 0`:
            * Match the minimum of both amounts: $\text{tradeAmount} = \min(\text{buyAmount}, \text{sellAmount})$.
            * Deduct $\text{tradeAmount}$ from both.
            * If the sell order is not fully filled, push its remaining quantity back into the sell heap.
        * If the buy order still has remaining amount after matching, add `[buyPrice, remainingAmount]` to the **Buy backlog (Max-Heap)**.
    * **Incoming Sell Order (`orderType == 1`)**:
        * A seller wants to receive as much as possible ($\ge \text{price}$).
        * Check the **Buy backlog**: While the highest buy price in the backlog is $\ge \text{sellPrice}$ and the incoming sell order still has `amount > 0`:
            * Match $\text{tradeAmount} = \min(\text{sellAmount}, \text{buyAmount})$.
            * Deduct $\text{tradeAmount}$ from both.
            * If the buy order is not fully filled, push its remaining quantity back into the buy heap.
        * If the sell order still has remaining amount after matching, add `[sellPrice, remainingAmount]` to the **Sell backlog (Min-Heap)**.

2. **Total Backlog Calculation**:
    * After processing all incoming orders, sum all remaining quantities across both the buy and sell heaps, modulo $10^9 + 7$.

---

### Go Code

```go
import (
    "container/heap"
)

const MOD = 1_000_000_007

type Order [2]int // [price, amount]

// Min-Heap for Sell Orders (ordered by lowest price)
type minHeap []Order
func (h minHeap) Len() int           { return len(h) }
func (h minHeap) Less(i, j int) bool { return h[i][0] < h[j][0] }
func (h minHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *minHeap) Push(x any)        { *h = append(*h, x.(Order)) }
func (h *minHeap) Pop() any {
    last := (*h)[len(*h)-1]
    *h = (*h)[:len(*h)-1]
    return last
}

// Max-Heap for Buy Orders (ordered by highest price)
type maxHeap []Order
func (h maxHeap) Len() int           { return len(h) }
func (h maxHeap) Less(i, j int) bool { return h[i][0] > h[j][0] }
func (h maxHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *maxHeap) Push(x any)        { *h = append(*h, x.(Order)) }
func (h *maxHeap) Pop() any {
    last := (*h)[len(*h)-1]
    *h = (*h)[:len(*h)-1]
    return last
}

func getNumberOfBacklogOrders(orders [][]int) int {
    sells := &minHeap{}
    buys := &maxHeap{}
    heap.Init(sells)
    heap.Init(buys)

    for _, order := range orders {
        price, amount, orderType := order[0], order[1], order[2]

        if orderType == 0 { // Buy Order
            for sells.Len() > 0 && (*sells)[0][0] <= price && amount > 0 {
                cheapestSell := heap.Pop(sells).(Order)
                sellPrice, sellAmount := cheapestSell[0], cheapestSell[1]

                if amount >= sellAmount {
                    amount -= sellAmount
                } else {
                    // Partially fill sell order and push remaining back
                    heap.Push(sells, Order{sellPrice, sellAmount - amount})
                    amount = 0
                }
            }
            if amount > 0 {
                heap.Push(buys, Order{price, amount})
            }
        } else { // Sell Order
            for buys.Len() > 0 && (*buys)[0][0] >= price && amount > 0 {
                highestBuy := heap.Pop(buys).(Order)
                buyPrice, buyAmount := highestBuy[0], highestBuy[1]

                if amount >= buyAmount {
                    amount -= buyAmount
                } else {
                    // Partially fill buy order and push remaining back
                    heap.Push(buys, Order{buyPrice, buyAmount - amount})
                    amount = 0
                }
            }
            if amount > 0 {
                heap.Push(sells, Order{price, amount})
            }
        }
    }

    // Sum remaining quantities in both backlogs
    totalOrders := 0
    for _, order := range *sells {
        totalOrders = (totalOrders + order[1]) % MOD
    }
    for _, order := range *buys {
        totalOrders = (totalOrders + order[1]) % MOD
    }

    return totalOrders
}
```

---

### Code Efficiency

- **Time Complexity**: $O(n \log n)$
    - Where $n$ is the total number of orders in `orders`.
    - Each order is pushed to and popped from a heap of size at most $n$. Heap operations take $O(\log n)$ time, resulting in an overall time complexity of $O(n \log n)$.
- **Space Complexity**: $O(n)$
    - In the worst case where no orders match, all $n$ orders are stored in the `buys` and `sells` heaps, requiring $O(n)$ auxiliary space.