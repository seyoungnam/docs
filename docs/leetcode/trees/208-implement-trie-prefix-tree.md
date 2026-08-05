# 208. Implement Trie (Prefix Tree)

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/implement-trie-prefix-tree/description/)

## Solution: Array-Based Trie Node

A Trie (Prefix Tree) is a tree-like data structure used to efficiently store and retrieve keys in a dataset of strings. We can implement it using a node structure that holds a fixed-size array of pointers to its child nodes.

### Thought Process

1.  **Node Structure**:
    *   Since the problem constraints guarantee that inputs consist only of lowercase English letters (`'a'` to `'z'`), each node can have at most 26 children.
    *   Define the `Trie` struct containing:
        *   `children`: An array of size 26 containing pointers to child `Trie` nodes (`[26]*Trie`).
        *   `isWord`: A boolean flag indicating whether the path from the root to this node represents a complete word.
2.  **Insertion**:
    *   Start at the root node.
    *   For each character in the string `word`, compute its alphabet index: `c = word[i] - 'a'`.
    *   If the child at `children[c]` is `nil`, create a new `Trie` node and link it.
    *   Move the current pointer to the child node.
    *   After inserting all characters, set the final node's `isWord` flag to `true`.
3.  **Search**:
    *   Start at the root node.
    *   For each character, check if `children[c]` exists. If it is `nil`, the word is not in the Trie; return `false`.
    *   Move to the child node.
    *   After traversing all characters, return the value of the final node's `isWord` flag.
4.  **Prefix Search (StartsWith)**:
    *   Use the same traversal logic as the `Search` method.
    *   If we successfully traverse all characters of the `prefix` without hitting a `nil` node, return `true`. (We do not check `isWord` because we are only looking for a matching prefix path).

### Go Code

``` go
type Trie struct {
    children [26]*Trie
    isWord   bool
}

func Constructor() Trie {
    return Trie{
        children: [26]*Trie{},
        isWord: false,
    }
}

func (this *Trie) Insert(word string)  {
    curr := this
    for i := range word {
        c := word[i]-'a'
        if curr.children[c] == nil {
            curr.children[c] = &Trie{}
        }
        curr = curr.children[c]
    }
    curr.isWord = true
}

func (this *Trie) Search(word string) bool {
    curr := this
    for i := range word {
        c := word[i]-'a'
        if curr.children[c] == nil {
            return false
        }
        curr = curr.children[c]
    }
    return curr.isWord
}

func (this *Trie) StartsWith(prefix string) bool {
    curr := this
    for i := range prefix {
        c := prefix[i]-'a'
        if curr.children[c] == nil {
            return false
        }
        curr = curr.children[c]
    }
    return true
}
```

### Code Efficiency

- **Time Complexity**:
    - **Insert**: $O(L)$ where $L$ is the length of the word. We perform $L$ array lookups and allocations.
    - **Search**: $O(L)$ where $L$ is the length of the word.
    - **StartsWith**: $O(P)$ where $P$ is the length of the prefix.
- **Space Complexity**: $O(T \times 26)$ where $T$ is the total number of nodes in the Trie. In the worst case (no common prefixes), the space is proportional to the total number of characters across all inserted words.