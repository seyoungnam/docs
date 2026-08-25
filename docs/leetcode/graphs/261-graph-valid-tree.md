# 261. Graph Valid Tree


``` go
func validTree(n int, edges [][]int) bool {
    if len(edges) > n-1 {
        return false
    }

    adj := make([][]int, n)
    for _, e := range edges {
        u, v := e[0], e[1]
        adj[u] = append(adj[u], v)
        adj[v] = append(adj[v], u)
    }
    
    visit := map[int]bool{}
    var hasCycle func(prev, curr int) bool
    hasCycle = func(prev, curr int) bool {
        if visit[curr] {
            return true
        }
        visit[curr] = true
        for _, next := range adj[curr] {
            if next == prev {
                continue
            }
            if hasCycle(curr, next) {
                return true
            }
        }
        return false
    } 
    return !hasCycle(-1, 0) && len(visit) == n
}
```

