# 937. Reorder Data in Log Files

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/reorder-data-in-log-files/description/)

## Solution : Sorting

### Thought Process

1.  **Categorization**: The problem defines two types of logs: **Letter-logs** and **Digit-logs**.
    -   Digit-logs must maintain their original relative order (stable property).
    -   Letter-logs must be sorted lexicographically by their contents. If contents are identical, sort them by their identifiers.
    -   All letter-logs come before all digit-logs.
2.  **Implementation Strategy**:
    -   Iterate through the logs and separate them into two slices: `letterLogs` and `digitLogs`.
    -   We identify the log type by looking at the first character after the first space (the identifier).
3.  **Custom Sorting**:
    -   Apply a custom sort to the `letterLogs` slice.
    -   For each log, split it into the `identifier` and the `content`.
    -   Compare the `content` first. If they are equal, compare the `identifier`.
4.  **Final Result**: Combine the sorted `letterLogs` with the original `digitLogs` using `append`.

### Go Code

``` go
func reorderLogFiles(logs []string) []string {
    var letterLogs []string
    var digitLogs []string

    for _, log := range logs {
        idx := strings.Index(log, " ")
        char := log[idx+1]
        if '0' <= char && char <= '9' {
            digitLogs = append(digitLogs, log)
        } else {
            letterLogs = append(letterLogs, log)
        }
    }
    
    sort.Slice(letterLogs, func(i, j int) bool {
        logA, logB := letterLogs[i], letterLogs[j]
        idxA, idxB := strings.Index(logA, " "), strings.Index(logB, " ")

        idfA, fieldA := logA[:idxA], logA[idxA+1:]
        idfB, fieldB := logB[:idxB], logB[idxB+1:]

        if fieldA == fieldB {
            return idfA < idfB
        }
        return fieldA < fieldB
    })
    return append(letterLogs, digitLogs...)
}
```

### Code Efficiency

- **Time Complexity**: $O(M \cdot N \log N)$
    - $N$ is the number of logs and $M$ is the maximum length of a single log.
    - Sorting $N$ logs takes $O(N \log N)$ comparisons.
    - Each comparison between two strings takes $O(M)$ time.
    - The initial categorization takes $O(N \cdot M)$.
- **Space Complexity**: $O(M \cdot N)$
    - We create two new slices (`letterLogs` and `digitLogs`) to store the categorized logs, which in total store all the characters of the original input.
