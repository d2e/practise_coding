---
title: "Minimum Path Sum"
aliases: []
tags: [leetcode, dp, grid-dp, medium]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 64
leetcode_difficulty: Medium
pattern: "Grid DP"
status: not-started
---

# Minimum Path Sum

## Problem Statement
Given an `m x n` grid filled with non-negative numbers, find a path from top-left to bottom-right which minimizes the sum of all numbers along its path. You can only move either down or right at any point in time.

## Examples
```
Input:
grid = [[1,3,1],
        [1,5,1],
        [4,2,1]]
Output: 7
Explanation: path 1 -> 3 -> 1 -> 1 -> 1 minimizes the sum.
```

```
Input:
grid = [[1,2,3],
        [4,5,6]]
Output: 12
```

## Intuition & Approach
Since you can only move down or right, the number of distinct paths from the top-left to any cell `(i, j)` can be large, and trying all of them explicitly is exponential. But there's a key structural fact: the *only* two ways to arrive at cell `(i, j)` are from directly above `(i-1, j)` or directly to the left `(i, j-1)`. So the minimum-cost path to `(i, j)` must pass through whichever of those two neighbors gives the smaller accumulated cost, then add the current cell's value.

That recursive structure — "the optimal solution to a subproblem is built from optimal solutions to strictly smaller subproblems, with overlapping subproblems reused across many paths" — is the definition of dynamic programming. Define `dp[i][j]` = minimum path sum to reach `(i, j)` from `(0, 0)`. Then:
```
dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1])
```
with the first row and first column handled as base cases (they only have one possible way to be reached — straight along the edge).

## Pattern Recognition
> [!tip] Pattern: Grid DP
> Movement restricted to two directions (down/right, or similar) through a grid, combined with an "optimize a running total along the path" objective (minimize/maximize sum, count paths, etc.) — build a DP table over grid cells where each cell's value depends only on the cell above and the cell to the left.

## Algorithm Steps
1. Handle the base case: `dp[0][0] = grid[0][0]`.
2. Fill the first row: each cell can only be reached from the left, so `dp[0][j] = dp[0][j-1] + grid[0][j]`.
3. Fill the first column: each cell can only be reached from above, so `dp[i][0] = dp[i-1][0] + grid[i][0]`.
4. Fill the rest of the table in row-major (or column-major) order: `dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1])`.
5. Return `dp[m-1][n-1]`.
6. (Space optimization) Since each row only depends on the row directly above it, the 2D table can be collapsed to a single 1D array updated in place, reducing space from O(m·n) to O(n).

## Solution (Python)
```python
from typing import List

def minPathSum(grid: List[List[int]]) -> int:
    if not grid or not grid[0]:
        return 0

    rows, cols = len(grid), len(grid[0])
    dp = [0] * cols
    dp[0] = grid[0][0]

    # First row: only reachable from the left.
    for j in range(1, cols):
        dp[j] = dp[j - 1] + grid[0][j]

    for i in range(1, rows):
        dp[0] += grid[i][0]  # first column: only reachable from above
        for j in range(1, cols):
            dp[j] = grid[i][j] + min(dp[j], dp[j - 1])  # dp[j] here is still "above" before this update

    return dp[-1]
```

## Solution (Java)
```java
class Solution {
    public int minPathSum(int[][] grid) {
        if (grid == null || grid.length == 0 || grid[0].length == 0) return 0;

        int rows = grid.length, cols = grid[0].length;
        int[] dp = new int[cols];
        dp[0] = grid[0][0];

        // First row: only reachable from the left.
        for (int j = 1; j < cols; j++) {
            dp[j] = dp[j - 1] + grid[0][j];
        }

        for (int i = 1; i < rows; i++) {
            dp[0] += grid[i][0]; // first column: only reachable from above
            for (int j = 1; j < cols; j++) {
                dp[j] = grid[i][j] + Math.min(dp[j], dp[j - 1]); // dp[j] here is still "above" before this update
            }
        }

        return dp[cols - 1];
    }
}
```

## Complexity Analysis
- **Time:** O(m·n) — every cell is computed exactly once via O(1) work.
- **Space:** O(n) with the rolling 1D array optimization (O(m·n) if using a full 2D DP table, which is also acceptable and often clearer to reason about first).

## Edge Cases
- 1x1 grid — the answer is just `grid[0][0]`, with no movement at all.
- Single row or single column grid — only one possible path exists (straight line), the DP should reduce to a simple prefix sum along that line.
- Grid containing zeros — should not cause any special-casing; zero costs simply don't add anything.
- Non-square grids (more rows than columns or vice versa) — algorithm works identically regardless of shape.

## Common Mistakes
- Forgetting to separately initialize the first row and first column before the main DP loop, causing an index-out-of-bounds or incorrect base values when `i == 0` or `j == 0` inside a naive `min(dp[i-1][j], dp[i][j-1])` expression.
- In the space-optimized 1D version, updating `dp[j]` before reading the "diagonal-up" value is fine here since Minimum Path Sum only needs "up" and "left" (not "up-left"), but this pattern breaks for problems that also need the diagonal predecessor — worth noting as a boundary of when the rolling-array trick applies cleanly.
- Mutating the input `grid` in place to save space without checking whether the caller expects `grid` to remain unmodified after the call.

## Related Problems
- [[Longest-Increasing-Subsequence]]
- [[Maximum-Subarray]]
- [[Counting-Bits]]

## Follow-up / Variants
A common follow-up is "Unique Paths" / "Unique Paths II" (LC 62 / 63), which uses the identical grid-DP recurrence but counts the *number* of paths rather than minimizing a sum, and adds obstacles that block certain cells entirely. Another follow-up asks you to also reconstruct the actual minimum-cost path (not just its sum), which requires storing a direction/parent choice at each cell during the DP fill and walking it back from `(m-1, n-1)` to `(0, 0)`.
