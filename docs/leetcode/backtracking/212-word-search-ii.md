# 212. Word Search II

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/word-search-ii/description/)

## Solution: Backtracking with Trie (DFS Closure)

Searching for multiple words on a board individually (using Word Search I logic) would result in a Time Limit Exceeded (TLE) error. To solve this efficiently, we insert all target words into a **Trie (Prefix Tree)**. This allows us to search for all words simultaneously as we navigate the grid, pruning search branches immediately if the prefix does not exist in the Trie.

### Thought Process

1.  **Trie (Prefix Tree) Struct**:
    *   `child [26]*Trie` holds child node references for lowercase letters `'a'` to `'z'`.
    *   `isWord bool` marks if a path ending at the current node represents a complete word in our dictionary.
    *   `Add(word)` inserts a string into the tree.
2.  **Guided DFS (Trie Traversal)**:
    *   Instead of passing index numbers, the DFS traversal is guided by Trie nodes (`node *Trie`).
    *   At cell $(r, c)$, we check the character `curr = board[r][c]`. If `node.child[curr-'a']` is `nil`, it means no word in our dictionary starts with this prefix. We prune this branch and return immediately.
3.  **Deduplication Optimization**:
    *   When we find a valid word (`node.isWord == true`), we add it to the results list `res` and set `node.isWord = false`. This guarantees we collect each word at most once, eliminating duplicates without the overhead of using a set.
4.  **Recursive Decisions**:
    *   **Pruning & Base Cases**:
        *   If coordinates are out of bounds, return.
        *   If the cell has already been visited in the current path (`board[r][c] == '*'`), return.
        *   If no matching letter exists in the Trie, return.
    *   **In-Place Visited Marking**:
        *   Temporarily mark `board[r][c] = '*'` to prevent reusing the cell.
    *   **Cardinal Search**:
        *   Recursively search the 4 neighboring directions: `(r+1, c)`, `(r-1, c)`, `(r, c+1)`, `(r, c-1)`.
    *   **Backtrack**:
        *   Restore the original character: `board[r][c] = curr`.

### Go Code

``` go
type Trie struct {
    child   [26]*Trie
    isWord  bool
}

func (this *Trie) Add(word string) {
    for i := range word {
        k := word[i]-'a'
        if this.child[k] == nil {
            this.child[k] = &Trie{}
        }
        this = this.child[k]
    }
    this.isWord = true
}

func findWords(board [][]byte, words []string) []string {
    res := make([]string, 0)
    root := &Trie{}
    for _, word := range words {
        root.Add(word)
    }

    ROWS, COLS := len(board), len(board[0])
    
    var dfs func(r int, c int, node *Trie, word string)
    dfs = func(r int, c int, node *Trie, word string) {
        if r < 0 || r >= ROWS || c < 0 || c >= COLS {
            return
        }
        curr := board[r][c]
        if curr == '*' {
            return
        }
        i := curr-'a'
        if node.child[i] == nil {
            return
        }
        node = node.child[i]
        word += string(curr)
        
        if node.isWord {
            res = append(res, word)
            node.isWord = false
        }

        board[r][c] = '*'
        dfs(r+1, c, node, word)
        dfs(r-1, c, node, word)
        dfs(r, c+1, node, word)
        dfs(r, c-1, node, word)
        board[r][c] = curr
    }

    for r := 0; r < ROWS; r++ {
        for c := 0; c < COLS; c++ {
            dfs(r, c, root, "")
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(M \times 4 \times 3^{L-1})$
    - Where $M$ is the number of cells on the board and $L$ is the maximum length of a word. At the starting cell, we have 4 directions, and at subsequent cells we explore at most 3 directions. The prefix tree limits searches, meaning we rarely explore deep paths. Inserting all words into the Trie takes $O(W \cdot L)$ where $W$ is the number of words.
- **Space Complexity**: $O(W \cdot L)$
    - The space required to store the Trie structure containing all words (which is bounded by the total number of characters across prefix paths). The recursion call stack depth uses $O(L)$ space.