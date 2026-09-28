---
title: "Random Pick with Weight"
aliases: []
tags: [leetcode, design, prefix-sum, binary-search, medium]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 528
leetcode_difficulty: Medium
pattern: "Prefix Sum + Binary Search (Weighted Sampling)"
status: not-started
---

# Random Pick with Weight

## Problem Statement
Given an array of positive integers `w` where `w[i]` describes the weight of index `i`, design a class that repeatedly picks a random index in proportion to its weight — i.e., the probability of picking index `i` should be `w[i] / sum(w)`. Each call to `pickIndex()` should return an index according to this distribution, and calls are independent.

## Examples
```
w = [1, 3]
pickIndex() should return index 0 with probability 1/4, and index 1 with probability 3/4.

w = [1]
pickIndex() always returns 0.
```

## Intuition & Approach
Picture the weights laid out on a number line as consecutive segments: index 0 occupies `[0, w[0])`, index 1 occupies `[w[0], w[0]+w[1])`, and so on, up to `total = sum(w)`. If you pick a uniformly random real number (or in practice, integer) in `[0, total)`, the probability it lands in index `i`'s segment is exactly `w[i] / total` — precisely the distribution required.

So the problem reduces to: (1) build the prefix sums of `w` once (these define the segment boundaries), and (2) on each query, draw a random integer in `[1, total]` (or `[0, total)` depending on convention) and find which segment it falls into. Because the prefix sums are sorted (strictly increasing, since weights are positive), "which segment does this value fall into" is exactly a **binary search for the first prefix sum that is `>=` the random value** (`bisect_left`).

This is the same "preprocess once, query cheaply" idea as Shortest Word Distance II, paired with binary search instead of a two-pointer merge, because here each query is independent (not a merge of two structures).

## Pattern Recognition
> [!tip] Pattern: Weighted Sampling via Prefix Sum + Binary Search
> Whenever you need to sample from a discrete distribution defined by weights, convert weights into a prefix-sum array (turning weights into cumulative boundaries), draw a uniform random value in the total range, and binary search for its segment. This generalizes to any "pick proportional to weight" requirement, including reservoir-style problems with fixed weight arrays.

## Algorithm Steps
**Constructor:**
1. Compute the prefix sums of `w`: `prefix[i] = w[0] + w[1] + ... + w[i]`.
2. Store `total = prefix[-1]`.

**`pickIndex()`:**
1. Draw `target = random.randint(1, total)` (1-indexed to match prefix sums cleanly).
2. Binary search `prefix` for the leftmost index `i` such that `prefix[i] >= target` (`bisect_left(prefix, target)`).
3. Return `i`.

## Solution (Python)
```python
import random
from bisect import bisect_left
from typing import List

class Solution:
    def __init__(self, w: List[int]):
        self.prefix = []
        running_total = 0
        for weight in w:
            running_total += weight
            self.prefix.append(running_total)
        self.total = running_total

    def pickIndex(self) -> int:
        target = random.randint(1, self.total)
        # First prefix sum >= target identifies which weight's segment
        # target falls into.
        return bisect_left(self.prefix, target)
```

## Solution (Java)
```java
import java.util.Random;

class Solution {
    private int[] prefix;
    private int total;
    private Random random = new Random();

    public Solution(int[] w) {
        prefix = new int[w.length];
        int runningTotal = 0;
        for (int i = 0; i < w.length; i++) {
            runningTotal += w[i];
            prefix[i] = runningTotal;
        }
        total = runningTotal;
    }

    public int pickIndex() {
        int target = random.nextInt(total) + 1; // uniform in [1, total]

        // First prefix sum >= target identifies which weight's segment
        // target falls into (equivalent to Python's bisect_left).
        int lo = 0, hi = prefix.length - 1;
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (prefix[mid] < target) {
                lo = mid + 1;
            } else {
                hi = mid;
            }
        }
        return lo;
    }
}
```

## Complexity Analysis
- **Time:** Constructor: O(N) to build prefix sums. `pickIndex`: O(log N) for the binary search.
- **Space:** O(N) for the prefix sum array.

## Edge Cases
- A single-element array (`w = [5]`) — every call must return index 0; `total = 5`, and any `target` in `[1, 5]` binary-searches to index 0.
- Very skewed weights (e.g., `w = [1, 1000000]`) — correctness doesn't depend on the magnitude ratio, only on `total` fitting comfortably in the integer range used by the random number generator.
- Repeated calls must be independent — don't accidentally introduce state that biases later calls (e.g., don't shrink `total` after a pick).

## Common Mistakes
- Building a flattened array of indices repeated by weight (e.g., `[0, 1,1,1]` for `w=[1,3]`) and picking uniformly from it — technically correct but wastes O(sum(w)) memory instead of O(N), which fails when weights are large.
- Off-by-one errors between 0-indexed and 1-indexed random ranges — using `random.randint(0, total-1)` requires `bisect_right` instead of `bisect_left` on the prefix array; mixing conventions is the most common bug here.
- Recomputing prefix sums inside `pickIndex` instead of once in the constructor, destroying the whole point of the "preprocess once" design.

## Related Problems
- [[Shortest-Word-Distance-II]]
- [[K-Closest-Points-to-Origin]]
- [[Ways-to-Make-a-Fair-Array]]

## Follow-up / Variants
A common follow-up: "what if weights can be updated dynamically between picks?" — that breaks the static prefix-sum array and pushes you toward a Fenwick tree (Binary Indexed Tree) that supports O(log N) weight updates alongside O(log N) prefix-sum queries.
