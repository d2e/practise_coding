---
title: "Longest Continuous Subarray With Absolute Diff Less Than or Equal to Limit"
aliases: []
tags: [leetcode, sliding-window, monotonic-deque, medium]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 1438
leetcode_difficulty: Medium
pattern: "Sliding Window + Monotonic Deque (running max/min)"
status: not-started
---

# Longest Continuous Subarray With Absolute Diff Less Than or Equal to Limit

## Problem Statement
Given an array of integers `nums` and an integer `limit`, return the size of the longest non-empty subarray such that the absolute difference between any two elements of this subarray is less than or equal to `limit`.

Constraints: `1 <= nums.length <= 10^5`, `1 <= nums[i] <= 10^9`, `0 <= limit <= 10^9`.

## Examples
```
Input: nums = [8,2,4,7], limit = 4
Output: 2
Explanation: [8,2] diff=6 (too big), [2,4] diff=2 (ok), [4,7] diff=3 (ok), [2,4,7] diff max-min=5 (too big).
The longest valid subarray is [2,4] or [4,7], size 2.

Input: nums = [10,1,2,4,7,2], limit = 5
Output: 4
Explanation: [2,4,7,2] has max=7, min=2, diff=5 <= limit. Size 4.

Input: nums = [4,2,2,2,4,4,2,2], limit = 0
Output: 3
```

## Intuition & Approach
For any subarray, "the absolute difference between any two elements is at most `limit`" is equivalent to "`max(subarray) - min(subarray) <= limit`" — you never need to compare all pairs, just the max and the min. This turns the problem into: find the longest window where `max - min <= limit`.

This is a classic sliding window: expand the right end of the window one element at a time; whenever the window's `max - min` exceeds `limit`, shrink from the left until it's valid again. The challenge is computing the window's max and min efficiently as the window slides — recomputing them from scratch on every shrink/expand would be O(n) per step, O(n^2) overall.

The standard trick: maintain two **monotonic deques** alongside the window.
- A **max-deque** that is monotonically decreasing (front holds the current window's maximum). When adding a new element, pop from the back any elements smaller than it (they can never be the max again while the new element is in the window), then push the new element.
- A **min-deque** that is monotonically increasing (front holds the current window's minimum), maintained symmetrically.

At each step, the window's max is `max_deque[0]` and its min is `min_deque[0]`, both in O(1). When shrinking the window from the left, if the element leaving equals the front of a deque, pop it from that deque's front too. This keeps both deques in sync with the window in amortized O(1) per operation (each element is pushed and popped at most once total across the whole run).

## Pattern Recognition
> [!tip] Pattern: Sliding Window + Monotonic Deque (running max/min)
> Whenever a sliding window's validity depends on its running maximum and/or minimum (not sum, not count), and you need O(1) access to that max/min as the window slides in both directions, reach for one or two monotonic deques instead of a heap (a heap can't efficiently remove arbitrary elements that fall out of the window on the left).

## Algorithm Steps
1. Initialize `left = 0`, an empty `max_deque` (stores indices, values decreasing front-to-back) and an empty `min_deque` (values increasing front-to-back).
2. For each `right` in `0..n-1`:
   - While `max_deque` is non-empty and `nums[max_deque[-1]] < nums[right]`: pop from the back of `max_deque`. Push `right`.
   - While `min_deque` is non-empty and `nums[min_deque[-1]] > nums[right]`: pop from the back of `min_deque`. Push `right`.
   - While `nums[max_deque[0]] - nums[min_deque[0]] > limit`: shrink the window — increment `left`, and if `max_deque[0] < left` pop it from the front, likewise for `min_deque[0]`.
   - Update `max_len = max(max_len, right - left + 1)`.
3. Return `max_len`.

## Solution (Python)
```python
from collections import deque
from typing import List

class Solution:
    def longestSubarray(self, nums: List[int], limit: int) -> int:
        max_deque: deque[int] = deque()  # indices, values decreasing
        min_deque: deque[int] = deque()  # indices, values increasing
        left = 0
        max_len = 0

        for right, val in enumerate(nums):
            while max_deque and nums[max_deque[-1]] < val:
                max_deque.pop()
            max_deque.append(right)

            while min_deque and nums[min_deque[-1]] > val:
                min_deque.pop()
            min_deque.append(right)

            while nums[max_deque[0]] - nums[min_deque[0]] > limit:
                left += 1
                if max_deque[0] < left:
                    max_deque.popleft()
                if min_deque[0] < left:
                    min_deque.popleft()

            max_len = max(max_len, right - left + 1)

        return max_len
```

## Solution (Java)
```java
import java.util.ArrayDeque;
import java.util.Deque;

class Solution {
    public int longestSubarray(int[] nums, int limit) {
        Deque<Integer> maxDeque = new ArrayDeque<>(); // indices, values decreasing
        Deque<Integer> minDeque = new ArrayDeque<>(); // indices, values increasing
        int left = 0;
        int maxLen = 0;

        for (int right = 0; right < nums.length; right++) {
            int val = nums[right];

            while (!maxDeque.isEmpty() && nums[maxDeque.peekLast()] < val) {
                maxDeque.pollLast();
            }
            maxDeque.addLast(right);

            while (!minDeque.isEmpty() && nums[minDeque.peekLast()] > val) {
                minDeque.pollLast();
            }
            minDeque.addLast(right);

            while (nums[maxDeque.peekFirst()] - nums[minDeque.peekFirst()] > limit) {
                left++;
                if (maxDeque.peekFirst() < left) {
                    maxDeque.pollFirst();
                }
                if (minDeque.peekFirst() < left) {
                    minDeque.pollFirst();
                }
            }

            maxLen = Math.max(maxLen, right - left + 1);
        }

        return maxLen;
    }
}
```

## Complexity Analysis
- **Time:** O(n) — each index is pushed onto each deque exactly once and popped at most once (from either end), so total deque operations across the whole run are O(n), amortized.
- **Space:** O(n) worst case for the two deques (e.g., a strictly increasing array fills the min-deque with every index).

## Edge Cases
- `limit == 0`: only subarrays of entirely identical elements qualify (as in example 3), since any difference at all exceeds 0.
- Strictly increasing or strictly decreasing array: one of the deques degenerates to holding just the single most extreme index at any time; verify both deques still update correctly.
- Single-element array: the window of size 1 trivially satisfies `max - min == 0 <= limit`, so the answer is `1`.

## Common Mistakes
- Using a single sorted structure (like a heap) for max/min tracking, which cannot efficiently evict elements that fall outside the window on the left (a heap doesn't support arbitrary removal in O(log n) without extra bookkeeping like lazy deletion).
- Forgetting to check `max_deque[0] < left` and `min_deque[0] < left` **both**, independently, when shrinking — an index might still be valid in one deque but stale in the other.
- Popping from the back with a `<=` comparison instead of `<` (or vice versa for the min-deque), which can incorrectly discard a legitimate tie-breaking candidate needed later.

## Related Problems
- [[Longest-Subarray-of-1s-After-Deleting-One-Element]]
- [[Daily-Temperatures]]
- [[Minimum-Absolute-Difference]]

## Follow-up / Variants
A common follow-up asks for an approach using two heaps (max-heap and min-heap of `(value, index)` pairs) with lazy deletion instead of deques — functionally equivalent but with O(log n) per operation instead of amortized O(1), useful if you're less comfortable with deque invariants. Another variant restricts to a fixed window size `k` (rather than a variable maximizing window), which is the more traditional "sliding window maximum" (LeetCode 239) using just the max-deque half of this technique.
