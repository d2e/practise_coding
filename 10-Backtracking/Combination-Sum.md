---
title: "Combination Sum"
aliases: []
tags: [leetcode, backtracking, recursion, medium]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 39
leetcode_difficulty: Medium
pattern: "Backtracking (Combinations with Reuse)"
status: not-started
---

# Combination Sum

## Problem Statement
Given an array of **distinct** positive integers `candidates` and a target integer `target`, return a list of all **unique combinations** of `candidates` where the chosen numbers sum to `target`. The same number may be chosen from `candidates` an **unlimited number of times**. Two combinations are unique if the frequency of at least one chosen number differs (order doesn't matter — `[2,2,3]` and `[3,2,2]` are the same combination and should appear only once).

Constraints: 1 ≤ candidates.length ≤ 30, 2 ≤ candidates[i] ≤ 40, all elements distinct, 1 ≤ target ≤ 40.

## Examples
```
Input: candidates = [2,3,6,7], target = 7
Output: [[2,2,3],[7]]

Input: candidates = [2,3,5], target = 8
Output: [[2,2,2,2],[2,3,3],[3,5]]

Input: candidates = [2], target = 1
Output: []
```

## Intuition & Approach
This is the template backtracking problem for "combinations that sum to a target with unlimited reuse." The search space is a decision tree: at each step, decide whether to include the current candidate again (since reuse is allowed) or move on to the next candidate. Recurse, and when the running sum equals `target`, record the current path as a valid combination; if it exceeds `target`, prune that branch (no point continuing down a path that's already too big, assuming positive integers).

The critical technique for avoiding duplicate combinations (like getting both `[2,3,3]` and `[3,2,3]`) is to **only allow moving forward through the candidates list**, never backward. Concretely: pass a `start` index into the recursive call, and when recursing after choosing `candidates[i]` again, call with `start=i` (not `start=i+1`, since reuse is allowed) — but when moving on to try the *next distinct* candidate without reusing the current one, that happens naturally by incrementing `i` in the loop. This "start index never decreases" rule is what enforces uniqueness without needing to sort-and-dedupe combinations after the fact.

Sorting `candidates` first isn't required for correctness here (since all elements are distinct and reuse is allowed), but it enables an important optimization: once the running sum plus the current candidate exceeds `target`, every later candidate (which is even larger, since the array is sorted) will also fail, so you can `break` out of the loop entirely instead of just `continue`-ing — pruning much more aggressively.

## Pattern Recognition
> [!tip] Pattern: Backtracking with Reuse
> "Find all combinations/subsets that satisfy a sum/property, with unlimited reuse of elements" is the backtracking-with-a-start-index pattern: recurse with `start=i` (allow re-picking the same index) rather than `start=i+1` (which would forbid reuse, as in the "each element used once" variant, Combination Sum II).

## Algorithm Steps
1. Sort `candidates` (enables the early-break pruning optimization).
2. Define a recursive `backtrack(start, remaining, path)`:
   - If `remaining == 0`, append a copy of `path` to results and return.
   - For `i` from `start` to `len(candidates) - 1`:
     - If `candidates[i] > remaining`, break (sorted array ⇒ nothing further can work either).
     - Choose `candidates[i]`: append it to `path`, recurse with `backtrack(i, remaining - candidates[i], path)` (note: `i`, not `i+1`, to allow reuse).
     - Un-choose: pop `path` to backtrack.
3. Call `backtrack(0, target, [])` and return the collected results.

## Solution (Python)
```python
from typing import List

def combinationSum(candidates: List[int], target: int) -> List[List[int]]:
    candidates.sort()
    results: List[List[int]] = []
    path: List[int] = []

    def backtrack(start: int, remaining: int) -> None:
        if remaining == 0:
            results.append(path[:])
            return

        for i in range(start, len(candidates)):
            if candidates[i] > remaining:
                break  # sorted, so no later candidate can work either
            path.append(candidates[i])
            backtrack(i, remaining - candidates[i])  # i, not i+1: allow reuse
            path.pop()

    backtrack(0, target)
    return results
```

## Solution (Java)
```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

class Solution {
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        Arrays.sort(candidates);
        List<List<Integer>> results = new ArrayList<>();
        List<Integer> path = new ArrayList<>();
        backtrack(candidates, target, 0, path, results);
        return results;
    }

    private void backtrack(int[] candidates, int remaining, int start,
                            List<Integer> path, List<List<Integer>> results) {
        if (remaining == 0) {
            results.add(new ArrayList<>(path));
            return;
        }

        for (int i = start; i < candidates.length; i++) {
            if (candidates[i] > remaining) {
                break; // sorted, so no later candidate can work either
            }
            path.add(candidates[i]);
            backtrack(candidates, remaining - candidates[i], i, path, results); // i, not i+1: allow reuse
            path.remove(path.size() - 1);
        }
    }
}
```

## Complexity Analysis
- **Time:** O(N^(T/M + 1)) in the worst case, where N = number of candidates, T = target, M = the minimal candidate value — this bounds the number of nodes in the recursion tree (the exact bound is loose; practically, it's exponential in the target divided by the smallest candidate, since that determines the maximum recursion depth). Each valid combination found also costs O(T/M) to copy into the results list.
- **Space:** O(T/M) for the recursion stack depth and the `path` list at its deepest point, plus O(number of valid combinations × average combination length) for the output.

## Edge Cases
- No combination sums to `target` (e.g., `candidates=[2]`, `target=1`) → return `[]`.
- `target` exactly equal to one of the candidates → single-element combination.
- All candidates larger than `target` → no valid combinations, the `break` pruning kicks in immediately.
- A candidate that evenly divides `target` many times (e.g., `candidates=[2]`, `target=8` → `[[2,2,2,2]]`).

## Common Mistakes
- Recursing with `start=i+1` after choosing `candidates[i]`, which incorrectly forbids reuse — that's the algorithm for Combination Sum II (each candidate used at most once), a different problem.
- Forgetting to copy `path` (`path[:]` or `list(path)`) when appending to `results` — appending the live list reference means later `pop()` calls mutate what's already "stored," corrupting all previously found results.
- Not sorting first and using `continue` instead of `break` on `candidates[i] > remaining` — still correct, just misses the pruning optimization that makes the search much faster in practice.
- Confusing this with subset-sum / partition problems that don't allow reuse, and copying that template’s `start+1` recursion by mistake.

## Related Problems
- [[Longest-Increasing-Subsequence]]
- [[Number-of-Islands]]
- [[Word-Ladder]]

## Follow-up / Variants
Classic follow-up progression: Combination Sum II (each candidate usable at most once, array may contain duplicates — requires sorting + skipping adjacent duplicates at the same recursion depth to avoid duplicate combinations), and Combination Sum III (choose exactly `k` numbers from 1-9 summing to `target`, each used at most once). All three share the same backtracking skeleton with small tweaks to the reuse/duplicate-skipping rules.
