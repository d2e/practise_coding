---
title: "Patterns Cheatsheet"
aliases: ["Cheatsheet"]
tags: [moc, cheatsheet, leetcode]
---

# Pattern Cheatsheet

One-page quick reference. See [[00-MOC|the MOC]] for the full problem list, and each folder's `00-index.md` for a deeper primer on that pattern.

| Pattern | Recognize it when... | Core idea | Typical complexity | Problems |
|---|---|---|---|---|
| **Two Pointers** | Sorted array, need pairs/merging, in-place compaction | Two indices moving toward/away from each other | O(n) | [[Merge-Sorted-Array]], [[Minimum-Absolute-Difference]], [[Remove-Nth-Node-From-End-of-List]] |
| **Sliding Window** | Contiguous subarray/substring with a constraint (sum, distinct count, max diff) | Expand right, shrink left when constraint breaks | O(n) | [[Longest-Subarray-of-1s-After-Deleting-One-Element]], [[Longest-Continuous-Subarray-With-Absolute-Diff-Limit]] |
| **Prefix Sum** | Range sum / balance queries, "does removing index i change something" | Precompute cumulative sums for O(1) range queries | O(n) build, O(1) query | [[Ways-to-Make-a-Fair-Array]], [[Random-Pick-with-Weight]] |
| **Monotonic Stack** | "Next greater/smaller element", span problems | Stack keeps elements in increasing/decreasing order, pop when broken | O(n) | [[Daily-Temperatures]] |
| **Monotonic Deque** | Sliding window max/min | Deque keeps window candidates in sorted order, evict from both ends | O(n) | [[Longest-Continuous-Subarray-With-Absolute-Diff-Limit]] |
| **Heap / Priority Queue** | "K closest/largest/smallest", merge sorted streams | Push/pop maintaining a size-k or ordered heap | O(n log k) | [[K-Closest-Points-to-Origin]], [[Merge-k-Sorted-Lists]] |
| **Union-Find (DSU)** | Grouping/merging by transitive relation, connectivity | Union nodes that co-occur, find() to check group membership | O(n α(n)) | [[Accounts-Merge]] |
| **Hashing / Design** | Fast lookup, frequency counting, precompute-then-query APIs | Hash map from key → data needed to answer queries in O(1)/O(log n) | O(n) build | [[Ransom-Note]], [[Shortest-Word-Distance-II]] |
| **BFS (unweighted shortest path)** | Grid/graph, "fewest steps/levels" | Level-order expansion from source(s) | O(V+E) | [[Number-of-Islands]], [[Word-Ladder]], [[Walls-and-Gates]], [[Check-Completeness-of-a-Binary-Tree]] |
| **Topological Sort** | Ordering with precedence constraints, cycle detection | Kahn's algorithm (indegree queue) or DFS postorder reversal | O(V+E) | [[Alien-Dictionary]] |
| **Dijkstra / Weighted Shortest Path** | Costs differ per edge/operation, minimize total cost | Min-heap of (cost, state), relax neighbors | O(E log V) | [[Digit-Operations-to-Make-Two-Integers-Equal]] |
| **Backtracking** | "Find all combinations/permutations satisfying X" | Choose → recurse → un-choose, prune when constraint violated | Exponential, pruned | [[Combination-Sum]] |
| **Divide & Conquer / Recursion on structure** | Problem decomposes into independent subregions (quadrants, halves) | Solve subproblems, combine | O(n log n) typical | [[Construct-Quad-Tree]], [[Merge-k-Sorted-Lists]] |
| **1D DP** | Optimal value at position i depends on a few previous positions | dp[i] = f(dp[i-1], dp[i-2], ...) | O(n) | [[Counting-Bits]], [[Maximum-Subarray]], [[Longest-Increasing-Subsequence]] |
| **2D Grid DP** | Path counting/optimization on a grid | dp[i][j] = f(dp[i-1][j], dp[i][j-1]) | O(m·n) | [[Minimum-Path-Sum]] |
| **Binary Search on Answer / Patience Sorting** | Sorted structure hidden in the problem, or "can we do it in X?" is monotonic | Binary search over value space or over a maintained sorted array | O(log n) per step | [[Longest-Increasing-Subsequence]], [[Sort-Integers-by-The-Power-Value]] |
| **State-space Memoization** | Repeated recursive computation over small bounded input (e.g. Collatz-like chains) | Cache computed results by key | Amortized O(n) | [[Sort-Integers-by-The-Power-Value]] |
| **FSM / String Parsing** | Validate a string against grammar rules (numbers, dates) | Explicit state machine or careful char-by-char scan with flags | O(n) | [[Valid-Number]] |
| **String Matching** | Find occurrence of pattern in text | Naive O(n·m) or KMP O(n+m) with prefix-function | O(n+m) optimal | [[Find-the-Index-of-the-First-Occurrence-in-a-String]] |

## General Interview Strategy

1. **Restate the problem** and confirm constraints (array size, value ranges, duplicates allowed, sorted or not).
2. **Say the brute force out loud** first — it anchors the interviewer and often reveals the pattern needed to optimize.
3. **Name the pattern explicitly** ("this looks like a sliding window because we need a contiguous range with a constraint") — interviewers are grading pattern recognition, not just correct code.
4. **Code incrementally**, narrating as you go; handle the empty/single-element edge case first.
5. **State complexity unprompted** at the end, and mention the follow-up/optimized variant if one exists (most notes in this vault have a *Follow-up / Variants* section for exactly this).
