---
title: "Ways to Make a Fair Array"
aliases: []
tags: [leetcode, array, prefix-sum, medium]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 1664
leetcode_difficulty: Medium
pattern: "Prefix Sum (split by parity)"
status: not-started
---

# Ways to Make a Fair Array

## Problem Statement
You are given an integer array `nums`. You can choose exactly one index `i` and remove `nums[i]` from the array. After removal, the remaining elements shift left to fill the gap and are **re-indexed** starting from `0`.

An array is **fair** if the sum of elements at even indices equals the sum of elements at odd indices (using the re-indexing after removal).

Return the number of indices `i` such that removing `nums[i]` results in a fair array.

Constraints: `1 <= nums.length <= 10^5`, `0 <= nums[i] <= 10^4`.

## Examples
```
Input: nums = [2,1,6,4]
Output: 1
Explanation: Removing index 0 -> [1,6,4], odd-sum=6, even-sum=1+4=5. Not fair.
             Removing index 1 -> [2,6,4], odd-sum=6, even-sum=2+4=6. Fair!
             Removing index 2 -> [2,1,4], odd-sum=1, even-sum=2+4=6. Not fair.
             Removing index 3 -> [2,1,6], odd-sum=1, even-sum=2+6=8. Not fair.
             Only 1 index works.

Input: nums = [1,1,1]
Output: 3

Input: nums = [1,2,3]
Output: 0
```

## Intuition & Approach
The brute-force approach removes each index one at a time, rebuilds the array, and recomputes the even/odd sums from scratch — O(n) per removal, O(n^2) overall. With `n` up to `10^5`, that's too slow.

The key realization: when you remove index `i`, only the parity of indices **after** `i` changes (everything before `i` keeps its original parity; everything after `i` flips parity because it shifts left by one position). So instead of recomputing sums, precompute:
- `prefix_even[i]` = sum of elements at even indices in `nums[0..i-1]`
- `prefix_odd[i]` = sum of elements at odd indices in `nums[0..i-1]`
- `suffix_even[i]` = sum of elements at even indices in `nums[i+1..n-1]`
- `suffix_odd[i]` = sum of elements at odd indices in `nums[i+1..n-1]`

When removing index `i`, the elements before `i` retain their parity, but the elements after `i` have their parity **flipped** in the new array (an element that was at an even index shifts into an odd index, and vice versa). So after removing `nums[i]`:
- new even-sum = `prefix_even[i] + suffix_odd[i]` (suffix odds become the new evens)
- new odd-sum = `prefix_odd[i] + suffix_even[i]` (suffix evens become the new odds)

If these two are equal, removing index `i` produces a fair array. This can be computed in O(1) per index after an O(n) prefix pass, giving O(n) total.

## Pattern Recognition
> [!tip] Pattern: Prefix Sum (split by parity)
> Whenever a problem asks "what happens if I remove/insert one element" and the answer depends on sums that are split by even/odd index (or by any positional property that shifts after a removal), precompute prefix and suffix sums split by that property, so each query resolves in O(1) instead of O(n).

## Algorithm Steps
1. Compute total `even_sum` and `odd_sum` over the whole array (these serve as the running "suffix" sums, decremented as we scan).
2. Iterate `i` from `0` to `n-1`, maintaining running `prefix_even` and `prefix_odd` sums for indices strictly before `i`.
3. Before processing index `i`, subtract `nums[i]` from the appropriate running total (even or odd, based on `i`'s original parity) to get the correct suffix sums for indices strictly after `i`.
4. Compute `new_even = prefix_even + suffix_odd` and `new_odd = prefix_odd + suffix_even`; if equal, increment the answer counter.
5. Add `nums[i]` to the appropriate prefix sum (even or odd) before moving to `i + 1`.
6. Return the total count.

## Solution (Python)
```python
from typing import List

class Solution:
    def waysToMakeFair(self, nums: List[int]) -> int:
        n = len(nums)

        # Suffix sums start as totals; we'll peel off elements as we scan.
        suffix_even = sum(nums[i] for i in range(0, n, 2))
        suffix_odd = sum(nums[i] for i in range(1, n, 2))

        prefix_even = 0
        prefix_odd = 0
        count = 0

        for i in range(n):
            # Remove nums[i] from suffix sums so they represent indices strictly after i.
            if i % 2 == 0:
                suffix_even -= nums[i]
            else:
                suffix_odd -= nums[i]

            # After removing index i, prefix parities stay the same,
            # but suffix parities flip (odd suffix becomes even, even suffix becomes odd).
            new_even = prefix_even + suffix_odd
            new_odd = prefix_odd + suffix_even

            if new_even == new_odd:
                count += 1

            # Add nums[i] back into the prefix sums before moving on.
            if i % 2 == 0:
                prefix_even += nums[i]
            else:
                prefix_odd += nums[i]

        return count
```

## Solution (Java)
```java
class Solution {
    public int waysToMakeFair(int[] nums) {
        int n = nums.length;

        // Suffix sums start as totals; we'll peel off elements as we scan.
        int suffixEven = 0, suffixOdd = 0;
        for (int i = 0; i < n; i += 2) suffixEven += nums[i];
        for (int i = 1; i < n; i += 2) suffixOdd += nums[i];

        int prefixEven = 0, prefixOdd = 0;
        int count = 0;

        for (int i = 0; i < n; i++) {
            // Remove nums[i] from suffix sums so they represent indices strictly after i.
            if (i % 2 == 0) {
                suffixEven -= nums[i];
            } else {
                suffixOdd -= nums[i];
            }

            // After removing index i, prefix parities stay the same,
            // but suffix parities flip (odd suffix becomes even, even suffix becomes odd).
            int newEven = prefixEven + suffixOdd;
            int newOdd = prefixOdd + suffixEven;

            if (newEven == newOdd) {
                count++;
            }

            // Add nums[i] back into the prefix sums before moving on.
            if (i % 2 == 0) {
                prefixEven += nums[i];
            } else {
                prefixOdd += nums[i];
            }
        }

        return count;
    }
}
```

## Complexity Analysis
- **Time:** O(n) — one pass to compute initial suffix sums, one more pass to evaluate every removal candidate in O(1) each.
- **Space:** O(1) — only a constant number of running sums are kept; no per-index array is materialized.

## Edge Cases
- `n == 1`: removing the only element leaves an empty array, which is trivially fair (both sums are 0) — verify the formula yields `new_even == new_odd == 0` in this case.
- All elements equal (e.g., `[1,1,1]`): often every removal is fair, as in the third example.
- Large `n` (`10^5`) with all max values (`10^4`) — check for integer overflow is not a concern in Python, but is worth noting if porting to a fixed-width-integer language.

## Common Mistakes
- Recomputing sums from scratch for each candidate index, leading to O(n^2) time that times out on large inputs.
- Forgetting that suffix parities **flip** after removal — using `suffix_even` for the new even-sum (instead of `suffix_odd`) is the single most common bug in this problem.
- Off-by-one errors in exactly when `nums[i]` is subtracted from the suffix sum vs. added to the prefix sum relative to evaluating index `i`.

## Related Problems
- [[Minimum-Absolute-Difference]]
- [[Maximum-Subarray]]
- [[Longest-Continuous-Subarray-With-Absolute-Diff-Limit]]

## Follow-up / Variants
A natural extension: instead of removing exactly one element, what if you could remove any number of elements (with some cost), and must minimize cost to make the array fair? That moves from an O(n) prefix-sum scan into DP/greedy territory. Another variant swaps "even/odd index sum" for "sum of every k-th index," which generalizes the same prefix/suffix split technique to `k` buckets instead of 2.
