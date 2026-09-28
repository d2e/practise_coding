---
title: "Walls and Gates"
aliases: []
tags: [leetcode, multi-source-bfs, medium]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 286
leetcode_difficulty: Medium
pattern: "Multi-Source BFS"
status: not-started
---

# Walls and Gates

## Problem Statement
You are given an `m x n` grid `rooms` initialized with three possible values:
- `-1` — a wall or obstacle.
- `0` — a gate.
- `2147483647` (`INF`) — an empty room.

Fill each empty room with the distance to its *nearest* gate. If a room cannot reach any gate, it should remain `INF`. You may move up, down, left, or right, and you may not move through walls.

## Examples
```
Input:
INF  -1   0   INF
INF  INF  INF -1
INF  -1   INF -1
0    -1   INF INF

Output:
3  -1   0   1
2   2   1  -1
1  -1   2  -1
0  -1   3   4
```

```
Input:
rooms = [[0]]
Output: [[0]]            # a single gate, nothing to fill
```

## Intuition & Approach
The naive instinct is: for every empty room, run a BFS/DFS out to find the nearest gate, and take the minimum over all gates. That works but is `O((mn)^2)` in the worst case, because every room can trigger its own search across the whole grid.

The key inversion: instead of asking "what is the nearest gate *from* this room?", ask "how far is *every* room from *any* gate?" — and answer all of those questions in a single BFS by starting from **all gates at once**. Push every gate into the BFS queue with distance 0, then expand outward layer by layer. Because BFS explores in increasing order of distance, the first time a room is visited is guaranteed to be via the shortest path from the nearest gate. This is the multi-source BFS pattern: whenever a problem asks for "distance to nearest X" where there are many X's, seed the queue with all X's simultaneously rather than searching from each one independently.

## Pattern Recognition
> [!tip] Pattern: Multi-Source BFS
> Signal words: "nearest gate/exit/rotten orange/fire", multiple starting points, and a request for per-cell distance to the closest one. Seed the BFS frontier with *all* sources at distance 0 instead of looping a single-source BFS once per source.

## Algorithm Steps
1. Scan the grid once to collect the coordinates of every gate (`0`) into a queue, each tagged with distance 0.
2. Run a standard BFS from this combined frontier: pop a cell, look at its 4 neighbors, and if a neighbor is `INF` (unvisited empty room), set it to `current distance + 1` and push it onto the queue.
3. Never revisit a cell — the `rooms` grid doubles as the "visited" marker: once a cell is no longer `INF`, skip it.
4. Continue until the queue is empty. Any room BFS never reaches stays `INF`, meaning it's walled off from every gate.

## Solution (Python)
```python
from collections import deque
from typing import List

def wallsAndGates(rooms: List[List[int]]) -> None:
    if not rooms or not rooms[0]:
        return

    INF = 2147483647
    m, n = len(rooms), len(rooms[0])
    queue = deque()

    # Seed the frontier with every gate simultaneously.
    for r in range(m):
        for c in range(n):
            if rooms[r][c] == 0:
                queue.append((r, c))

    directions = [(1, 0), (-1, 0), (0, 1), (0, -1)]
    while queue:
        r, c = queue.popleft()
        for dr, dc in directions:
            nr, nc = r + dr, c + dc
            if 0 <= nr < m and 0 <= nc < n and rooms[nr][nc] == INF:
                # First visit is guaranteed shortest, because BFS expands by distance layer.
                rooms[nr][nc] = rooms[r][c] + 1
                queue.append((nr, nc))
```

## Solution (Java)
```java
import java.util.ArrayDeque;
import java.util.Deque;

class Solution {
    public void wallsAndGates(int[][] rooms) {
        if (rooms == null || rooms.length == 0 || rooms[0].length == 0) return;

        int INF = Integer.MAX_VALUE;
        int m = rooms.length, n = rooms[0].length;
        Deque<int[]> queue = new ArrayDeque<>();

        // Seed the frontier with every gate simultaneously.
        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                if (rooms[r][c] == 0) queue.offer(new int[]{r, c});
            }
        }

        int[][] directions = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
        while (!queue.isEmpty()) {
            int[] cell = queue.poll();
            int r = cell[0], c = cell[1];
            for (int[] d : directions) {
                int nr = r + d[0], nc = c + d[1];
                if (nr >= 0 && nr < m && nc >= 0 && nc < n && rooms[nr][nc] == INF) {
                    // First visit is guaranteed shortest, because BFS expands by distance layer.
                    rooms[nr][nc] = rooms[r][c] + 1;
                    queue.offer(new int[]{nr, nc});
                }
            }
        }
    }
}
```

## Complexity Analysis
- **Time:** O(m·n) — every cell is enqueued and dequeued at most once.
- **Space:** O(m·n) — the BFS queue can hold up to all cells in the worst case (e.g., a grid full of gates or a wide-open room).

## Edge Cases
- Grid with no gates at all — every empty room correctly stays `INF`.
- Grid with no empty rooms (all walls/gates) — BFS queue drains immediately, nothing to do.
- Rooms fully enclosed by walls with no path to any gate — must remain `INF`, not accidentally overwritten.
- A grid that is entirely gates, or a 1x1 grid.

## Common Mistakes
- Running a separate BFS/DFS per gate (or per empty room) instead of a single combined multi-source BFS — correct but far too slow.
- Forgetting to check `rooms[nr][nc] == INF` before overwriting, which can cause re-processing of already-finalized cells and blow up the queue.
- Using DFS with a "compare and take minimum" strategy, which requires revisiting cells multiple times and re-propagating updated distances — much more error-prone than BFS's single-pass guarantee.

## Related Problems
- [[Number-of-Islands]]
- [[Word-Ladder]]
- [[Alien-Dictionary]]

## Follow-up / Variants
A common follow-up is "Rotting Oranges" (LC 994), which is structurally identical multi-source BFS but asks for the number of BFS layers (time steps) rather than per-cell distances. Another variant asks you to also reconstruct the actual shortest path from each room to its nearest gate, which requires storing a parent pointer during BFS.
