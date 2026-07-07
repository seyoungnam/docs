# 1220. Count Vowels Permutation

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/count-vowels-permutation/description/)

## Solution 1: Backtracking (Time Limit Exceeded)

A naive approach to solving this problem is to generate all valid permutations using depth-first backtracking. We recursively choose the next vowel while obeying the character transition constraints.

### Thought Process

1.  **Vowel Transition Constraints**:
    *   `a` can only be followed by `e`.
    *   `e` can only be followed by `a` or `i`.
    *   `i` cannot be followed by another `i`.
    *   `o` can only be followed by `i` or `u`.
    *   `u` can only be followed by `a`.
2.  **Reverse Mapping (Incoming Vowels)**:
    *   Alternatively, we can track what vowels can precede each character:
        *   `e`, `i`, `u` can transition to `a`.
        *   `a`, `i` can transition to `e`.
        *   `e`, `o` can transition to `i`.
        *   `i` can transition to `o`.
        *   `i`, `o` can transition to `u`.
3.  **Recursive DFS**:
    *   We define a closure `dfs(i int, prev string)` to count combinations. At each index `i`, we try appending all vowels. If `i > 0`, we only recurse if the previous vowel matches the transition rules.
4.  **Why it TLEs**:
    *   The total number of valid permutations grows exponentially ($O(2.68^n)$). For $n = 2 \times 10^4$, the search space is far too large, resulting in a Time Limit Exceeded (TLE) error.

### Go Code

``` go
const MOD = 1e9 + 7

func countVowelPermutation(n int) int {
    count := 0
    vowels := []string{"a", "e", "i", "o", "u"}

    var dfs func(i int, prev string)
    dfs = func(i int, prev string) {
        if i == n {
            count++
            count %= MOD
            return
        }
        for _, curr := range vowels {
            if i == 0 {
                dfs(i+1, curr)
            } else {
                switch curr {
                case "a":
                    if prev == "e" || prev == "i" || prev == "u"{
                        dfs(i+1, curr)
                    }
                case "e":
                    if prev == "a" || prev == "i"{
                        dfs(i+1, curr)
                    }
                case "i":
                    if prev == "e" || prev == "o" {
                        dfs(i+1, curr)
                    }
                case "o":
                    if prev == "i" {
                        dfs(i+1, curr)
                    }
                case "u":
                    if prev == "i" || prev == "o" {
                        dfs(i+1, curr)
                    }
                }
            }
        }
    }
    dfs(0, "")
    return count % MOD
}
```

### Code Efficiency

- **Time Complexity**: $O(2.68^n)$ - The branching factor causes exponential time complexity, leading to TLE.
- **Space Complexity**: $O(n)$ auxiliary space for the recursion stack.

---

## Solution 2: Dynamic Programming (Optimal)

To count the permutations efficiently, we can use **Dynamic Programming**. Instead of tracking individual paths, we count how many valid strings of length `len` end with each vowel.

### Thought Process

1.  **State Definition**:
    Let $a_k, e_k, i_k, o_k, u_k$ be the number of valid sequences of length $k$ ending with `a`, `e`, `i`, `o`, and `u` respectively.
2.  **Transitions**:
    Using our rules of which vowels can precede each character:
    *   A sequence of length $k+1$ ending in `a` must have its $k$-th character as `e`, `i`, or `u`. Thus:
        $$a_{k+1} = (e_k + i_k + u_k) \pmod{10^9+7}$$
    *   Following the same logic for the other vowels:
        $$e_{k+1} = (a_k + i_k) \pmod{10^9+7}$$
        $$i_{k+1} = (e_k + o_k) \pmod{10^9+7}$$
        $$o_{k+1} = i_k \pmod{10^9+7}$$
        $$u_{k+1} = (i_k + o_k) \pmod{10^9+7}$$
3.  **Space Compression**:
    *   The state at step $k+1$ depends only on the values at step $k$. Thus, we don't need a 2D table or array. We can keep 5 scalar variables and update them concurrently for each iteration.
4.  **Base Case**:
    *   For $n = 1$, each vowel can start a valid sequence of length 1, so:
        $$a = e = i = o = u = 1$$

### Go Code

``` go
const MOD = 1e9 + 7

func countVowelPermutation(n int) int {
    a, e, i, o, u := 1, 1, 1, 1, 1

    for length := 1; length < n; length++ {
        nextA := (e + i + u) % MOD
        nextE := (a + i) % MOD
        nextI := (e + o) % MOD
        nextO := i % MOD
        nextU := (i + o) % MOD

        a, e, i, o, u = nextA, nextE, nextI, nextO, nextU
    }
    return (a + e + i + o + u) % MOD
}
```

### Code Efficiency

- **Time Complexity**: $O(n)$
    - We loop exactly $n - 1$ times, executing constant-time additions and modulo operations in each iteration.
- **Space Complexity**: $O(1)$
    - We only track five scalar state variables, using constant auxiliary space.