---
title: "Single Number III"
tags: [bit-manipulation, xor, partitioning]
difficulty: Medium
---

# Single Number III

## 🎯 What problem are we solving?

> [!question]
> Given an array where every number appears **exactly twice** except for **two distinct numbers**, which each appear exactly once, find those two numbers (in either order).

## 💡 Intuition

> [!tip]
> XOR-ing the entire array still cancels every properly-paired duplicate, but this time it leaves behind `a ^ b` — the XOR of the *two* unique numbers, not a clean single value. That combined XOR is still useful, though: since `a` and `b` are distinct, `a ^ b` is guaranteed **non-zero**, and every bit that's set in it marks a position where `a` and `b` **differ**.
>
> Pick *any* one such differing bit — the standard choice is the **rightmost set bit** of `a ^ b` (found the same way as [Remove the Last Set Bit](07-Remove-the-Last-Set-Bit.md): `diff & (-diff)`, or equivalently `diff & ~(diff - 1)`). Use that bit to **partition the entire array into two buckets**: numbers with that bit set go in bucket 1, numbers with that bit unset go in bucket 2. Because `a` and `b` differ at exactly that bit, they're guaranteed to land in *different* buckets. Every properly-paired duplicate, meanwhile, always lands in the *same* bucket as its partner (both copies of any given number obviously share the same bits). XOR each bucket independently — the duplicates cancel within their bucket, leaving exactly `a` in one bucket's result and `b` in the other's.

## 🖼️ Visualizing it

```
nums = [1, 2, 1, 3, 2, 5]   (1 and 2 are duplicated; 3 and 5 are the two unique numbers)

Step 1: XOR everything -> 1^2^1^3^2^5 = 3^5 = 011 ^ 101 = 110 (6)
  (this is a^b, where a=3, b=5)

Step 2: find the rightmost set bit of 6 (110) -> bit 1 (010)

Step 3: partition by whether bit 1 is set:
  3 (011): bit 1 set    -> bucket 1
  5 (101): bit 1 unset  -> bucket 2
  1 (001): bit 1 unset  -> bucket 2
  1 (001): bit 1 unset  -> bucket 2
  2 (010): bit 1 set    -> bucket 1
  2 (010): bit 1 set    -> bucket 1

  bucket 1: {3, 2, 2} -> XOR -> 3^2^2 = 3
  bucket 2: {5, 1, 1} -> XOR -> 5^1^1 = 5
```

**Result: `{3, 5}`** — both duplicated pairs (1s and 2s) canceled correctly within their buckets, leaving exactly the two unique numbers.

## 🛠️ Algorithm / Approach

> [!abstract]
> 1. XOR the entire array into `xorAll` — this equals `a ^ b`.
> 2. Isolate the rightmost set bit: `rightmostSetBit = xorAll & (-xorAll)`.
> 3. Initialize `bucket1 = 0`, `bucket2 = 0`. For each number: if `num & rightmostSetBit` is non-zero, XOR it into `bucket1`; otherwise XOR it into `bucket2`.
> 4. Return `{bucket1, bucket2}` — the two unique numbers, in either order.

## 👨‍💻 Code

### Java

```java
public class SingleNumberIII {

    public int[] singleNumberIII(int[] nums) {
        int xorAll = 0;
        for (int num : nums) {
            xorAll ^= num;
        }

        int rightmostSetBit = xorAll & (-xorAll);

        int bucket1 = 0, bucket2 = 0;
        for (int num : nums) {
            if ((num & rightmostSetBit) != 0) {
                bucket1 ^= num;
            } else {
                bucket2 ^= num;
            }
        }
        return new int[]{bucket1, bucket2};
    }
}
```

### Python

```python
def single_number_iii(nums: list[int]) -> list[int]:
    xor_all = 0
    for num in nums:
        xor_all ^= num

    rightmost_set_bit = xor_all & (-xor_all)

    bucket1, bucket2 = 0, 0
    for num in nums:
        if num & rightmost_set_bit:
            bucket1 ^= num
        else:
            bucket2 ^= num

    return [bucket1, bucket2]
```

## ⏱️ Complexity Analysis

> [!note]
> - **Time:** O(N) — two linear passes over the array.
> - **Space:** O(1) — no data structure needed, just a few integer accumulators.

## ⚠️ Common Pitfalls

> [!warning]
> - **Trying to apply the plain [Single Number I](12-Single-Number-I.md) trick directly.** XOR-ing everything only gives `a ^ b`, not either individual value — the partitioning step is what's needed to actually separate them.
> - **Picking an arbitrary bit position instead of one that's actually set in `xorAll`.** Only a bit where `a` and `b` genuinely differ is guaranteed to separate them into different buckets — an arbitrary or unset bit position could put both in the same bucket, breaking the whole approach.
> - **Forgetting `a ^ b` is guaranteed non-zero.** This relies on `a` and `b` being distinct (per the problem's guarantee) — if they could be equal, this whole technique wouldn't apply.
> - **`xorAll & (-xorAll)` on languages without two's-complement negative integers** (or with arbitrary-precision integers behaving differently) — verify this idiom behaves as expected in the target language; it relies on `-xorAll` being `xorAll`'s two's complement.

## 🔗 Related Problems / Next Up

> [!success]
> - **Builds on:** [Single Number I](12-Single-Number-I.md), [Single Number II](13-Single-Number-II.md)
> - [↑ Back to Bit Manipulation Index](index.md)
