# 126. Word Ladder II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/word-ladder-ii/description/)

## Solution: Forward BFS (Distance Mapping) + Backward DFS (Backtracking)

Finding all shortest transformation sequences requires a two-step approach:
1.  **Forward BFS**: Starting from `beginWord`, perform a level-order search to find the shortest distance to each reachable word. We map these distances in a map. Once we find a path to `endWord` in the current level, we complete the level and stop the BFS (since any path extending to subsequent levels would be longer than the shortest path).
2.  **Backward DFS**: Start from `endWord` and backtrack to `beginWord`. At each step, from the current word `curr`, we only transition to neighbor words `prevWord` that satisfy the strict distance constraint: `distance[prevWord] == distance[curr] - 1`. Reversing this path once we reach `beginWord` yields a valid shortest transformation sequence.

### Thought Process

1.  **Setup and Validation**:
    - Load the `wordList` into a hash set `wordSet` for $O(1)$ lookups.
    - If `endWord` is not in `wordSet`, no transformation is possible. Return an empty slice.
2.  **Forward BFS (Distance Calculation)**:
    - Maintain a `distance` map initialized with `distance[beginWord] = 0` and a BFS queue with `beginWord`.
    - Run level-order BFS:
        - For each word in the current queue level, generate all 1-character mutations (substituting each character position with `'a'` through `'z'`).
        - If the mutated word `nextWord` is in `wordSet` and is unvisited, set `distance[nextWord] = distance[curr] + 1`.
        - If `nextWord == endWord`, set `foundEnd = true`. Else, add `nextWord` to the queue for the next level.
    - Stop the BFS after completing the level where `foundEnd == true`.
3.  **Backward DFS (Path Backtracking)**:
    - If `foundEnd` is false, no path exists. Return `[][]string{}`.
    - Initiate a recursive DFS from `endWord`, keeping track of the traversed path.
    - From the current word `curr`, generate all 1-character mutations. If a mutation `prevWord` exists in `distance` and `distance[prevWord] == distance[curr] - 1`, recursively call DFS on `prevWord`.
    - When `curr == beginWord`, reverse the accumulated path list (since we walked backwards) and add it to the final result slice.

### Go Code

``` go
func findLadders(beginWord string, endWord string, wordList []string) [][]string {
    wordSet := make(map[string]bool)
    for _, w := range wordList {
        wordSet[w] = true
    }

    if !wordSet[endWord] {
        return [][]string{}
    }

    distance := make(map[string]int)
    distance[beginWord] = 0

    queue := []string{beginWord}
    foundEnd := false

    for len(queue) > 0 && !foundEnd {
        size := len(queue)
        for k := 0; k < size; k++ {
            curr := queue[0]
            queue = queue[1:]

            currBytes := []byte(curr)
            currDist := distance[curr]

            for i := 0; i < len(currBytes); i++ {
                original := currBytes[i]
                for j := 0; j < 26; j++ {
                    currBytes[i] = byte('a'+j)
                    nextWord := string(currBytes)
                    if wordSet[nextWord] {
                        if _, visited := distance[nextWord]; !visited {
                            distance[nextWord] = currDist + 1
                            if nextWord == endWord {
                                foundEnd = true
                            } else {
                                queue = append(queue, nextWord)
                            }
                        }
                    }
                }
                currBytes[i] = original
            }
        }
    }

    if !foundEnd {
        return [][]string{}
    }
    
    var res [][]string

    var dfs func(curr string, path []string)
    dfs = func(curr string, path []string) {
        if curr == beginWord {
            reversed := make([]string, len(path))
            for i, v := range path {
                reversed[len(path)-1-i] = v
            }
            res = append(res, reversed)
            return
        }

        currBytes := []byte(curr)
        currDist := distance[curr]

        for i := 0; i < len(currBytes); i++ {
            original := currBytes[i]
            for j := 0; j < 26; j++ {
                currBytes[i] = byte('a'+j)
                prevWord := string(currBytes)
                if dist, ok := distance[prevWord]; ok && dist == currDist-1 {
                    dfs(prevWord, append(path, prevWord))
                }
            }
            currBytes[i] = original
        }
    }
    dfs(endWord, []string{endWord})
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N \cdot L \cdot 26 + P \cdot L)$
    - Where $N$ is the number of words in `wordList`, $L$ is the length of each word, and $P$ is the number of shortest paths.
    - Generating mutations for each word visited in BFS takes $O(L \cdot 26)$ time. The BFS explores at most all $N$ words, taking $O(N \cdot L \cdot 26)$ time.
    - The DFS searches and reconstructs the paths. Backtracking directly to the start without dead ends limits the search to valid shortest paths.
- **Space Complexity**: $O(N \cdot L + P \cdot L)$
    - The `wordSet` and `distance` map store at most $N$ words of length $L$, taking $O(N \cdot L)$ space.
    - The recursive call stack for DFS can reach a depth of $O(N)$.
    - The output slice stores $P$ paths, each containing up to $N$ words of length $L$.