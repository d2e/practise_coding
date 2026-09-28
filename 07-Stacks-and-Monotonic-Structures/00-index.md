---
title: "Stacks & Monotonic Structures — Pattern Primer"
tags: [moc, pattern-primer, stack, monotonic-stack]
---

# Stacks & Monotonic Structures

## When to use this pattern
A monotonic stack (or deque) keeps its elements in strictly increasing or strictly decreasing order at all times, by popping elements that would violate that order before pushing a new one. This is the go-to technique for **"next/previous greater/smaller element"** style problems: for every element, you want to know the nearest element to its left or right that is bigger (or smaller) than it.

The reason this beats brute force: a naive solution scans forward (or backward) from every index looking for the answer, which is O(n^2) in the worst case (e.g., a strictly decreasing sequence, where every element has to scan almost to the end). With a monotonic stack, every element is pushed exactly once and popped at most once over the entire run, because once an element is popped (resolved), it never needs to be reconsidered — its answer is now known. That guarantees O(n) total work despite the nested-looking loop.

The general recipe: iterate through the array once. At each new element, while the stack's top violates the monotonic invariant relative to the new element, pop it and resolve its answer using the current index/value. Then push the new element.

## How to recognize it in a problem statement
- "For each element, find the next/previous element that is greater/smaller" — the canonical signal.
- "Days until a warmer temperature," "next larger element," "trapping rain water," "largest rectangle in histogram" — all variations on the same monotonic-stack skeleton.
- If you find yourself writing a nested loop that rescans from each index, and the inner scan direction only ever moves forward (never backtracks past a point once skipped), that's a strong hint the nested loop collapses to O(n) via a monotonic stack.

## Generic template (next greater element, to the right)
```python
n = len(arr)
answer = [-1] * n  # or 0, depending on problem semantics
stack = []  # indices, values strictly decreasing bottom-to-top

for i, val in enumerate(arr):
    while stack and arr[stack[-1]] < val:
        j = stack.pop()
        answer[j] = i  # or i - j, or val, depending on what's asked
    stack.append(i)
# anything left on the stack at the end never found its answer
```

## Problems in this folder
- [[Daily-Temperatures]] — monotonic decreasing stack of indices, resolving "days until warmer" in O(n).

See also [[Longest-Continuous-Subarray-With-Absolute-Diff-Limit]] in `03-Sliding-Window` — uses monotonic **deques** (not stacks) to track a sliding window's running max/min, a closely related but distinct use of the same "keep it monotonic to avoid re-scanning" idea.
