---
title: "Graphs & BFS/DFS — Pattern Primer"
aliases: [Graphs Index]
tags: [pattern-primer, graphs]
---

# Graphs & BFS/DFS

Grid and graph traversal problems all boil down to the same three primitives — **BFS**, **DFS**, and **Union-Find** — combined with a small number of recurring framings. Recognizing *which* framing a problem is (rather than just "it's a graph, I'll BFS it") is most of the battle.

## When to reach for each traversal
- **BFS** — whenever the problem asks for a *shortest* path/sequence/number-of-steps in an **unweighted** graph (or a graph where every edge costs the same). BFS explores in strictly increasing distance order, so the first time you reach a target node, you have the shortest path to it, guaranteed.
- **DFS** — when you just need *any* path, need to explore/enumerate all paths, or need to detect structure like cycles or connected components and the order of exploration doesn't matter. Also natural for recursive tree/grid flood-fills.
- **Dijkstra (min-heap)** — the moment edges have **different weights/costs**. BFS's FIFO-queue shortest-path guarantee only holds for uniform edge weights; as soon as costs vary, you need a priority queue that always expands the currently-cheapest frontier node.
- **Topological Sort (Kahn's / DFS post-order)** — whenever the problem gives "A must come before B" pairwise constraints and asks for *a* valid global ordering, or to detect that no such ordering exists (a cycle).
- **Union-Find (Disjoint Set Union)** — whenever the problem is fundamentally about *connectivity/grouping* — "are these two things in the same group," "how many distinct groups exist," or "merge these groups together" — especially when merges happen incrementally over time.

## Generic BFS template
```python
from collections import deque

def bfs(start, get_neighbors, is_target):
    visited = {start}
    queue = deque([(start, 0)])
    while queue:
        node, dist = queue.popleft()
        if is_target(node):
            return dist
        for neighbor in get_neighbors(node):
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, dist + 1))
    return -1  # unreachable
```

## Problems in this folder
- [[Walls-and-Gates]] — multi-source BFS (seed the queue with *all* sources at once)
- [[Alien-Dictionary]] — topological sort over a graph derived from adjacent-word comparisons
- [[Word-Ladder]] — unweighted shortest path via BFS with on-the-fly neighbor generation
- [[Number-of-Islands]] — connected components via grid flood fill (BFS/DFS)
- [[Digit-Operations-to-Make-Two-Integers-Equal]] — shortest path over an implicit state graph (BFS when uniform cost, Dijkstra when weighted)

See also [[Accounts-Merge]] in the Hashing & Design folder — it's a graph-connectivity problem solved with Union-Find rather than BFS/DFS, worth comparing directly against [[Number-of-Islands]] to see the same underlying idea (connected components) solved two different ways.
