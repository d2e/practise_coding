---
title: "Check Completeness of a Binary Tree"
aliases: []
tags: [leetcode, tree, bfs, medium]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 958
leetcode_difficulty: Medium
pattern: "BFS Level-Order with a 'Seen a Gap' Flag"
status: not-started
---

# Check Completeness of a Binary Tree

## Problem Statement
Given the root of a binary tree, determine if it is a **complete binary tree**. In a complete binary tree, every level except possibly the last is completely filled, and all nodes in the last level are as far left as possible (there can be a gap on the right of the last level, but never a gap followed by a node further right).

## Examples
```
Input: root = [1,2,3,4,5,6]
Output: true
(every level is full except the last, and the last level's nodes [4,5,6] are packed left)

Input: root = [1,2,3,5,6,null,7]
Output: false
(node 7 exists at a position to the right of a null — a gap followed by a real node)
```

## Intuition & Approach
The definition of "complete" is fundamentally about the *positions* nodes occupy relative to a gap-free left-packed layout — which is exactly what a standard **level-order BFS traversal** naturally exposes, as long as you push `None` placeholders into the queue for missing children instead of skipping them.

The key insight: walk the tree level by level (BFS), and for every node dequeued — including `None` placeholders — push its two children (or `None` if a child is absent). The tree is complete if and only if, once you encounter the first `None` in this traversal, every node dequeued *after* that point is also `None`. In other words: real nodes must all appear before the first gap; no real node is allowed to appear after a gap.

This works because a complete tree's level-order traversal (with explicit `None` placeholders for missing children) is exactly a sequence of real nodes followed by a run of `None`s — any real node found after the first `None` proves a "hole" exists before it, violating left-packing.

## Pattern Recognition
> [!tip] Pattern: BFS Level-Order with Explicit Null Placeholders
> Whenever a tree property depends on *positional* structure (not just values) — completeness, checking if it matches a specific shape, or serialization comparisons — do a level-order BFS that pushes `None` for missing children rather than skipping them. This turns positional structure into a simple sequential scan.

## Algorithm Steps
1. If `root` is `None`, the empty tree is trivially complete; return `True`.
2. Initialize a queue with `root`.
3. Initialize `seen_null = False`.
4. While the queue is non-empty, pop the front node:
   - If it is `None`, set `seen_null = True`.
   - Else, if `seen_null` is already `True`, a real node appeared after a gap — return `False`. Otherwise, push its `left` and `right` children (even if they're `None`) onto the queue.
5. If the loop completes without returning `False`, return `True`.

## Solution (Python)
```python
from collections import deque
from typing import Optional

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class Solution:
    def isCompleteTree(self, root: Optional[TreeNode]) -> bool:
        if root is None:
            return True

        queue = deque([root])
        seen_null = False

        while queue:
            node = queue.popleft()
            if node is None:
                seen_null = True
                continue

            if seen_null:
                # A real node appeared after a gap: not left-packed.
                return False

            queue.append(node.left)
            queue.append(node.right)

        return True
```

## Solution (Java)
```java
import java.util.LinkedList;
import java.util.Queue;

class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode() {}
    TreeNode(int val) { this.val = val; }
    TreeNode(int val, TreeNode left, TreeNode right) {
        this.val = val;
        this.left = left;
        this.right = right;
    }
}

class Solution {
    public boolean isCompleteTree(TreeNode root) {
        if (root == null) return true;

        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        boolean seenNull = false;

        while (!queue.isEmpty()) {
            TreeNode node = queue.poll();
            if (node == null) {
                seenNull = true;
                continue;
            }

            if (seenNull) {
                // A real node appeared after a gap: not left-packed.
                return false;
            }

            queue.offer(node.left);
            queue.offer(node.right);
        }

        return true;
    }
}
```

## Complexity Analysis
- **Time:** O(N) — every node (and its null-child placeholders) is visited exactly once during the BFS.
- **Space:** O(N) — the queue can hold up to O(N) elements in the worst case (e.g., a wide, shallow tree at its widest level), plus the `None` placeholders enqueued alongside real nodes.

## Edge Cases
- Empty tree (`root is None`) — trivially complete.
- A single node with no children — trivially complete.
- A tree that is "complete" in the everyday sense of "every level full" (a perfect binary tree) — must also return `True`; the algorithm handles this naturally since it never encounters a gap before the very end.
- A left child missing but a right child present at the same node (e.g., node has `right` but not `left`) — this should immediately fail, and the algorithm catches it because `None` (for the missing left) is enqueued before the real `right` child, which is then read as "a real node after a gap" once that `None` is dequeued and a subsequent real node follows.

## Common Mistakes
- Skipping `None` children instead of pushing them into the queue — this loses the positional information needed to detect gaps, since a normal BFS that skips nulls can't distinguish a complete tree from one with holes.
- Confusing "complete binary tree" with "full binary tree" (every node has 0 or 2 children) or "perfect binary tree" (all levels fully filled) — these are different, stricter or unrelated definitions.
- Returning `False` too early on encountering *any* `None`, rather than only failing when a real node is found *after* a `None`.

## Related Problems
- [[Construct-Quad-Tree]]
- [[Word-Ladder]]
- [[Walls-and-Gates]]

## Follow-up / Variants
A natural follow-up is checking whether a tree is a **perfect** binary tree instead (stricter: every level fully filled, tree height known in advance) — solvable by computing depth and node count and checking `count == 2^depth - 1`, without needing the BFS gap-tracking used here.
