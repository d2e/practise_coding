---
title: "Check If a Word Occurs As a Prefix of Any Word in a Sentence"
aliases: []
tags: [leetcode, string-parsing, easy]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 1455
leetcode_difficulty: Easy
pattern: "String Parsing / Prefix Check"
status: not-started
---

# Check If a Word Occurs As a Prefix of Any Word in a Sentence

## Problem Statement
Given a `sentence` (a string of lowercase letters and single spaces, with no leading/trailing spaces and no consecutive spaces) and a `searchWord`, find the index of the first word in `sentence` that has `searchWord` as a prefix. Word indices are **1-indexed**. Return the 1-indexed position of the first matching word, or `-1` if no word in the sentence starts with `searchWord`.

Constraints: 1 ≤ sentence length ≤ 100, 1 ≤ searchWord length ≤ 30, both consist of lowercase English letters and, for the sentence, single spaces between words.

## Examples
```
Input: sentence = "i love eating burger", searchWord = "burg"
Output: 4          # "burger" is the 4th word and starts with "burg"

Input: sentence = "this problem is an easy problem", searchWord = "pro"
Output: 2          # first match wins even though word 6 also matches

Input: sentence = "i am tired", searchWord = "you"
Output: -1
```

## Intuition & Approach
This is a direct simulation problem — there's no hidden trick, the challenge is just executing the string handling cleanly. Split the sentence into words (using `str.split()`, which handles the "single space" separation for you), then walk the words in order checking `word.startswith(searchWord)`. Return as soon as you find a match, using the 1-indexed position. If you exhaust the list without a match, return -1.

The only subtlety is remembering the index is 1-based, not 0-based, and returning **on the first match**, not just any match — since the problem says "first word," iterating in order and returning early is required (rather than, say, collecting all matches and picking the min index, which would be unnecessary extra work).

## Pattern Recognition
> [!tip] Pattern: String Parsing / Prefix Check
> Whenever a problem gives you a sentence of words and asks something per-word ("first word that...", "does any word..."), the move is: split on whitespace, then use Python's built-in `str.startswith`/`str.endswith` rather than manually slicing and comparing characters.

## Algorithm Steps
1. Split `sentence` into a list of words on whitespace.
2. Iterate over the list with `enumerate`, starting the index at 1.
3. For each word, check `word.startswith(searchWord)`.
4. Return the 1-indexed position on the first match.
5. If the loop completes with no match, return -1.

## Solution (Python)
```python
def isPrefixOfWord(sentence: str, searchWord: str) -> int:
    for i, word in enumerate(sentence.split(), start=1):
        if word.startswith(searchWord):
            return i
    return -1
```

## Solution (Java)
```java
class Solution {
    public int isPrefixOfWord(String sentence, String searchWord) {
        String[] words = sentence.split(" ");
        for (int i = 0; i < words.length; i++) {
            if (words[i].startsWith(searchWord)) {
                return i + 1; // 1-indexed
            }
        }
        return -1;
    }
}
```

## Complexity Analysis
- **Time:** O(n) where n is the total length of `sentence` — splitting is O(n), and `startswith` on each word costs at most O(len(searchWord)), so the whole scan is bounded by O(n).
- **Space:** O(n) for the list produced by `sentence.split()`.

## Edge Cases
- `searchWord` longer than every word in the sentence (no match possible, `startswith` correctly returns `False` without erroring).
- `searchWord` equal to an entire word (still counts as a prefix — `"burger".startswith("burger")` is `True`).
- Sentence with a single word.
- No word matches at all → must return -1, not 0 or an exception.

## Common Mistakes
- Using 0-indexed `enumerate` and forgetting to add 1 before returning.
- Returning the *last* matching index instead of the first (e.g., by not breaking/returning immediately inside the loop).
- Manually slicing `word[:len(searchWord)] == searchWord` instead of `startswith` — works, but is more error-prone if `searchWord` is longer than `word` in some languages/edge conditions (Python handles it fine, but it's needless reinvention).

## Related Problems
- [[Reverse-Words-in-a-String]]
- [[Ransom-Note]]
- [[Find-the-Index-of-the-First-Occurrence-in-a-String]]

## Follow-up / Variants
A natural follow-up: what if you need to find *all* matching word indices, not just the first? Same loop, just append to a results list instead of returning early. Another variant asks for suffix matching instead of prefix — swap `startswith` for `endswith`.
