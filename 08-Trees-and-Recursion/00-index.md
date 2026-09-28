---
title: "Trees & Recursion — Pattern Primer"
tags: [moc, tree, recursion]
---

# Trees & Recursion

Tree problems generally split into two families: **recursive divide-and-conquer** (build/transform a tree or grid by combining solutions to sub-regions) and **BFS level-order traversal** (when the question is about level/position rather than pure structure).

## Divide and Conquer on Trees/Grids
Recurse on a region, check a base condition (uniformity, a leaf, an empty subtree), and if it doesn't hold, split into sub-regions, solve each recursively, and combine. The combining step defines the shape of the result (e.g., building an internal Quad Tree node from four children).

- Recognize it when: the problem defines a structure recursively in terms of smaller versions of itself (grid quadrants, subtrees).

## BFS Level-Order with Placeholders
When a property depends on *position* (is a node too far right, is a level partially filled), don't skip missing children during traversal — push explicit `None` placeholders so the traversal order encodes position, not just values.

- Recognize it when: the problem talks about "levels," "as far left as possible," or shape/positional constraints rather than just node values.

## Problems in this folder
- [[Construct-Quad-Tree]] — divide and conquer over a grid
- [[Check-Completeness-of-a-Binary-Tree]] — BFS with null placeholders to detect gaps

## See also
- [[Number-of-Islands]] and [[Walls-and-Gates]] for BFS/DFS on grids (a close cousin of tree recursion).
- [[Minimum-Path-Sum]] for grid-based DP, a different way to process 2D structures.
