---
title: Bit Manipulation
description: Striver's Bit Manipulation playlist — binary/decimal conversion, the core bitwise operators, and the classic bit-trick interview problems built on top of them.
---

# Bit Manipulation

Notes built from take U forward's Bit Manipulation playlist (Strivers A2Z DSA Course). Covers how numbers are actually stored in memory, the 5 core bitwise operators, and the recurring bit-trick patterns that show up across interview problems.

## Notes in this section — 16/16 ✅

- [x] [Introduction to Bit Manipulation](01-Introduction-to-Bit-Manipulation.md) — *conversions, two's complement, the 5 operators*
- [x] [Swap Two Numbers](02-Swap-Two-Numbers.md)
- [x] [Check if the ith Bit is Set](03-Check-if-the-ith-Bit-is-Set.md)
- [x] [Set the ith Bit](04-Set-the-ith-Bit.md)
- [x] [Clear the ith Bit](05-Clear-the-ith-Bit.md)
- [x] [Toggle the ith Bit](06-Toggle-the-ith-Bit.md)
- [x] [Remove the Last Set Bit](07-Remove-the-Last-Set-Bit.md)
- [x] [Check if a Number is a Power of Two](08-Check-if-a-Number-is-a-Power-of-Two.md)
- [x] [Count the Number of Set Bits](09-Count-the-Number-of-Set-Bits.md)
- [x] [Minimum Bit Flips to Convert a Number](10-Minimum-Bit-Flips-to-Convert-a-Number.md)
- [x] [Power Set (Print All Subsets)](11-Power-Set-Print-All-Subsets.md)
- [x] [Single Number I](12-Single-Number-I.md)
- [x] [Single Number II](13-Single-Number-II.md)
- [x] [Single Number III](14-Single-Number-III.md)
- [x] [XOR of Numbers in a Given Range](15-XOR-of-Numbers-in-a-Given-Range.md)
- [x] [Divide Two Integers](16-Divide-Two-Integers.md)

✅ **Section complete (16/16)**

## The 5 core operators, at a glance

| Operator | Symbol | Rule |
|---|---|---|
| AND | `&` | All bits `1` → `1`, otherwise `0` |
| OR | `\|` | Any bit `1` → `1`, otherwise `0` |
| XOR | `^` | Odd number of `1`s → `1`, even → `0` |
| Right Shift | `>>` | `x >> k` = `x / 2^k` |
| Left Shift | `<<` | `x << k` = `x * 2^k` |
| NOT | `~` | Flips every bit; result depends on two's complement if negative |

**Recurring threads across the playlist:**
- `n & (n - 1)` removes the rightmost set bit — the basis for power-of-two checks, set-bit counting (Brian Kernighan's algorithm), and separating numbers into "buckets" by a differing bit.
- XOR of identical values cancels to `0` — the basis for swapping without a temp variable, and every "find the number that appears differently" problem (Single Number I/II/III).
- `1 << i` isolates/targets a single bit position — the basis for checking, setting, clearing, and toggling the *i*th bit.

[← Back to All Topics](../index.md)
