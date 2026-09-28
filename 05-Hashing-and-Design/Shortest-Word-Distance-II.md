---
title: "Shortest Word Distance II"
aliases: []
tags: [leetcode, design, hashing, two-pointers, medium]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 244
leetcode_difficulty: Medium
pattern: "Preprocessing + Two-Pointer Merge (Amortized Design)"
status: not-started
---

# Shortest Word Distance II

## Problem Statement
Design a class that is initialized once with an array of words (which may contain duplicates), and then supports many repeated calls to a `shortest(word1, word2)` method, each returning the shortest distance (in index positions) between an occurrence of `word1` and an occurrence of `word2` in the array. The class should be optimized for the case where `shortest` is called many times.

This differs from "Shortest Word Distance I" (a single query, so a single linear scan suffices) — here the *repeated-query* nature is the whole point: you should do expensive work once in the constructor and cheap work per query.

## Examples
```
wordsDict = ["practice", "makes", "perfect", "coding", "makes"]

WordDistance(wordsDict)
shortest("coding", "practice")  -> 3   (index 3 and index 0)
shortest("makes", "coding")     -> 1   (index 1 or 4, and index 3 -> closest is |4-3|=1)
```

## Intuition & Approach
If you handled every `shortest` call with a fresh linear scan over the whole array (as in Shortest Word Distance I), each query would cost O(N), which is wasteful when the same array is queried many times. The problem is explicitly telling you: "the array doesn't change between queries, but the query pairs do — invest work upfront."

The key idea: in the constructor, build a hash map from each distinct word to a **sorted** list of the indices where it occurs (sorted for free, since you build it by scanning the array left to right). Given two sorted lists of indices — one for `word1`, one for `word2` — finding the minimum absolute difference between an element of list A and an element of list B is a classic **two-pointer merge** problem: walk both lists simultaneously, always advancing the pointer at the smaller index (since advancing the larger one can only increase the gap), tracking the minimum difference seen.

This is the same "two sorted lists, walk together" idea used in merging sorted arrays — it turns an O(len(A) * len(B)) brute-force comparison into O(len(A) + len(B)).

## Pattern Recognition
> [!tip] Pattern: Preprocess Once, Query Cheaply
> When a problem structure is "one-time setup, then many queries," that's a signal to move the expensive work into the constructor/preprocessing step, even if it means using more memory, so that each query is fast. Combined here with the "two sorted lists -> two-pointer merge" pattern for finding closest pairs across two sorted sequences.

## Algorithm Steps
**Constructor:**
1. Iterate over `wordsDict` with the index.
2. Append each index to `index_map[word]` (a `defaultdict(list)`).

**`shortest(word1, word2)`:**
1. Retrieve `indices1 = index_map[word1]` and `indices2 = index_map[word2]` (both sorted ascending).
2. Use two pointers `i = 0`, `j = 0` over `indices1` and `indices2`.
3. Track `best = min(best, abs(indices1[i] - indices2[j]))`.
4. Advance whichever pointer points at the smaller index (`i += 1` if `indices1[i] < indices2[j]`, else `j += 1`).
5. Stop when either pointer runs off the end of its list; return `best`.

## Solution (Python)
```python
from typing import List
from collections import defaultdict

class WordDistance:
    def __init__(self, wordsDict: List[str]):
        self.index_map = defaultdict(list)
        for i, word in enumerate(wordsDict):
            self.index_map[word].append(i)

    def shortest(self, word1: str, word2: str) -> int:
        indices1 = self.index_map[word1]
        indices2 = self.index_map[word2]

        i = j = 0
        best = float("inf")
        while i < len(indices1) and j < len(indices2):
            best = min(best, abs(indices1[i] - indices2[j]))
            # Advance the pointer at the smaller index: moving the
            # larger one can never shrink the gap.
            if indices1[i] < indices2[j]:
                i += 1
            else:
                j += 1

        return best
```

## Solution (Java)
```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

class WordDistance {
    private Map<String, List<Integer>> indexMap = new HashMap<>();

    public WordDistance(String[] wordsDict) {
        for (int i = 0; i < wordsDict.length; i++) {
            indexMap.computeIfAbsent(wordsDict[i], k -> new ArrayList<>()).add(i);
        }
    }

    public int shortest(String word1, String word2) {
        List<Integer> indices1 = indexMap.get(word1);
        List<Integer> indices2 = indexMap.get(word2);

        int i = 0, j = 0;
        int best = Integer.MAX_VALUE;
        while (i < indices1.size() && j < indices2.size()) {
            best = Math.min(best, Math.abs(indices1.get(i) - indices2.get(j)));
            // Advance the pointer at the smaller index: moving the
            // larger one can never shrink the gap.
            if (indices1.get(i) < indices2.get(j)) {
                i++;
            } else {
                j++;
            }
        }

        return best;
    }
}
```

## Complexity Analysis
- **Time:** Constructor: O(N), scanning the word list once. `shortest`: O(A + B), where A and B are the occurrence counts of `word1` and `word2` — far better than the O(N) per-query cost of the naive repeated-scan approach when the same words are queried many times.
- **Space:** O(N) for the index map, storing every word's positions once.

## Edge Cases
- `word1 == word2`? (Not part of the standard problem's guarantees, but worth considering: if allowed, the answer should be the minimum gap between two *distinct* occurrences, not 0 — a naive two-pointer walk would need a guard against comparing an index with itself.)
- A word that appears only once — its index list has length 1, and the two-pointer loop still works correctly (terminates after one comparison per exhaustion of the shorter list).
- Very skewed frequency (one word appears 1 time, the other 10,000 times) — the two-pointer approach is still O(A + B), not O(A * B), so this remains efficient.

## Common Mistakes
- Re-scanning the whole array inside `shortest` (this is the *first* version of this problem's brute-force approach; here it defeats the purpose of the "many repeated queries" setup).
- Comparing every index in `indices1` against every index in `indices2` (O(A*B)) instead of using the two-pointer merge (O(A+B)).
- Forgetting that `index_map` needs to preserve indices in sorted order — this happens for free if you build it via a single left-to-right scan, but would break if you built it in a different order (e.g., from a set).

## Related Problems
- [[Random-Pick-with-Weight]]
- [[Merge-Sorted-Array]]
- [[Accounts-Merge]]

## Follow-up / Variants
A natural extension: "Shortest Word Distance III" allows `word1 == word2`, requiring you to find the minimum distance between two *different* occurrences of the same word — solvable with a single pass tracking the previous index of that word, rather than the two-pointer merge shown here.
