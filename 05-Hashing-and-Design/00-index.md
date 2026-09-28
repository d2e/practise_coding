---
title: "Hashing & Design — Pattern Primer"
tags: [moc, hashing, design]
---

# Hashing & System Design (Class Design) Problems

Two related but distinct skills live in this folder:

## Hashing for Grouping / Connectivity
Hash maps turn "does this share something with that?" into an O(1) lookup. When the sharing relationship is transitive (A shares with B, B shares with C, therefore A, B, C are all one group), a hash map alone isn't enough — you need **Union-Find** on top of it to track connected components efficiently as unions are discovered incrementally.

- Recognize it when: "merge groups that share X," "connected if," or "same person/entity if any overlap."

## Design Problems (Constructor + Repeated Queries)
A distinct shape: the problem gives you data once (a constructor) and then asks for many repeated queries against that fixed data. The right mental model is always the same: **push as much work as possible into the constructor**, so each query is cheap (ideally O(log N) or better). Two common toolkits for the query side:
- **Prefix sums + binary search** — for weighted sampling or range-sum queries ([[Random-Pick-with-Weight]]).
- **Sorted index lists + two-pointer merge** — for "closest pair across two sets" queries ([[Shortest-Word-Distance-II]]).

- Recognize it when: the problem statement literally says "design a class" or shows a constructor signature followed by method calls that will be invoked many times.

## Problems in this folder
- [[Accounts-Merge]] — Union-Find over emails
- [[Shortest-Word-Distance-II]] — preprocessed index lists + two-pointer merge
- [[Random-Pick-with-Weight]] — prefix sum + binary search sampling

## See also
- [[Number-of-Islands]] and [[Word-Ladder]] for more connectivity/graph problems.
- [[Merge-Sorted-Array]] for the base two-pointer merge technique used in Shortest Word Distance II.
