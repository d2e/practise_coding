---
title: "Minimum Absolute Difference"
aliases: []
tags: [leetcode, array, sorting, easy]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 1200
leetcode_difficulty: Easy
pattern: "Sort then Scan Adjacent Pairs"
status: not-started
---

# Minimum Absolute Difference

## Problem Statement
Given an array of distinct integers `arr`, find all pairs of elements with the minimum absolute difference of any two elements.

Return a list of pairs in ascending order (with respect to pairs), each pair `[a, b]` follows `a, b` are from `arr`, `a < b`, and `b - a` equals the minimum absolute difference of any two elements in `arr`.

Constraints: `2 <= arr.length <= 10^5`, `-10^6 <= arr[i] <= 10^6`, all elements are distinct.

## Examples
```
Input: arr = [4,2,1,3]
Output: [[1,2],[2,3],[3,4]]
Explanation: The minimum absolute difference is 1. All pairs achieving this difference: [1,2],[2,3],[3,4].

Input: arr = [1,3,6,10,15]
Output: [[1,3]]

Input: arr = [3,8,-10,23,19,-4,-14,27]
Output: [[-14,-10],[19,23],[23,27]]
```

## Intuition & Approach
The naive approach checks every pair of elements — O(n^2) comparisons — to find the minimum absolute difference and then collect all pairs achieving it. With `n` up to `10^5`, that's roughly 10^10 operations, far too slow.

The key insight is that the minimum absolute difference between any two elements of an array can **only** be achieved by elements that are adjacent to each other once the array is sorted. This is because if two elements `a` and `b` achieve the minimum difference but are not adjacent after sorting, there must be some element `c` strictly between them in sorted order, and either `|a - c|` or `|c - b|` would be strictly smaller than `|a - b|` — contradicting that `|a - b|` was minimum. So after sorting, you only ever need to compare **consecutive** elements.

So: sort the array (O(n log n)), then do a single linear scan comparing each element to its predecessor, tracking the minimum difference seen. Do a second pass (or combine into one pass with a result list) to collect every consecutive pair whose difference equals that minimum.

## Pattern Recognition
> [!tip] Pattern: Sort then Scan Adjacent Pairs
> Whenever a problem asks about "minimum difference between any two elements" (or similar closeness/pairing questions) with no other structural constraint, sorting first reduces the search space from all O(n^2) pairs down to just the O(n) adjacent pairs in sorted order — because the extremal pair is always adjacent post-sort.

## Algorithm Steps
1. Sort `arr` in ascending order.
2. Scan through consecutive pairs `(arr[i-1], arr[i])`, computing `arr[i] - arr[i-1]` for each, and track the minimum difference `min_diff` found.
3. Scan again (or track during the first pass), collecting every consecutive pair whose difference equals `min_diff` into the result list.
4. Return the result list.

## Solution (Python)
```python
from typing import List

class Solution:
    def minimumAbsDifference(self, arr: List[int]) -> List[List[int]]:
        arr.sort()

        min_diff = min(arr[i] - arr[i - 1] for i in range(1, len(arr)))

        result = []
        for i in range(1, len(arr)):
            if arr[i] - arr[i - 1] == min_diff:
                result.append([arr[i - 1], arr[i]])

        return result
```

## Solution (Java)
```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

class Solution {
    public List<List<Integer>> minimumAbsDifference(int[] arr) {
        Arrays.sort(arr);

        int minDiff = Integer.MAX_VALUE;
        for (int i = 1; i < arr.length; i++) {
            minDiff = Math.min(minDiff, arr[i] - arr[i - 1]);
        }

        List<List<Integer>> result = new ArrayList<>();
        for (int i = 1; i < arr.length; i++) {
            if (arr[i] - arr[i - 1] == minDiff) {
                result.add(Arrays.asList(arr[i - 1], arr[i]));
            }
        }

        return result;
    }
}
```

## Complexity Analysis
- **Time:** O(n log n) — dominated by the sort; the two linear scans afterward are O(n).
- **Space:** O(log n) to O(n) for the sort's internal recursion/stack (implementation-dependent), plus O(k) for the output list where `k` is the number of pairs achieving the minimum difference; excluding output, auxiliary space is effectively O(1) beyond the sort itself.

## Edge Cases
- Array of exactly 2 elements: trivially the only pair, and it is by definition the minimum difference pair.
- Large negative and positive values mixed together (e.g., `[-10^6, 10^6]`) — sorting handles sign correctly, no special-casing needed.
- Multiple pairs tied for the minimum difference scattered across the array (not just one) — the second scan must not `break` early after finding the first match.

## Common Mistakes
- Comparing all O(n^2) pairs instead of sorting first, causing a timeout on large inputs.
- Forgetting that after sorting, only *adjacent* pairs can possibly achieve the minimum — trying to also check non-adjacent pairs is unnecessary (wasted work) but not actually a correctness bug, just an efficiency one, and often a source of confusion when debugging.
- Off-by-one indexing when scanning "each element vs. its predecessor" (starting the loop at `i = 0` instead of `i = 1`, causing an out-of-bounds access at `arr[-1]`).

## Related Problems
- [[Merge-Sorted-Array]]
- [[Ways-to-Make-a-Fair-Array]]
- [[Longest-Continuous-Subarray-With-Absolute-Diff-Limit]]

## Follow-up / Variants
A related but distinct problem is finding the minimum absolute difference between elements of **two different** sorted arrays (rather than within one array) — that's typically solved with a two-pointer merge-style walk across both arrays simultaneously. Another variant restricts pairs to only those at least `k` positions apart in the original (unsorted) array, which breaks the simple "sort and scan adjacent" trick and requires a different approach (e.g., a sliding window with a balanced structure).
