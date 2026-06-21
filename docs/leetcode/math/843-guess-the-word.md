# 843. Guess the Word

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/guess-the-word/description/)

## Solution: Heuristic Filtering (Most Representative Word)

We can solve this interactive game by maintaining a pool of possible candidate words and shrinking this pool after each guess. Rather than picking guesses at random, we pick a "most representative" word that shares the most common characters with other words in the pool, maximizing the information gain from each feedback.

### Thought Process

1.  **Candidate Pool**: Start with a pool containing all words in the input list.
2.  **Representative Word Selection**:
    - For each position $0 \le i < 6$ and each lowercase letter, count its occurrence across all words currently in the pool.
    - Score each candidate word by summing the frequencies of its characters at their respective positions.
    - Choose the word with the highest score as the next guess. This heuristic prioritizes words containing highly common letters, which helps to partition the pool more effectively regardless of the match count.
3.  **Guessing and Filtering**:
    - Guess the chosen representative word. If `Guess` returns 6, we have found the secret word and can terminate.
    - Otherwise, the secret word must have exactly `matches` characters in common with our guess word. We filter the candidate pool, keeping only the words that have exactly `matches` overlapping characters with our guess word.
4.  **Looping**: Repeat the guessing and filtering process up to 30 times.

### Go Code

``` go
/**
 * // This is the Master's API interface.
 * // You should not implement it, or speculate about its implementation
 * type Master struct {
 * }
 *
 * func (this *Master) Guess(word string) int {}
 */
func findSecretWord(words []string, master *Master) {
    pool := words

    for range 30 {
        if len(pool) == 0 {
            return
        }
        guessWord := getMostRepresentativeWord(pool)

        matches := master.Guess(guessWord)
        if matches == 6 {
            return
        }

        nextPool := make([]string, 0)
        for _, w := range pool {
            if getMatchCount(w, guessWord) == matches {
                nextPool = append(nextPool, w)
            }
        }
        pool = nextPool
    }
}

func getMostRepresentativeWord(pool []string) string {
    var count[6][26]int
    for _, w := range pool {
        for i := 0; i < 6; i++ {
            count[i][w[i]-'a']++
        }
    }
    bestWord := pool[0]
    maxScore := -1

    for _, w := range pool {
        score := 0
        for i := 0; i < 6; i++ {
            score += count[i][w[i]-'a']
        }
        if score > maxScore {
            maxScore = score
            bestWord = w
        }
    }
    return bestWord
}

func getMatchCount(w1, w2 string) int {
    matches := 0
    for i := 0; i < 6; i++ {
        if w1[i] == w2[i] {
            matches++
        }
    }
    return matches
}
```

### Code Efficiency

- **Time Complexity**: $O(g \cdot n \cdot L)$
    - $g \le 30$ is the maximum number of guesses, $n$ is the number of words in the pool (initially up to 100), and $L = 6$ is the word length.
    - Calculating character frequencies, scoring words, and filtering the pool all run in $O(n \cdot L)$ per iteration. Since $L$ and $g$ are small constants, the overall running time is extremely fast and effectively linear, $O(n)$.
- **Space Complexity**: $O(n)$
    - We use additional space to store the filtered candidate pools.

---

## Alternative Solution: Minimax Strategy

Instead of using a simple frequency heuristic, we can use a Minimax strategy to minimize the worst-case size of the candidate pool in subsequent guesses. For every word, we calculate how the remaining words would be distributed into buckets based on their match count (0 to 6) with it. We then select the word whose largest bucket is as small as possible.

### Thought Process

1.  **Bucketing Candidates**: For each candidate word, compare it against all other words in the current pool. Group these other words into buckets (from index 0 to 6) based on their match count.
2.  **Find the Worst-Case Bucket**: The worst-case outcome when guessing a word is that the secret word lands in its largest bucket. We record this size as `currLargestBucketSize`.
3.  **Minimize the Maximum**: To find the optimal guess, we scan all candidate words and select the one that has the smallest maximum bucket size (`largestBucketSize`). This guarantees that no matter what match count the master returns, our remaining candidate pool size is minimized.
4.  **Filtering**: Call `master.Guess(guessWord)`. If we get 6 matches, we stop. Otherwise, filter the pool to keep only the words that have exactly `matches` characters in common with our guess.

### Go Code

``` go
func findSecretWord(words []string, master *Master) {
    pool := words

    for range 30 {
        if len(pool) == 0 {
            return
        }

        guessWord := getMinimaxWord(pool)
        matches := master.Guess(guessWord)
        if matches == 6 {
            return
        }

        nextPool := make([]string, 0)
        for _, w := range pool {
            if getMatchCount(guessWord, w) == matches {
                nextPool = append(nextPool, w)
            }
        }
        pool = nextPool
    }
}

func getMinimaxWord(pool []string) string {
    bestWord := pool[0]
    largestBucketSize := len(pool) + 1

    for _, word := range pool {
        bucket := [7]int{}
        currLargestBucketSize := 0
        for _, other := range pool {
            if word != other {
                matchCount := getMatchCount(word, other)
                bucket[matchCount]++
                currLargestBucketSize = max(currLargestBucketSize, bucket[matchCount])
            }
        }
        if currLargestBucketSize < largestBucketSize {
            largestBucketSize = currLargestBucketSize
            bestWord = word
        }
    }
    return bestWord
}

func getMatchCount(w1, w2 string) int {
    cnt := 0
    for i := 0; i < 6; i++ {
        if w1[i] == w2[i] {
            cnt++
        }
    }
    return cnt
}
```

### Code Efficiency

- **Time Complexity**: $O(g \cdot n^2 \cdot L)$
    - $g \le 30$ is the maximum number of guesses, $n$ is the pool size (initially up to 100), and $L = 6$ is the word length.
    - In each iteration, comparing every pair of words in the pool to build buckets takes $O(n^2 \cdot L)$ time. Since $n \le 100$ and $L = 6$, $O(n^2 \cdot L)$ is extremely small (around $6 \times 10^4$ operations in the worst case) and easily fits within LeetCode's execution limits.
- **Space Complexity**: $O(n)$
    - We store the candidate pools and use a small constant array of size 7 (`bucket`) for frequency counts.