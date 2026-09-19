# 192. Word Frequency

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/word-frequency/description/)

Given a text file `words.txt`, calculate the frequency of each word and output the results sorted by descending frequency. Each line of output should display the word followed by its count, separated by a single space.

**Assumptions**:
- `words.txt` contains only lowercase English letters and spaces `' '`.
- Words are separated by one or more whitespace characters.
- Each word's frequency is unique (no need to handle frequency ties).

---

## Solution 1: Unix Pipeline (`tr` + `sort` + `uniq` + `awk`)

A classic one-line Unix pipeline that streams the file through standard Unix coreutils.

### Thought Process

1. **Tokenize Words (`tr -s ' ' '\n'`)**:
   - `tr -s ' ' '\n'` replaces and squeezes consecutive spaces (`-s`) into a single newline character (`\n`), placing each word on its own line.
2. **Group Identical Words (`sort`)**:
   - The `uniq` utility only counts adjacent identical lines. Sorting the words alphabetically ensures that all identical tokens are grouped consecutively.
3. **Count Frequencies (`uniq -c`)**:
   - `uniq -c` suppresses consecutive duplicate lines and prefixes each unique line with its occurrence count (formatted as `<count> <word>`).
4. **Sort by Frequency Descending (`sort -nr`)**:
   - Sort numerically (`-n`) and in reverse/descending order (`-r`) based on the frequency count in the first column.
5. **Format Output (`awk '{print $2, $1}'`)**:
   - Reorder the columns to match the requested format: print field 2 (the word) followed by field 1 (the frequency).

### Bash Script

``` bash
cat words.txt | tr -s ' ' '\n' | sort | uniq -c | sort -nr | awk '{print $2, $1}'
```

> **Alternative one-liner using `xargs`**:
> ``` bash
> xargs -n 1 < words.txt | sort | uniq -c | sort -nr | awk '{print $2, $1}'
> ```
> `xargs -n 1` splits by arbitrary whitespace (spaces, tabs, and newlines) and outputs one word per line.

### Code Efficiency

- **Time Complexity**: $O(N \log N)$
  - Where $N$ is the total number of words in `words.txt`. Tokenizing takes $O(N)$, but sorting all $N$ words takes $O(N \log N)$ time.
- **Space Complexity**: $O(N)$
  - The pipeline buffers words in memory or temporary files during the sorting phases.

---

## Solution 2: Single-Pass `awk` (Hash Map)

Using `awk` with an associative array (hash table) avoids the costly initial $O(N \log N)$ sorting step by counting word frequencies in a single pass directly in memory.

### Thought Process

1. **Line and Field Splitting**:
   - `awk` automatically splits each line into fields `$1, $2, \dots, \$NF` using whitespace as the default delimiter, where `NF` represents the total number of fields in the current record.
2. **One-Pass Counting**:
   - For every line, loop from `i = 1` through `NF` and increment the counter in the associative array: `count[$i]++`.
3. **Emit Results in `END` Block**:
   - After reaching the end of the file, iterate through all keys `w` in `count` and print `w` and `count[w]`.
4. **Sort by Frequency (`sort -rn -k2`)**:
   - Pipe the output to `sort -rn -k2` to sort numerically (`-n`) in descending order (`-r`) based on the second column (`-k2`, which is the count).

### Bash Script

``` bash
awk '{
    for (i = 1; i <= NF; i++) {
        count[$i]++
    }
}
END {
    for (w in count) {
        print w, count[w]
    }
}' words.txt | sort -rn -k2
```

> **One-line format**:
> ``` bash
> awk '{for(i=1;i<=NF;i++) count[$i]++} END {for(w in count) print w, count[w]}' words.txt | sort -rn -k2
> ```

### Code Efficiency

- **Time Complexity**: $O(N + U \log U)$
  - Where $N$ is the total number of words and $U$ is the number of unique words ($U \le N$).
  - Counting occurs in a single linear pass $O(N)$.
  - We only sort the $U$ distinct words at the end, which is $O(U \log U)$ and significantly faster than sorting all $N$ tokens when $U \ll N$.
- **Space Complexity**: $O(U)$
  - The associative array in `awk` stores only the $U$ unique words and their counts.

---

## Solution 3: Go Hash Map & Slice Sort

In Go, we can implement the canonical array/hashing solution using a `map[string]int` and `bufio.Scanner` with `bufio.ScanWords` for streaming file processing.

### Thought Process

1. **Stream Words**:
   - Open `words.txt` and wrap it with `bufio.NewScanner`. Configure `scanner.Split(bufio.ScanWords)` to scan token by token without reading the entire file into memory at once.
2. **Frequency Counting**:
   - Use `counts := make(map[string]int)` to record the frequency of each scanned word.
3. **Collect & Sort**:
   - Transfer the entries of the frequency map into a slice of `WordFreq{Word, Count}` structs.
   - Sort the slice in descending order of frequency using `sort.Slice`.

### Go Code

``` go
package main

import (
    "bufio"
    "fmt"
    "os"
    "sort"
)

type WordFreq struct {
    Word  string
    Count int
}

func countWordFrequency(filename string) ([]WordFreq, error) {
    file, err := os.Open(filename)
    if err != nil {
        return nil, err
    }
    defer file.Close()

    counts := make(map[string]int)
    scanner := bufio.NewScanner(file)
    scanner.Split(bufio.ScanWords)

    for scanner.Scan() {
        word := scanner.Text()
        counts[word]++
    }

    if err := scanner.Err(); err != nil {
        return nil, err
    }

    res := make([]WordFreq, 0, len(counts))
    for word, count := range counts {
        res = append(res, WordFreq{Word: word, Count: count})
    }

    sort.Slice(res, func(i, j int) bool {
        if res[i].Count == res[j].Count {
            return res[i].Word < res[j].Word
        }
        return res[i].Count > res[j].Count
    })

    return res, nil
}

func main() {
    freqs, err := countWordFrequency("words.txt")
    if err != nil {
        fmt.Fprintln(os.Stderr, "Error:", err)
        return
    }

    for _, wf := range freqs {
        fmt.Printf("%s %d\n", wf.Word, wf.Count)
    }
}
```

### Code Efficiency

- **Time Complexity**: $O(N + U \log U)$
  - Scanning and hashing $N$ words takes $O(N)$ time. Sorting the $U$ unique words takes $O(U \log U)$ time.
- **Space Complexity**: $O(U)$
  - The map and slice store $U$ distinct words. The buffered scanner uses $O(L)$ auxiliary memory per token, where $L$ is the maximum word length.

---

## Solution 4: Concurrent MapReduce (Worker Pool & Channels)

For large-scale files or streaming chunks, we can parallelize the counting phase using a worker pool (Map step), then aggregate the partial count maps into a unified result (Reduce step).

### Thought Process

1. **Map Step (Worker Pool)**:
   - Chunk input text and dispatch chunks over a buffered `jobs` channel.
   - Spawn a pool of `numWorkers` goroutines. Each worker consumes chunks, splits them into tokens via `strings.FieldsFunc`, counts local occurrences, and sends its partial `map[string]int` to a `res` channel.
2. **Synchronization**:
   - A `sync.WaitGroup` tracks worker completion. Once all workers finish, a monitoring goroutine safely closes the `res` channel.
3. **Reduce Step (Aggregation)**:
   - The main goroutine reads every local map from `res` and sums counts into a `final` map.
4. **Sort and Output**:
   - Extract map entries into a slice and sort by frequency descending.

### Go Code

``` go
package main

import (
    "fmt"
    "sort"
    "strings"
    "sync"
    "unicode"
)

type WordFreq struct {
    Word  string
    Count int
}

func countChunk(chunk string) map[string]int {
    counts := map[string]int{}

    isSeparator := func(r rune) bool {
        return unicode.IsSpace(r) || unicode.IsPunct(r) || unicode.IsSymbol(r)
    }

    for _, token := range strings.FieldsFunc(chunk, isSeparator) {
        word := strings.ToLower(token)
        counts[word]++
    }
    return counts
}

func parallelWordCount(chunks []string, numWorkers int) []WordFreq {
    var wg sync.WaitGroup

    jobs := make(chan string, len(chunks))
    res := make(chan map[string]int, len(chunks))

    // 1. Worker pool (Map phase)
    for w := 0; w < numWorkers; w++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for job := range jobs {
                res <- countChunk(job)
            }
        }()
    }

    // 2. Dispatch chunks
    for _, chunk := range chunks {
        jobs <- chunk
    }
    close(jobs)

    // 3. Close result channel once all workers finish
    go func() {
        wg.Wait()
        close(res)
    }()

    // 4. Aggregate counts (Reduce phase)
    final := map[string]int{}
    for jobMap := range res {
        for word, cnt := range jobMap {
            final[word] += cnt
        }
    }

    // 5. Sort descending by frequency
    sortedFreqs := make([]WordFreq, 0, len(final))
    for word, count := range final {
        sortedFreqs = append(sortedFreqs, WordFreq{Word: word, Count: count})
    }
    sort.Slice(sortedFreqs, func(i, j int) bool {
        return sortedFreqs[i].Count > sortedFreqs[j].Count
    })

    return sortedFreqs
}

func main() {
    chunks := []string{
        "the day is sunny the the",
        "the sunny is is",
    }

    counts := parallelWordCount(chunks, 4)

    for _, wf := range counts {
        fmt.Printf("%s %d\n", wf.Word, wf.Count)
    }
}
```

### Code Efficiency

- **Time Complexity**: $O(N / P + U \log U)$
  - Where $N$ is the total number of words, $P$ is the number of worker goroutines, and $U$ is the number of unique words. Chunk processing is distributed over $P$ cores, followed by $O(U)$ aggregation and $O(U \log U)$ sorting.
- **Space Complexity**: $O(P \cdot U_{\text{local}} + U)$
  - Each worker allocates a local frequency map before sending it across the channel, and the final reducer aggregates up to $U$ distinct words.
