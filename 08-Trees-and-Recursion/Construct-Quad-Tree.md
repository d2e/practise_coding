---
title: "Construct Quad Tree"
aliases: []
tags: [leetcode, tree, divide-and-conquer, recursion, medium]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 427
leetcode_difficulty: Medium
pattern: "Divide and Conquer on a Grid"
status: not-started
---

# Construct Quad Tree

## Problem Statement
Given a `n x n` binary grid (where `n` is a power of 2), construct a Quad Tree representation of it. A Quad Tree node has two possible forms:
- A **leaf** node, storing a boolean `val` and `isLeaf = True`, when the region it represents is entirely 0s or entirely 1s.
- An **internal** node, storing `isLeaf = False` (its `val` is conventionally `True` but unused) and four children — `topLeft`, `topRight`, `bottomLeft`, `bottomRight` — each representing one quadrant of the current region, when the region contains a mix of 0s and 1s.

Return the root of the constructed Quad Tree.

## Examples
```
Input: grid = [[0,1],[1,0]]
Output: internal node with four leaf children:
  topLeft = leaf(0), topRight = leaf(1),
  bottomLeft = leaf(1), bottomRight = leaf(0)

Input: grid = [[1,1],[1,1]]
Output: a single leaf node with val = 1
(the entire grid is uniform, so no subdivision is needed)
```

## Intuition & Approach
This is a direct application of **divide and conquer**: to build the Quad Tree for a region, first check if the whole region is uniform (all 0s or all 1s) — if so, it's a leaf, done. If not, split the region into four equal quadrants (top-left, top-right, bottom-left, bottom-right) and recursively build the Quad Tree for each quadrant, then combine the four resulting subtrees into one internal node.

The "is this region uniform" check can be done naively by scanning every cell in the region and comparing to the first cell — but note that this same region gets scanned again during the "should I recurse" check even after we've already decided to recurse (redundant work is bounded, but it's worth noting). Given `n` is a power of 2 down to a 1x1 base case (always uniform, trivially a leaf), the recursion terminates cleanly since each recursive call operates on a region a quarter the area of its parent, and a 1x1 region is always uniform.

A key detail: the problem guarantees `n` is a power of 2, so every quadrant split is exact (no remainder rows/columns to handle), which is what makes divide-and-conquer clean here — no need for the "off-by-one when splitting an odd-length range" handling you'd need in other grid-recursion problems.

## Pattern Recognition
> [!tip] Pattern: Divide and Conquer on a Grid
> Whenever a problem defines a data structure recursively in terms of "smaller grid regions of the same shape" — Quad Trees, 2D range-uniformity checks, or matrix-based divide and conquer (like Strassen's algorithm) — the recursion is naturally over (row_start, row_end, col_start, col_end) boundaries, splitting each dimension in half at every level.

## Algorithm Steps
1. Define `build(row, col, size)` representing the sub-grid starting at `(row, col)` with side length `size`.
2. Check if all cells in that sub-grid are equal (uniform). If yes, return a leaf `Node(val=grid[row][col], isLeaf=True)`.
3. If not uniform, let `half = size // 2` and recursively build:
   - `topLeft = build(row, col, half)`
   - `topRight = build(row, col + half, half)`
   - `bottomLeft = build(row + half, col, half)`
   - `bottomRight = build(row + half, col + half, half)`
4. Return `Node(val=True, isLeaf=False, topLeft, topRight, bottomLeft, bottomRight)`.
5. Kick off the recursion with `build(0, 0, n)`.

## Solution (Python)
```python
from typing import List, Optional

class Node:
    def __init__(self, val, isLeaf, topLeft=None, topRight=None, bottomLeft=None, bottomRight=None):
        self.val = val
        self.isLeaf = isLeaf
        self.topLeft = topLeft
        self.topRight = topRight
        self.bottomLeft = bottomLeft
        self.bottomRight = bottomRight

class Solution:
    def construct(self, grid: List[List[int]]) -> "Node":
        n = len(grid)

        def is_uniform(row: int, col: int, size: int) -> bool:
            first = grid[row][col]
            for r in range(row, row + size):
                for c in range(col, col + size):
                    if grid[r][c] != first:
                        return False
            return True

        def build(row: int, col: int, size: int) -> "Node":
            if is_uniform(row, col, size):
                return Node(val=bool(grid[row][col]), isLeaf=True)

            half = size // 2
            top_left = build(row, col, half)
            top_right = build(row, col + half, half)
            bottom_left = build(row + half, col, half)
            bottom_right = build(row + half, col + half, half)
            return Node(val=True, isLeaf=False,
                        topLeft=top_left, topRight=top_right,
                        bottomLeft=bottom_left, bottomRight=bottom_right)

        return build(0, 0, n)
```

## Solution (Java)
```java
class Node {
    public boolean val;
    public boolean isLeaf;
    public Node topLeft;
    public Node topRight;
    public Node bottomLeft;
    public Node bottomRight;

    public Node() {
        this.val = false;
        this.isLeaf = false;
    }

    public Node(boolean val, boolean isLeaf) {
        this.val = val;
        this.isLeaf = isLeaf;
    }

    public Node(boolean val, boolean isLeaf, Node topLeft, Node topRight, Node bottomLeft, Node bottomRight) {
        this.val = val;
        this.isLeaf = isLeaf;
        this.topLeft = topLeft;
        this.topRight = topRight;
        this.bottomLeft = bottomLeft;
        this.bottomRight = bottomRight;
    }
}

class Solution {
    private int[][] grid;

    public Node construct(int[][] grid) {
        this.grid = grid;
        return build(0, 0, grid.length);
    }

    private boolean isUniform(int row, int col, int size) {
        int first = grid[row][col];
        for (int r = row; r < row + size; r++) {
            for (int c = col; c < col + size; c++) {
                if (grid[r][c] != first) {
                    return false;
                }
            }
        }
        return true;
    }

    private Node build(int row, int col, int size) {
        if (isUniform(row, col, size)) {
            return new Node(grid[row][col] == 1, true);
        }

        int half = size / 2;
        Node topLeft = build(row, col, half);
        Node topRight = build(row, col + half, half);
        Node bottomLeft = build(row + half, col, half);
        Node bottomRight = build(row + half, col + half, half);
        return new Node(true, false, topLeft, topRight, bottomLeft, bottomRight);
    }
}
```

## Complexity Analysis
- **Time:** O(n^2 log n) in the naive version shown — at each of the O(log n) recursion levels, the uniformity check across all cells does O(n^2) total work per level (since each level's checks together cover the grid, possibly with some overlap from failed uniform checks). This can be improved to O(n^2) total using a 2D prefix-sum of 1s to check uniformity in O(1) per region instead of scanning it.
- **Space:** O(n^2) for the output tree in the worst case (a checkerboard pattern forces subdivision all the way down to 1x1 leaves); O(log n) recursion stack depth.

## Edge Cases
- A fully uniform grid (all 0s or all 1s) — should return a single leaf node with no children, without any unnecessary subdivision.
- A "checkerboard" grid where every cell differs from its neighbors — forces subdivision all the way down to 1x1 leaves, maximizing tree size.
- `n == 1` (a single cell) — trivially a leaf, base case of the recursion.

## Common Mistakes
- Forgetting that an internal node's `val` is conventionally unused/`True` and mistakenly trying to compute a "representative value" for it.
- Off-by-one errors when computing quadrant boundaries — relying on `n` being a power of 2 (guaranteed by the problem) to make `half = size // 2` split evenly; this approach would break on odd-sized grids.
- Re-scanning the same cells redundantly across the uniform-check without memoizing or using a prefix-sum optimization, which is fine for LeetCode's constraints but worth flagging as an optimization opportunity in an interview.

## Related Problems
- [[Check-Completeness-of-a-Binary-Tree]]
- [[Number-of-Islands]]
- [[Minimum-Path-Sum]]

## Follow-up / Variants
A common optimization follow-up: precompute a 2D prefix-sum array of the grid so that "is this region uniform" and "what's the sum of this region" both become O(1) queries, dropping total construction time to O(n^2). Another variant asks you to implement "Quad Tree Intersection" (LC 558), which merges two Quad Trees representing an OR of their grids without ever materializing the underlying grids.
