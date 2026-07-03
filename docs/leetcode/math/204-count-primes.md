# 204. Count Primes

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/count-primes/description/)

## Solution: Sieve of Eratosthenes

The **Sieve of Eratosthenes** is a classic and highly efficient algorithm to find all prime numbers less than a given number $n$. Instead of testing each number individually for primality, we start from the first prime ($2$) and mark all of its multiples as composite (non-prime). We then repeat this process for the next unmarked number.

### Thought Process

1.  **Boolean Array Track**:
    *   Create a boolean slice `nonPrimes` of size $n$, where `nonPrimes[i] = true` means the number $i$ is composite (not prime).
    *   By default, all elements are `false`, meaning we initially assume all numbers are prime.
2.  **Sieving Multiples**:
    *   Iterate $i$ from $2$ up to $\sqrt{n}$ (i.e., $i \times i < n$). We only need to check up to $\sqrt{n}$ because if $n$ has a divisor, it must have a factor less than or equal to its square root.
    *   If `nonPrimes[i]` is `false`, it means $i$ is prime. We then loop through its multiples starting from $i \times i$ (since any smaller multiple of $i$, such as $i \times (i - 1)$, would have already been marked by a prime factor smaller than $i$) and mark them as composite (`nonPrimes[j] = true`).
3.  **Count Results**:
    *   Iterate through the `nonPrimes` array and increment our counter for every index $i$ that remains `false`, making sure to exclude $0$ and $1$ as they are not prime numbers.

### Go Code

``` go
func countPrimes(n int) int {
    if n <= 1 {
        return 0
    }
    nonPrimes := make([]bool, n)
    for i := 2; i*i < n; i++ {
        if !nonPrimes[i] {
            for j := i*i; j < n; j += i {
                nonPrimes[j] = true
            }
        }
    }
    res := 0
    for i, tf := range nonPrimes {
        if !tf && i != 0 && i != 1 {
            res++
        }
    }
    return res
}
```

### Code Efficiency

- **Time Complexity**: $O(n \log \log n)$
    - The outer loop runs up to $\sqrt{n}$. The inner loop marks multiples of each prime $p$, running $n/p$ times. The sum of the reciprocals of prime numbers up to $n$ diverges as $\log \log n$, resulting in $O(n \log \log n)$ total time complexity.
- **Space Complexity**: $O(n)$
    - We allocate a boolean array of size $n$ to keep track of the primality of numbers.