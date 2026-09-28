---
title: "Digit Operations to Make Two Integers Equal"
aliases: []
tags: [leetcode, dijkstra, medium]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: unknown
leetcode_difficulty: Medium
pattern: "Dijkstra over an Implicit State Graph"
status: not-started
---

# Digit Operations to Make Two Integers Equal

> [!warning] Verify exact constraints
> This is a newer LeetCode problem and the precise rule set (which intermediate states are disallowed, exact cost model) should be double-checked against the current LeetCode listing before relying on this note for an exact submission. What follows is accurate to the *general shape* of the problem as commonly described: transform one integer into another via single-digit increments/decrements, at minimum total cost, subject to a validity constraint on every intermediate value (commonly: the intermediate number must not be prime, or must stay within some numeric range) — and the note focuses on teaching the underlying technique (Dijkstra / 0-1 BFS over an implicit graph of numbers) that solves this entire family of problems, rather than asserting an exact constraint wording.

## Problem Statement
You are given two integers, `n` and `m`, that have the same number of digits. In one operation, you may pick any digit of the current number and either increment or decrement it by 1 (digits wrap or are restricted to stay within `0-9` depending on the exact variant, and the leading digit typically cannot become `0`). Each such operation costs 1. Additionally, at every step, the *resulting* number must satisfy some validity constraint (commonly: it must not be a prime number). Find the minimum total cost to transform `n` into `m` using only valid intermediate numbers; if it's impossible, return `-1`.

## Examples
```
Input: n = 10, m = 12
Output: 2
Explanation (illustrative): 10 -> 11 -> 12, each step costs 1, and assuming
11 is a valid intermediate state under the problem's constraint, total cost = 2.
```

```
Input: n = 4, m = 8
Output: -1 (illustrative)
Explanation: if every path from 4 to 8 is forced to pass through a disallowed
intermediate value (e.g., a prime, under a "must not be prime" constraint),
no valid transformation exists.
```

*(Treat the exact numbers above as illustrative of the mechanics, not verified LeetCode test cases — the point is the shape of the transformation and cost, not the specific digits.)*

## Intuition & Approach
Strip away the specific "which numbers are disallowed" rule for a moment, and what's left is a very recognizable shape: you have a *start state* (`n`) and a *target state* (`m`), a set of *moves* you can make from any state (increment or decrement one digit), each move has a *cost* (here, uniformly 1), and some states are simply **not allowed to be visited**. That is exactly a shortest-path problem on an implicit graph, where:
- Nodes = valid numbers (with the same digit count as `n`/`m`).
- Edges = "change one digit by ±1," each with weight 1.
- Forbidden numbers are removed from the graph entirely (never enqueued/relaxed).

Because every edge has the *same* weight (cost 1 per digit change), this collapses to plain BFS (BFS finds shortest paths in unweighted, or uniformly-weighted, graphs just as well as Dijkstra does — Dijkstra is only strictly necessary when edge weights differ). If a variant of this problem gives *different* costs for different digit changes (e.g., changing digit 5 costs more than changing digit 2), then it becomes genuine Dijkstra with a min-heap priority queue, since BFS's FIFO-queue guarantee (shortest-path-first) only holds for equal-weight edges.

The graph is never built explicitly — with up to `10^d` possible numbers for `d` digits, materializing it would be wasteful. Instead, neighbors are generated on the fly: for a number's digit string, try `+1`/`-1` on each digit position, skip anything invalid (wrong digit count, leading zero, fails the validity constraint), and treat those as edges to explore next.

## Pattern Recognition
> [!tip] Pattern: Shortest Path over an Implicit Graph
> Whenever a problem says "minimum number of operations to turn state A into state B," where each operation is a small local edit and some intermediate states are forbidden, don't think "DP over all states" first — think "BFS/Dijkstra where nodes are states and edges are generated on the fly." Use BFS when all operations cost the same; use Dijkstra (min-heap) the moment operations have differing costs.

## Algorithm Steps
1. Represent each number as its digit string so digit-level edits are easy and digit count (and leading-zero constraints) can be validated.
2. Precompute a fast "is this number in a valid state" check (e.g., a sieve of non-primes up to the max possible value, if the constraint is primality-based).
3. Run BFS (or Dijkstra, if costs vary) from `n`: maintain a `distance` map (or array) initialized to infinity, with `distance[n] = 0`.
4. From the current number, generate all neighbors by changing one digit by ±1 (respecting digit bounds and leading-digit rules); skip neighbors that fail the validity constraint or have already been finalized with a shorter distance.
5. Relax/enqueue valid neighbors; for BFS, the first time you pop `m` from the queue, its distance is the answer. For Dijkstra, pop from a min-heap by current best distance and stop once `m` is popped.
6. If the search exhausts without ever reaching `m`, return `-1`.

## Solution (Python)
```python
from collections import deque

def min_digit_operations(n: int, m: int, is_valid) -> int:
    """
    Generic BFS template for this family of problems.
    `is_valid(number: int) -> bool` encapsulates whatever the problem's
    intermediate-state constraint actually is (e.g., "is not prime").
    Assumes n and m have the same digit count and all operations cost 1.
    """
    digits = len(str(n))

    def neighbors(num: int):
        digit_list = list(str(num).zfill(digits))
        for i in range(digits):
            original = digit_list[i]
            for delta in (1, -1):
                new_digit = int(original) + delta
                if new_digit < 0 or new_digit > 9:
                    continue
                if i == 0 and new_digit == 0 and digits > 1:
                    continue  # avoid an illegal leading zero
                digit_list[i] = str(new_digit)
                candidate = int("".join(digit_list))
                digit_list[i] = original
                if is_valid(candidate):
                    yield candidate

    if n == m:
        return 0

    visited = {n}
    queue = deque([(n, 0)])
    while queue:
        current, cost = queue.popleft()
        for nxt in neighbors(current):
            if nxt == m:
                return cost + 1
            if nxt not in visited:
                visited.add(nxt)
                queue.append((nxt, cost + 1))

    return -1
```

## Solution (Java)
```java
import java.util.*;
import java.util.function.IntPredicate;

class Solution {
    /**
     * Generic BFS template for this family of problems.
     * `isValid` encapsulates whatever the problem's intermediate-state
     * constraint actually is (e.g., "is not prime").
     * Assumes n and m have the same digit count and all operations cost 1.
     */
    public int minDigitOperations(int n, int m, IntPredicate isValid) {
        if (n == m) return 0;

        int digits = String.valueOf(n).length();
        Set<Integer> visited = new HashSet<>();
        visited.add(n);
        Deque<int[]> queue = new ArrayDeque<>(); // {number, cost}
        queue.offer(new int[]{n, 0});

        while (!queue.isEmpty()) {
            int[] cur = queue.poll();
            int current = cur[0], cost = cur[1];

            for (int next : neighbors(current, digits, isValid)) {
                if (next == m) return cost + 1;
                if (visited.add(next)) {
                    queue.offer(new int[]{next, cost + 1});
                }
            }
        }

        return -1;
    }

    private List<Integer> neighbors(int num, int digits, IntPredicate isValid) {
        char[] digitChars = String.format("%0" + digits + "d", num).toCharArray();
        List<Integer> result = new ArrayList<>();

        for (int i = 0; i < digits; i++) {
            char original = digitChars[i];
            for (int delta : new int[]{1, -1}) {
                int newDigit = (original - '0') + delta;
                if (newDigit < 0 || newDigit > 9) continue;
                if (i == 0 && newDigit == 0 && digits > 1) continue; // avoid an illegal leading zero

                digitChars[i] = (char) ('0' + newDigit);
                int candidate = Integer.parseInt(new String(digitChars));
                digitChars[i] = original;

                if (isValid.test(candidate)) result.add(candidate);
            }
        }

        return result;
    }
}
```

## Complexity Analysis
- **Time:** O(V · D) where V is the number of distinct valid numbers reachable (bounded by `10^d` for d digits) and D is the number of digits — each visited number generates up to `2·D` neighbors.
- **Space:** O(V) for the visited set / distance map and the BFS queue.

## Edge Cases
- `n == m` already — answer is 0 with no operations needed.
- `n` or `m` itself fails the validity constraint — depending on problem wording, the *endpoints* may be exempt from the constraint even though intermediate states aren't; this must be verified against the exact problem statement.
- No valid path exists at all (target is isolated by forbidden numbers) — must return `-1`, not loop forever; BFS naturally terminates once the queue empties.
- Leading zero produced by decrementing the leading digit — must be excluded as an invalid state for numbers with more than one digit.
- Numbers with different digit counts — outside the problem's stated constraints, but defensive code should not assume equal length without the problem guaranteeing it.

## Common Mistakes
- Reaching for Dijkstra/a priority queue by default when all edges actually cost the same — plain BFS is simpler, faster, and sufficient whenever weights are uniform.
- Forgetting to mark `m` as "found" the moment it's *generated* as a neighbor rather than waiting to pop it from the queue — for BFS this optimization avoids one extra queue round-trip, though it isn't required for correctness.
- Not validating leading-zero digit edits, which can silently allow transformations through numbers with fewer digits than intended.
- Recomputing the validity check (e.g., primality) from scratch for every candidate instead of precomputing it once (e.g., via a sieve), which can dominate runtime for larger digit counts.

## Related Problems
- [[Walls-and-Gates]]
- [[Word-Ladder]]
- [[Counting-Bits]]

## Follow-up / Variants
A natural follow-up is to allow differing costs per digit position or per digit value (e.g., changing a `9` costs more than changing a `1`), which forces genuine Dijkstra with a min-heap instead of plain BFS. Another variant restricts operations to changing a digit by more than ±1 in a single move (e.g., "set any digit to any value for cost equal to the absolute difference"), which changes the neighbor-generation step but leaves the overall shortest-path framing intact.
