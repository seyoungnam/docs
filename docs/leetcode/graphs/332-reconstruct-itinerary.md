# 332. Reconstruct Itinerary

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/reconstruct-itinerary/description/)

You are given a list of airline tickets where `tickets[i] = [from_i, to_i]` represent the departure and the arrival airports of one flight. Reconstruct the itinerary in order and return it.

All of the tickets belong to a man who departs from `"JFK"`. Thus, the itinerary must begin with `"JFK"`. If there are multiple valid itineraries, you should return the itinerary that has the smallest lexical order when read as a single string.

*   For example, the itinerary `["JFK", "LGA"]` has a smaller lexical order than `["JFK", "LGB"]`.

You may assume all tickets form at least one valid itinerary. You must use all the tickets once and only once.

---

## Solution: Eulerian Path (Hierholzer's Algorithm)

Finding a path that visits every edge in a directed graph exactly once is known as finding an **Eulerian Path**. We can solve this efficiently using **Hierholzer's Algorithm**, which uses Depth-First Search (DFS) and post-order backtracking to build the itinerary.

### Thought Process

1.  **Adjacency List & Lexicographical Ordering**:
    *   Build an adjacency list `adj` mapping each departure airport to a list of arrival airports.
    *   Since we want the lexicographically smallest itinerary, sort each airport's destination list alphabetically using `sort.Strings(adj[src])`.
2.  **Hierholzer's Traversal**:
    *   Start a DFS at `"JFK"`.
    *   From the current airport `src`, while there are remaining outbound flights:
        *   Retrieve the alphabetically smallest destination: `dst := adj[src][0]`.
        *   Remove this ticket from the adjacency list to ensure it is only used once: `adj[src] = adj[src][1:]`.
        *   Recursively call DFS on the destination `dst`.
    *   **Post-Order Backtracking**: When an airport has no outbound flights left (it is a dead-end or the end of a cycle), we prepend it to the beginning of our result slice:
        $$\text{res} = \text{append}([\text{src}], \text{res}...)$$
        This guarantees that any dead-ends (like the final destination) are placed at the end of the itinerary, and any cycles are correctly integrated back into the main path.

> [!TIP]
> **Slice Appending Optimization**:
> Appending to the front of a slice (`append([]string{src}, res...)`) takes $O(K)$ time where $K$ is the length of the slice. Repeating this $E$ times takes $O(E^2)$ time.
> We can optimize this to $O(E)$ time by appending to the end of the slice (`res = append(res, src)`) and then reversing the slice just before returning.

### Go Code

``` go
import "sort"

func findItinerary(tickets [][]string) []string {
    adj := map[string][]string{}
    for _, t := range tickets {
        from, to := t[0], t[1]
        adj[from] = append(adj[from], to)
    }
    
    // Sort destinations alphabetically to satisfy the lexical ordering rule
    for src := range adj {
        sort.Strings(adj[src])
    }
    
    res := []string{}
    var dfs func(src string)
    dfs = func(src string) {
        for len(adj[src]) > 0 {
            dst := adj[src][0]
            adj[src] = adj[src][1:] // Remove the edge after using it
            dfs(dst)
        }
        // Prepend to the itinerary when no outgoing flights remain
        res = append([]string{src}, res...)
    }

    dfs("JFK")
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(E \log E)$
    - Where $E$ is the number of tickets. Sorting each airport's destination list takes $O(E \log (\text{outdegree})) = O(E \log E)$ in the worst case. The DFS visits each flight exactly once, taking $O(E)$ steps. (Note: Appending to the front of the slice takes $O(E^2)$ overall, which can be optimized to $O(E)$ by appending to the end and reversing the result).
- **Space Complexity**: $O(E + V)$
    - We use $O(E + V)$ space to store the adjacency list, where $V$ is the number of unique airports, and $O(E)$ space for the DFS recursion call stack in the worst case.