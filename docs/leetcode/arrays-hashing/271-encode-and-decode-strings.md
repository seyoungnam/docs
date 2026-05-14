# 271. Encode and Decode Strings

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/encode-and-decode-strings/description/)

## Solution: Length-Prefixing (Chunked Transfer Encoding)

The challenge in encoding a list of strings into a single string is handling delimiters. If we use a simple character like a comma, the strings themselves might contain commas, leading to ambiguity. To solve this, we use a **Length-Prefixing** strategy, similar to how chunked transfer works in network protocols.

### Thought Process

1.  **Objective**: Encode a list of strings into a single string and decode it back accurately, even if the strings contain special characters or delimiters.
2.  **The Strategy**: For each string, store its **length** followed by a **delimiter** (e.g., `#`), then the **string content** itself.
    - Format: `[length]#[string]`
3.  **Encoding**:
    - Iterate through each string in the input list.
    - Use a `strings.Builder` for efficiency (to avoid repeated allocations).
    - Convert the length of the current string to a string (`strconv.Itoa`).
    - Append the length, the `#` delimiter, and then the actual string content.
    - Example: `["lint", "code", "love", "you"]` -> `"4#lint4#code4#love3#you"`
4.  **Decoding**:
    - Maintain a pointer `i` to track our position in the encoded string.
    - For each chunk:
        - Find the position of the next `#` starting from `i`.
        - The substring between `i` and the `#` is the length of the next string.
        - Convert this substring to an integer.
        - Extract the substring of that length immediately following the `#`.
        - Move the pointer `i` to the end of the extracted string and repeat.

### Go Code

``` go
import (
    "strconv"
    "strings"
)

type Codec struct {
    
}

// Encodes a list of strings to a single string.
func (codec *Codec) Encode(strs []string) string {
    var sb strings.Builder
    for _, s := range strs {
        sb.WriteString(strconv.Itoa(len(s)))
        sb.WriteByte('#')
        sb.WriteString(s)
    }
    return sb.String()
}

// Decodes a single string to a list of strings.
func (codec *Codec) Decode(strs string) []string {
    var res []string
    i := 0

    for i < len(strs) {
        j := i
        // Find the delimiter to get the length prefix
        for strs[j] != '#' {
            j++
        }

        length, _ := strconv.Atoi(strs[i:j])
        
        // Extract the string based on the parsed length
        start := j+1
        end := start + length
        res = append(res, strs[start:end])
        
        // Move pointer to the beginning of the next chunk
        i = end
    }
    return res
}
```


### Code Efficiency

- **Time Complexity**: $O(N)$
    - Let $N$ be the total number of characters across all strings.
    - **Encoding**: Each character is visited once to be written to the `strings.Builder`.
    - **Decoding**: We iterate through the encoded string once to parse lengths and slice substrings.
- **Space Complexity**: $O(N)$
    - **Encoding**: The output string requires $O(N)$ space.
    - **Decoding**: The resulting slice of strings requires $O(N)$ space to store the original characters.
    - Auxiliary space for pointers and temporary variables is $O(1)$.




