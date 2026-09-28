---
title: "K Closest Points to Origin"
aliases: []
tags: [leetcode, heap, quickselect, medium]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 973
leetcode_difficulty: Medium
pattern: "Top-K via Heap (or Quickselect)"
status: not-started
---

# K Closest Points to Origin

## Problem Statement
Given an array of points on the 2D plane and an integer `k`, return the `k` points closest to the origin `(0, 0)`. Distance is Euclidean, but since we only ever compare distances (never need the actual value), squared distance (`x^2 + y^2`) works just as well and avoids floating point. The answer can be returned in any order.

## Examples
```
Input: points = [[1,3],[-2,2]], k = 1
Output: [[-2,2]]
Explanation: distance((1,3)) = sqrt(10), distance((-2,2)) = sqrt(8) < sqrt(10), so (-2,2) is closer.

Input: points = [[3,3],[5,-1],[-2,4]], k = 2
Output: [[3,3],[-2,4]]
```

## Intuition & Approach
This is the "top-K" pattern: out of N items, find the K with the smallest (or largest) value of some key. The instinct to fully sort all N points by distance and take the first K works but costs O(N log N) — more than necessary when K is much smaller than N.

**Max-heap of size K** is the standard efficient approach: maintain a heap containing the K closest points seen *so far*. For each new point, if the heap has fewer than K elements, push it in. Otherwise, compare the new point's distance to the heap's maximum (the root of a max-heap); if the new point is closer, pop the max and push the new point. By the end, the heap holds exactly the K closest points. This costs O(N log K) — better than O(N log N) when K << N.

**Quickselect** (based on the partitioning step of quicksort) does even better on average: O(N) expected time. Partition the array around a pivot distance; if the pivot ends up at index K after partitioning, everything to its left is exactly the K closest (order among them doesn't matter, matching the problem's "any order" requirement). Recurse into only the side that contains index K. This is the interview follow-up worth mentioning after the heap solution.

Python's heap module only provides a min-heap, so to simulate a max-heap of size K we negate the distances when pushing.

## Pattern Recognition
> [!tip] Pattern: Top-K via Bounded Heap
> "Find the K smallest/largest/closest out of N" is the signature of this pattern. Use a heap of size K (opposite polarity of what you're optimizing for — a *max*-heap when you want the K *smallest*, so the heap's root is always the worst of your current K candidates and thus the one to evict). This avoids the wasted work of fully sorting.

## Algorithm Steps
**Heap approach:**
1. Maintain a max-heap (via negated squared distances) of size at most `k`.
2. For each point, compute `dist = x*x + y*y`.
3. Push `(-dist, point)` onto the heap.
4. If the heap size exceeds `k`, pop (removes the current farthest point among the candidates).
5. After processing all points, the heap contains the `k` closest; extract and return them.

## Solution (Python)
```python
import heapq
from typing import List

class Solution:
    def kClosest(self, points: List[List[int]], k: int) -> List[List[int]]:
        heap = []  # max-heap via negated squared distance
        for x, y in points:
            dist = x * x + y * y
            heapq.heappush(heap, (-dist, x, y))
            if len(heap) > k:
                heapq.heappop(heap)

        return [[x, y] for _, x, y in heap]

    def kClosestQuickselect(self, points: List[List[int]], k: int) -> List[List[int]]:
        def dist(p):
            return p[0] * p[0] + p[1] * p[1]

        def partition(left: int, right: int) -> int:
            pivot_dist = dist(points[right])
            i = left
            for j in range(left, right):
                if dist(points[j]) <= pivot_dist:
                    points[i], points[j] = points[j], points[i]
                    i += 1
            points[i], points[right] = points[right], points[i]
            return i

        left, right = 0, len(points) - 1
        while left < right:
            pivot_index = partition(left, right)
            if pivot_index == k:
                break
            elif pivot_index < k:
                left = pivot_index + 1
            else:
                right = pivot_index - 1

        return points[:k]
```

## Solution (Java)
```java
import java.util.Arrays;
import java.util.PriorityQueue;

class Solution {
    public int[][] kClosest(int[][] points, int k) {
        // Max-heap keyed by squared distance: root is the farthest of our
        // current k candidates, so it's the one to evict.
        PriorityQueue<int[]> heap = new PriorityQueue<>(
            (a, b) -> (b[0] * b[0] + b[1] * b[1]) - (a[0] * a[0] + a[1] * a[1])
        );

        for (int[] point : points) {
            heap.offer(point);
            if (heap.size() > k) {
                heap.poll();
            }
        }

        int[][] result = new int[heap.size()][];
        int i = 0;
        for (int[] point : heap) {
            result[i++] = point;
        }
        return result;
    }

    public int[][] kClosestQuickselect(int[][] points, int k) {
        int left = 0, right = points.length - 1;
        while (left < right) {
            int pivotIndex = partition(points, left, right);
            if (pivotIndex == k) {
                break;
            } else if (pivotIndex < k) {
                left = pivotIndex + 1;
            } else {
                right = pivotIndex - 1;
            }
        }
        return Arrays.copyOfRange(points, 0, k);
    }

    private int dist(int[] p) {
        return p[0] * p[0] + p[1] * p[1];
    }

    private int partition(int[][] points, int left, int right) {
        int pivotDist = dist(points[right]);
        int i = left;
        for (int j = left; j < right; j++) {
            if (dist(points[j]) <= pivotDist) {
                int[] tmp = points[i];
                points[i] = points[j];
                points[j] = tmp;
                i++;
            }
        }
        int[] tmp = points[i];
        points[i] = points[right];
        points[right] = tmp;
        return i;
    }
}
```

## Complexity Analysis
- **Heap approach — Time:** O(N log K), since each of the N points does at most one O(log K) push/pop against a heap bounded at size K. **Space:** O(K) for the heap.
- **Quickselect — Time:** O(N) expected (each partitioning pass eliminates roughly half the remaining candidates on average), O(N²) worst case with a poor pivot choice. **Space:** O(1) extra (in-place partitioning), ignoring recursion/iteration overhead.

## Edge Cases
- `k` equals the total number of points — return all of them (heap approach naturally handles this: the heap never exceeds size K = N).
- Points with identical distances — any valid subset of size K containing all minimal-distance ties is acceptable, since the problem allows any order and doesn't require a unique tie-break.
- A point exactly at the origin `(0, 0)` — distance 0, should sort first among closest.

## Common Mistakes
- Using a min-heap of *all* N points and popping K times — this works but costs O(N log N) to build the full heap, defeating the purpose of the bounded-size optimization.
- Computing actual Euclidean distance with `sqrt` — unnecessary floating-point overhead when only relative comparisons matter; squared distance preserves ordering.
- In the heap approach, forgetting to negate the distance, which turns Python's min-heap into a heap that evicts the *closest* point instead of the farthest.

## Related Problems
- [[Merge-k-Sorted-Lists]]
- [[Sort-Integers-by-The-Power-Value]]
- [[Minimum-Area-Rectangle]]

## Follow-up / Variants
A frequent follow-up: "what if points arrive as a continuous stream and you need the K closest *so far* at any point in time?" — the max-heap approach adapts directly (it's already incremental), whereas quickselect requires the full array up front and doesn't stream.
