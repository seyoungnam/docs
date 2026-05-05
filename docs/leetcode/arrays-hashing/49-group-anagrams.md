# 49. Group Anagrams

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/group-anagrams/description/)

## Solution : Frequency Counting

### Thought Process

1.  **Core Idea**: Two strings are anagrams if they have the same character frequencies.
2.  **Mapping Technique**: Use a hash map where the key is a unique representation of character counts and the value is a list of strings sharing those counts.
3.  **Key Construction**:
    -   In Go, fixed-size arrays like `[26]int` are comparable and can be used directly as map keys.
    -   For each word, iterate through its characters and increment the corresponding index (`char - 'a'`) in a 26-element array.
4.  **Grouping**:
    -   As we process each string, generate its frequency array and add the string to the map under that key.
5.  **Finalization**: Convert the map's values (slices of strings) into a 2D slice to return.

### Go Code

=== "approach 1"

    ``` go
    func groupAnagrams(strs []string) [][]string {
        kv := map[[26]int][]string{}
        for _, word := range strs {
            counter := [26]int{}
            for i := 0; i < len(word); i++ {
                key := word[i] - 'a'
                counter[key]++
            }
            kv[counter] = append(kv[counter], word)
        }
        res := make([][]string, 0, len(kv))
        for _, values := range kv {
            res = append(res, values)
        }
        return res
    }
    ```

=== "approach 2"

    ``` go
    import (
        "slices"
    )

    func groupAnagrams(strs []string) [][]string {
        kv := map[string][]int{}
        for i, word := range strs {
            runes := []rune(word)
            slices.Sort(runes)
            key := string(runes)
            kv[key] = append(kv[key], i)
        }
        res := make([][]string, 0, len(kv))
        for _, indexes := range kv {
            subset := make([]string, len(indexes))
            for j, idx := range indexes {
                subset[j] = strs[idx]
            }
            res = append(res, subset)
        }
        return res
    }
    ```

### Code Efficiency

- **Time Complexity**: $O(N \cdot L)$
    - $N$ is the number of strings and $L$ is the average length of each string.
    - We iterate through every character of every string exactly once to build the frequency arrays.
- **Space Complexity**: $O(N \cdot L)$
    - We store all characters of the input strings in the hash map and the resulting 2D slice.
    - The auxiliary space for the frequency array is constant ($O(26)$).
