# 68. Text Justification

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/text-justification/description/)

## Solution: Greedy Word Packing & Space Distribution

The problem asks us to format an array of words into lines of length `maxWidth` such that each line is fully justified (left and right aligned). For lines with multiple words (except the last line), spaces should be distributed as evenly as possible between words, with extra spaces assigned to the left-most gaps when uneven. The last line and lines containing only a single word must be left-justified with trailing space padding.

### Thought Process

1. **Greedy Line Packing**:
   - Iterate through `words` using a pointer `i`.
   - Maintain `currWords` (words placed in the current line) and `currLen` (sum of lengths of words in `currWords`).
   - For word `words[i]`, check if adding it to the line satisfies: `currLen + len(currWords) + len(words[i]) <= maxWidth`. 
   - `len(currWords)` accounts for the minimum single space required between words.
   - If it fits, append `words[i]` to `currWords`, update `currLen`, and move to the next word.

2. **Formatting Fully Justified Lines**:
   - When the next word doesn't fit, the current line is full and must be formatted.
   - Calculate total space needed: `extraSpace = maxWidth - currLen`.
   - Calculate number of gaps: `gaps = max(1, len(currWords) - 1)`.
   - Base spaces per gap: `space = extraSpace / gaps`.
   - Extra spaces to distribute to left slots: `remainder = extraSpace % gaps`.
   - Loop through `j` from `0` to `gaps - 1`:
     - Append `strings.Repeat(" ", space)` to `currWords[j]`.
     - If `remainder > 0`, append one additional space and decrement `remainder`.
   - Join `currWords` into a single string and append to the result slice `res`.
   - Reset `currWords` and `currLen` for the next line.

3. **Formatting the Final Line**:
   - After processing all words, the remaining `currWords` belong to the final line.
   - Per requirements, the last line is left-justified: join `currWords` with a single space (`strings.Join(currWords, " ")`).
   - Calculate trailing spaces: `trailSpace = maxWidth - len(lastWords)`.
   - Append `lastWords + strings.Repeat(" ", trailSpace)` to `res`.

### Go Code

``` go
import "strings"

func fullJustify(words []string, maxWidth int) []string {
    res := []string{}
    currWords, currLen := []string{}, 0
    i := 0

    for i < len(words) {
        if currLen + len(currWords) + len(words[i]) <= maxWidth {
            currWords = append(currWords, words[i])
            currLen += len(words[i])
            i++
        } else {
            extraSpace := maxWidth - currLen
            gaps := max(1, len(currWords)-1)
            remainder := extraSpace % gaps
            space := extraSpace / gaps

            for j := 0; j < max(1, len(currWords)-1); j++ {
                currWords[j] += strings.Repeat(" ", space)
                if remainder > 0 {
                    currWords[j] += " "
                    remainder--
                }
            }

            res = append(res, strings.Join(currWords, ""))
            currWords, currLen = []string{}, 0
        }
    }

    lastWords := strings.Join(currWords, " ")
    trailSpace := maxWidth - len(lastWords)
    res = append(res, lastWords + strings.Repeat(" ", trailSpace))

    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(N)$
    - Where $N$ is the total number of characters across all words. Each word is processed once, and string formatting constructs lines of total length proportional to the final formatted text.
- **Space Complexity**: $O(N)$
    - The output slice stores all lines, requiring $O(N)$ space. Auxiliary space per line is $O(\text{maxWidth})$ to maintain current words and format space padding.