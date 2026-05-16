# 121. Best Time to Buy and Sell Stock

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/description/)

## Solution: Suffix Maximum

To maximize profit, we want to find the largest difference between a price and a subsequent higher price. This can be solved by precomputing the maximum future price for every day.

### Thought Process

1.  **Precompute Future Highs**: Create a `suffix` array where `suffix[i]` represents the maximum price from day `i` to the end of the array.
2.  **Calculate Potential Profit**: Iterate through the prices. For each day `i`, the best we can do if we buy on that day is to sell at the maximum price available in the future, which is `suffix[i+1]`.
3.  **Track Maximum**: Maintain a `res` variable to store the highest profit found.

### Go Code

``` go
func maxProfit(prices []int) int {
    n := len(prices)
    if n < 2 {
        return 0
    }
    suffix := make([]int, n)
    suffix[n-1] = prices[n-1]
    for i := n-2; i >=0; i-- {
        suffix[i] = max(suffix[i+1], prices[i])
    }
    res := 0
    for i := 0; i < n-1; i++ {
        res = max(res, suffix[i+1] - prices[i])
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We perform two linear passes over the prices array.
- **Space Complexity**: $O(n)$
    - We store a suffix array of size $n$.

---

## Optimized Solution: One-Pass (Greedy)

### Thought Process

1.  **Track Minimum Price**: As we iterate, keep track of the lowest price seen so far (`minPrice`).
2.  **Calculate Local Profit**: For each price, the current potential profit is `price - minPrice`.
3.  **Update Global Maximum**: Keep track of the highest profit encountered during the single pass.

### Go Code

``` go
func maxProfit(prices []int) int {
    minPrice := prices[0]
    maxProf := 0
    
    for i := 1; i < len(prices); i++ {
        if prices[i] < minPrice {
            minPrice = prices[i]
        } else {
            maxProf = max(maxProf, prices[i] - minPrice)
        }
    }
    return maxProf
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - The algorithm performs a single pass over the prices array.
- **Space Complexity**: $O(1)$
    - We only use two variables (`minPrice`, `maxProf`) regardless of input size.
