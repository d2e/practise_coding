---
title: "Counting Bits"
aliases: []
tags: [leetcode, dp, bit-manipulation, easy]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 338
leetcode_difficulty: Easy
pattern: "DP Built on Bit Manipulation"
status: not-started
---

# Counting Bits

## Problem Statement
Given an integer `n`, return an array `ans` of length `n + 1` such that for each `i` (`0 <= i <= n`), `ans[i]` is the number of `1`'s in the binary representation of `i`.

## Examples
```
Input: n = 2
Output: [0,1,1]
Explanation: 0 --> 0, 1 --> 1, 2 --> 10
```

```
Input: n = 5
Output: [0,1,1,2,1,2]
Explanation: 0-->0, 1-->1, 2-->10, 3-->11, 4-->100, 5-->101
```

## Intuition & Approach
The brute-force baseline is: for each number from `0` to `n`, count its set bits directly using **Brian Kernighan's algorithm** — repeatedly do `num &= (num - 1)`, which clears the lowest set bit each time, and count how many iterations it takes to reach 0. That's `O(n)` numbers times `O(log n)` bits each, so `O(n log n)` overall — correct, but wasteful, because it recomputes similar work for every number independently.

The DP insight is that the bit count of `i` can be derived in O(1) from the bit count of a *smaller* number already computed, via one of two equivalent recurrences:
- **Right-shift recurrence:** `i >> 1` is `i` with its last bit dropped, and `i & 1` tells you whether that dropped bit was a 1. So `popcount(i) = popcount(i >> 1) + (i & 1)`.
- **Lowest-set-bit recurrence:** `i & (i - 1)` clears the lowest set bit of `i`, meaning `popcount(i) = popcount(i & (i - 1)) + 1`.

Either way, you build the answer array bottom-up, where each entry only depends on an entry strictly earlier in the array — a classic 1D DP over integers rather than over array indices with more complex transitions.

## Pattern Recognition
> [!tip] Pattern: DP over Bit Structure
> Whenever a problem asks for a property of "every number from 0 to n" (bit count, digit sum, etc.) and that property can be expressed recursively in terms of a smaller/related number (like `i >> 1` or `i - 1`), it's a strong hint to build a DP table indexed by the number itself, reusing already-computed smaller entries, rather than recomputing from scratch for every value.

## Algorithm Steps
1. Allocate `ans` of size `n + 1`, with `ans[0] = 0` as the base case.
2. For `i` from `1` to `n`: set `ans[i] = ans[i >> 1] + (i & 1)`.
3. Return `ans`.

## Solution (Python)
```python
from typing import List

def countBits(n: int) -> List[int]:
    ans = [0] * (n + 1)
    for i in range(1, n + 1):
        ans[i] = ans[i >> 1] + (i & 1)
    return ans


def countBits_brute_force(n: int) -> List[int]:
    """Baseline using Brian Kernighan's algorithm, O(n log n), for comparison."""
    def popcount(num: int) -> int:
        count = 0
        while num:
            num &= num - 1  # clears the lowest set bit
            count += 1
        return count

    return [popcount(i) for i in range(n + 1)]
```

## Solution (Java)
```java
class Solution {
    public int[] countBits(int n) {
        int[] ans = new int[n + 1];
        for (int i = 1; i <= n; i++) {
            ans[i] = ans[i >> 1] + (i & 1);
        }
        return ans;
    }

    // Baseline using Brian Kernighan's algorithm, O(n log n), for comparison.
    public int[] countBitsBruteForce(int n) {
        int[] ans = new int[n + 1];
        for (int i = 0; i <= n; i++) {
            ans[i] = popcount(i);
        }
        return ans;
    }

    private int popcount(int num) {
        int count = 0;
        while (num != 0) {
            num &= num - 1; // clears the lowest set bit
            count++;
        }
        return count;
    }
}
```

## Complexity Analysis
- **Time:** O(n) for the DP solution — each entry is computed in O(1) from an already-computed entry. (The brute-force baseline is O(n log n).)
- **Space:** O(n) for the output array (which is required regardless, since it's the return value); O(1) auxiliary space beyond that.

## Edge Cases
- `n = 0` — result is simply `[0]`.
- `n = 1` — result is `[0, 1]`.
- Powers of two (e.g., `i = 8 = 1000₂`) — should correctly report a bit count of 1, verifying the recurrence handles the "single high bit, rest zero" case.
- Large `n` — solution should remain linear, not blow up due to redundant recomputation.

## Common Mistakes
- Recomputing `popcount` from scratch for each `i` (the brute-force approach) when an O(n) DP is straightforward and expected as the "optimal" solution in interviews.
- Using `i >> 1` correctly but forgetting the `+ (i & 1)` term, which silently produces wrong counts for all odd numbers.
- Off-by-one on array size — the array must have `n + 1` entries (indices `0` through `n` inclusive), not `n`.

## Related Problems
- [[Digit-Operations-to-Make-Two-Integers-Equal]]
- [[Minimum-Path-Sum]]

## Follow-up / Variants
A common follow-up is to do this without any built-in bit-count function *and* without extra space beyond the output array — the shown DP solution already satisfies both. Another variant asks for the count of numbers in `[0, n]` whose bit count itself has a certain property (e.g., is prime), which layers a second pass on top of this same `ans` array.
