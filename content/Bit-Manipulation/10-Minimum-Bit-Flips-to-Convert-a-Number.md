---
title: "Minimum Bit Flips to Convert a Number"
tags: [bit-manipulation, xor, brian-kernighan]
difficulty: Easy
---

# Minimum Bit Flips to Convert a Number

## 🎯 What problem are we solving?

> [!question]
> Given a positive integer `start` and a non-negative integer `goal`, find the minimum number of bit flips needed to convert `start` into `goal`.

## 💡 Intuition

> [!tip]
> AND and OR can't distinguish "these bits differ" from "these bits are both something" — but **XOR can**, since it's specifically `1` exactly where two bits *disagree* and `0` where they agree. So `start ^ goal` produces a number whose set bits mark **precisely the positions where `start` and `goal` differ** — exactly the bit positions that need to be flipped to turn one into the other.
>
> Once that's understood, the rest of the problem collapses into [Count the Number of Set Bits](09-Count-the-Number-of-Set-Bits.md): the answer is simply the number of `1`s in `start ^ goal`.

## 🖼️ Visualizing it

```
start = 10 (1010), goal = 7 (0111)

start ^ goal:
1010
^0111
-----
1101   -> 3 set bits -> answer = 3

(Positions 0, 2, 3 differ between 10 and 7; position 1 agrees -- both have 1 there.)
```

```
start = 3 (011), goal = 4 (100)

011 ^ 100 = 111 -> 3 set bits -> answer = 3
```

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Compute `diff = start ^ goal`.
> 2. Count the set bits in `diff` (Brian Kernighan's algorithm: repeatedly `diff = diff & (diff - 1)`, counting iterations until `diff` becomes `0`).
> 3. Return the count.

## 👨‍💻 Code

### Java

```java
public class MinimumBitFlips {

    public int minBitFlips(int start, int goal) {
        int diff = start ^ goal;
        int count = 0;

        while (diff != 0) {
            diff = diff & (diff - 1);
            count++;
        }
        return count;
    }
}
```

### Python

```python
def min_bit_flips(start: int, goal: int) -> int:
    diff = start ^ goal
    count = 0

    while diff != 0:
        diff = diff & (diff - 1)
        count += 1

    return count
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(number of set bits in `start ^ goal`) using Brian Kernighan's algorithm — or O(log(start ^ goal)) using the simpler per-position scan, since the loop only needs to run until the XOR value's highest set bit has been passed (not a fixed 31 iterations).
> - **Space:** O(1).

## ⚠️ Common Pitfalls

> [!warning]
> - **Trying to count differing bits by comparing `start` and `goal` digit-by-digit after separately converting each to binary.** Unnecessary — XOR does the "find where they differ" step directly and in one operation.
> - **Using AND or OR instead of XOR.** Neither can express "these two bits disagree" the way XOR can — this is precisely the property that makes XOR the right tool here.
> - **Iterating a hardcoded 32 times instead of stopping once the XOR value reaches `0`.** Wastes time on numbers that need far fewer bit checks.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Count the Number of Set Bits](09-Count-the-Number-of-Set-Bits.md), [Remove the Last Set Bit](07-Remove-the-Last-Set-Bit.md)
> - [↑ Back to Bit Manipulation Index](index.md)
