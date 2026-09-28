---
title: "Strings — Pattern Primer"
tags: [moc, pattern-primer, strings]
---

# Strings

String problems in interviews are rarely about a single named algorithm — they're about **careful, correct handling of a specific parsing or validation rule**, occasionally combined with a well-known technique (hashing for frequency counts, a finite state machine for validation, KMP for substring search).

## When to reach for this bucket
- The problem statement talks about words, sentences, characters, prefixes/suffixes, or "build/validate a string according to rule X."
- There's no obvious numeric optimization structure (that's more likely sliding window, DP, etc.) — the challenge is *correctness of parsing*, not asymptotic complexity.

## Recurring sub-techniques in this folder
- **Split/normalize + iterate**: `str.split()` (no-arg) collapses whitespace and drops leading/trailing spaces — the standard tool for "words in a sentence" problems. See [[Check-If-a-Word-Occurs-As-a-Prefix-of-Any-Word-in-a-Sentence]], [[Reverse-Words-in-a-String]].
- **Frequency counting**: `collections.Counter` for "can A be built from B's characters" problems. See [[Ransom-Note]].
- **Finite state machine**: for validating strings against a grammar with several ordered/optional components (sign, digits, decimal point, exponent). See [[Valid-Number]].
- **Substring search**: brute force vs. KMP's O(n+m) prefix-function trick for avoiding re-comparison after a partial match fails. See [[Find-the-Index-of-the-First-Occurrence-in-a-String]].
- **Recursive chunking / simulation**: breaking a large formatting problem into small, well-tested helper functions over natural groupings (e.g., 3-digit chunks for number-to-words). See [[Integer-to-English-Words]].

## Problems
- [[Check-If-a-Word-Occurs-As-a-Prefix-of-Any-Word-in-a-Sentence]] — Easy
- [[Reverse-Words-in-a-String]] — Medium
- [[Ransom-Note]] — Easy
- [[Valid-Number]] — Hard
- [[Find-the-Index-of-the-First-Occurrence-in-a-String]] — Easy
- [[Integer-to-English-Words]] — Hard

## Study tip
Do the two Easy problems first to build fluency with `split`/`Counter`, then Valid Number (FSM) and Integer to English Words (recursive chunking) as the "big" problems in this bucket — both are Hard-rated but for bookkeeping reasons, not algorithmic depth, so they reward writing a clean helper function up front.
