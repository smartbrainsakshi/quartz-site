---
title: "Check if a Number is a Power of Two"
tags: [bit-manipulation, and]
difficulty: Easy
---

# Check if a Number is a Power of Two

## 🎯 What problem are we solving?

> [!question]
> Given a number `n`, determine whether it is a power of two (i.e. `2^0, 2^1, 2^2, ...`).

## 💡 Intuition

> [!tip]
> Every power of two has **exactly one set bit** in its binary form (`1 = 0001`, `2 = 0010`, `4 = 0100`, `8 = 1000`, ...). [Remove the Last Set Bit](07-Remove-the-Last-Set-Bit.md) (`n & (n - 1)`) clears the rightmost set bit — if that was the *only* set bit, the result is `0`. If `n` had any other set bits (i.e. it isn't a power of two), those remaining bits survive the AND and the result is non-zero. So: **a number is a power of two if and only if `n & (n - 1) == 0`.**

## 🖼️ Visualizing it

```
n = 16 (10000)  -- one set bit
n - 1 = 15 (01111)
10000 & 01111 = 00000  -> zero -> IS a power of two
```

```
n = 13 (1101)  -- three set bits
n - 1 = 12 (1100)
1101 & 1100 = 1100  -> non-zero -> NOT a power of two
```

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Compute `n & (n - 1)`.
> 2. If the result equals `0`, `n` is a power of two; otherwise it isn't.

## 👨‍💻 Code

### Java

```java
public class PowerOfTwoCheck {

    public boolean isPowerOfTwo(int n) {
        if (n <= 0) return false;   // 0 and negatives are never powers of two
        return (n & (n - 1)) == 0;
    }
}
```

### Python

```python
def is_power_of_two(n: int) -> bool:
    if n <= 0:
        return False   # 0 and negatives are never powers of two
    return (n & (n - 1)) == 0
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(1).
> - **Space:** O(1).

## ⚠️ Common Pitfalls

> [!warning]
> - **Forgetting to special-case `n <= 0`.** `n = 0` gives `0 & (-1)`, which — depending on how negative numbers/underflow are handled in a given language — can incorrectly evaluate to `0` and be misreported as "a power of two." `0` is never a power of two, and neither is any negative number.
> - **Converting to binary and manually counting set bits instead.** Correct, but unnecessarily slow — the whole point of this trick is avoiding an O(log N) bit-counting pass in favor of a single O(1) check.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Remove the Last Set Bit](07-Remove-the-Last-Set-Bit.md)
> - [↑ Back to Bit Manipulation Index](index.md)
