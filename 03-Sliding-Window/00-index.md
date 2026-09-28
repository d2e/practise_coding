---
title: "Sliding Window — Pattern Primer"
tags: [moc, pattern-primer, sliding-window]
---

# Sliding Window

## When to use this pattern
Sliding window applies whenever you need to find the longest, shortest, or otherwise optimal **contiguous** subarray/substring satisfying some condition, and that condition can be checked/maintained incrementally as you add or remove one element at a time from the edges of a window. The giveaway is that brute force would check all O(n^2) subarrays, but the condition is "monotonic" in the sense that growing the window can only make it harder to satisfy (or growing never un-satisfies something already satisfied) — which lets you avoid re-checking from scratch.

There are two flavors:
- **Fixed-size window** — window size `k` is given; slide it across the array, updating an incremental aggregate (sum, count, max) as one element leaves and one enters.
- **Variable-size window** — expand the right edge greedily; when the window becomes invalid, shrink from the left until valid again. The answer is typically tracked as the best window length seen.

When the window's validity depends on something more complex than a running sum/count — specifically a running **max or min** — a plain running aggregate isn't enough, because removing an element from the left might have been the max/min, and you can't recover the new max/min in O(1) without extra structure. That's when a **monotonic deque** (or two, one for max and one for min) is paired with the sliding window, so the current max/min is always readable in O(1) at the front of the deque.

## How to recognize it in a problem statement
- "Longest/shortest **contiguous** subarray/substring such that [condition]" is the single strongest signal.
- "At most k [defects]" (at most one zero, at most k distinct characters, etc.) — classic bounded-defect sliding window.
- "Max - min <= limit" or similar range-based validity condition — sliding window + monotonic deque(s).
- If the condition can hold, then stop holding, then hold again as you scan (non-monotonic), sliding window does NOT apply directly — you likely need prefix sums + hashing instead.

## Generic template (variable-size window)
```python
left = 0
best = 0
# window_state tracks whatever makes the window valid/invalid
for right in range(len(arr)):
    add(arr[right])  # expand window, update window_state
    while window_is_invalid():
        remove(arr[left])  # shrink window, update window_state
        left += 1
    best = max(best, right - left + 1)
```

## Problems in this folder
- [[Longest-Subarray-of-1s-After-Deleting-One-Element]] — window with at most one zero (bounded defect count).
- [[Longest-Continuous-Subarray-With-Absolute-Diff-Limit]] — window validity depends on running max/min, tracked via two monotonic deques.

See also [[Daily-Temperatures]] in `07-Stacks-and-Monotonic-Structures` — a different problem shape, but shares the "monotonic stack/deque avoids re-scanning" spirit with the deque technique used here.
