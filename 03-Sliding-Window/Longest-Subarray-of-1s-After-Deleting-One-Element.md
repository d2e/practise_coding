---
title: "Longest Subarray of 1's After Deleting One Element"
aliases: []
tags: [leetcode, sliding-window, array, medium]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 1493
leetcode_difficulty: Medium
pattern: "Sliding Window (bounded defect count)"
status: not-started
---

# Longest Subarray of 1's After Deleting One Element

## Problem Statement
Given a binary array `nums`, you must delete **exactly one element** from it. Return the size of the longest non-empty subarray containing only `1`'s in the resulting array. If there is no such subarray, return `0`.

Constraints: `1 <= nums.length <= 10^5`, `nums[i]` is either `0` or `1`.

## Examples
```
Input: nums = [1,1,0,1]
Output: 3
Explanation: Delete the 0 to get [1,1,1], the longest run of 1's has length 3.

Input: nums = [0,1,1,1,0,1,1,0,1]
Output: 5
Explanation: Delete the 0 at index 4 to get [0,1,1,1,1,1,0,1], the longest run of 1's is length 5.

Input: nums = [1,1,1]
Output: 2
Explanation: You must delete one element even though the whole array is 1's, so the best you can do is length 2.
```

## Intuition & Approach
Deleting exactly one element from a run of 1's is equivalent to finding the longest contiguous window that contains **at most one 0** — because that one 0 is exactly the element you delete, and everything else in the window is a 1 that survives. This reframes "delete one element to maximize the run of 1's" into a classic sliding-window problem: find the longest window with at most one "defect" (a 0).

Maintain a window `[left, right]` and a count of zeros currently inside it. Expand `right` one step at a time, incrementing the zero count whenever `nums[right] == 0`. Whenever the zero count exceeds 1, shrink from the left (incrementing `left`, decrementing the zero count if the element leaving was a 0) until the count is back to at most 1. At every step, the current window length represents "elements that survive after deleting the (at most one) zero inside it" — but since we must delete a real element, the actual answer for that window is `window_length - 1` (accounting for the one deleted slot, whether it was a 0 or, in the all-1's case, a 1 you're forced to remove).

The `window_length - 1` subtlety is what handles the tricky all-ones case (example 3): even though `[1,1,1]` has zero defects, you're still required to delete exactly one element, so the answer is `3 - 1 = 2`, not `3`.

## Pattern Recognition
> [!tip] Pattern: Sliding Window (bounded defect count)
> Whenever a problem allows "at most k bad elements" inside a contiguous window (here, at most one zero) and asks for the longest/shortest such window, use a two-pointer sliding window that expands greedily and shrinks only when the defect budget is exceeded.

## Algorithm Steps
1. Initialize `left = 0` and `zero_count = 0`.
2. Iterate `right` from `0` to `n-1`, and if `nums[right] == 0`, increment `zero_count`.
3. While `zero_count > 1`: if `nums[left] == 0`, decrement `zero_count`; then increment `left` (shrinking the window from the left).
4. Track the maximum window length seen: `max_len = max(max_len, right - left + 1)`.
5. Return `max_len - 1` (since exactly one element — the deleted one — must always be removed from the window).

## Solution (Python)
```python
from typing import List

class Solution:
    def longestSubarray(self, nums: List[int]) -> int:
        left = 0
        zero_count = 0
        max_len = 0

        for right in range(len(nums)):
            if nums[right] == 0:
                zero_count += 1

            while zero_count > 1:
                if nums[left] == 0:
                    zero_count -= 1
                left += 1

            max_len = max(max_len, right - left + 1)

        # Must delete exactly one element, even if the window is all 1's.
        return max_len - 1
```

## Solution (Java)
```java
class Solution {
    public int longestSubarray(int[] nums) {
        int left = 0;
        int zeroCount = 0;
        int maxLen = 0;

        for (int right = 0; right < nums.length; right++) {
            if (nums[right] == 0) {
                zeroCount++;
            }

            while (zeroCount > 1) {
                if (nums[left] == 0) {
                    zeroCount--;
                }
                left++;
            }

            maxLen = Math.max(maxLen, right - left + 1);
        }

        // Must delete exactly one element, even if the window is all 1's.
        return maxLen - 1;
    }
}
```

## Complexity Analysis
- **Time:** O(n) — `left` and `right` each traverse the array at most once; the window never expands and shrinks more than 2n times total (amortized two-pointer analysis).
- **Space:** O(1) — only a constant number of counters (`left`, `zero_count`, `max_len`) are maintained.

## Edge Cases
- Array is all zeros (e.g., `[0,0,0]`): every window has at most one zero only when the window size is 1; `max_len` ends up as `1`, and the answer is `1 - 1 = 0`, correctly signaling no valid subarray of 1's remains.
- Array is all ones: as in example 3, the answer is `n - 1`, not `n`, because deletion is mandatory.
- Single-element array (`n == 1`): whatever the value, after forced deletion nothing remains, so the answer must be `0` — verify the formula produces this (window length 1, minus 1, equals 0).

## Common Mistakes
- Forgetting the final `- 1`, which is easy to miss since the window itself represents "elements after deletion" conceptually, but the deletion of the one bad (or, in the all-1s case, one good) element still has to be accounted for.
- Allowing `zero_count` to exceed 1 without shrinking (off-by-one in the `while` condition, e.g., using `>` vs `>=`).
- Not decrementing `zero_count` when the element leaving the window on the left was itself a zero, causing the count to drift incorrect over the rest of the scan.

## Related Problems
- [[Longest-Continuous-Subarray-With-Absolute-Diff-Limit]]
- [[Ways-to-Make-a-Fair-Array]]
- [[Daily-Temperatures]]

## Follow-up / Variants
The natural generalization (and a very common interview follow-up) is "Max Consecutive Ones III": given a binary array and an integer `k`, find the longest subarray after flipping at most `k` zeros to ones — same sliding window shape, but the defect budget is `k` instead of a fixed 1, and there's no forced `-1` adjustment since flipping (not deleting) is allowed.
