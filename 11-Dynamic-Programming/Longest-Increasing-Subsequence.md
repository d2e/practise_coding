---
title: "Longest Increasing Subsequence"
aliases: [LIS]
tags: [leetcode, dp, binary-search, medium]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 300
leetcode_difficulty: Medium
pattern: "DP + Patience Sorting / Binary Search"
status: not-started
---

# Longest Increasing Subsequence

## Problem Statement
Given an integer array `nums`, return the length of the longest strictly increasing subsequence. A subsequence is derived by deleting zero or more elements without changing the order of the remaining elements — the elements do **not** need to be contiguous.

## Examples
```
Input: nums = [10,9,2,5,3,7,101,18]
Output: 4
Explanation: The longest increasing subsequence is [2,3,7,101], length 4.
```

```
Input: nums = [0,1,0,3,2,3]
Output: 4
```

```
Input: nums = [7,7,7,7,7,7,7]
Output: 1          # strictly increasing, so repeated equal values don't extend it
```

## Intuition & Approach

### O(n²) DP — the natural first solution
Define `dp[i]` = the length of the longest increasing subsequence that *ends exactly at* index `i`. For each `i`, look back at every earlier index `j < i`: if `nums[j] < nums[i]`, then `nums[i]` could extend whatever subsequence ends at `j`, giving a candidate length `dp[j] + 1`. Taking the best such candidate over all valid `j` (or `1` if none extend, since `nums[i]` alone is a subsequence of length 1) gives:
```
dp[i] = 1 + max(dp[j] for j < i if nums[j] < nums[i], default=0)
```
The final answer is `max(dp)` over all `i`, since the longest subsequence can end anywhere.

### O(n log n) — patience sorting / binary search
The O(n²) approach recomputes a lot of redundant comparisons. The faster approach maintains an auxiliary array `tails`, where `tails[k]` = the *smallest possible tail value* of any increasing subsequence of length `k + 1` found so far. This array is always sorted, which is the crucial property that unlocks binary search.

For each new number `x`:
- If `x` is larger than every element in `tails`, it extends the longest subsequence found so far — append it.
- Otherwise, find the leftmost position in `tails` where `x` could replace an existing value (via binary search) — i.e., the first tail that is `>= x` — and overwrite it with `x`.

The "replace" step is the subtle part: overwriting `tails[k]` with a smaller value doesn't mean an increasing subsequence of that exact length using the *original* elements changed — `tails` is **not** a real subsequence at any point in the algorithm. It only tracks, for each achievable length, the best (smallest) possible tail value, because a smaller tail value strictly improves future extension chances (more numbers can be appended after a smaller tail than after a larger one). The length of the final `tails` array is the answer, even though `tails` itself does not represent an actual valid subsequence from the input.

## Pattern Recognition
> [!tip] Pattern: DP with an Optimized "Best Tail" Structure
> "Longest strictly increasing/decreasing subsequence" (not necessarily contiguous) is the canonical trigger for this pattern. Start by reasoning about the straightforward O(n²) DP (`dp[i]` = best subsequence ending at `i`), then recognize the O(n log n) upgrade whenever the DP's transition can be reframed as "maintain a sorted list of best-possible-tails per length" — this generalizes to problems like "minimum number of piles in patience sorting" and box-stacking variants.

## Algorithm Steps
**O(n²) DP:**
1. Initialize `dp = [1] * n` (every single element is a subsequence of length 1 on its own).
2. For each `i` from 1 to n-1, for each `j` from 0 to i-1: if `nums[j] < nums[i]`, set `dp[i] = max(dp[i], dp[j] + 1)`.
3. Return `max(dp)`.

**O(n log n) patience sorting:**
1. Initialize an empty list `tails`.
2. For each `x` in `nums`: binary search `tails` for the leftmost index where `tails[idx] >= x` (bisect_left).
3. If that index equals `len(tails)`, append `x` (it's larger than everything, extends the longest run). Otherwise, overwrite `tails[idx] = x`.
4. Return `len(tails)`.

## Solution (Python)
```python
from bisect import bisect_left
from typing import List

def lengthOfLIS(nums: List[int]) -> int:
    tails: List[int] = []
    for x in nums:
        idx = bisect_left(tails, x)
        if idx == len(tails):
            tails.append(x)
        else:
            tails[idx] = x
    return len(tails)


def lengthOfLIS_dp(nums: List[int]) -> int:
    """O(n^2) baseline — clearer to derive first, useful to present before optimizing."""
    if not nums:
        return 0
    n = len(nums)
    dp = [1] * n
    for i in range(1, n):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    return max(dp)
```

## Solution (Java)
```java
import java.util.Arrays;

class Solution {
    public int lengthOfLIS(int[] nums) {
        int[] tails = new int[nums.length];
        int size = 0; // number of valid entries in tails

        for (int x : nums) {
            int idx = lowerBound(tails, size, x);
            tails[idx] = x;
            if (idx == size) size++;
        }

        return size;
    }

    // Leftmost index where tails[idx] >= x, searching only within [0, size).
    private int lowerBound(int[] tails, int size, int x) {
        int lo = 0, hi = size;
        while (lo < hi) {
            int mid = (lo + hi) / 2;
            if (tails[mid] < x) lo = mid + 1;
            else hi = mid;
        }
        return lo;
    }

    // O(n^2) baseline — clearer to derive first, useful to present before optimizing.
    public int lengthOfLISDp(int[] nums) {
        if (nums.length == 0) return 0;
        int n = nums.length;
        int[] dp = new int[n];
        Arrays.fill(dp, 1);
        int best = 1;

        for (int i = 1; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if (nums[j] < nums[i]) {
                    dp[i] = Math.max(dp[i], dp[j] + 1);
                }
            }
            best = Math.max(best, dp[i]);
        }

        return best;
    }
}
```

## Complexity Analysis
- **O(n²) DP:** Time O(n²) (nested loop over all pairs), Space O(n) for the `dp` array.
- **O(n log n) patience sorting:** Time O(n log n) — n insertions, each doing an O(log n) binary search. Space O(n) for the `tails` array.

## Edge Cases
- Empty array — should return 0.
- All elements equal (e.g., `[7,7,7]`) — strictly increasing means no element can extend another; answer is 1.
- Already strictly increasing array — `tails` grows by one element every iteration, answer equals array length.
- Strictly decreasing array — every new element replaces `tails[0]`; final `tails` length stays 1.
- Single-element array — answer is 1.

## Common Mistakes
- Believing `tails` (in the O(n log n) approach) represents an actual increasing subsequence at every step — it doesn't; only its *length* is meaningful, which trips people up when asked to also reconstruct the actual subsequence (that requires additional parent-pointer bookkeeping, not just reading off `tails`).
- Using `bisect_left` vs `bisect_right` incorrectly — for a *strictly* increasing subsequence, you want `bisect_left` (find the first tail `>= x` and replace it); using `bisect_right` would incorrectly allow equal values to extend the subsequence, which is the correct choice only for *non-decreasing* subsequences instead.
- In the O(n²) DP, initializing `dp[i] = 0` instead of `1` — every element is trivially a subsequence of length 1 by itself, and forgetting this base case undercounts.
- Confusing "longest increasing subsequence" (elements need not be contiguous) with "longest increasing *subarray*" (must be contiguous, solvable with a simple linear scan, no DP or binary search needed).

## Related Problems
- [[Maximum-Subarray]]
- [[Minimum-Path-Sum]]
- [[Longest-Continuous-Subarray-With-Absolute-Diff-Limit]]

## Follow-up / Variants
A very common follow-up is "Russian Doll Envelopes" (LC 354), which is LIS in two dimensions: sort by one dimension, then run LIS-style patience sorting on the other, with a careful tie-breaking sort trick to avoid incorrectly nesting envelopes of equal width. Another follow-up asks to reconstruct the actual longest increasing subsequence (not just its length), which requires storing, for each element, a pointer to its predecessor in the best subsequence found so far, then walking that chain backward from the end.
