---
title: "Maximum Subarray"
aliases: []
tags: [leetcode, dp, kadane, medium]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 53
leetcode_difficulty: Medium
pattern: "Kadane's Algorithm"
status: not-started
---

# Maximum Subarray

## Problem Statement
Given an integer array `nums`, find the contiguous subarray (containing at least one number) which has the largest sum, and return that sum.

## Examples
```
Input: nums = [-2,1,-3,4,-1,2,1,-5,4]
Output: 6
Explanation: [4,-1,2,1] has the largest sum = 6.
```

```
Input: nums = [-1]
Output: -1
```

```
Input: nums = [5,4,-1,7,8]
Output: 23
```

## Intuition & Approach
The brute-force approach tries every possible subarray (`O(n²)` pairs of start/end, or `O(n³)` if you recompute sums naively each time) and tracks the maximum sum seen. This is correct but slow.

The key insight — **Kadane's algorithm** — is a local decision made at every position: for the subarray ending exactly at index `i`, either it extends the best subarray ending at `i-1` by including `nums[i]`, or it's more profitable to *start fresh* at `i` alone. Formally, define `current_max` = the maximum sum of a subarray ending at the current index. Then:
```
current_max = max(nums[i], current_max + nums[i])
```
If `current_max` (the running sum so far) is negative, it can only ever *drag down* any future subarray it's appended to — so it's strictly better to abandon it and restart from `nums[i]` alone. This is why the recurrence takes `max(nums[i], current_max + nums[i])` instead of always extending. The overall answer is the maximum `current_max` seen across all positions, since the best subarray must end *somewhere*, and we're tracking the best ending at every possible position.

This is technically a DP formulation (`dp[i]` = best subarray ending at `i`, computed from `dp[i-1]`), just with the DP array collapsed to a single rolling variable since each state only depends on the immediately preceding one.

## Pattern Recognition
> [!tip] Pattern: Kadane's Algorithm
> "Maximum/minimum sum of a contiguous subarray" is the canonical signal. The recurrence "extend the previous run, or restart here" — `best_ending_here = max(nums[i], best_ending_here + nums[i])` — applies any time a running total can be reset to a fresh start whenever continuing it becomes counter-productive (negative for sums, but the same shape recurs for products with sign-flip handling, etc.).

## Algorithm Steps
1. Initialize `current_max = nums[0]` and `global_max = nums[0]` (handles the "at least one number" requirement and negative-only arrays correctly).
2. For each subsequent element `nums[i]`: set `current_max = max(nums[i], current_max + nums[i])`.
3. Update `global_max = max(global_max, current_max)`.
4. After the loop, `global_max` holds the answer.

## Solution (Python)
```python
from typing import List

def maxSubArray(nums: List[int]) -> int:
    current_max = global_max = nums[0]

    for num in nums[1:]:
        current_max = max(num, current_max + num)
        global_max = max(global_max, current_max)

    return global_max


def maxSubArray_divide_and_conquer(nums: List[int]) -> int:
    """O(n log n) alternative, useful as a follow-up discussion point."""
    def helper(lo: int, hi: int) -> int:
        if lo == hi:
            return nums[lo]

        mid = (lo + hi) // 2
        left_max = helper(lo, mid)
        right_max = helper(mid + 1, hi)

        # Best subarray crossing the midpoint.
        left_cross = float("-inf")
        total = 0
        for i in range(mid, lo - 1, -1):
            total += nums[i]
            left_cross = max(left_cross, total)

        right_cross = float("-inf")
        total = 0
        for i in range(mid + 1, hi + 1):
            total += nums[i]
            right_cross = max(right_cross, total)

        return max(left_max, right_max, left_cross + right_cross)

    return helper(0, len(nums) - 1)
```

## Solution (Java)
```java
class Solution {
    public int maxSubArray(int[] nums) {
        int currentMax = nums[0];
        int globalMax = nums[0];

        for (int i = 1; i < nums.length; i++) {
            currentMax = Math.max(nums[i], currentMax + nums[i]);
            globalMax = Math.max(globalMax, currentMax);
        }

        return globalMax;
    }

    // O(n log n) alternative, useful as a follow-up discussion point.
    public int maxSubArrayDivideAndConquer(int[] nums) {
        return helper(nums, 0, nums.length - 1);
    }

    private int helper(int[] nums, int lo, int hi) {
        if (lo == hi) return nums[lo];

        int mid = (lo + hi) / 2;
        int leftMax = helper(nums, lo, mid);
        int rightMax = helper(nums, mid + 1, hi);

        // Best subarray crossing the midpoint.
        int leftCross = Integer.MIN_VALUE;
        int total = 0;
        for (int i = mid; i >= lo; i--) {
            total += nums[i];
            leftCross = Math.max(leftCross, total);
        }

        int rightCross = Integer.MIN_VALUE;
        total = 0;
        for (int i = mid + 1; i <= hi; i++) {
            total += nums[i];
            rightCross = Math.max(rightCross, total);
        }

        return Math.max(Math.max(leftMax, rightMax), leftCross + rightCross);
    }
}
```

## Complexity Analysis
- **Time:** O(n) for Kadane's algorithm — a single linear pass. (The divide-and-conquer alternative is O(n log n), included for interview follow-up discussion since it's a common "can you think of another way?" prompt.)
- **Space:** O(1) for Kadane's — just two rolling variables. (Divide-and-conquer uses O(log n) recursion stack space.)

## Edge Cases
- All-negative array (e.g., `[-3, -1, -2]`) — the answer must still be the least-negative single element (`-1`), not `0`, since an empty subarray isn't allowed; initializing both trackers to `nums[0]` handles this correctly.
- Single-element array — trivially returns that element.
- Array with a single very large positive number surrounded by negatives — Kadane's "restart if negative" logic correctly isolates just that number.
- All-positive array — the entire array is the answer, and the running sum never resets.

## Common Mistakes
- Initializing `current_max`/`global_max` to `0` instead of `nums[0]` — this silently breaks on all-negative arrays by allowing an implicit "empty subarray with sum 0," which isn't a valid answer per the problem's "at least one number" requirement.
- Writing the recurrence as `current_max = max(0, current_max) + nums[i]` (clamping to zero) without separately tracking whether an all-negative array needs special handling — this is a common way the all-negative edge case slips through.
- Confusing this with the "maximum product subarray" variant, where the recurrence must track *both* a running max and running min (because a negative number can flip a very negative running product into the new maximum).

## Related Problems
- [[Minimum-Path-Sum]]
- [[Longest-Increasing-Subsequence]]
- [[Longest-Continuous-Subarray-With-Absolute-Diff-Limit]]

## Follow-up / Variants
The most common follow-up is "Maximum Product Subarray" (LC 152), which requires tracking both a running max *and* running min at each step, since multiplying by a negative number can turn the smallest running product into the largest. Another common ask is to also return the *indices* of the maximum subarray, not just its sum, which requires tracking the start index whenever the running sum resets.
