# 2115. Find All Possible Recipes from Given Supplies

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/find-all-possible-recipes-from-given-supplies/description/)

## Solution: BFS Topological Sort (Kahn's Algorithm)

To find all recipes that can be prepared, we can model the ingredients, supplies, and recipes as a Directed Dependency Graph:
- A directed edge points from a dependency (ingredient or sub-recipe) to the recipe that requires it.
- We track the count of remaining required ingredients for each recipe in an `inbound` (in-degree) map.
- We perform a topological sort starting with the initial `supplies` (nodes with an in-degree of 0). When a recipe's dependencies are fully met (in-degree drops to 0), it is added to the queue and can be used to make subsequent recipes.

### Thought Process

1.  **Recipe Verification**:
    - Store the target recipes in a hash set `recipeSet` for $O(1)$ verification when a item is successfully produced.
2.  **Dependency Graph Construction**:
    - Create an adjacency list `graph` where `graph[ing]` stores all recipes that require `ing`.
    - Maintain an `inbound` map to track the number of dependencies (ingredients) required to make each recipe.
3.  **Queue Initialization**:
    - Push the initial `supplies` onto the queue, since they require no ingredients to make.
4.  **Topological Traversal**:
    - While the queue is not empty, dequeue `curr`.
    - If `curr` is a target recipe (exists in `recipeSet`), add it to the final result list `res`.
    - For each recipe `next` that depends on `curr`:
        - Decrement the dependency count: `inbound[next]--`.
        - If `inbound[next]` reaches `0` (all its required ingredients are available), add `next` to the queue so it can be used for downstream recipes.

### Go Code

``` go
func findAllRecipes(recipes []string, ingredients [][]string, supplies []string) []string {
    recipeSet := make(map[string]bool)
    for _, r := range recipes {
        recipeSet[r] = true
    }

    graph := make(map[string][]string)
    inbound := make(map[string]int)

    for i, r := range recipes {
        for _, ing := range ingredients[i] {
            graph[ing] = append(graph[ing], r)
            inbound[r]++
        }
    }

    queue := make([]string, 0)
    for _, s := range supplies {
        queue = append(queue, s)
    }

    res := make([]string, 0)
    for len(queue) > 0 {
        curr := queue[0]
        queue = queue[1:]

        if recipeSet[curr] {
            res = append(res, curr)
        }

        for _, next := range graph[curr] {
            inbound[next]--
            if inbound[next] == 0 {
                queue = append(queue, next)
            }
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(V + E)$
    - Where $V$ is the total number of unique items (recipes + supplies) and $E$ is the total number of ingredients in all recipes combined. Building the dependency graph takes $O(E)$ time. In the BFS traversal, each node and edge is processed at most once, taking $O(V + E)$ time.
- **Space Complexity**: $O(V + E)$
    - We use $O(V + E)$ space to store the adjacency list representation of the graph, and $O(V)$ space for the `recipeSet`, `inbound` map, queue, and result slice.