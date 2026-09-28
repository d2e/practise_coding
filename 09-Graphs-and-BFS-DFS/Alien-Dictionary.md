---
title: "Alien Dictionary"
aliases: []
tags: [leetcode, topological-sort, hard]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 269
leetcode_difficulty: Hard
pattern: "Topological Sort"
status: not-started
---

# Alien Dictionary

## Problem Statement
There is a new alien language that uses the English alphabet, but the order among the letters is unknown. You are given a list of strings `words` from the alien language's dictionary, where the strings in `words` are sorted lexicographically according to this unknown alien alphabet's rules.

Derive the order of letters in this alien language, and return it as a string. If the given list of words is not a valid ordering under *any* possible character ordering, return `""`. If there are multiple valid orderings, return any one of them.

A word `a` is lexicographically smaller than `b` if, at the first position where they differ, `a`'s character comes before `b`'s character in the alien order — with the additional rule that if `a` is a strict prefix of `b`, `a` must come first (like normal dictionaries); if `b` is a proper prefix of `a` instead, the ordering is invalid.

## Examples
```
Input: words = ["wrt","wrf","er","ett","rftt"]
Output: "wertf"
```

```
Input: words = ["z","x"]
Output: "zx"
```

```
Input: words = ["z","x","z"]
Output: ""            # x < z and z < x can't both hold -> cycle -> invalid
```

## Intuition & Approach
The list is already sorted according to *some* unknown alphabet — our job is to reverse-engineer that alphabet. The only information we're given about ordering comes from *adjacent* word pairs: comparing `words[i]` to `words[i+1]` lets us find the first index where their characters differ, which tells us "this character comes before that character" in the alien alphabet. Comparing non-adjacent pairs gives no new information beyond what transitivity through adjacent pairs already implies, so we only need `n-1` comparisons for `n` words.

Each such finding is an edge in a directed graph: `char_a -> char_b` meaning `char_a` precedes `char_b`. Once we've built this graph from every adjacent pair, a valid global letter ordering is exactly a **topological sort** of the graph. If the graph has a cycle, no valid ordering exists (the input is inconsistent), and we return `""`.

There's one more subtlety, the prefix rule: if we ever find that `words[i]` is *longer than* `words[i+1]` but `words[i+1]` is a prefix of `words[i]` (e.g., `"abc"` before `"ab"`), that's an immediate contradiction — no character ordering can make a longer word that starts with a shorter word come first. That must be checked before extracting edges from the pair.

## Pattern Recognition
> [!tip] Pattern: Topological Sort
> Whenever a problem gives you pairwise "this must come before that" constraints (build order, course prerequisites, alien word order, compilation dependencies) and asks for *a* valid global order — or to detect that no such order exists — it's topological sort on a directed graph built from those constraints (Kahn's algorithm / BFS with in-degrees, or DFS with a post-order reversal).

## Algorithm Steps
1. Initialize a graph (adjacency set) and an in-degree counter for every unique character seen across all words.
2. For each adjacent pair `(words[i], words[i+1])`:
   a. If `words[i]` is longer than `words[i+1]` and `words[i+1]` is a prefix of `words[i]`, the input is invalid — return `""`.
   b. Otherwise, find the first index where the two words differ, add a directed edge `words[i][j] -> words[i+1][j]` (if not already present), and increment the in-degree of the destination character.
3. Run Kahn's algorithm: seed a queue with all characters that have in-degree 0, repeatedly pop a character, append it to the result, and decrement the in-degree of its neighbors, pushing any that drop to 0.
4. If the result contains all unique characters, return it as a string; otherwise a cycle exists among the remaining characters — return `""`.

## Solution (Python)
```python
from collections import defaultdict, deque
from typing import List

def alienOrder(words: List[str]) -> str:
    graph = defaultdict(set)
    in_degree = {c: 0 for word in words for c in word}

    for first, second in zip(words, words[1:]):
        min_len = min(len(first), len(second))
        found_difference = False
        for i in range(min_len):
            if first[i] != second[i]:
                if second[i] not in graph[first[i]]:
                    graph[first[i]].add(second[i])
                    in_degree[second[i]] += 1
                found_difference = True
                break
        # "abc" before "ab" is impossible under any ordering.
        if not found_difference and len(first) > len(second):
            return ""

    queue = deque([c for c in in_degree if in_degree[c] == 0])
    order = []
    while queue:
        c = queue.popleft()
        order.append(c)
        for nxt in graph[c]:
            in_degree[nxt] -= 1
            if in_degree[nxt] == 0:
                queue.append(nxt)

    if len(order) != len(in_degree):
        return ""  # cycle detected — not every character could be ordered
    return "".join(order)
```

## Solution (Java)
```java
import java.util.*;

class Solution {
    public String alienOrder(String[] words) {
        Map<Character, Set<Character>> graph = new HashMap<>();
        Map<Character, Integer> inDegree = new HashMap<>();
        for (String word : words) {
            for (char c : word.toCharArray()) {
                graph.putIfAbsent(c, new HashSet<>());
                inDegree.putIfAbsent(c, 0);
            }
        }

        for (int i = 0; i < words.length - 1; i++) {
            String first = words[i], second = words[i + 1];
            int minLen = Math.min(first.length(), second.length());
            boolean foundDifference = false;
            for (int j = 0; j < minLen; j++) {
                char a = first.charAt(j), b = second.charAt(j);
                if (a != b) {
                    if (graph.get(a).add(b)) {
                        inDegree.merge(b, 1, Integer::sum);
                    }
                    foundDifference = true;
                    break;
                }
            }
            // "abc" before "ab" is impossible under any ordering.
            if (!foundDifference && first.length() > second.length()) {
                return "";
            }
        }

        Deque<Character> queue = new ArrayDeque<>();
        for (char c : inDegree.keySet()) {
            if (inDegree.get(c) == 0) queue.offer(c);
        }

        StringBuilder order = new StringBuilder();
        while (!queue.isEmpty()) {
            char c = queue.poll();
            order.append(c);
            for (char next : graph.get(c)) {
                inDegree.merge(next, -1, Integer::sum);
                if (inDegree.get(next) == 0) queue.offer(next);
            }
        }

        if (order.length() != inDegree.size()) return ""; // cycle detected
        return order.toString();
    }
}
```

## Complexity Analysis
- **Time:** O(C) where C is the total number of characters across all words — building edges is proportional to total character count (each adjacent pair contributes at most one edge lookup), and Kahn's algorithm visits each node and edge once, and the number of distinct characters/edges is bounded by the alphabet size (≤ 26) and total input size.
- **Space:** O(1) to O(26²) for the graph and in-degree map (bounded by alphabet size), plus O(C) for the input itself.

## Edge Cases
- A single word in the list — no edges can be derived, any permutation of its unique characters is technically valid; returning them in any order (e.g., insertion order) is acceptable.
- Words that produce a cycle, like `["z", "x", "z"]` — must return `""`.
- The invalid-prefix case, like `["abc", "ab"]` — must return `""` even though no character-level contradiction exists.
- Characters that never appear in any comparison (isolated nodes with in-degree 0) — they still belong in the output, just with no ordering constraint relative to others beyond what Kahn's algorithm naturally assigns.
- Duplicate words adjacent to each other — comparing them finds no difference and isn't a length violation, so it's simply a no-op for edge extraction.

## Common Mistakes
- Comparing every pair of words instead of only adjacent pairs — unnecessary and can introduce misleading "evidence" if done incorrectly, though transitivity means it's redundant at best.
- Forgetting the prefix-violation check (`"abc"` before `"ab"`), which silently produces a wrong non-empty answer instead of detecting the contradiction.
- Adding a duplicate edge each time the same character pair recurs across multiple adjacent-word comparisons, which double-counts in-degree and breaks Kahn's algorithm — always guard with `if second[i] not in graph[first[i]]`.
- Only extracting the *first* differing character and forgetting to `break` out of the inner loop, which can create edges from characters that come after the true first difference.

## Related Problems
- [[Word-Ladder]]
- [[Walls-and-Gates]]
- [[Accounts-Merge]]

## Follow-up / Variants
A natural follow-up is "Course Schedule" / "Course Schedule II" (LC 207 / 210), which is the same topological-sort machinery with explicit prerequisite pairs instead of derived character edges — good practice for recognizing that the *graph-building step* is what changes between problems, while the topological sort itself is a reusable template.
