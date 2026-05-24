# 2268. Minimum Number of Keypresses

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/minimum-number-of-keypresses/description/)

## Solution: Greedy (Frequency Sorting)

The goal is to map 26 letters to 9 keypad buttons to minimize total presses. The optimal strategy is **greedy**: letters that appear most frequently in `s` must be mapped to the 1st press (1 press), the next 9 most frequent to the 2nd press (2 presses), and the remaining to the 3rd press (3 presses).

### Thought Process

1.  **Count Frequencies**: Create a fixed-size frequency array of size 26 to count the occurrences of each letter.
2.  **Sort Descending**: Sort the frequencies in descending order to prioritize the most frequent letters.
3.  **Assign Keypresses**: Loop through the sorted frequencies:
    *   The first 9 letters (indices `0` to `8`) get `1` press.
    *   The next 9 letters (indices `9` to `17`) get `2` presses.
    *   The remaining letters (indices `18` to `25`) get `3` presses.
    *   This is modeled by: `presses := i / 9 + 1`.
4.  **Accumulate**: Sum `frequency * presses` for each character.

### Go Code

``` go
import "sort"

func minimumKeypresses(s string) int {
    // 1. Count character frequencies
    freq := make([]int, 26)
    for i := 0; i < len(s); i++ {
        freq[s[i]-'a']++
    }
    
    // 2. Sort frequencies in descending order
    sort.Slice(freq, func(i, j int) bool {
        return freq[i] > freq[j]
    })
    
    // 3. Greedily assign keypress count based on frequency rank
    res := 0
    for i := 0; i < 26; i++ {
        if freq[i] == 0 {
            break // No more characters present in s
        }
        
        presses := i/9 + 1
        res += freq[i] * presses
    }
    
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - Counting frequencies takes $O(n)$ time. Sorting a fixed-size array of size 26 takes $O(26 \log 26) = O(1)$ constant time. Thus, the total time is linear.
- **Space Complexity**: $O(1)$
    - We use a fixed-size array of size 26, requiring $O(1)$ auxiliary space.
