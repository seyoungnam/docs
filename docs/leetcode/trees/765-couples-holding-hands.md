# 765. Couples Holding Hands

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/couples-holding-hands/description/)

## Solution 1: Disjoint Set Union (DSU / Union-Find)

We can model this problem as finding connected components in a graph where each couple represents a node and each couch (pair of adjacent seats) represents an edge between the couples sitting there.

### Thought Process

1. **Problem Formulation & Graph Representation**:
    * There are $N = \text{len}(row) / 2$ couples, numbered from $0$ to $N-1$.
    * Person $x$ belongs to couple ID $\lfloor x / 2 \rfloor$ (or `x / 2`). For example, persons $0$ and $1$ belong to couple $0$, persons $2$ and $3$ belong to couple $1$, etc.
    * There are $N$ couches located at seat pairs $(2i, 2i+1)$ for $i \in [0, N-1]$.
    * If couch $i$ contains persons from couple $C_1$ and couple $C_2$, it implies an edge between couple $C_1$ and couple $C_2$.
2. **Component Size & Minimum Swaps**:
    * Because every couch has 2 seats and every couple has 2 people, the graph decomposes into a set of disjoint connected components (cycles).
    * For any connected component with $k$ couples sitting across $k$ couches, a single swap can resolve at most one couple, breaking the component into a resolved couple (size 1) and a remaining component of size $k - 1$.
    * Therefore, resolving a component of size $k$ requires exactly $k - 1$ swaps.
    * If there are $M$ connected components across all $N$ couples, the total minimum swaps required is:
      $$\text{Total Swaps} = \sum_{j=1}^{M} (k_j - 1) = \sum k_j - \sum 1 = N - M$$
3. **DSU Strategy**:
    * Initialize $N$ disjoint sets, one for each couple (initial component count $M = N$).
    * For each couch $(2i, 2i+1)$:
        * Identify the two couples: $C_1 = \text{row}[2i] / 2$ and $C_2 = \text{row}[2i+1] / 2$.
        * If $C_1 \neq C_2$, union their sets. Each successful union reduces the component count $M$ by $1$.
    * The answer is $N - M$ (which is equal to the total number of successful unions performed).

### Go Code

```go
type UnionFind struct {
    parent []int
    rank   []int
    count  int
}

func NewUnionFind(n int) *UnionFind {
    parent, rank := make([]int, n), make([]int, n)
    for i := 0; i < n; i++ {
        parent[i] = i
        rank[i] = 1
    }
    return &UnionFind{
        parent: parent,
        rank:   rank,
        count:  n,
    }
}

func (uf *UnionFind) Find(x int) int {
    if uf.parent[x] != x {
        uf.parent[x] = uf.Find(uf.parent[x]) // Path compression
    }
    return uf.parent[x]
}

func (uf *UnionFind) Union(x, y int) bool {
    rootX := uf.Find(x)
    rootY := uf.Find(y)
    if rootX == rootY {
        return false
    }

    // Union by rank
    if uf.rank[rootX] < uf.rank[rootY] {
        uf.parent[rootX] = rootY
    } else if uf.rank[rootX] > uf.rank[rootY] {
        uf.parent[rootY] = rootX
    } else {
        uf.parent[rootY] = rootX
        uf.rank[rootX]++
    }
    uf.count--
    return true
}

func minSwapsCouples(row []int) int {
    n := len(row) / 2
    uf := NewUnionFind(n)

    for i := 0; i < len(row); i += 2 {
        couple1 := row[i] / 2
        couple2 := row[i+1] / 2
        uf.Union(couple1, couple2)
    }

    return n - uf.count
}
```

### Code Efficiency

- **Time Complexity**: $O(N \cdot \alpha(N)) \approx O(N)$
    - $N = \text{len}(row) / 2$ is the number of couples.
    - We iterate through the $N$ pairs once and perform $O(1)$ amortized `Find` and `Union` operations via Path Compression and Union by Rank ($\alpha$ is the Inverse Ackermann function).
- **Space Complexity**: $O(N)$
    - The `parent` and `rank` slices in `UnionFind` store state for $N$ couples.

---

## Solution 2: Greedy Swap with Position Map (Cycle Resolution)

Instead of building a DSU graph, we can greedily resolve each couch one by one from left to right.

### Thought Process

1. **Partner Identification**:
    * For any person $x$, their partner is $x \oplus 1$ (e.g., $0 \oplus 1 = 1$, $1 \oplus 1 = 0$, $2 \oplus 1 = 3$, $3 \oplus 1 = 2$).
2. **Greedy Choice**:
    * Iterate through seats $2i$ for $i = 0, 1, \dots, N-1$.
    * Let the person at seat $2i$ be $p_1$. Their partner must be $partner = p_1 \oplus 1$.
    * If seat $2i+1$ already contains $partner$, no swap is needed.
    * If seat $2i+1$ contains someone else ($p_2$), find where $partner$ is currently sitting using a position lookup array `pos`.
    * Swap the person at $2i+1$ with the person at `pos[partner]` and update their indices in `pos`.
3. **Why Greedy is Optimal**:
    * Swapping $partner$ into seat $2i+1$ immediately resolves couple $p_1$'s placement and reduces the remaining cycle size by 1 without altering already settled couples.

### Go Code

```go
func minSwapsCouples(row []int) int {
    n := len(row)
    pos := make([]int, n)
    for i, person := range row {
        pos[person] = i
    }

    swaps := 0
    for i := 0; i < n; i += 2 {
        firstPerson := row[i]
        partner := firstPerson ^ 1

        if row[i+1] != partner {
            partnerPos := pos[partner]

            // Swap row[i+1] with row[partnerPos]
            personToMove := row[i+1]
            row[i+1], row[partnerPos] = row[partnerPos], row[i+1]

            // Update position map
            pos[personToMove] = partnerPos
            pos[partner] = i + 1

            swaps++
        }
    }
    return swaps
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Building the initial `pos` map takes $O(N)$ time.
    - We iterate through the array of length $2N$ in steps of 2, performing $O(1)$ lookups and swaps.
- **Space Complexity**: $O(N)$
    - The `pos` slice stores the seat position of each of the $2N$ people.