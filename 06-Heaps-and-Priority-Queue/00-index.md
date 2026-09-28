---
title: "Heaps & Priority Queue — Pattern Primer"
tags: [moc, heap]
---

# Heaps & Priority Queue

A heap (priority queue) is the go-to structure whenever you repeatedly need "the current min/max" out of a changing collection, without paying the cost of fully sorting.

## Top-K Pattern
To find the K smallest/largest/closest items out of N, maintain a heap of size K with the *opposite* polarity of what you want (a max-heap when hunting for the K smallest, so the root is always the worst of your current K candidates and the one to evict when a better candidate shows up). This is O(N log K), strictly better than sorting everything (O(N log N)) when K is small relative to N. Quickselect is the O(N)-expected alternative once you can mutate the input array and don't need a stable full ordering.

- Recognize it when: "return the K closest / smallest / most frequent," especially when K is much smaller than N or the input streams in.

## Memoized Recursion
Not a heap technique per se, but grouped here because [[Sort-Integers-by-The-Power-Value]] combines a recursive per-element computation with a final sort — recognize when many inputs share overlapping recursive subproblems (chains that merge), and cache aggressively.

## Problems in this folder
- [[K-Closest-Points-to-Origin]] — bounded max-heap, with quickselect follow-up
- [[Sort-Integers-by-The-Power-Value]] — memoized recursion + custom sort key

## See also
- [[Merge-k-Sorted-Lists]] for the other major heap pattern: k-way merge.
- [[Daily-Temperatures]] for a different kind of "keep the best candidate" structure (monotonic stack instead of heap).
