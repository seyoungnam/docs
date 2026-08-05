# 211. Design Add and Search Words Data Structure

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/design-add-and-search-words-data-structure/description/)

## Solution: Trie with DFS Wildcard Matching

We can implement this dictionary efficiently using a **Trie (Prefix Tree)** combined with recursive **Depth-First Search (DFS)** to handle wildcard matching.

### Thought Process

1.  **Trie Node Representation**:
    *   Like a standard Trie, each node contains a fixed array of 26 pointers (`children`) for lowercase letters `'a'`–`'z'` and a boolean flag (`isWord`) to mark the end of a word.
2.  **AddWord Operation**:
    *   Iterate through the characters of the word, traversing down the children nodes and creating new nodes when necessary. Finally, set `isWord = true` on the terminal node.
3.  **Search Operation with Wildcard**:
    *   Since the search string can contain `'.'` (which matches any character), we implement a recursive search function:
        *   **Base Case**: If the word slice is empty (`len(word) == 0`), return `this.isWord`.
        *   **Wildcard Case (`'.'`)**: If the current character is `'.'`, the search can match any branch. Loop through all 26 pointers in `children`. For each non-nil child, recursively call `Search(word[1:])`. If any path returns `true`, return `true`. If none succeed, return `false`.
        *   **Standard Character Case**: If it is a normal letter, compute its index `c = word[0] - 'a'`. If `children[c]` is `nil`, the path does not exist; return `false`. Otherwise, recursively call `Search(word[1:])` on the child node.

### Go Code

``` go
type WordDictionary struct {
    children    [26]*WordDictionary
    isWord      bool
}

func Constructor() WordDictionary {
    return WordDictionary{}
}

func (this *WordDictionary) AddWord(word string)  {
    curr := this
    for i := range word {
        c := word[i]-'a'
        if curr.children[c] == nil {
            curr.children[c] = &WordDictionary{}
        }
        curr = curr.children[c]
    }
    curr.isWord = true
    return
}

func (this *WordDictionary) Search(word string) bool {
    if len(word) == 0 {
        return this.isWord
    }
    if word[0] == '.' {
        for _, next := range this.children {
            if next != nil && next.Search(word[1:]) {
                return true
            }
        }
        return false
    }
    c := word[0]-'a'
    next := this.children[c]
    if next == nil {
        return false
    }
    return next.Search(word[1:])
}
```

### Code Efficiency

- **Time Complexity**:
    - **AddWord**: $O(L)$ where $L$ is the length of the word.
    - **Search**:
        - **No wildcards**: $O(L)$ since we trace a single path.
        - **With wildcards (worst case, e.g., `....`)**: $O(26^L)$ as we might need to explore all branches of the Trie recursively.
- **Space Complexity**:
    - **Trie Nodes**: $O(T \times 26)$ where $T$ is the total number of nodes.
    - **Call Stack**: $O(L)$ auxiliary space for the recursion stack during `Search`.