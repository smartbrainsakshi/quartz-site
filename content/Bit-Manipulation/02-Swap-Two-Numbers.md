---
title: "Swap Two Numbers"
tags: [bit-manipulation, xor]
difficulty: Easy
---

# Swap Two Numbers

## 🎯 What problem are we solving?

> [!question]
> Given two numbers `a` and `b`, swap their values **without using a third (temporary) variable**.

## 💡 Intuition

> [!tip]
> XOR-ing a number with itself gives `0` (`x ^ x = 0`), and XOR-ing anything with `0` gives back the original (`x ^ 0 = x`). Chaining three XOR assignments exploits this: `a = a ^ b` first "packs" both values into `a` — then unpacking that pack against the other variable recovers each original value in the other slot, one at a time.

## 🖼️ Visualizing it

```
a = 5, b = 6

step 1: a = a ^ b        -> a = 5^6 = 3   (b unchanged: 6)
step 2: b = a ^ b        -> b = 3^6 = 5   (this "5" is the original a!)
step 3: a = a ^ b        -> a = 3^5 = 6   (this "6" is the original b!)

Result: a = 6, b = 5 -- swapped, no third variable used.
```

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. `a = a ^ b`
> 2. `b = a ^ b` — this now equals the *original* `a` (since the packed `a` XOR `b` cancels `b` out).
> 3. `a = a ^ b` — this now equals the *original* `b` (since the packed `a` XOR the just-recovered original `a` cancels down to `b`).

## 👨‍💻 Code

### Java

```java
public class SwapTwoNumbers {

    public void swap(int[] nums, int i, int j) {
        if (i == j) return;   // guard against self-swap (see pitfalls)
        nums[i] = nums[i] ^ nums[j];
        nums[j] = nums[i] ^ nums[j];
        nums[i] = nums[i] ^ nums[j];
    }
}
```

### Python

```python
def swap(nums: list[int], i: int, j: int) -> None:
    if i == j:
        return   # guard against self-swap (see pitfalls)
    nums[i] = nums[i] ^ nums[j]
    nums[j] = nums[i] ^ nums[j]
    nums[i] = nums[i] ^ nums[j]
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(1) — three fixed bitwise operations.
> - **Space:** O(1) — no extra variable used at all.

## ⚠️ Common Pitfalls

> [!warning]
> - **Swapping a value with itself (`swap(arr, i, i)` where both indices refer to the same memory location).** `x ^ x = 0`, so `a = a ^ a` zeroes the value out at the very first step, permanently destroying it. Always guard against `i == j` (or `&a == &b`) before applying the XOR swap.
> - **Forgetting this only works cleanly for integer types.** XOR swap doesn't generalize to floating-point numbers the same way.
> - **Using the third-variable version and thinking it's "worse."** It isn't — it's clearer and safer in production code. The XOR trick is primarily an interview/embedded-systems technique for when a temp variable is genuinely unavailable or costly.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Introduction to Bit Manipulation](01-Introduction-to-Bit-Manipulation.md) — specifically the XOR operator
> - [↑ Back to Bit Manipulation Index](index.md)
