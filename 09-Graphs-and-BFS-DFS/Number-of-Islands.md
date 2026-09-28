---
title: "Number of Islands"
aliases: []
tags: [leetcode, flood-fill, medium]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 200
leetcode_difficulty: Medium
pattern: "Grid Flood Fill (BFS/DFS)"
status: not-started
---

# Number of Islands

## Problem Statement
Given an `m x n` 2D binary grid `grid` which represents a map of `'1'`s (land) and `'0'`s (water), return the number of islands. An island is surrounded by water and is formed by connecting adjacent lands horizontally or vertically (not diagonally). Assume all four edges of the grid are surrounded by water.

## Examples
```
Input:
grid = [
  ["1","1","1","1","0"],
  ["1","1","0","1","0"],
  ["1","1","0","0","0"],
  ["0","0","0","0","0"]
]
Output: 1
```

```
Input:
grid = [
  ["1","1","0","0","0"],
  ["1","1","0","0","0"],
  ["0","0","1","0","0"],
  ["0","0","0","1","1"]
]
Output: 3
```

## Intuition & Approach
An "island" is precisely a **connected component** in the graph where grid cells are nodes and edges connect orthogonally-adjacent land cells. So the problem reduces to: count the number of connected components of `'1'`s.

The classic approach: scan every cell in the grid in order. Whenever you find an unvisited `'1'`, that's the *first* cell of a brand-new island you haven't counted yet — increment the island counter, then flood-fill (BFS or DFS) outward from that cell, marking every reachable land cell as visited so the outer scan never double-counts it. By the time the flood fill finishes, the entire island has been "consumed," and the outer loop continues scanning for the next unvisited `'1'`.

Marking visited can be done with a separate visited set, or — more space-efficiently — by mutating the grid in place (flipping visited `'1'`s to `'0'`s), since we don't need the original grid afterward.

## Pattern Recognition
> [!tip] Pattern: Grid Flood Fill / Connected Components
> Any time a problem describes a grid of cells where adjacency (usually 4-directional) implies connection, and asks "how many connected groups / regions / islands," it's a connected-components count: scan every cell, and every time you find an unvisited "start" cell, flood-fill outward (BFS or DFS) to mark its entire component, incrementing a counter once per fill.

## Algorithm Steps
1. Initialize `islands = 0`.
2. Iterate over every cell `(r, c)` in the grid in row-major order.
3. If `grid[r][c] == '1'` (unvisited land), increment `islands`, then flood-fill from `(r, c)`: using BFS (queue) or DFS (recursion/stack), visit every orthogonally-connected `'1'`, marking each as `'0'` (or a visited marker) as you go so it's never revisited.
4. After the full scan, `islands` holds the answer.

## Solution (Python)
```python
from collections import deque
from typing import List

def numIslands(grid: List[List[str]]) -> int:
    if not grid or not grid[0]:
        return 0

    rows, cols = len(grid), len(grid[0])
    islands = 0

    def bfs(sr: int, sc: int) -> None:
        queue = deque([(sr, sc)])
        grid[sr][sc] = "0"  # mark visited immediately on enqueue
        while queue:
            r, c = queue.popleft()
            for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
                nr, nc = r + dr, c + dc
                if 0 <= nr < rows and 0 <= nc < cols and grid[nr][nc] == "1":
                    grid[nr][nc] = "0"
                    queue.append((nr, nc))

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == "1":
                islands += 1
                bfs(r, c)

    return islands
```

## Solution (Java)
```java
import java.util.ArrayDeque;
import java.util.Deque;

class Solution {
    private int rows, cols;
    private static final int[][] DIRECTIONS = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};

    public int numIslands(char[][] grid) {
        if (grid == null || grid.length == 0 || grid[0].length == 0) return 0;

        rows = grid.length;
        cols = grid[0].length;
        int islands = 0;

        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (grid[r][c] == '1') {
                    islands++;
                    bfs(grid, r, c);
                }
            }
        }

        return islands;
    }

    private void bfs(char[][] grid, int sr, int sc) {
        Deque<int[]> queue = new ArrayDeque<>();
        queue.offer(new int[]{sr, sc});
        grid[sr][sc] = '0'; // mark visited immediately on enqueue

        while (!queue.isEmpty()) {
            int[] cell = queue.poll();
            int r = cell[0], c = cell[1];
            for (int[] d : DIRECTIONS) {
                int nr = r + d[0], nc = c + d[1];
                if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && grid[nr][nc] == '1') {
                    grid[nr][nc] = '0';
                    queue.offer(new int[]{nr, nc});
                }
            }
        }
    }
}
```

## Complexity Analysis
- **Time:** O(m·n) — every cell is visited a constant number of times (once by the outer scan, and at most once by a flood fill).
- **Space:** O(m·n) — worst case, the BFS queue (or DFS recursion stack) holds a large fraction of the grid, e.g., when the entire grid is one giant island.

## Edge Cases
- Empty grid, or a grid with empty rows — return 0 immediately.
- Grid entirely water — loop finds no `'1'`s, returns 0.
- Grid entirely land — single flood fill consumes everything, returns 1.
- A 1x1 grid with `"1"` or `"0"`.
- Diagonally-adjacent land cells that are *not* orthogonally connected — must be counted as separate islands (a common source of off-by-one/logic errors if 8-directional adjacency is mistakenly used).

## Common Mistakes
- Using 8-directional (including diagonal) adjacency instead of the required 4-directional adjacency.
- Marking a cell visited only when *dequeued* instead of when *enqueued*, which allows the same cell to be pushed onto the queue multiple times from different neighbors.
- Recursing with DFS on a very large all-land grid without considering recursion-depth limits (Python's default recursion limit can be hit on large grids) — BFS or an explicit stack avoids this.
- Forgetting that `grid` is a list of lists of strings (`'1'`/`'0'`), not integers — comparisons like `grid[r][c] == 1` (integer) silently always fail.

## Related Problems
- [[Walls-and-Gates]]
- [[Word-Ladder]]
- [[Accounts-Merge]]

## Follow-up / Variants
A frequent follow-up is "Max Area of Island" (LC 695), which uses the identical flood-fill skeleton but tracks and returns the size of the largest connected component instead of just the count. Another common follow-up asks you to solve it with **Union-Find** instead of BFS/DFS — union every land cell with its land neighbors, and the number of islands is the number of distinct root parents among land cells; this variant is also the standard approach for "Number of Islands II" (LC 305), where land cells are added one at a time and you must report the island count after each addition.
