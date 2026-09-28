---
title: "Ransom Note"
aliases: []
tags: [leetcode, hashing, counting, easy]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 383
leetcode_difficulty: Easy
pattern: "Frequency Counting / Hash Map"
status: not-started
---

# Ransom Note

## Problem Statement
Given two strings `ransomNote` and `magazine`, return `True` if `ransomNote` can be constructed by using the letters from `magazine`, where each letter in `magazine` can be used **at most once**. Otherwise return `False`.

Constraints: both strings consist of lowercase English letters, length up to 10^5.

## Examples
```
Input: ransomNote = "a", magazine = "b"
Output: false

Input: ransomNote = "aa", magazine = "ab"
Output: false      # only one 'a' available in magazine

Input: ransomNote = "aa", magazine = "aab"
Output: true
```

## Intuition & Approach
This is the canonical "can I build string A out of the letters of string B" problem, and it always reduces to comparing character frequency counts. Count how many of each letter appear in `magazine`, then subtract the counts needed for `ransomNote`. If at any point you need more of a letter than `magazine` has (or the letter never appears in `magazine` at all), the answer is `False`.

Python's `collections.Counter` makes this a near one-liner: `Counter` supports subtraction-like comparison, and specifically `Counter(ransomNote) - Counter(magazine)` produces a Counter containing only the *positive* leftover counts — i.e., the letters `ransomNote` needs that `magazine` didn't supply enough of. If that leftover Counter is empty, construction is possible.

## Pattern Recognition
> [!tip] Pattern: Frequency Counting
> Any problem phrased as "can string/set A be built from the characters/elements of string/set B" (anagram checks, ransom notes, "can construct from letters") is a frequency-count comparison — reach for `Counter` or a fixed-size array of 26 counts immediately, don't reach for sorting or nested loops.

## Algorithm Steps
1. Build a frequency count of letters in `magazine`.
2. Build a frequency count of letters in `ransomNote`.
3. For every letter required by `ransomNote`, check that `magazine`'s count for that letter is at least as large.
4. If any letter fails that check, return `False`; otherwise return `True`.

## Solution (Python)
```python
from collections import Counter

def canConstruct(ransomNote: str, magazine: str) -> bool:
    note_counts = Counter(ransomNote)
    magazine_counts = Counter(magazine)
    for letter, needed in note_counts.items():
        if magazine_counts[letter] < needed:
            return False
    return True
```

```python
# Equivalent one-liner using Counter subtraction (drops non-positive results).
from collections import Counter

def canConstruct_oneliner(ransomNote: str, magazine: str) -> bool:
    return not (Counter(ransomNote) - Counter(magazine))
```

## Solution (Java)
```java
class Solution {
    public boolean canConstruct(String ransomNote, String magazine) {
        int[] noteCounts = new int[26];
        int[] magazineCounts = new int[26];
        for (char c : ransomNote.toCharArray()) noteCounts[c - 'a']++;
        for (char c : magazine.toCharArray()) magazineCounts[c - 'a']++;

        for (int i = 0; i < 26; i++) {
            if (noteCounts[i] > magazineCounts[i]) {
                return false;
            }
        }
        return true;
    }
}
```

```java
// Equivalent single-array "subtraction" style: decrement as we consume, bail on the first shortfall.
class Solution {
    public boolean canConstruct(String ransomNote, String magazine) {
        int[] counts = new int[26];
        for (char c : magazine.toCharArray()) {
            counts[c - 'a']++;
        }
        for (char c : ransomNote.toCharArray()) {
            if (--counts[c - 'a'] < 0) {
                return false;
            }
        }
        return true;
    }
}
```

## Complexity Analysis
- **Time:** O(n + m) where n = len(ransomNote), m = len(magazine) — one pass to build each Counter, one pass over the note's distinct letters (at most 26) to compare.
- **Space:** O(1) effectively, since there are only 26 lowercase letters possible — the Counters are bounded by a constant size regardless of input length.

## Edge Cases
- `ransomNote` longer than `magazine` (immediately impossible, but the counting approach handles it correctly without a special case).
- `ransomNote` is empty (`""` can always be constructed — the loop simply does nothing and returns `True`).
- `magazine` is empty but `ransomNote` is not (returns `False`).
- Repeated letters requiring more copies than available.

## Common Mistakes
- Using a set intersection check instead of counts — sets ignore how many times a letter is needed vs. available, giving wrong answers when quantities matter (e.g., `ransomNote="aa"`, `magazine="a"` would incorrectly pass a set-based check).
- Iterating over `magazine_counts.items()` and checking `note_counts[letter] > needed` — this is backwards and doesn't correctly bound by what's needed vs. available.
- Forgetting that `Counter` subtraction (`-`) drops zero/negative entries, so `bool(Counter(...) - Counter(...))` being falsy is the correct "nothing left over" check — don't compare with `== Counter()` and worry about extra letters in magazine (those are fine and irrelevant).

## Related Problems
- [[Check-If-a-Word-Occurs-As-a-Prefix-of-Any-Word-in-a-Sentence]]
- [[Accounts-Merge]]
- [[Minimum-Absolute-Difference]]

## Follow-up / Variants
Common follow-up: what if letters could be reused unlimited times from magazine (not "at most once")? Then it reduces to a simple set-membership check per required letter. Another variant: support Unicode/multi-byte characters — `Counter` still works unchanged since it's character-based, not limited to ASCII.
