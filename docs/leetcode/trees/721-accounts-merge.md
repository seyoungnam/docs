# 721. Accounts Merge

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/accounts-merge/description/)

## Solution: Disjoint Set Union (DSU / Union-Find) on Email Associations

We can model this problem as a graph where each account index represents a node, and sharing an email address represents an edge between two accounts. We can merge connected components efficiently using the **Disjoint Set Union (DSU)** data structure.

### Thought Process

1.  **DSU Initialization**:
    *   Initialize a DSU structure where each account index $i$ (from $0$ to $N-1$) is a separate element.
2.  **Mapping and Unioning**:
    *   Maintain a hash map `emailToAcc` to map each `email` (string) to the first `account index` (int) that owns it.
    *   Iterate through each account `accounts[i]` and its email list:
        *   If an email has been seen before (already exists in `emailToAcc` at index `idx`), union the current account index `i` with `idx` (`uf.Union(i, idx)`).
        *   If the email is seen for the first time, record it: `emailToAcc[email] = i`.
3.  **Grouping Emails by Component Leader**:
    *   Initialize a hash map `emailGroup` mapping the representative account index (`leader`) to a list of emails (`[]string`).
    *   Iterate through all registered emails in `emailToAcc`:
        *   Find the DSU representative for the associated account ID: `leader = uf.Find(accId)`.
        *   Append the email to that leader's list: `emailGroup[leader] = append(emailGroup[leader], email)`.
4.  **Format Results**:
    *   For each leader and its gathered emails:
        *   Retrieve the corresponding owner's name: `name = accounts[leader][0]`.
        *   Sort the list of emails alphabetically as required by the problem description.
        *   Prepend the name to the sorted email slice and append the result to `res`.

### Go Code

``` go
type UnionFind struct {
    parent  []int
    rank    []int
}

func NewUnionFind(n int) *UnionFind {
    parent, rank := make([]int, n), make([]int, n)
    for i := 0; i < n; i++ {
        parent[i] = i
        rank[i] = 1
    }
    return &UnionFind{parent, rank}
}

func (uf *UnionFind) Find(x int) int {
    if x != uf.parent[x] {
        uf.parent[x] = uf.Find(uf.parent[x])
    }
    return uf.parent[x]
}

func (uf *UnionFind) Union(x, y int) bool {
    px, py := uf.Find(x), uf.Find(y)
    if px == py {
        return false
    }
    if uf.rank[px] > uf.rank[py] {
        uf.parent[py] = px
        uf.rank[px] += uf.rank[py]
    } else {
        uf.parent[px] = py
        uf.rank[py] += uf.rank[px]
    }
    return true
}

func accountsMerge(accounts [][]string) [][]string {
    n := len(accounts)
    uf := NewUnionFind(n)
    emailToAcc := make(map[string]int)

    for i, account := range accounts {
        for j := 1; j < len(account); j++ {
            email := account[j]
            if idx, exists := emailToAcc[email]; exists {
                uf.Union(i, idx)
            } else {
                emailToAcc[email] = i
            }
        }
    }

    emailGroup := make(map[int][]string)
    for email, accId := range emailToAcc {
        leader := uf.Find(accId)
        emailGroup[leader] = append(emailGroup[leader], email)
    }

    res := [][]string{}
    for accId, emails := range emailGroup {
        name := accounts[accId][0]
        sort.Strings(emails)
        merged := append([]string{name}, emails...)
        res = append(res, merged)
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(E \log E + E \cdot \alpha(N))$
    - Let $N$ be the number of accounts and $E$ be the total number of emails across all accounts.
    - Mapping and DSU operations take $O(E \cdot \alpha(N))$ time, where $\alpha$ is the Inverse Ackermann function.
    - Sorting the emails inside each merged component dominates the overall time complexity, requiring $O(E \log E)$ in the worst case (where all emails belong to a single user).
- **Space Complexity**: $O(E + N)$
    - The DSU parent and rank slices use $O(N)$ space.
    - The `emailToAcc` map and `emailGroup` map store up to $E$ elements, requiring $O(E)$ space.