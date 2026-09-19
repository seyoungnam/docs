# 692. Top K Frequent Words

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/top-k-frequent-words/description/)

Given an array of strings `words` and an integer `k`, return the `k` most frequent strings.

Return the answer sorted by the frequency from highest to lowest. Sort the words with the same frequency by their lexicographical order.

---

## Solution 1: Min-Heap of Size $k$

### Thought Process

1. **Count Frequencies**:
    * Count the occurrences of each unique word using a hash map `counter := make(map[string]int)`.
2. **Min-Heap Custom Comparator**:
    * Maintain a Min-Heap of size at most $k$ to store `Data{word, count}` items.
    * In a Min-Heap, elements at the top are evicted first when `heap.Len() > k`.
    * If counts differ (`h[i].count != h[j].count`), order by smaller count (`h[i].count < h[j].count`) so words with fewer occurrences get evicted first.
    * If counts are identical (`h[i].count == h[j].count`), order by lexicographically **larger** string (`h[i].word > h[j].word`). This ensures that lexicographically larger words are evicted first, leaving lexicographically smaller words in the heap.
3. **Bounded Heap Maintenance**:
    * Iterate through the frequency map and push each word into the heap.
    * If the heap size exceeds $k$, pop the top element (`heap.Pop`).
4. **Extract Result**:
    * Since the heap is a Min-Heap, popping elements gives items from lowest to highest priority among the top $k$.
    * Fill the result slice from right to left (index $k-1$ down to $0$) to obtain decreasing frequency order and ascending lexicographical order for ties.

### Go Code

```go
import (
    "container/heap"
)

type Data struct {
    word  string
    count int
}

type minHeap []Data

func (h minHeap) Len() int { return len(h) }
func (h minHeap) Less(i, j int) bool {
    if h[i].count == h[j].count {
        return h[i].word > h[j].word
    }
    return h[i].count < h[j].count
}
func (h minHeap) Swap(i, j int)       { h[i], h[j] = h[j], h[i] }
func (h *minHeap) Push(x interface{}) { *h = append(*h, x.(Data)) }
func (h *minHeap) Pop() interface{} {
    last := (*h)[len(*h)-1]
    *h = (*h)[:len(*h)-1]
    return last
}

func topKFrequent(words []string, k int) []string {
    counter := make(map[string]int)
    for _, word := range words {
        counter[word]++
    }

    h := &minHeap{}
    heap.Init(h)

    for word, cnt := range counter {
        heap.Push(h, Data{word: word, count: cnt})
        if h.Len() > k {
            heap.Pop(h)
        }
    }

    res := make([]string, k)
    for i := k - 1; i >= 0; i-- {
        res[i] = heap.Pop(h).(Data).word
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N + U \log k)$
    - Where $N$ is the total number of words in `words` and $U$ is the number of unique words ($U \le N$).
    - Counting frequencies takes $O(N)$ time.
    - Pushing $U$ unique words into a heap of bounded size $k$ takes $O(U \log k)$ time.
    - Extracting $k$ elements takes $O(k \log k)$ time.
    - Total time complexity is $O(N \log k)$ in the worst case where $U \approx N$.
- **Space Complexity**: $O(N)$
    - The frequency map stores $U$ unique words ($O(U) \le O(N)$), and the heap stores at most $k + 1$ elements ($O(k)$).

---

## Solution 2: Bucket Sort

### Thought Process

1. **Count Frequencies**:
    * Count word frequencies using a hash map `freq := make(map[string]int)`.
2. **Bucket Grouping**:
    * The maximum possible frequency of any word is $n = \text{len}(words)$.
    * Create a bucket array `bucket := make([][]string, n+1)` where `bucket[c]` holds all words with frequency `c`.
3. **Sort Within Buckets & Collect Results**:
    * Traverse buckets from highest frequency $n$ down to $1$.
    * If `bucket[i]` is not empty, sort the words within the bucket in alphabetical order (`sort.Strings(bucket[i])`).
    * Append words to the result slice until `len(res) == k`.

### Go Code

```go
import (
    "sort"
)

func topKFrequent(words []string, k int) []string {
    freq := make(map[string]int)
    for _, word := range words {
        freq[word]++
    }

    n := len(words)
    bucket := make([][]string, n+1)
    for word, cnt := range freq {
        bucket[cnt] = append(bucket[cnt], word)
    }

    res := make([]string, 0, k)
    for i := n; i > 0; i-- {
        if len(bucket[i]) == 0 {
            continue
        }
        sort.Strings(bucket[i])
        for _, word := range bucket[i] {
            res = append(res, word)
            if len(res) == k {
                return res
            }
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N + U \log U)$
    - Counting frequencies takes $O(N)$ time.
    - Distributing $U$ unique words into buckets takes $O(U)$ time.
    - Sorting words in each bucket takes $O(\sum B_i \log B_i)$ where $B_i$ is the number of words in bucket $i$. In the worst case (all words have equal frequency), this is $O(U \log U)$.
    - Total time complexity is $O(N + U \log U)$.
- **Space Complexity**: $O(N)$
    - The hash map and the bucket array collectively store $U$ unique words and require $O(N)$ space.