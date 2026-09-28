---
title: "Remove Nth Node From End of List"
aliases: []
tags: [leetcode, two-pointers, linked-list, medium]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 19
leetcode_difficulty: Medium
pattern: "Two Pointers (Fast/Slow with Gap)"
status: not-started
---

# Remove Nth Node From End of List

## Problem Statement
Given the head of a singly linked list, remove the n-th node from the end of the list and return the head of the modified list. You may be asked to do it in one pass (a single traversal of the list).

Constraints: the number of nodes is between 1 and 30, node values are in [0, 100], and `n` is guaranteed to be valid (`1 <= n <= length`).

## Examples
```
Input: head = [1,2,3,4,5], n = 2
Output: [1,2,3,5]

Input: head = [1], n = 1
Output: []

Input: head = [1,2], n = 1
Output: [1]
```

## Intuition & Approach
The naive approach is two passes: walk the list once to compute its length `L`, then walk again to the `(L - n)`-th node (0-indexed from head) and unlink the next node. That works but touches the list twice.

The one-pass insight: if you advance one pointer `n` steps ahead of a second pointer first, then move both pointers forward together one step at a time, the gap between them stays exactly `n`. When the front pointer reaches the end (`None`), the trailing pointer is sitting exactly one node before the node that needs to be removed. This is the classic "fast/slow pointer with a fixed gap" technique.

The tricky edge case is removing the head itself (e.g., `n == length`). A dummy node placed before `head` sidesteps this: the trailing pointer starts at `dummy` instead of `head`, so it can always safely do `prev.next = prev.next.next` even when the node being removed is the original head.

## Pattern Recognition
> [!tip] Pattern: Two Pointers (Fast/Slow with Gap)
> Whenever a linked-list problem asks you to find something relative to "the end" of the list without knowing the length up front, and asks for a single pass, think: advance one pointer `n` (or `k`) steps first, then move both together. The fixed gap turns an "end-relative" position into something the trailing pointer reaches naturally.

## Algorithm Steps
1. Create a `dummy` node with `dummy.next = head`. Set `slow = fast = dummy`.
2. Advance `fast` by `n + 1` steps (so `fast` ends up `n+1` nodes ahead of `slow`, i.e., `n` nodes ahead of `slow.next`).
3. While `fast` is not `None`, move both `slow` and `fast` one step at a time.
4. When `fast` becomes `None`, `slow.next` is the node to remove. Set `slow.next = slow.next.next`.
5. Return `dummy.next` (handles the case where the original head was removed).

## Solution (Python)
```python
from typing import Optional

class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

class Solution:
    def removeNthFromEnd(self, head: Optional[ListNode], n: int) -> Optional[ListNode]:
        dummy = ListNode(0, head)
        slow = fast = dummy

        # Move fast n+1 steps ahead so the gap between slow and the
        # node-to-remove's predecessor is exactly n.
        for _ in range(n + 1):
            fast = fast.next

        while fast is not None:
            slow = slow.next
            fast = fast.next

        slow.next = slow.next.next
        return dummy.next
```

## Solution (Java)
```java
class ListNode {
    int val;
    ListNode next;
    ListNode() {}
    ListNode(int val) { this.val = val; }
    ListNode(int val, ListNode next) { this.val = val; this.next = next; }
}

class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode dummy = new ListNode(0, head);
        ListNode slow = dummy, fast = dummy;

        // Move fast n+1 steps ahead so the gap between slow and the
        // node-to-remove's predecessor is exactly n.
        for (int i = 0; i < n + 1; i++) {
            fast = fast.next;
        }

        while (fast != null) {
            slow = slow.next;
            fast = fast.next;
        }

        slow.next = slow.next.next;
        return dummy.next;
    }
}
```

## Complexity Analysis
- **Time:** O(L) — one full pass over the list, where L is the list length.
- **Space:** O(1) — only a constant number of pointers are used (the dummy node is a single extra node, not proportional to input size).

## Edge Cases
- List has exactly one node and `n == 1` — must return an empty list (`None`), which is why the dummy node matters.
- `n` equals the length of the list — the head itself must be removed.
- `n == 1` — removing the last (tail) node, must correctly set the new tail's `next` to `None`.

## Common Mistakes
- Forgetting the dummy node and mishandling the case where the head is the node to be removed.
- Advancing `fast` by only `n` steps instead of `n + 1`, which leaves `slow` pointing *at* the node to remove instead of at its predecessor.
- Assuming you need the list length first (two-pass solution) when a one-pass solution is expected in an interview follow-up.

## Related Problems
- [[Merge-k-Sorted-Lists]]
- [[Longest-Subarray-of-1s-After-Deleting-One-Element]]
- [[Longest-Continuous-Subarray-With-Absolute-Diff-Limit]]

## Follow-up / Variants
A common follow-up is "remove the n-th node from the *start*" (trivial single pointer walk), or "remove all nodes with duplicate values," which combines this gap technique with a hash set of seen values.
