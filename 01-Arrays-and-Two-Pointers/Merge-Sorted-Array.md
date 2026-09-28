---
title: "Merge Sorted Array"
aliases: []
tags: [leetcode, two-pointers, easy]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 88
leetcode_difficulty: Easy
pattern: "Two Pointers (merge from the back)"
status: not-started
---

# Merge Sorted Array

## Problem Statement
You are given two integer arrays `nums1` and `nums2`, sorted in non-decreasing order, and two integers `m` and `n` representing the number of elements in `nums1` and `nums2` respectively.

`nums1` has a total length of `m + n`: the first `m` elements are the real data, and the last `n` elements are `0` and should be ignored — they exist only to give you room to write the merged result.

Merge `nums1` and `nums2` into a single array sorted in non-decreasing order, stored **in-place inside `nums1`**. The final sorted array should not be returned; it should be stored inside `nums1`, with the first `m + n` elements holding the merged values.

Constraints: `nums1.length == m + n`, `nums2.length == n`, `0 <= m, n <= 200`, `1 <= m + n <= 200`.

## Examples
```
Input: nums1 = [1,2,3,0,0,0], m = 3, nums2 = [2,5,6], n = 3
Output: [1,2,2,3,5,6]

Input: nums1 = [1], m = 1, nums2 = [], n = 0
Output: [1]

Input: nums1 = [0], m = 0, nums2 = [1], n = 1
Output: [1]
```

## Intuition & Approach
The naive approach is to overwrite `nums1[0:m]` and `nums2` into a fresh array, sort it, and copy it back — O((m+n) log(m+n)) time and O(m+n) extra space. That works but throws away the fact that both inputs are already sorted, and it ignores the "in-place" spirit of the problem.

The key insight: `nums1` has exactly `n` trailing zero slots — exactly enough room for all of `nums2`. If you merge from the **front**, you'd immediately start overwriting `nums1` elements you haven't read yet, because the write pointer and the unread real data occupy the same region. Merging from the **back** avoids this entirely: the tail of `nums1` is empty space, so you can safely write the largest remaining value there without clobbering anything you still need to read.

So: keep two read pointers, one at the end of the real data in `nums1` (`i = m - 1`) and one at the end of `nums2` (`j = n - 1`), and a write pointer at the very end of `nums1` (`k = m + n - 1`). Repeatedly compare `nums1[i]` and `nums2[j]`, write the larger one at `nums1[k]`, and decrement the corresponding pointers. This is standard merge-step logic from merge sort, just run in reverse.

## Pattern Recognition
> [!tip] Pattern: Two Pointers (merge from the back)
> Whenever you must merge two sorted sequences in-place and one of them has trailing unused space exactly sized to fit the other, merging from the back (largest-to-smallest) lets you write without overwriting data you still need to read.

## Algorithm Steps
1. Set `i = m - 1` (last real element of `nums1`), `j = n - 1` (last element of `nums2`), `k = m + n - 1` (last index of `nums1`, the next write slot).
2. While `i >= 0` and `j >= 0`: compare `nums1[i]` and `nums2[j]`; write the larger at `nums1[k]`; decrement that source pointer and `k`.
3. When the loop ends, if `j >= 0` still has remaining elements, copy them into `nums1[0..j]` (any remaining `nums1[i]` elements are already in their correct final position and need no copying).
4. Return (the array is merged in place).

## Solution (Python)
```python
from typing import List

class Solution:
    def merge(self, nums1: List[int], m: int, nums2: List[int], n: int) -> None:
        i, j, k = m - 1, n - 1, m + n - 1

        while i >= 0 and j >= 0:
            if nums1[i] > nums2[j]:
                nums1[k] = nums1[i]
                i -= 1
            else:
                nums1[k] = nums2[j]
                j -= 1
            k -= 1

        # Any leftover nums1 elements are already correctly placed.
        # Only leftover nums2 elements still need copying.
        while j >= 0:
            nums1[k] = nums2[j]
            j -= 1
            k -= 1
```

## Solution (Java)
```java
class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int i = m - 1, j = n - 1, k = m + n - 1;

        while (i >= 0 && j >= 0) {
            if (nums1[i] > nums2[j]) {
                nums1[k--] = nums1[i--];
            } else {
                nums1[k--] = nums2[j--];
            }
        }

        // Any leftover nums1 elements are already correctly placed.
        // Only leftover nums2 elements still need copying.
        while (j >= 0) {
            nums1[k--] = nums2[j--];
        }
    }
}
```

## Complexity Analysis
- **Time:** O(m + n) — each element from both arrays is visited and written exactly once.
- **Space:** O(1) — merging happens entirely within the existing `nums1` array, no auxiliary array is allocated.

## Edge Cases
- `m == 0`: `nums1` starts with no real data; the whole result should just be a copy of `nums2`.
- `n == 0`: `nums2` is empty; `nums1` is already correctly merged and no writes are needed (the trailing `while j >= 0` loop naturally does nothing).
- All elements of `nums2` are smaller than all elements of `nums1`, or vice versa — verify the comparison logic handles a fully one-sided merge.

## Common Mistakes
- Merging from the front instead of the back, which overwrites `nums1` elements before they've been read, corrupting the result.
- Forgetting the final "copy leftover `nums2`" loop — if `nums2` still has smaller elements left when `nums1`'s real data is exhausted, they must still be copied over (the leftover `nums1` elements, by contrast, are already correctly positioned and need no action).

## Related Problems
- [[Minimum-Absolute-Difference]]
- [[Merge-k-Sorted-Lists]]
- [[Longest-Continuous-Subarray-With-Absolute-Diff-Limit]]

## Follow-up / Variants
A natural follow-up: what if `nums1` did *not* have extra trailing space (i.e., you must return a new merged array)? That reduces to the classic merge-sort merge step with O(m+n) auxiliary space. Another variant asks you to merge `k` sorted arrays at once — see [[Merge-k-Sorted-Lists]] for the heap-based generalization.
