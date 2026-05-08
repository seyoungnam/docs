# 127. Word Ladder

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/word-ladder/description/)

## Solution: BFS

### Thought Process

1.  **Shortest Path in Unweighted Graph**: The problem asks for the shortest transformation sequence between two words. This can be modeled as finding the shortest path in an unweighted graph where each word is a node, and an edge exists between two words if they differ by exactly one character. BFS is ideal for finding the shortest path in such graphs.
2.  **Word Preprocessing**: Convert the \`wordList\` into a hash set (\`map[string]bool\`) for \$O(1)\$ average-time lookups. If \`endWord\` is not in the set, a transformation is impossible, and we could return \`0\` early.
3.  **BFS Initialization**:
    *   Initialize a \`queue\` with \`beginWord\`.
    *   Maintain a \`visited\` set to prevent cycles and redundant processing.
    *   Start \`step\` at \`1\` (representing the sequence length including \`beginWord\`).
4.  **Level-by-Level Traversal**:
    *   For each word at the current level, try to generate all possible one-letter variations.
    *   For each character position in the word, substitute it with every letter from 'a' to 'z'.
    *   If a variation exists in the \`words\` set and hasn't been \`visited\`:
        *   If it matches \`endWord\`, return the current \`step + 1\` (or return \`step\` inside the loop if the check is performed after dequeueing).
        *   Otherwise, add it to the \`queue\` and mark it as \`visited\`.
5.  **Termination**: If the queue becomes empty and \`endWord\` hasn't been reached, return \`0\`.

### Go Code

\`\`\` go
func ladderLength(beginWord string, endWord string, wordList []string) int {
    words := map[string]bool{}
    for _, w := range wordList {
        words[w] = true
    }
    visited := map[string]bool{beginWord: true}
    queue := []string{beginWord}

    step := 1
    for len(queue) > 0 {
        curLen := len(queue)
        for range curLen {
            curr := queue[0]
            queue = queue[1:]

            if curr == endWord {
                return step
            }

            currBytes := []byte(curr)
            for i := 0; i < len(currBytes); i++ {
                original := currBytes[i]
                for j := 0; j < 26; j++ {
                    currBytes[i] = byte(int('a')+j)
                    currString := string(currBytes)
                    if words[currString] && !visited[currString] {
                        queue = append(queue, currString)
                        visited[currString] = true
                    }
                }
                currBytes[i] = original
            }
        }
        step++
    }
    return 0
}
\`\`\`

### Code Efficiency

-   **Time Complexity**: \$O(N \times L^2)\$
    -   \$N\$ is the number of words in the \`wordList\`, and \$L\$ is the length of each word.
    -   In the worst case, we process each word once (\$N\$).
    -   For each word, we iterate through its length (\$L\$) and try 26 characters. 
    -   String creation \`string(currBytes)\` and map lookups take \$O(L)\$ time.
    -   Total: \$O(N \times L \times 26 \times L) = O(N \times L^2)\$.
-   **Space Complexity**: \$O(N \times L)\$
    -   The \`words\` map, \`visited\` map, and \`queue\` can each store up to \$N\$ words of length \$L\$.

---
