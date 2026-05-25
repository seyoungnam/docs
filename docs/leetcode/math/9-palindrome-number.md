# 9. Palindrome Number

[:paperclip: LeetCode Problem Description](https://leetcode.com/problems/palindrome-number/description/)

## Solution: Complete Integer Reversal

We can determine if an integer is a palindrome by reversing its digits entirely and comparing the reversed value to the original number.

### Thought Process

1.  **Negative Numbers**: Any negative number (e.g. `-121`) cannot be a palindrome due to the minus sign. Return `false` immediately.
2.  **Reverse Digits**: Create a copy of `x` (let's call it `n`) and reconstruct the number in reverse order:
    *   Extract the last digit: `n % 10`.
    *   Build the reversed number: `reversed = 10 * reversed + (n % 10)`.
    *   Truncate the last digit: `n /= 10`.
3.  **Verify**: Return `true` if `reversed == x`.

### Go Code

``` go
func isPalindrome(x int) bool {
    if x < 0 {
        return false
    }
    n := x
    reversed := 0
    for n > 0 {
        reversed = 10*reversed + n%10
        n /= 10
    }
    return reversed == x
}
```

### Code Efficiency

- **Time Complexity**: $O(\log_{10} x)$
    - The number of iterations is proportional to the number of digits in `x`, which is $\lfloor \log_{10} x \rfloor + 1$.
- **Space Complexity**: $O(1)$
    - We only use a few integer variables, requiring $O(1)$ auxiliary space.

---

## Alternative Solution: Half-Integer Reversal (Preventing Overflow)

Instead of reversing the entire integer (which could cause integer overflow for large numbers in languages with strict 32-bit limits), we can reverse only the **second half** of the number and compare it to the first half.

### Thought Process

1.  **Discard Edge Cases**: Negative numbers, and positive numbers ending in `0` (like `10`, `20`), cannot be palindromes. Return `false` (except for `0` itself).
2.  **Reverse Halfway**: Reverse the digits of the second half of `x` into `reversed` until `x <= reversed` (which signifies we have reached or passed the middle of the number).
3.  **Compare**: 
    *   If the number has an even number of digits, `x` must equal `reversed`.
    *   If the number has an odd number of digits, the middle digit will be the last digit of `reversed`. We can discard it by doing `reversed / 10`, and check if `x == reversed / 10`.

### Go Code

``` go
func isPalindrome(x int) bool {
    // Negative numbers and numbers ending in 0 (except 0 itself) are not palindromes
    if x < 0 || (x%10 == 0 && x != 0) {
        return false
    }
    
    reversed := 0
    // Stop when we reach the middle of the number
    for x > reversed {
        reversed = 10*reversed + x%10
        x /= 10
    }
    
    // Even length: x == reversed
    // Odd length: x == reversed/10 (discards middle digit)
    return x == reversed || x == reversed/10
}
```

### Code Efficiency

- **Time Complexity**: $O(\log_{10} x)$
    - We only iterate through half of the digits, which is twice as fast in practice.
- **Space Complexity**: $O(1)$
    - Constant auxiliary space is used with zero risk of integer overflow.
