---
title: "Find the Index of the First Occurrence in a String"
aliases: ["strStr"]
tags: [leetcode, string-matching, kmp, easy]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 28
leetcode_difficulty: Easy
pattern: "String Matching (Substring Search)"
status: not-started
---

# Find the Index of the First Occurrence in a String

## Problem Statement
Given two strings `haystack` and `needle`, return the index of the first occurrence of `needle` in `haystack`, or `-1` if `needle` is not part of `haystack`. This is the classic `strStr()` problem.

Constraints: 1 ≤ haystack.length, needle.length ≤ 10^4 (in the harder framing used for KMP practice; LeetCode's own constraints are smaller but the algorithmic lesson is the same).

## Examples
```
Input: haystack = "sadbutsad", needle = "sad"
Output: 0

Input: haystack = "leetcode", needle = "leeto"
Output: -1

Input: haystack = "hello", needle = "ll"
Output: 2
```

## Intuition & Approach
The brute-force approach is: for every starting position `i` in `haystack`, check whether `haystack[i:i+len(needle)] == needle`. This is O(n·m) in the worst case (e.g., `haystack = "aaaa...a"`, `needle = "aaa...ab"`), because after a partial match fails you throw away all the progress you made and restart from `i+1`.

The insight behind **Knuth-Morris-Pratt (KMP)** is that when a match fails partway through, you already know a lot about the characters you just compared (they matched `needle`'s prefix up to that point) — so instead of restarting the needle pointer from 0, you can jump it forward to the longest proper prefix of `needle` that is also a suffix of what you've matched so far. This "failure function" (a.k.a. the prefix function, `lps` — longest proper prefix which is also suffix) is precomputed once on `needle` in O(m), and then the main scan over `haystack` never backtracks, giving O(n + m) total.

For interviews, know both: the brute-force version is what most people reach for and is often accepted, but understanding *why* it's O(n·m) and how KMP fixes it is the actual signal an interviewer is probing for when they ask this question.

## Pattern Recognition
> [!tip] Pattern: String Matching
> "Find occurrence(s) of a pattern inside a larger text" is substring search. Default to the brute-force double loop for simplicity unless asked to optimize, but always be ready to explain KMP (or mention Python's built-in `str.find`/`in`, which use an optimized algorithm internally) as the follow-up.

## Algorithm Steps
**Brute force:**
1. For each start index `i` from `0` to `len(haystack) - len(needle)`:
2. Compare `haystack[i:i+len(needle)]` against `needle`.
3. Return `i` on the first match; return -1 if no `i` works.

**KMP (follow-up):**
1. Build the `lps` (longest-prefix-suffix) array for `needle`: `lps[i]` = length of the longest proper prefix of `needle[:i+1]` that is also a suffix of it.
2. Scan `haystack` with pointer `i`, and `needle` with pointer `j`.
3. On a character match, advance both `i` and `j`.
4. On a mismatch with `j > 0`, don't move `i` — instead set `j = lps[j-1]` (skip ahead using the precomputed table) and retry.
5. On a mismatch with `j == 0`, just advance `i`.
6. If `j` ever reaches `len(needle)`, a match starting at `i - j` has been found.

## Solution (Python)
```python
def strStr(haystack: str, needle: str) -> int:
    n, m = len(haystack), len(needle)
    for i in range(n - m + 1):
        if haystack[i:i + m] == needle:
            return i
    return -1
```

## Solution (Java)
```java
class Solution {
    public int strStr(String haystack, String needle) {
        int n = haystack.length(), m = needle.length();
        for (int i = 0; i <= n - m; i++) {
            if (haystack.substring(i, i + m).equals(needle)) {
                return i;
            }
        }
        return -1;
    }
}
```

### Follow-up: KMP in O(n+m)
The key building block is the `lps` prefix-function table, computed once on `needle`, which lets the main scan skip redundant comparisons on a mismatch instead of restarting from scratch.
```python
def strStr_kmp(haystack: str, needle: str) -> int:
    if not needle:
        return 0

    m = len(needle)
    lps = [0] * m
    length = 0  # length of the previous longest prefix-suffix
    i = 1
    while i < m:
        if needle[i] == needle[length]:
            length += 1
            lps[i] = length
            i += 1
        elif length > 0:
            length = lps[length - 1]  # fall back, don't advance i
        else:
            lps[i] = 0
            i += 1

    n = len(haystack)
    i = j = 0
    while i < n:
        if haystack[i] == needle[j]:
            i += 1
            j += 1
            if j == m:
                return i - j
        elif j > 0:
            j = lps[j - 1]
        else:
            i += 1
    return -1
```

### Follow-up: KMP in O(n+m) (Java)
```java
class Solution {
    public int strStr(String haystack, String needle) {
        if (needle.isEmpty()) {
            return 0;
        }

        int m = needle.length();
        int[] lps = new int[m];
        int length = 0; // length of the previous longest prefix-suffix
        int i = 1;
        while (i < m) {
            if (needle.charAt(i) == needle.charAt(length)) {
                length++;
                lps[i] = length;
                i++;
            } else if (length > 0) {
                length = lps[length - 1]; // fall back, don't advance i
            } else {
                lps[i] = 0;
                i++;
            }
        }

        int n = haystack.length();
        int hi = 0, ni = 0;
        while (hi < n) {
            if (haystack.charAt(hi) == needle.charAt(ni)) {
                hi++;
                ni++;
                if (ni == m) {
                    return hi - ni;
                }
            } else if (ni > 0) {
                ni = lps[ni - 1];
            } else {
                hi++;
            }
        }
        return -1;
    }
}
```

## Complexity Analysis
- **Brute force — Time:** O(n·m) worst case, O(n) best case. **Space:** O(1) (Python's slice comparison technically allocates a temporary substring, so realistically O(m) per comparison, but conceptually the algorithm is O(1) extra state).
- **KMP — Time:** O(n + m) — the `lps` table build is O(m), and the main scan is O(n) because `i` never decreases and `j` decreases only in amounts bounded by prior increases. **Space:** O(m) for the `lps` table.

## Edge Cases
- `needle` is an empty string → by convention (and LeetCode's own rule) should return 0.
- `needle` longer than `haystack` → immediately -1 (the brute-force range `range(n - m + 1)` naturally produces an empty range when `m > n`).
- `needle` occurs at the very last possible position.
- `haystack == needle` exactly → match at index 0.
- Repetitive patterns like `haystack = "aaaaaa"`, `needle = "aaa"` — worth mentally tracing through KMP's `lps` table (`[0,1,2]`) to confirm it skips correctly.

## Common Mistakes
- Off-by-one in the brute-force loop bound — must be `range(n - m + 1)`, not `range(n - m)`, or the last valid starting position gets skipped.
- In KMP, updating `i` inside the `elif length > 0: length = lps[length-1]` branch of the `lps`-building loop (it must NOT advance `i` there — only the fallback length changes).
- Forgetting the special case for `needle == ""`.
- Using slicing (`haystack[i:i+m] == needle`) in a tight loop over very large strings without realizing each slice is itself an O(m) copy — fine for brute force at this problem's scale, but worth naming as the reason KMP avoids repeated re-comparison work.

## Related Problems
- [[Reverse-Words-in-a-String]]
- [[Check-If-a-Word-Occurs-As-a-Prefix-of-Any-Word-in-a-Sentence]]
- [[Word-Ladder]]

## Follow-up / Variants
Follow-up: find *all* occurrences of `needle` in `haystack` (not just the first) — same KMP scan, but instead of returning immediately at `j == m`, record the match and fall back `j = lps[j-1]` to keep scanning for overlapping occurrences. Another well-known related algorithm for the same problem class is Rabin-Karp (rolling hash), useful when you need to search for multiple patterns simultaneously.
