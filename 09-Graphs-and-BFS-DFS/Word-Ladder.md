---
title: "Word Ladder"
aliases: []
tags: [leetcode, bfs, hard]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 127
leetcode_difficulty: Hard
pattern: "Unweighted Shortest Path (BFS)"
status: not-started
---

# Word Ladder

## Problem Statement
Given two words, `beginWord` and `endWord`, and a dictionary `wordList`, return the number of words in the *shortest* transformation sequence from `beginWord` to `endWord`, such that:
- Only one letter can be changed at a time.
- Every intermediate word in the sequence must exist in `wordList` (`beginWord` does not need to be in `wordList`).

If no such transformation sequence exists, return `0`. All words are the same length and consist of lowercase letters.

## Examples
```
Input: beginWord = "hit", endWord = "cog", wordList = ["hot","dot","dog","lot","log","cog"]
Output: 5
Explanation: hit -> hot -> dot -> dog -> cog  (5 words in the sequence)
```

```
Input: beginWord = "hit", endWord = "cog", wordList = ["hot","dot","dog","lot","log"]
Output: 0
Explanation: endWord "cog" is not in wordList, so no valid sequence exists.
```

## Intuition & Approach
Think of every word in `wordList` (plus `beginWord`) as a node in a graph, with an edge between two words if they differ by exactly one letter. "Shortest transformation sequence" is then just "shortest path between two nodes" in an *unweighted* graph — the textbook signal for BFS, since BFS explores nodes in strictly increasing order of distance from the source, so the first time you reach `endWord` you're guaranteed it's via the minimum number of steps.

The naive way to find neighbors of a word is to compare it against every other word in the dictionary (`O(L · N)` per word, where `L` is word length and `N` is dictionary size). A much faster way exploits the fixed alphabet: for a word of length `L`, there are only `26 * L` possible one-letter-substitution neighbors to *generate* directly, rather than compare against. Generate each of those candidate words and check membership in a hash set of the dictionary in `O(1)`. This is the standard optimization that turns Word Ladder from "compare against everyone" into "generate and look up."

## Pattern Recognition
> [!tip] Pattern: BFS for Unweighted Shortest Path
> "Minimum number of transformations/moves/steps to go from A to B," with each move being an equally-weighted single edit — that's unweighted shortest path, solved with plain BFS layer-by-layer, not Dijkstra (which is reserved for weighted edges) and not DFS (which finds *a* path, not the shortest one).

## Algorithm Steps
1. Put `wordList` into a hash set for O(1) membership checks; if `endWord` isn't in it, return `0` immediately.
2. BFS from `beginWord`, tracking `(word, steps)`; use a visited set to avoid revisiting words (mark a word visited the moment it's enqueued, not when dequeued, to avoid duplicate enqueues).
3. For the current word, generate all `26 * L` single-letter-substitution variants; for each variant that is in the dictionary set and not yet visited, mark it visited and enqueue it with `steps + 1`.
4. The moment you generate/reach `endWord`, return `steps + 1` (the transformation count including both endpoints).
5. If the queue empties without reaching `endWord`, return `0`.

## Solution (Python)
```python
from collections import deque
from typing import List

def ladderLength(beginWord: str, endWord: str, wordList: List[str]) -> int:
    word_set = set(wordList)
    if endWord not in word_set:
        return 0

    queue = deque([(beginWord, 1)])
    visited = {beginWord}
    alphabet = "abcdefghijklmnopqrstuvwxyz"

    while queue:
        word, steps = queue.popleft()
        if word == endWord:
            return steps

        for i in range(len(word)):
            for ch in alphabet:
                if ch == word[i]:
                    continue
                candidate = word[:i] + ch + word[i + 1:]
                if candidate in word_set and candidate not in visited:
                    visited.add(candidate)
                    queue.append((candidate, steps + 1))

    return 0
```

## Solution (Java)
```java
import java.util.*;

class Solution {
    public int ladderLength(String beginWord, String endWord, List<String> wordList) {
        Set<String> wordSet = new HashSet<>(wordList);
        if (!wordSet.contains(endWord)) return 0;

        Deque<String> queue = new ArrayDeque<>();
        queue.offer(beginWord);
        Set<String> visited = new HashSet<>();
        visited.add(beginWord);
        int steps = 1;

        while (!queue.isEmpty()) {
            int levelSize = queue.size();
            for (int k = 0; k < levelSize; k++) {
                String word = queue.poll();
                if (word.equals(endWord)) return steps;

                char[] chars = word.toCharArray();
                for (int i = 0; i < chars.length; i++) {
                    char original = chars[i];
                    for (char ch = 'a'; ch <= 'z'; ch++) {
                        if (ch == original) continue;
                        chars[i] = ch;
                        String candidate = new String(chars);
                        if (wordSet.contains(candidate) && !visited.contains(candidate)) {
                            visited.add(candidate);
                            queue.offer(candidate);
                        }
                    }
                    chars[i] = original;
                }
            }
            steps++;
        }

        return 0;
    }
}
```

## Complexity Analysis
- **Time:** O(N · L²) — for each of up to N words we generate `L * 26` candidates, and constructing/hashing each candidate string costs O(L).
- **Space:** O(N · L) — the word set, visited set, and BFS queue each hold up to N words of length L.

## Edge Cases
- `endWord` not present in `wordList` at all — return 0 immediately without doing any BFS.
- `beginWord == endWord` — depending on interpretation this is typically outside problem constraints, but guard logic naturally returns 1 if reached trivially.
- `wordList` is empty — `endWord not in word_set` catches this and returns 0.
- Words of length 1 — the inner double loop still works correctly (single position, 26 candidates).
- No path exists even though `endWord` is in the list (isolated by letter changes) — BFS queue drains and returns 0.

## Common Mistakes
- Marking a word as visited only when *dequeued* rather than when *enqueued* — this allows the same word to be pushed onto the queue multiple times from different neighbors, wasting time and in some implementations causing incorrect step counts.
- Comparing the current word against every dictionary word to find neighbors instead of generating candidates directly — correct but far slower, and often times out on large dictionaries.
- Forgetting to remove/not re-add `beginWord` to the dictionary set, which is harmless here since we track visited separately, but can cause subtle bugs in variants that mutate the word set directly (e.g., removing found words from the set as you go).
- Off-by-one on the step count — the answer counts *words* in the sequence, not the number of transformations, so the initial queue entry must start at `steps = 1`, not `0`.

## Related Problems
- [[Alien-Dictionary]]
- [[Walls-and-Gates]]
- [[Number-of-Islands]]

## Follow-up / Variants
The direct follow-up is "Word Ladder II" (LC 126), which asks you to return *all* shortest transformation sequences rather than just the count — this requires BFS to build a layered predecessor graph first, then a DFS/backtracking pass to reconstruct every shortest path. Another classic optimization is **bidirectional BFS**, alternating expansion from both `beginWord` and `endWord` and stopping when the two frontiers meet, which significantly reduces the search space in practice.
