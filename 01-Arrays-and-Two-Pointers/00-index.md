---
title: "Arrays & Two Pointers — Pattern Primer"
tags: [moc, pattern-primer, two-pointers, array]
---

# Arrays & Two Pointers

## When to use this pattern
Reach for two pointers whenever you're working with an array (often sorted, or with a fixed structure like "extra space at the end") and can make progress by advancing one or more indices toward each other, or in the same direction, instead of re-scanning the whole array repeatedly. It shows up in three common shapes:

- **Opposite-direction pointers** (`left` at the start, `right` at the end, moving toward each other) — classic for problems on sorted arrays like finding pairs with a target sum, or reversing in place.
- **Same-direction pointers merging from one end** — e.g. merging two sorted arrays by writing from the back when there's guaranteed free space, avoiding overwriting unread data.
- **Fast/slow pointers over positional structure** — used more in linked lists, but the same spirit (one pointer tracks a "write" position, another tracks a "read" position) shows up in plain arrays too (e.g. removing duplicates in place).

Alongside two pointers, this folder also covers general **array manipulation techniques**: in-place matrix transforms (decomposing a rotation into transpose + reverse), hash-set-based geometric lookups, and prefix-sum tricks for "what if I remove one element" questions.

## How to recognize it in a problem statement
- "In-place," "no extra array," "O(1) extra space" — signals you should look for a two-pointer or in-place transform instead of allocating a new structure.
- "Sorted array" + "find a pair/triplet" — almost always two pointers from both ends.
- "What is the minimum/maximum difference between any two elements" — sort first, then only adjacent elements in sorted order matter.
- "Remove/insert one element, how does some property change" — prefix/suffix sums split by the relevant property (parity, running total) turn an O(n) per-query brute force into O(1) per query.

## Generic template (opposite-direction two pointers)
```python
left, right = 0, len(arr) - 1
while left < right:
    if <condition met>:
        # do something, possibly move both
        left += 1
        right -= 1
    elif <need bigger value>:
        left += 1
    else:
        right -= 1
```

## Problems in this folder
- [[Merge-Sorted-Array]] — merge from the back to avoid overwriting unread data.
- [[Rotate-Image]] — decompose a 90° rotation into transpose + row-reverse, both in-place.
- [[Minimum-Area-Rectangle]] — hash set + enumerate diagonal corner pairs.
- [[Ways-to-Make-a-Fair-Array]] — prefix/suffix sums split by even/odd index parity.
- [[Minimum-Absolute-Difference]] — sort, then only adjacent pairs can be extremal.
