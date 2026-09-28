---
title: "Minimum Area Rectangle"
aliases: []
tags: [leetcode, array, hash-table, geometry, medium]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 939
leetcode_difficulty: Medium
pattern: "Hash Set + Diagonal Pair Enumeration"
status: not-started
---

# Minimum Area Rectangle

## Problem Statement
You are given an array of points on the X-Y plane, where `points[i] = [xi, yi]`. Find the minimum area of a rectangle formed from these points, with sides parallel to the X and Y axes. If there is no such rectangle, return `0`.

Constraints: `1 <= points.length <= 500`, `0 <= xi, yi <= 4 * 10^4`, all points are distinct.

## Examples
```
Input: points = [[1,1],[1,3],[3,1],[3,3],[2,2]]
Output: 4
Explanation: the rectangle (1,1),(1,3),(3,1),(3,3) has area (3-1)*(3-1) = 4.

Input: points = [[1,1],[1,3],[3,1],[3,3],[4,1],[4,3]]
Output: 2
Explanation: rectangle (3,1),(3,3),(4,1),(4,3) has area (4-3)*(3-1) = 2, smaller than the 2x2 one.
```

## Intuition & Approach
Since the rectangle's sides must be axis-parallel, any valid rectangle is fully determined by two of its **diagonally opposite corners**: `(x1, y1)` and `(x2, y2)` with `x1 != x2` and `y1 != y2`. The other two corners are forced to be `(x1, y2)` and `(x2, y1)`. This is the key simplification — you never need to reason about arbitrary quadrilaterals, only about pairs of points that could be diagonal corners.

The naive approach checks all `C(n, 4)` combinations of four points to see if they form a rectangle — far too slow for `n = 500`.

Instead: put every point into a hash set for O(1) membership checks. Then iterate over **all pairs of points** `((x1,y1), (x2,y2))` with `x1 < x2` and `y1 < y2` (i.e., candidate diagonal corners). For each pair, check whether the other two corners `(x1, y2)` and `(x2, y1)` also exist in the set. If they do, this pair of points forms a valid rectangle with area `(x2 - x1) * (y2 - y1)`; track the minimum such area seen.

This reduces the problem from O(n^4) to O(n^2) pair checks, each O(1) thanks to the hash set — O(n^2) total, which comfortably handles `n = 500` (250,000 pairs).

## Pattern Recognition
> [!tip] Pattern: Hash Set + Diagonal Pair Enumeration
> Whenever a geometric shape is fully determined by two of its corners (as an axis-aligned rectangle is by its diagonal), you can avoid enumerating all combinations of points — just enumerate pairs, and use a hash set to verify the remaining corners exist in O(1).

## Algorithm Steps
1. Insert every point (as a tuple) into a hash set for O(1) lookups.
2. For every unordered pair of points `p1 = (x1, y1)` and `p2 = (x2, y2)`:
   - Skip if `x1 == x2` or `y1 == y2` (they can't be diagonal corners of a rectangle with positive area).
   - Check whether `(x1, y2)` and `(x2, y1)` are both in the set.
   - If so, compute `area = abs(x2 - x1) * abs(y2 - y1)` and update the running minimum.
3. Return the minimum area found, or `0` if no valid rectangle was found.

## Solution (Python)
```python
from typing import List

class Solution:
    def minAreaRect(self, points: List[List[int]]) -> int:
        point_set = {(x, y) for x, y in points}
        n = len(points)
        min_area = float("inf")

        for i in range(n):
            x1, y1 = points[i]
            for j in range(i + 1, n):
                x2, y2 = points[j]
                if x1 == x2 or y1 == y2:
                    continue  # can't be diagonal corners
                if (x1, y2) in point_set and (x2, y1) in point_set:
                    area = abs(x2 - x1) * abs(y2 - y1)
                    min_area = min(min_area, area)

        return min_area if min_area != float("inf") else 0
```

## Solution (Java)
```java
import java.util.HashSet;
import java.util.Set;

class Solution {
    public int minAreaRect(int[][] points) {
        Set<Long> pointSet = new HashSet<>();
        for (int[] p : points) {
            pointSet.add(encode(p[0], p[1]));
        }

        int n = points.length;
        long minArea = Long.MAX_VALUE;

        for (int i = 0; i < n; i++) {
            int x1 = points[i][0], y1 = points[i][1];
            for (int j = i + 1; j < n; j++) {
                int x2 = points[j][0], y2 = points[j][1];
                if (x1 == x2 || y1 == y2) continue; // can't be diagonal corners

                if (pointSet.contains(encode(x1, y2)) && pointSet.contains(encode(x2, y1))) {
                    long area = (long) Math.abs(x2 - x1) * Math.abs(y2 - y1);
                    minArea = Math.min(minArea, area);
                }
            }
        }

        return minArea == Long.MAX_VALUE ? 0 : (int) minArea;
    }

    // Coordinates are bounded by 4*10^4, so a fixed multiplier keeps encodings unique.
    private long encode(int x, int y) {
        return (long) x * 100_000 + y;
    }
}
```

## Complexity Analysis
- **Time:** O(n^2) — every pair of points is examined once, with O(1) hash-set lookups per pair.
- **Space:** O(n) — the hash set stores every point once.

## Edge Cases
- Fewer than 4 points: no rectangle is possible; the algorithm naturally never finds a valid pair and returns `0`.
- Points that are collinear (all same x or all same y): every pair is skipped by the `x1 == x2 or y1 == y2` check, correctly yielding `0`.
- Multiple rectangles of the same minimum area — the algorithm just needs the minimum value, not the specific rectangle, so ties don't need special handling.

## Common Mistakes
- Iterating over all 4-point combinations directly (O(n^4)), which times out for larger inputs.
- Forgetting to skip pairs sharing an x or y coordinate, which would otherwise attempt to compute a zero-width or zero-height "rectangle."
- Using a list instead of a hash set for membership checks, degrading lookups from O(1) to O(n) and the whole algorithm to O(n^3).

## Related Problems
- [[Rotate-Image]]
- [[K-Closest-Points-to-Origin]]
- [[Accounts-Merge]]

## Follow-up / Variants
A well-known follow-up (LeetCode 963, "Minimum Area Rectangle II") removes the axis-parallel restriction and allows rectangles at any angle — that variant requires grouping points by their diagonal midpoint and diagonal length (any two pairs of points sharing both are the diagonals of some rectangle), a different but related hashing trick.
