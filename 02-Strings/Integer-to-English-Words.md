---
title: "Integer to English Words"
aliases: []
tags: [leetcode, string-parsing, recursion, hard]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 273
leetcode_difficulty: Hard
pattern: "Recursive Chunking / Simulation"
status: not-started
---

# Integer to English Words

## Problem Statement
Convert a non-negative integer `num` to its English words representation.

Constraints: 0 ≤ num ≤ 2^31 - 1.

## Examples
```
Input: num = 123
Output: "One Hundred Twenty Three"

Input: num = 12345
Output: "Twelve Thousand Three Hundred Forty Five"

Input: num = 1234567
Output: "One Million Two Hundred Thirty Four Thousand Five Hundred Sixty Seven"

Input: num = 0
Output: "Zero"
```

## Intuition & Approach
This problem has no clever algorithmic trick — it's a **simulation** problem where the difficulty is entirely in careful bookkeeping. The key structural insight: English numbers are naturally grouped in chunks of three digits (ones/tens/hundreds), and each chunk gets a scale word appended (thousand, million, billion). So the strategy is:

1. Split `num` into groups of 3 digits from the right (e.g., 1,234,567 → [1, 234, 567]).
2. Write a helper that converts any number from 0-999 into words (this is the fiddly part — handling teens, tens, and hundreds correctly).
3. Combine each 3-digit group with its scale word ("Thousand", "Million", "Billion"), skipping groups that are zero, and join everything with spaces.

The 0-999 helper itself needs three small lookup tables: words for 1-9 (ones), words for 10-19 (the irregular "teens" — eleven, twelve, ..., nineteen don't follow the tens+ones pattern), and words for the tens (20, 30, ..., 90). Given those three tables, converting any 3-digit chunk is: hundreds digit → "X Hundred", then handle the remaining 2-digit remainder as either a teen (11-19), or tens digit + ones digit.

## Pattern Recognition
> [!tip] Pattern: Recursive Chunking / Simulation
> When a problem asks you to "spell out" or "format" a number (English words, Roman numerals, currency), look for a natural grouping/place-value structure to recurse or iterate over, and isolate the messy formatting logic for a *single small unit* into its own helper function so the top-level logic stays clean.

## Algorithm Steps
1. Handle `num == 0` as a special case → return `"Zero"`.
2. Define lookup tables: `ones` (1-9 word list), `teens` (10-19 word list), `tens` (20,30,...,90 word list).
3. Define `three_digit_to_words(n)` for 0 ≤ n < 1000: handle the hundreds digit, then the last two digits (teens vs. tens+ones).
4. Define scale words in descending order: `[(10**9, "Billion"), (10**6, "Million"), (10**3, "Thousand"), (1, "")]`.
5. For each scale, extract `num // scale`, convert that chunk with the helper, append the scale word if the chunk is non-zero, and reduce `num %= scale`.
6. Join all non-empty pieces with single spaces.

## Solution (Python)
```python
def numberToWords(num: int) -> str:
    if num == 0:
        return "Zero"

    ones = ["", "One", "Two", "Three", "Four", "Five", "Six", "Seven", "Eight", "Nine"]
    teens = ["Ten", "Eleven", "Twelve", "Thirteen", "Fourteen", "Fifteen",
             "Sixteen", "Seventeen", "Eighteen", "Nineteen"]
    tens = ["", "", "Twenty", "Thirty", "Forty", "Fifty", "Sixty", "Seventy", "Eighty", "Ninety"]

    def three_digit_to_words(n: int) -> str:
        parts = []
        if n >= 100:
            parts.append(ones[n // 100] + " Hundred")
            n %= 100
        if n >= 20:
            parts.append(tens[n // 10])
            if n % 10:
                parts.append(ones[n % 10])
        elif n >= 10:
            parts.append(teens[n - 10])
        elif n > 0:
            parts.append(ones[n])
        return " ".join(parts)

    scales = [(10**9, "Billion"), (10**6, "Million"), (10**3, "Thousand"), (1, "")]

    result_parts = []
    for scale, scale_word in scales:
        if num >= scale:
            chunk = num // scale
            num %= scale
            chunk_words = three_digit_to_words(chunk)
            result_parts.append(chunk_words + (f" {scale_word}" if scale_word else ""))

    return " ".join(result_parts)
```

## Solution (Java)
```java
class Solution {
    private static final String[] ONES = {
        "", "One", "Two", "Three", "Four", "Five", "Six", "Seven", "Eight", "Nine"
    };
    private static final String[] TEENS = {
        "Ten", "Eleven", "Twelve", "Thirteen", "Fourteen", "Fifteen",
        "Sixteen", "Seventeen", "Eighteen", "Nineteen"
    };
    private static final String[] TENS = {
        "", "", "Twenty", "Thirty", "Forty", "Fifty", "Sixty", "Seventy", "Eighty", "Ninety"
    };

    public String numberToWords(int num) {
        if (num == 0) {
            return "Zero";
        }

        int[] scales = {1_000_000_000, 1_000_000, 1_000, 1};
        String[] scaleWords = {"Billion", "Million", "Thousand", ""};

        StringBuilder result = new StringBuilder();
        for (int s = 0; s < scales.length; s++) {
            int scale = scales[s];
            if (num >= scale) {
                int chunk = num / scale;
                num %= scale;
                if (result.length() > 0) {
                    result.append(' ');
                }
                result.append(threeDigitToWords(chunk));
                if (!scaleWords[s].isEmpty()) {
                    result.append(' ').append(scaleWords[s]);
                }
            }
        }

        return result.toString();
    }

    private String threeDigitToWords(int n) {
        StringBuilder sb = new StringBuilder();
        if (n >= 100) {
            sb.append(ONES[n / 100]).append(" Hundred");
            n %= 100;
        }
        if (n >= 20) {
            if (sb.length() > 0) sb.append(' ');
            sb.append(TENS[n / 10]);
            if (n % 10 != 0) {
                sb.append(' ').append(ONES[n % 10]);
            }
        } else if (n >= 10) {
            if (sb.length() > 0) sb.append(' ');
            sb.append(TEENS[n - 10]);
        } else if (n > 0) {
            if (sb.length() > 0) sb.append(' ');
            sb.append(ONES[n]);
        }
        return sb.toString();
    }
}
```

## Complexity Analysis
- **Time:** O(1) — the input is bounded by 2^31 - 1, so the number of chunks (at most 4: billions/millions/thousands/ones) and the work per chunk is constant regardless of input size.
- **Space:** O(1) — fixed-size lookup tables and a bounded number of result parts.

## Edge Cases
- `num == 0` → must special-case to `"Zero"`, since the general loop would otherwise produce an empty string.
- Numbers exactly on a scale boundary (e.g., `1000`, `1000000`) → the "ones" chunk is 0 and must be skipped entirely (no "Zero" appearing mid-string).
- Teens crossing into tens confusingly (e.g., `n=115` → "One Hundred Fifteen", not "One Hundred Ten Five") — this is why teens must be checked before the generic tens/ones split.
- Maximum value `2147483647` → exercises all four scale groups (billion, million, thousand, ones) simultaneously.
- Numbers like `100`, `1000`, `1000000` that are a bare scale with no remainder in lower chunks.

## Common Mistakes
- Treating 10-19 with the generic `tens[n//10] + ones[n%10]` formula — this breaks because "One Ten" / "One-Five" isn't how English works; teens are irregular and need their own lookup table.
- Inserting a scale word for a chunk that is zero (e.g., producing "Zero Thousand" for `num = 1000000`) — always guard with `if chunk_words:` / `if num >= scale` type checks before appending.
- Off-by-one in scale checks — using `>` instead of `>=` for whether a scale applies, which can silently skip a chunk equal to the scale itself.
- Extra or missing spaces when joining parts — safest to build a list of non-empty string fragments and `" ".join(...)` at the very end rather than concatenating with manual `+ " "` throughout.

## Related Problems
- [[Valid-Number]]
- [[Counting-Bits]]
- [[Reverse-Words-in-a-String]]

## Follow-up / Variants
A natural follow-up is converting to a different numeral system's words (e.g., Indian numbering with lakh/crore instead of thousand/million), which just changes the scale-word table and grouping boundaries. Another common variant: convert a number to Roman numerals (LC 12), which uses a similar greedy lookup-table technique but with a different grouping structure (subtractive notation like IV, IX).
