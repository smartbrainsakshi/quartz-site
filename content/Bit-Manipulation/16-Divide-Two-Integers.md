---
title: "Divide Two Integers"
tags: [bit-manipulation, shift, overflow]
difficulty: Hard
---

# Divide Two Integers

## 🎯 What problem are we solving?

> [!question]
> Given a `dividend` and a `divisor` (both in the 32-bit integer range, divisor never `0`), compute their integer division — **truncated toward zero** — **without using the multiplication, division, or modulo operators.**

## 💡 Intuition

> [!tip]
> Repeatedly subtracting the divisor one copy at a time (`sum += divisor` until it would exceed the dividend, counting steps) works, but is O(dividend / divisor) — far too slow for large inputs (e.g. dividing by `1`).
>
> The speedup: instead of subtracting the divisor **one copy at a time**, subtract the **largest possible power-of-two multiple of the divisor** at each step. Any quotient can be decomposed into a sum of powers of two (its own binary representation) — e.g. `22 / 3 = 7`, and `7 = 4 + 2 + 1`, so `22 = 3×4 + 3×2 + 3×1 = 12 + 6 + 3 + (1 left over)`. Concretely: find the largest `k` such that `divisor × 2^k ≤ remaining dividend`, subtract that amount, add `2^k` to the answer, and repeat with what's left — each round removes a much bigger chunk than plain repeated subtraction, collapsing the total number of steps to logarithmic.
>
> Signs are handled separately: compute the answer treating both operands as **positive**, then negate the result at the end if exactly one of the original operands was negative.

## 🖼️ Visualizing it

```
dividend = 22, divisor = 3

n = 22 (remaining), d = 3, answer = 0

Find the largest 2^count such that 3 * 2^count <= 22:
  3*2^0=3 (<=22, keep checking)  3*2^1=6 (<=22)  3*2^2=12 (<=22)  3*2^3=24 (>22, stop)
  -> largest is count=2 (3*4=12)
  answer += 2^2 = 4 -> answer = 4
  n -= 12 -> n = 10

Repeat with n=10:
  3*2^0=3 (<=10)  3*2^1=6 (<=10)  3*2^2=12 (>10, stop)
  -> largest is count=1 (3*2=6)
  answer += 2^1 = 2 -> answer = 6
  n -= 6 -> n = 4

Repeat with n=4:
  3*2^0=3 (<=4)  3*2^1=6 (>4, stop)
  -> largest is count=0 (3*1=3)
  answer += 2^0 = 1 -> answer = 7
  n -= 3 -> n = 1

n=1 < divisor=3 -> stop. answer = 7
```

**Result: 7** — matches `22 / 3 = 7` (truncated).

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. Handle the trivial case: if `dividend == divisor`, return `1`.
> 2. Determine the sign of the result: negative if exactly one of `dividend`/`divisor` is negative; positive otherwise.
> 3. Work with `n = abs(dividend)` and `d = abs(divisor)` from here on (using a wider type like `long` to safely hold `abs(INT_MIN)`, which overflows a 32-bit `int`).
> 4. While `n >= d`:
>    - Find the largest `count` such that `d << (count + 1) ≤ n` (i.e. keep doubling `d` via left shift while it still fits).
>    - Add `1 << count` to the answer.
>    - Subtract `d << count` from `n`.
> 5. Apply the sign determined in step 2.
> 6. Clamp: if the (positive) answer exceeds `INT_MAX` and the sign is positive, return `INT_MAX`; if it exceeds the magnitude bound and the sign is negative, return `INT_MIN`.

## 👨‍💻 Code

### Java

```java
public class DivideTwoIntegers {

    public int divide(int dividend, int divisor) {
        if (dividend == divisor) return 1;

        boolean isPositive = (dividend > 0) == (divisor > 0);

        long n = Math.abs((long) dividend);
        long d = Math.abs((long) divisor);
        long answer = 0;

        while (n >= d) {
            int count = 0;
            while (n >= (d << (count + 1))) {
                count++;
            }
            answer += (1L << count);
            n -= (d << count);
        }

        if (!isPositive) answer = -answer;

        if (answer > Integer.MAX_VALUE) return Integer.MAX_VALUE;
        if (answer < Integer.MIN_VALUE) return Integer.MIN_VALUE;
        return (int) answer;
    }
}
```

### Python

```python
def divide(dividend: int, divisor: int) -> int:
    if dividend == divisor:
        return 1

    is_positive = (dividend > 0) == (divisor > 0)

    n = abs(dividend)
    d = abs(divisor)
    answer = 0

    while n >= d:
        count = 0
        while n >= (d << (count + 1)):
            count += 1
        answer += (1 << count)
        n -= (d << count)

    if not is_positive:
        answer = -answer

    INT_MAX, INT_MIN = 2**31 - 1, -2**31
    if answer > INT_MAX:
        return INT_MAX
    if answer < INT_MIN:
        return INT_MIN
    return answer
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(log(dividend) × log(dividend)) — the outer loop removes a power-of-two-sized chunk each time (logarithmic number of outer iterations), and each outer iteration's inner loop (finding the largest fitting power of two) is itself logarithmic.
> - **Space:** O(1).

## ⚠️ Common Pitfalls

> [!warning]
> - **Not using a wider type (e.g. `long`) for intermediate values.** `abs(INT_MIN)` overflows a 32-bit `int` (since `INT_MIN`'s magnitude is `2^31`, one more than `INT_MAX`) — this single edge case is the most common source of bugs in this problem.
> - **Forgetting the clamp to `INT_MAX`/`INT_MIN` at the end.** `INT_MIN / -1` mathematically equals `2^31`, which itself overflows a 32-bit signed int — the problem expects this to be clamped to `INT_MAX`, not silently wrapped or left as an out-of-range value.
> - **Using the multiplication, division, or modulo operators anywhere** (including implicitly, e.g. via `Math.pow`) — the whole point of the exercise is restricting to addition, subtraction, and bit shifts.
> - **Determining the sign incorrectly.** The result is negative if and only if *exactly one* of `dividend`/`divisor` is negative — both positive or both negative both yield a positive result.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Introduction to Bit Manipulation](01-Introduction-to-Bit-Manipulation.md) — specifically left shift as multiplication by powers of two, and `INT_MAX`/`INT_MIN` overflow reasoning
> - This wraps up the Bit Manipulation section
> - [↑ Back to Bit Manipulation Index](index.md)
