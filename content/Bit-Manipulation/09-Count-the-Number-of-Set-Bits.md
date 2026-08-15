---
title: "Count the Number of Set Bits"
tags: [bit-manipulation, and, shift, brian-kernighan]
difficulty: Easy
---

# Count the Number of Set Bits

## 🎯 What problem are we solving?

> [!question]
> Given a number `n`, count how many bits in its binary representation are `1`.

## 💡 Intuition

> [!tip]
> Two approaches, both simple:
>
> **Approach 1 — check every bit position.** The last bit of any odd number is always `1` (every odd number is "some even base + 1"); the last bit of an even number is always `0`. So `n & 1` cheaply tells you the last bit, and `n >> 1` (equivalent to `n / 2`) slides the next bit into that position. Loop until `n` becomes `0`, summing up `n & 1` at each step. This costs one iteration **per bit position** — up to 31 iterations for a 32-bit int, regardless of how many bits are actually set.
>
> **Approach 2 — Brian Kernighan's algorithm.** Recall `n & (n - 1)` clears exactly the rightmost set bit ([Remove the Last Set Bit](07-Remove-the-Last-Set-Bit.md)). Repeatedly apply it, counting each application, until `n` becomes `0` — the number of applications *is* the number of set bits. This costs one iteration **per set bit**, not per bit position — strictly faster whenever `n` has fewer set bits than total bit width (i.e. almost always).

## 🖼️ Visualizing it

```
n = 13 (1101), using Brian Kernighan's algorithm:

n=1101, n-1=1100, n&(n-1)=1100 (12)   count=1
n=1100, n-1=1011, n&(n-1)=1000 (8)    count=2
n=1000, n-1=0111, n&(n-1)=0000 (0)    count=3
n=0 -> stop

Result: 3 set bits
```

Compare to Approach 1, which would run a fixed 4 iterations here (down to the point where `n` hits `1`) regardless of how many of those bits were actually `1`.

## 🛠️ Algorithm / Approach

> [!abstract]
> **Approach 1:**
> 1. `count = 0`. While `n != 0`: if `n & 1` is `1`, increment `count`; then `n = n >> 1`.
> 2. Return `count`.
>
> **Approach 2 (Brian Kernighan's — preferred):**
> 1. `count = 0`. While `n != 0`: `n = n & (n - 1)`; increment `count`.
> 2. Return `count`.

## 👨‍💻 Code

### Java

```java
public class CountSetBits {

    public int countBitsApproach1(int n) {
        int count = 0;
        while (n != 0) {
            count += (n & 1);
            n = n >> 1;
        }
        return count;
    }

    public int countBitsBrianKernighan(int n) {
        int count = 0;
        while (n != 0) {
            n = n & (n - 1);
            count++;
        }
        return count;
    }
}
```

### Python

```python
def count_bits_approach1(n: int) -> int:
    count = 0
    while n != 0:
        count += (n & 1)
        n >>= 1
    return count

def count_bits_brian_kernighan(n: int) -> int:
    count = 0
    while n != 0:
        n = n & (n - 1)
        count += 1
    return count
```

## ⏱️ Complexity Analysis

> [!note]
> - **Approach 1 — Time:** O(log N) (~31 iterations at worst for a 32-bit int, regardless of the actual number of set bits). **Space:** O(1).
> - **Brian Kernighan's — Time:** O(number of set bits) — strictly better whenever `n` isn't close to having every bit set. **Space:** O(1).
> - C++'s STL provides a built-in `__builtin_popcount(n)` for this exact operation — worth mentioning to an interviewer as a real-world shortcut, even when asked to implement it manually.

## ⚠️ Common Pitfalls

> [!warning]
> - **Using `n % 2` instead of `n & 1`.** Functionally equivalent, but bitwise AND is the idiomatic choice here and is what competitive programmers reach for by default (also true for `n / 2` vs `n >> 1`).
> - **Not recognizing when Brian Kernighan's algorithm is worth using.** For numbers with few set bits relative to their bit width, it's a meaningful constant-factor win over the naive per-position scan — worth explicitly mentioning as the "optimized" follow-up in an interview.
> - **Off-by-one assumptions about loop bounds in Approach 1.** The loop should run until `n` becomes exactly `0`, not for a hardcoded 32 iterations — hardcoding wastes time on numbers that need far fewer bits.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Remove the Last Set Bit](07-Remove-the-Last-Set-Bit.md)
> - This wraps up the "bit-trick toolbox" video — next up: [Minimum Bit Flips to Convert a Number](10-Minimum-Bit-Flips-to-Convert-a-Number.md), which uses set-bit counting directly
> - [↑ Back to Bit Manipulation Index](index.md)
