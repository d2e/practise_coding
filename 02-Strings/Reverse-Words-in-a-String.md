---
title: "Reverse Words in a String"
aliases: []
tags: [leetcode, string-parsing, two-pointers, medium]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 151
leetcode_difficulty: Medium
pattern: "String Parsing / In-Place Reversal"
status: not-started
---

# Reverse Words in a String

## Problem Statement
Given an input string `s`, reverse the order of the **words**. A word is a maximal sequence of non-space characters. Return a string of the words in reverse order, joined by a single space. The returned string should not have leading or trailing spaces, and multiple spaces between words in the input should be reduced to a single space in the output.

Constraints: 1 ≤ s.length ≤ 10^4, s contains English letters, digits, spaces, and printable characters; s contains at least one non-space character.

## Examples
```
Input: s = "the sky is blue"
Output: "blue is sky the"

Input: s = "  hello world  "
Output: "world hello"        # leading/trailing spaces removed

Input: s = "a good   example"
Output: "example good a"     # multiple spaces collapsed to one
```

## Intuition & Approach
The "obvious" Python solution is `" ".join(reversed(s.split()))` — `s.split()` with no arguments already collapses runs of whitespace and drops leading/trailing spaces, so this one-liner is both correct and idiomatic. It's worth knowing *why* this works: `str.split()` (no separator argument) treats any run of whitespace as a single delimiter and ignores leading/trailing runs, which is exactly the normalization the problem demands.

The more interesting version of this problem — and the one interviewers actually probe — is the **O(1) extra space, in-place** variant on a mutable character array (as it's framed in C++/Java). The technique: reverse the entire character array first, then reverse each individual word within the already-reversed array, and finally clean up whitespace by writing valid words into a "read/write pointer" pass. This is the same three-reversal trick used to rotate arrays, and it's worth practicing by hand even if Python's `str` immutability makes the built-in one-liner the practical answer.

## Pattern Recognition
> [!tip] Pattern: String Parsing / In-Place Reversal
> "Reverse order of tokens while normalizing whitespace" almost always reduces to split → reverse → join in high-level languages. When the constraint is O(1) space on a mutable buffer, reach for the "reverse the whole thing, then reverse each piece back" trick — the same idea powers array rotation.

## Algorithm Steps
**Pythonic approach:**
1. Split `s` on whitespace (collapses multiple spaces, strips leading/trailing).
2. Reverse the resulting list of words.
3. Join with a single space.

**In-place (O(1) space) approach, for interview follow-up:**
1. Reverse the whole character array.
2. Walk through the reversed array, reversing each word's characters back to normal order in place (word boundaries are found by scanning for spaces).
3. Do a second pass with a slow/fast pointer to compact runs of spaces into single spaces and trim leading/trailing spaces.

## Solution (Python)
```python
def reverseWords(s: str) -> str:
    return " ".join(reversed(s.split()))
```

```python
# In-place style (simulated on a list, since Python strings are immutable),
# demonstrating the O(1)-extra-space technique interviewers look for.
def reverseWordsInPlace(s: str) -> str:
    chars = list(s)

    def reverse(lo: int, hi: int) -> None:
        while lo < hi:
            chars[lo], chars[hi] = chars[hi], chars[lo]
            lo += 1
            hi -= 1

    # 1. Reverse the entire buffer.
    reverse(0, len(chars) - 1)

    # 2. Reverse each word back to normal order.
    n = len(chars)
    start = 0
    for i in range(n + 1):
        if i == n or chars[i] == ' ':
            reverse(start, i - 1)
            start = i + 1

    # 3. Compact spaces: single pass with a write pointer.
    write = 0
    read = 0
    while read < n:
        if chars[read] != ' ':
            if write != 0:
                chars[write] = ' '
                write += 1
            while read < n and chars[read] != ' ':
                chars[write] = chars[read]
                write += 1
                read += 1
        else:
            read += 1

    return "".join(chars[:write])
```

## Solution (Java)
```java
class Solution {
    public String reverseWords(String s) {
        String[] words = s.trim().split("\\s+");
        StringBuilder sb = new StringBuilder();
        for (int i = words.length - 1; i >= 0; i--) {
            sb.append(words[i]);
            if (i > 0) {
                sb.append(' ');
            }
        }
        return sb.toString();
    }
}
```

```java
// In-place style on a mutable char[] — the O(1)-extra-space technique interviewers look for.
class Solution {
    public String reverseWordsInPlace(String s) {
        char[] chars = s.toCharArray();
        int n = chars.length;

        // 1. Reverse the entire buffer.
        reverse(chars, 0, n - 1);

        // 2. Reverse each word back to normal order.
        int start = 0;
        for (int i = 0; i <= n; i++) {
            if (i == n || chars[i] == ' ') {
                reverse(chars, start, i - 1);
                start = i + 1;
            }
        }

        // 3. Compact spaces: single pass with a write pointer.
        int write = 0, read = 0;
        while (read < n) {
            if (chars[read] != ' ') {
                if (write != 0) {
                    chars[write++] = ' ';
                }
                while (read < n && chars[read] != ' ') {
                    chars[write++] = chars[read++];
                }
            } else {
                read++;
            }
        }

        return new String(chars, 0, write);
    }

    private void reverse(char[] chars, int lo, int hi) {
        while (lo < hi) {
            char tmp = chars[lo];
            chars[lo] = chars[hi];
            chars[hi] = tmp;
            lo++;
            hi--;
        }
    }
}
```

## Complexity Analysis
- **Time:** O(n) — every character is visited a constant number of times across the split/reverse/join (or the three in-place passes).
- **Space:** O(n) for the Pythonic solution (the split produces a new list of words); O(1) extra space for the in-place variant beyond the input buffer itself (Python's string immutability forces `list(s)`, but the *algorithm* is O(1) auxiliary space).

## Edge Cases
- Leading and/or trailing spaces.
- Multiple consecutive spaces between words.
- A single word with no spaces at all.
- String composed of only spaces is disallowed by constraints (at least one non-space char), but defensive code should still not crash on it.

## Common Mistakes
- Using `s.split(" ")` instead of `s.split()` — the former does *not* collapse multiple spaces and produces empty-string tokens.
- Forgetting to strip leading/trailing spaces before or after reversing when implementing the in-place version by hand.
- Off-by-one errors in the word-boundary reversal pass (forgetting to reverse the final word when the loop ends at `i == n`).

## Related Problems
- [[Check-If-a-Word-Occurs-As-a-Prefix-of-Any-Word-in-a-Sentence]]
- [[Rotate-Image]]
- [[Ransom-Note]]

## Follow-up / Variants
Follow-up commonly asked: implement it truly in-place with O(1) extra memory in a language with mutable strings (C++ `vector<char>`, Java `char[]`). Another variant: reverse only the characters *within* each word while keeping word order intact (a different, simpler problem — LC 557).
