---
title: "Sort Integers by The Power Value"
aliases: []
tags: [leetcode, memoization, sorting, medium]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 1387
leetcode_difficulty: Medium
pattern: "Memoized Recursion + Custom Sort Key"
status: not-started
---

# Sort Integers by The Power Value

## Problem Statement
The "power value" of an integer `x` is the number of steps it takes to transform `x` into 1, using this process:
- If `x` is even, `x = x / 2`.
- If `x` is odd, `x = 3 * x + 1`.

(This is the Collatz conjecture sequence — empirically every positive integer reaches 1 eventually, and the problem assumes this holds within its constraints.)

Given integers `lo`, `hi`, and `k`, for every integer in the range `[lo, hi]`, compute its power value, sort the range by power value ascending (breaking ties by the integer's own value ascending), and return the `k`-th integer (1-indexed) in that sorted order.

## Examples
```
Input: lo = 12, hi = 15, k = 2
Output: 13
Explanation:
The power values are: 12 -> 9, 13 -> 9, 14 -> 17, 15 -> 17
(12 and 13 both have power value 9; 14 and 15 both have power value 17)
Sorted ascending by power value, tie-break by integer value: [12, 13, 14, 15]
The 2nd smallest is 13.

Input: lo = 1, hi = 1, k = 1
Output: 1
```

## Intuition & Approach
The problem has two independent parts: (1) compute the power value of every integer in `[lo, hi]`, and (2) sort by that computed key. Part 2 is trivial once part 1 is done — a single `sorted(range(lo, hi+1), key=power_value)` call handles both the primary sort and the tie-break, since Python's sort is stable and using `(power_value(x), x)` as an explicit key makes the tie-break deterministic and correct.

Part 1 is where the interesting technique lives. Computing each integer's power value via direct simulation (repeatedly halving or `3x+1`) is fine for the given range (`hi <= 1000`), but there's a classic optimization worth demonstrating: **memoization**. The power value of `x` is `1 + power_value(next(x))`. Many different starting values collapse into the same intermediate values on their way down to 1 (e.g., both 12 and 6 eventually pass through 3), so caching `power_value(x)` for every `x` you've already computed avoids redundant recursive work — this turns overlapping subproblems (a Collatz "chain") into a memoized recursion, similar in spirit to memoized DP, even though the recursion isn't over a simple linear index.

## Pattern Recognition
> [!tip] Pattern: Memoized Recursion for Repeated Subproblems
> When a recursive/iterative process on integer `x` depends only on the result for some smaller/related value (here, `x/2` or `3x+1`), and you're asked to run this over a *range* of starting values, caching results avoids recomputation across the range — even when the recursion doesn't have an obvious "smaller subproblem" shape like classic DP.

## Algorithm Steps
1. Build a `memo` dictionary mapping integer -> power value, seeded with `memo[1] = 0`.
2. Write a recursive (or iterative with an explicit stack) `power_value(x)` that: if `x` is in `memo`, return it; otherwise compute `next_x` per the even/odd rule, recursively find `power_value(next_x)`, store `memo[x] = 1 + power_value(next_x)`, and return it.
3. Build the list `range(lo, hi + 1)`.
4. Sort that list using `key=lambda x: (power_value(x), x)`.
5. Return the element at index `k - 1` (1-indexed).

## Solution (Python)
```python
from functools import lru_cache

class Solution:
    def getKth(self, lo: int, hi: int, k: int) -> int:
        @lru_cache(maxsize=None)
        def power_value(x: int) -> int:
            if x == 1:
                return 0
            next_x = x // 2 if x % 2 == 0 else 3 * x + 1
            return 1 + power_value(next_x)

        candidates = list(range(lo, hi + 1))
        candidates.sort(key=lambda x: (power_value(x), x))
        return candidates[k - 1]
```

## Solution (Java)
```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

class Solution {
    private Map<Integer, Integer> memo = new HashMap<>();

    public int getKth(int lo, int hi, int k) {
        List<Integer> candidates = new ArrayList<>();
        for (int i = lo; i <= hi; i++) {
            candidates.add(i);
        }

        candidates.sort((a, b) -> {
            int pa = powerValue(a), pb = powerValue(b);
            if (pa != pb) return pa - pb;
            return a - b;
        });

        return candidates.get(k - 1);
    }

    private int powerValue(int x) {
        if (x == 1) return 0;
        if (memo.containsKey(x)) return memo.get(x);

        int nextX = (x % 2 == 0) ? x / 2 : 3 * x + 1;
        int result = 1 + powerValue(nextX);
        memo.put(x, result);
        return result;
    }
}
```

## Complexity Analysis
- **Time:** O(R log R * C) where R = hi - lo + 1 is the range size and C is the average (amortized, thanks to memoization) chain length per value — the R log R factor comes from sorting; computing all power values is close to linear in R once memoized because chains overlap heavily. In the worst case without memoization it would be O(R * C).
- **Space:** O(R + M) where M is the number of distinct intermediate values visited across all chains, stored in the memo cache.

## Edge Cases
- `lo == hi` — range of size 1; `k` must be 1, return that single value directly.
- Two integers with identical power values — must be tie-broken by ascending integer value, not left in arbitrary order.
- `lo == 1` — power value of 1 is 0 by definition (base case), the smallest possible power value in the range.

## Common Mistakes
- Recomputing the power value from scratch for every integer without memoization — correct but wasteful, and a missed opportunity to demonstrate the caching insight.
- Forgetting the tie-break on integer value, which under Python's stable sort might accidentally work if the range is already in ascending input order, but will break if the input processing order changes (e.g., if you deduplicate or reorder before sorting).
- Off-by-one on `k` — the problem is 1-indexed, so the answer is `candidates[k - 1]`, not `candidates[k]`.

## Related Problems
- [[K-Closest-Points-to-Origin]]
- [[Counting-Bits]]
- [[Minimum-Absolute-Difference]]

## Follow-up / Variants
A natural extension: "return the power value itself for the k-th integer, not the integer" — trivial change to the return statement. A harder variant: prove/bound the maximum chain length within a given range to argue a tighter complexity bound (related to open questions about the Collatz conjecture, though not required for the LeetCode constraints).
