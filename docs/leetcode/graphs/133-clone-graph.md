# 133. Clone Graph

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/clone-graph/description/)

Given a reference of a node in a **connected** undirected graph.

Return a **deep copy** (clone) of the graph.

Each node in the graph contains a value (`int`) and a list (`List[Node]`) of its neighbors.

``` java
class Node {
    public int val;
    public List<Node> neighbors;
}
```

---

## Solution: DFS with Visited Node Mapping

Since the graph is undirected and can contain cycles, we must track visited nodes to avoid infinite recursion. We can do this using a hash map that maps each original node to its newly created clone.

### Thought Process

1.  **Clone Map**:
    *   Initialize a map `memo := map[*Node]*Node{}` to store the mapping from original nodes to their cloned counterparts. This acts as both our cache and our visited tracker.
2.  **DFS Cloning Strategy**:
    *   Define a recursive helper function `clone(node)`:
        *   **Base Case**: If `node == nil`, return `nil`.
        *   **Cache Hit**: Check if `node` is already in the map. If `memo[node]` exists, it means the node has already been cloned. Return the existing clone pointer immediately.
        *   **Create Clone**: If not present, create a new clone: `newNode := &Node{Val: node.Val}`.
        *   **Register in Map**: Insert the mapping `memo[node] = newNode` **before** traversing the neighbors. This is crucial for handling cycles; if a neighbor points back to the current node, the recursion will find the clone in the map instead of starting a new clone process.
        *   **Recurse Neighbors**: Iterate through `node.Neighbors`. For each neighbor `nei`, recursively clone it and append it to `newNode.Neighbors`:
            `newNode.Neighbors = append(newNode.Neighbors, clone(nei))`
        *   Return `newNode`.
3.  **Result**:
    *   Return `clone(node)`.

### Go Code

``` go
func cloneGraph(node *Node) *Node {
    memo := map[*Node]*Node{}

    var clone func(node *Node) *Node
    clone = func(node *Node) *Node {
        if node == nil {
            return nil
        }
        
        // If the node has already been cloned, return the clone
        if val, ok := memo[node]; ok {
            return val
        }
        
        // Create the clone and register it in the map
        newNode := &Node{Val: node.Val}
        memo[node] = newNode
        
        // Clone all the neighbors recursively
        for _, nei := range node.Neighbors {
            newNode.Neighbors = append(newNode.Neighbors, clone(nei))
        }
        
        return newNode
    }
    
    return clone(node)
}
```

### Code Efficiency

- **Time Complexity**: $O(V + E)$
    - Where $V$ is the number of vertices (nodes) and $E$ is the number of edges. We visit every vertex and its adjacency list exactly once.
- **Space Complexity**: $O(V)$
    - We store up to $V$ nodes in the `memo` map. In the worst case (e.g., a path graph), the DFS recursion stack can also grow up to $O(V)$ deep.