---
title: "XOR of Numbers in a Given Range"
tags: [bit-manipulation, xor, pattern-recognition]
difficulty: Medium
---

# XOR of Numbers in a Given Range

## 🎯 What problem are we solving?

> [!question]
> Given an integer `n`, find the XOR of all numbers from `1` to `n`. As a natural follow-up: given `L` and `R`, find the XOR of all numbers from `L` to `R`.

## 💡 Intuition

> [!tip]
> The brute-force loop (`answer = answer ^ i` for `i` from `1` to `n`) is O(N) — but XOR-ing `1` through `n` turns out to follow a clean **repeating pattern based on `n % 4`**, discoverable by writing out the first several values by hand:
> ```
> n=1: 1        n=5: 1        n=9: 1
> n=2: 3        n=6: 7        n=10: 11
> n=3: 0        n=7: 0        
> n=4: 4        n=8: 8        
> ```
> The pattern by `n % 4`:
> - `n % 4 == 1` → answer is `1`
> - `n % 4 == 2` → answer is `n + 1`
> - `n % 4 == 3` → answer is `0`
> - `n % 4 == 0` → answer is `n`
>
> This gives an **O(1)** function for "XOR from 1 to n." For the **range** version (`L` to `R`), the same trick used for prefix sums applies to XOR: XOR-ing `f(1..R)` with `f(1..L-1)` cancels every number from `1` to `L-1` (each appears in both and XORs to `0`), leaving exactly the XOR of `L` to `R`.

## 🖼️ Visualizing it

```
XOR from 1 to n, using the pattern:

n=4  (4%4=0) -> answer = n = 4        (check: 1^2^3^4 = 4 ✓)
n=7  (7%4=3) -> answer = 0            (check: 1^2^...^7 = 0 ✓)
```

```
XOR from L=4 to R=7:

f(R) = f(7) = 0        (7%4=3 -> 0)
f(L-1) = f(3) = 0       (3%4=3 -> 0)

answer = f(7) ^ f(3) = 0 ^ 0 = 0

Check by brute force: 4^5^6^7 = 1^6^7 = 7^7 = 0  ✓
```

## 🛠️ Algorithm / Approach

> [!abstract]
> **`xorUpTo(n)` — O(1):**
> 1. If `n % 4 == 1`, return `1`.
> 2. If `n % 4 == 2`, return `n + 1`.
> 3. If `n % 4 == 3`, return `0`.
> 4. Otherwise (`n % 4 == 0`), return `n`.
>
> **`xorInRange(L, R)`:**
> 1. Return `xorUpTo(L - 1) ^ xorUpTo(R)`.

## 👨‍💻 Code

### Java

```java
public class XorInRange {

    public int xorUpTo(int n) {
        switch (n % 4) {
            case 1: return 1;
            case 2: return n + 1;
            case 3: return 0;
            default: return n;   // n % 4 == 0
        }
    }

    public int xorInRange(int l, int r) {
        return xorUpTo(l - 1) ^ xorUpTo(r);
    }
}
```

### Python

```python
def xor_up_to(n: int) -> int:
    remainder = n % 4
    if remainder == 1:
        return 1
    elif remainder == 2:
        return n + 1
    elif remainder == 3:
        return 0
    else:
        return n

def xor_in_range(l: int, r: int) -> int:
    return xor_up_to(l - 1) ^ xor_up_to(r)
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(1) for both `xorUpTo` and `xorInRange` — a fixed handful of comparisons and one extra XOR for the range version. (Compare to the O(N) brute-force loop.)
> - **Space:** O(1).

## ⚠️ Common Pitfalls

> [!warning]
> - **Not recognizing this requires a memorized pattern rather than a derivable-on-the-spot trick.** Like some other patterns in this series, it's most practical to know the `n % 4` rule ahead of time rather than expect to rediscover it live in an interview — writing out the first 8-10 values by hand is the fastest way to internalize it.
> - **Forgetting the `L - 1` in the range formula.** The prefix-cancellation trick requires XOR-ing `f(R)` with `f(L - 1)`, *not* `f(L)` — using `f(L)` would incorrectly cancel `L` itself out of the final range.
> - **Handling `L = 1` incorrectly.** `xorUpTo(0)` should return `0` (an empty XOR range) — verify the `n % 4 == 0` branch handles `n = 0` sensibly, since `0 % 4 == 0` and the "otherwise return n" branch correctly yields `0` here.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Single Number I](12-Single-Number-I.md) — same XOR-cancellation idea, generalized into a range-query pattern
> - [↑ Back to Bit Manipulation Index](index.md)
