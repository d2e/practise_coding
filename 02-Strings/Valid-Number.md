---
title: "Valid Number"
aliases: []
tags: [leetcode, string-parsing, finite-state-machine, hard]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 65
leetcode_difficulty: Hard
pattern: "Finite State Machine / String Validation"
status: not-started
---

# Valid Number

## Problem Statement
Given a string `s`, return `True` if `s` is a valid number. A valid number can be:
- An **integer** or **decimal** number, optionally preceded by a sign (`+`/`-`).
- Optionally followed by an exponent part: `'e'` or `'E'`, an optional sign, and an integer.

A decimal number is either digits followed by an optional `.` and optional trailing digits (`"123."`, `"123.456"`), or a `.` followed by digits (`".456"`) — but a bare `"."` alone or a bare sign alone is invalid. There must be at least one digit in the mantissa somewhere.

Valid examples: `"2"`, `"0089"`, `"-0.1"`, `"+3.14"`, `"4."`, `"-.9"`, `"2e10"`, `"-90E3"`, `"3e+7"`, `"+6e-1"`, `"53.5e93"`, `"-123.456e789"`.
Invalid examples: `"abc"`, `"1a"`, `"1e"`, `"e3"`, `"99e2.5"`, `"--6"`, `"-+3"`, `"95a54e53"`, `"."`, `"."`, `" "` .

## Examples
```
Input: s = "0"
Output: true

Input: s = "e"
Output: false

Input: s = "."
Output: false

Input: s = "-90E3"
Output: true

Input: s = "1e"
Output: false      # exponent marker with no digits after it
```

## Intuition & Approach
This problem is famous for having a deceptively long list of edge cases if you try to handle it with ad-hoc string slicing and boolean flags scattered everywhere — that approach works but is extremely bug-prone (this is exactly why it's rated Hard despite no real algorithmic complexity). The clean way to think about it is as a **finite state machine (FSM)**: define a small set of named states representing "where we are in parsing a valid number" (start, sign, integer digits, decimal point, digits after decimal, exponent marker, exponent sign, exponent digits), and define legal transitions between states based on the next character's category (digit / sign / decimal point / exponent marker / other). If you ever hit a character with no legal transition, or you finish the string in a non-accepting state, the string is invalid.

The FSM approach converts "did I handle every edge case" into "did I define every state and transition correctly" — a much smaller, more mechanical thing to get right, and the standard technique for interview string-validation problems of this shape.

A lighter-weight alternative (still robust) is to manually track a handful of booleans while scanning: `seen_digit`, `seen_dot`, `seen_exponent`, resetting `seen_digit` to `False` when an exponent is entered (since the exponent needs its own digit). This is shown below as the primary solution since it needs no auxiliary state-transition table, but the FSM table version is included as the "gold standard" approach.

## Pattern Recognition
> [!tip] Pattern: Finite State Machine
> Whenever a problem requires validating that a string matches a specific grammar with several optional/ordered components (sign, digits, decimal point, exponent), and naive if/else branching is getting unmanageable, model it explicitly as a state machine. This generalizes to validating IP addresses, parsing dates, tokenizing expressions, etc.

## Algorithm Steps
**Boolean-flag scan (primary solution):**
1. Track three flags: `seen_digit`, `seen_dot`, `seen_exponent`, all `False`.
2. Walk the string character by character.
3. If it's a digit → set `seen_digit = True`.
4. If it's `+`/`-` → only legal as the very first character, or immediately after `e`/`E`; otherwise invalid.
5. If it's `.` → invalid if we've already seen a dot, or if we've already seen an exponent (no decimal point after `e`); mark `seen_dot = True`.
6. If it's `e`/`E` → invalid if we've already seen an exponent, or if no digit has been seen yet (mantissa must have at least one digit before the exponent); mark `seen_exponent = True` and **reset `seen_digit = False`** so the exponent is forced to have its own digit(s).
7. Any other character → invalid immediately.
8. After the scan, the string is valid only if `seen_digit` is `True` (at least one digit was seen overall, and specifically after the last `e` if one existed).

## Solution (Python)
```python
def isNumber(s: str) -> bool:
    seen_digit = seen_dot = seen_exponent = False

    for i, ch in enumerate(s):
        if ch.isdigit():
            seen_digit = True
        elif ch in '+-':
            # Sign only valid at the very start, or right after e/E.
            if i > 0 and s[i - 1] not in 'eE':
                return False
        elif ch == '.':
            # No repeated dots, and no dot after an exponent has started.
            if seen_dot or seen_exponent:
                return False
            seen_dot = True
        elif ch in 'eE':
            # No repeated exponents, and mantissa must have a digit already.
            if seen_exponent or not seen_digit:
                return False
            seen_exponent = True
            seen_digit = False  # exponent must supply its own digit(s)
        else:
            return False

    return seen_digit
```

## Solution (Java)
```java
class Solution {
    public boolean isNumber(String s) {
        boolean seenDigit = false, seenDot = false, seenExponent = false;

        for (int i = 0; i < s.length(); i++) {
            char ch = s.charAt(i);
            if (Character.isDigit(ch)) {
                seenDigit = true;
            } else if (ch == '+' || ch == '-') {
                // Sign only valid at the very start, or right after e/E.
                if (i > 0 && s.charAt(i - 1) != 'e' && s.charAt(i - 1) != 'E') {
                    return false;
                }
            } else if (ch == '.') {
                // No repeated dots, and no dot after an exponent has started.
                if (seenDot || seenExponent) {
                    return false;
                }
                seenDot = true;
            } else if (ch == 'e' || ch == 'E') {
                // No repeated exponents, and mantissa must have a digit already.
                if (seenExponent || !seenDigit) {
                    return false;
                }
                seenExponent = true;
                seenDigit = false; // exponent must supply its own digit(s)
            } else {
                return false;
            }
        }

        return seenDigit;
    }
}
```

## Complexity Analysis
- **Time:** O(n) — single left-to-right scan of the string, constant work per character.
- **Space:** O(1) — a fixed number of boolean flags regardless of input length.

## Edge Cases
- Bare sign with nothing else (`"+"`, `"-"`) → invalid, caught because `seen_digit` stays `False`.
- Bare dot (`"."`) → invalid, `seen_digit` never set.
- Trailing dot after digits (`"4."`) → valid, since a dot with no digits after it is fine as long as digits appeared before it.
- Leading dot before digits (`"-.9"`) → valid.
- Exponent with no mantissa digit before it (`"e3"`, `".e3"`) → invalid, caught by `not seen_digit` check at the `e`/`E` branch.
- Exponent with no digit after it (`"1e"`) → invalid, since `seen_digit` is reset to `False` on entering the exponent and never set again.
- Two signs in a row (`"--6"`, `"-+3"`) → invalid, since the second sign isn't at index 0 and isn't right after `e`/`E`.
- Whitespace anywhere (LeetCode's actual constraints disallow it entirely) → falls into the `else: return False` branch.

## Common Mistakes
- Forgetting to reset `seen_digit = False` when entering the exponent, which would incorrectly accept `"1e"` as valid (since a digit was seen earlier in the mantissa).
- Allowing a `.` after an exponent has started (`"1e2.5"` must be invalid — only integers are allowed in the exponent).
- Checking sign legality with `i == 0 or s[i-1] in 'eE'` but forgetting parentheses precedence bugs when translating to other languages (Python's `and`/`or` are unambiguous here, but it's a common transcription error).
- Trying to use a regex — it works (`^[+-]?(\d+\.?\d*|\.\d+)([eE][+-]?\d+)?$`) but is easy to get subtly wrong and much harder to reason about under interview pressure than the explicit scan; know the regex exists but prefer explaining the FSM/flag approach out loud.

## Related Problems
- [[Integer-to-English-Words]]
- [[Reverse-Words-in-a-String]]
- [[Find-the-Index-of-the-First-Occurrence-in-a-String]]

## Follow-up / Variants
A common follow-up is to formalize the solution as an explicit state-transition table (dict of dicts mapping `state -> char_category -> next_state`) rather than ad-hoc flags — useful if the interviewer wants to see you generalize the technique. Another variant: validate a string as valid JSON or a valid IPv4/IPv6 address, which use the same "scan + track legal transitions" philosophy.
