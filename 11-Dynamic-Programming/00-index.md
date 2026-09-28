---
title: "Dynamic Programming — Pattern Primer"
aliases: [DP Index]
tags: [pattern-primer, dynamic-programming]
---

# Dynamic Programming

DP applies whenever a problem has **overlapping subproblems** (the same smaller question gets asked repeatedly) and **optimal substructure** (the optimal answer to the whole problem can be built from optimal answers to its subproblems). Recognizing DP is mostly about spotting a recursive relationship where a brute-force recursion would redo the same work exponentially — the fix is to cache (memoize) or build bottom-up (tabulate) so each subproblem is solved exactly once.

## How to recognize a DP problem
- The problem asks to **optimize** something (min/max/count) over a sequence, grid, or set, subject to constraints on how you can move through it.
- A brute-force recursive solution is easy to write but clearly re-solves identical subproblems (draw the recursion tree — if it branches and re-converges, that's your signal).
- You can describe the answer to `f(n)` in terms of `f` of *strictly smaller* arguments.

## Generic template
```python
# Top-down (memoized)
from functools import lru_cache

@lru_cache(maxsize=None)
def solve(state):
    if base_case(state):
        return base_value
    return combine(solve(smaller_state) for smaller_state in transitions(state))

# Bottom-up (tabulated)
dp = [base_value] * (n + 1)
for i in range(1, n + 1):
    dp[i] = combine(dp[j] for j in relevant_smaller_indices(i))
return dp[n]
```
Bottom-up tabulation is usually preferred once the recurrence is clear, since it avoids recursion overhead/stack depth and often reveals a rolling-array space optimization (only the last one or two rows/states are ever needed).

## Problems in this folder
- [[Counting-Bits]] — DP over bit structure: `dp[i] = dp[i >> 1] + (i & 1)`
- [[Minimum-Path-Sum]] — grid DP: each cell depends only on the cell above and to the left
- [[Maximum-Subarray]] — Kadane's algorithm: "extend the previous run, or restart here"
- [[Longest-Increasing-Subsequence]] — O(n²) DP, then upgraded to O(n log n) via patience sorting / binary search

Cross-reference [[Digit-Operations-to-Make-Two-Integers-Equal]] in the Graphs folder — it's tempting to reach for DP on "minimum operations" problems, but when the state space is really a graph of reachable values with a validity constraint, BFS/Dijkstra over an implicit graph is usually cleaner than trying to force a DP recurrence over numeric states.
