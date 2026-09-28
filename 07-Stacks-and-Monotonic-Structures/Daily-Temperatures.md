---
title: "Daily Temperatures"
aliases: []
tags: [leetcode, monotonic-stack, array, medium]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 739
leetcode_difficulty: Medium
pattern: "Monotonic Stack (next greater element)"
status: not-started
---

# Daily Temperatures

## Problem Statement
Given an array of integers `temperatures` representing daily temperatures, return an array `answer` such that `answer[i]` is the number of days you have to wait after day `i` to get a warmer temperature. If there is no future day for which this is possible, `answer[i] == 0`.

Constraints: `1 <= temperatures.length <= 10^5`, `30 <= temperatures[i] <= 100`.

## Examples
```
Input: temperatures = [73,74,75,71,69,72,76,73]
Output: [1,1,4,2,1,1,0,0]

Input: temperatures = [30,40,50,60]
Output: [1,1,1,0]

Input: temperatures = [30,60,90]
Output: [1,1,0]
```

## Intuition & Approach
The brute-force approach is, for each day `i`, scan forward day by day until you find a warmer temperature, recording the distance — O(n) per day, O(n^2) overall. With `n` up to `10^5`, this is too slow.

This is a classic **"next greater element"** problem. The key insight: maintain a stack of day-indices for which we have *not yet* found a warmer future day, kept in a way that the temperatures at those indices are in **decreasing** order from bottom to top (a monotonic decreasing stack). When processing a new day `i` with temperature `temperatures[i]`, this new temperature might be the "answer" for some or all of the indices currently sitting on the stack — specifically, every index on top of the stack whose temperature is strictly less than `temperatures[i]` has just found its warmer day. Pop each of those, and compute `i - popped_index` as their wait time. Once no more stack-top elements are colder than the current day, push `i` onto the stack (it now waits for its own future warmer day).

Because the stack is decreasing, once you pop an index and resolve it, it's *never revisited* — every index is pushed once and popped at most once, giving O(n) total work despite the nested-looking `while` loop inside the `for` loop.

## Pattern Recognition
> [!tip] Pattern: Monotonic Stack (next greater element)
> Whenever a problem asks "for each element, find the next element to the right that is greater (or smaller)," and a brute-force scan would be O(n^2), reach for a monotonic stack. It resolves each element exactly once, achieving O(n) by ensuring only "still waiting" candidates are ever kept on the stack.

## Algorithm Steps
1. Initialize `answer` as an array of zeros with the same length as `temperatures`, and an empty stack (holding day-indices).
2. For each day `i` with temperature `temperatures[i]`:
   - While the stack is non-empty and `temperatures[i] > temperatures[stack_top]`: pop the top index `j`, and set `answer[j] = i - j`.
   - Push `i` onto the stack.
3. After processing all days, any indices remaining on the stack never found a warmer day, so their `answer` entries stay `0` (already initialized).
4. Return `answer`.

## Solution (Python)
```python
from typing import List

class Solution:
    def dailyTemperatures(self, temperatures: List[int]) -> List[int]:
        n = len(temperatures)
        answer = [0] * n
        stack: List[int] = []  # indices with temperatures in decreasing order

        for i, temp in enumerate(temperatures):
            while stack and temperatures[stack[-1]] < temp:
                j = stack.pop()
                answer[j] = i - j
            stack.append(i)

        return answer
```

## Solution (Java)
```java
import java.util.ArrayDeque;
import java.util.Deque;

class Solution {
    public int[] dailyTemperatures(int[] temperatures) {
        int n = temperatures.length;
        int[] answer = new int[n];
        Deque<Integer> stack = new ArrayDeque<>(); // indices with temperatures in decreasing order

        for (int i = 0; i < n; i++) {
            while (!stack.isEmpty() && temperatures[stack.peek()] < temperatures[i]) {
                int j = stack.pop();
                answer[j] = i - j;
            }
            stack.push(i);
        }

        return answer;
    }
}
```

## Complexity Analysis
- **Time:** O(n) — each index is pushed onto the stack exactly once and popped at most once, so the total number of stack operations across the entire run is bounded by 2n.
- **Space:** O(n) worst case for the stack (e.g., a strictly decreasing temperature sequence pushes every index without ever popping), plus O(n) for the output array.

## Edge Cases
- Strictly decreasing temperatures (e.g., `[90,80,70]`): no day ever finds a warmer future day; every entry in `answer` stays `0`, and the stack grows to hold every index until the end.
- Strictly increasing temperatures (e.g., `[30,40,50,60]` from example 2): every day resolves on the very next iteration, so the stack rarely holds more than one element at a time.
- All equal temperatures (e.g., `[50,50,50]`): the strict `<` comparison means equal temperatures never trigger a pop, so all entries stay `0` — verify the problem wants strictly warmer, not warmer-or-equal (it does).

## Common Mistakes
- Using `<=` instead of `<` in the stack-popping comparison, which would incorrectly treat equal temperatures as "warmer," producing wrong wait times.
- Storing temperatures (values) on the stack instead of indices — you need the index to compute `i - j`, not just to know which value was warmer.
- Attempting an O(n^2) nested-loop brute force under time pressure without recognizing the monotonic stack pattern, leading to a timeout on large inputs.

## Related Problems
- [[Longest-Continuous-Subarray-With-Absolute-Diff-Limit]]
- [[Longest-Subarray-of-1s-After-Deleting-One-Element]]
- [[K-Closest-Points-to-Origin]]

## Follow-up / Variants
The same monotonic-stack shape solves "Next Greater Element I/II" (LeetCode 496/503) directly — those ask for the *value* of the next greater element rather than the distance to it, and the circular-array variant (503) additionally requires conceptually doubling the array (or wrapping the index with modulo) to handle wraparound. Another follow-up: adapt this to find the next **smaller** element instead, which only requires flipping the comparison direction to build an increasing stack.
