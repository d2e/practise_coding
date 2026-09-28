---
title: "Linked Lists — Pattern Primer"
tags: [moc, linked-list]
---

# Linked Lists

Linked list problems are rarely about the data structure itself — they're about pointer manipulation under constraints (one pass, O(1) extra space, no peeking at length up front). The two dominant techniques in this folder:

## Two Pointers (Fast/Slow, or Fixed Gap)
Used when you need to find something relative to the *end* of the list, detect a cycle, or find a middle node, all without knowing the list's length ahead of time. The core trick: give one pointer a head start (in steps, or in speed), then move both together. The invariant (the gap, or the 2x speed difference) does the work for you.

- Recognize it when: the problem says "in one pass," "without knowing the length," or involves "the middle" / "the nth from the end."
- Watch for the edge case where the node being removed/found is the head — a **dummy node** before the real head almost always simplifies this.

## K-Way Merge (Heap or Divide & Conquer)
Used when you must combine multiple already-sorted sequences into one sorted sequence. A min-heap holding the "current candidate" from each sequence gives you the global minimum in O(log k) per step. Divide-and-conquer pairwise merging achieves the same time complexity with less overhead and is the strategy to reach for once the heap solution is understood.

- Recognize it when: you're given a list of sorted lists/arrays/streams and asked to merge them.

## Problems in this folder
- [[Remove-Nth-Node-From-End-of-List]] — two-pointer fixed gap, dummy node
- [[Merge-k-Sorted-Lists]] — min-heap and divide-and-conquer k-way merge

## See also
- [[K-Closest-Points-to-Origin]] and [[Sort-Integers-by-The-Power-Value]] for more heap practice.
- [[Accounts-Merge]] for a different flavor of "merge" problem (graph/union-find, not sorted sequences).
