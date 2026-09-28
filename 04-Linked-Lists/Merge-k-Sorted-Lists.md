---
title: "Merge k Sorted Lists"
aliases: []
tags: [leetcode, heap, linked-list, divide-and-conquer, hard]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 23
leetcode_difficulty: Hard
pattern: "K-Way Merge (Heap or Divide & Conquer)"
status: not-started
---

# Merge k Sorted Lists

## Problem Statement
You are given an array of `k` linked lists, each sorted in ascending order. Merge all the linked lists into one sorted linked list and return its head.

Constraints: `k` can be up to 10,000 lists, and the total number of nodes across all lists can be up to 10,000.

## Examples
```
Input: lists = [[1,4,5],[1,3,4],[2,6]]
Output: [1,1,2,3,4,4,5,6]

Input: lists = []
Output: []

Input: lists = [[]]
Output: []
```

## Intuition & Approach
This is "merge two sorted lists" (LC 21) generalized to `k` lists. The brute-force approach — merge list 1 with list 2, then merge the result with list 3, and so on — costs O(N*k) where N is the total number of nodes, because each merge step scans the whole accumulated result again.

There are two efficient strategies:

**1. Min-heap (priority queue).** At any moment, the smallest remaining unmerged value across all `k` lists must be the head of one of the `k` lists (since each list is individually sorted). Push the head node of each of the `k` lists into a min-heap keyed by value. Repeatedly pop the smallest, append it to the result, and if that node has a `next`, push `next` into the heap. This is O(N log k).

**2. Divide and conquer (pairwise merge).** Instead of merging lists one-by-one into a growing accumulator, merge them in pairs, then merge the results in pairs, recursively — like the merge step of merge sort. This also gives O(N log k) time but without the constant-factor overhead of heap operations, and O(log k) recursion stack instead of O(k) heap space.

The divide-and-conquer approach is generally preferred in an interview once you've shown the heap approach, because it demonstrates you can reason about the time complexity trade-off (heap push/pop overhead vs. straightforward pairwise merging).

## Pattern Recognition
> [!tip] Pattern: K-Way Merge
> Any time you need to merge `k` already-sorted sequences (lists, arrays, or streams) into one sorted output, think "min-heap of size k" or "pairwise divide and conquer." This pattern also shows up in external sorting and merging sorted log files from multiple sources.

## Algorithm Steps

**Heap approach:**
1. Push `(node.val, i, node)` for the head of each non-empty list into a min-heap (the index `i` breaks ties between equal values so nodes aren't compared directly).
2. Pop the smallest tuple, append its node to the result list.
3. If the popped node has a `next`, push `(next.val, i, next)` back into the heap.
4. Repeat until the heap is empty.

**Divide and conquer:**
1. If the list of lists is empty, return `None`. If it has one list, return it.
2. Split the array of lists into two halves.
3. Recursively merge each half into a single sorted list.
4. Merge the two resulting sorted lists using the standard two-list merge.

## Solution (Python)
```python
import heapq
from typing import List, Optional

class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

class Solution:
    def mergeKLists(self, lists: List[Optional[ListNode]]) -> Optional[ListNode]:
        # --- Min-heap approach ---
        heap = []
        for i, node in enumerate(lists):
            if node:
                # i as tiebreaker avoids comparing ListNode objects directly
                heapq.heappush(heap, (node.val, i, node))

        dummy = ListNode()
        tail = dummy
        while heap:
            val, i, node = heapq.heappop(heap)
            tail.next = node
            tail = tail.next
            if node.next:
                heapq.heappush(heap, (node.next.val, i, node.next))

        return dummy.next

    def mergeKListsDivideAndConquer(self, lists: List[Optional[ListNode]]) -> Optional[ListNode]:
        # --- Divide and conquer approach ---
        if not lists:
            return None
        if len(lists) == 1:
            return lists[0]

        mid = len(lists) // 2
        left = self.mergeKListsDivideAndConquer(lists[:mid])
        right = self.mergeKListsDivideAndConquer(lists[mid:])
        return self._merge_two(left, right)

    def _merge_two(self, a: Optional[ListNode], b: Optional[ListNode]) -> Optional[ListNode]:
        dummy = ListNode()
        tail = dummy
        while a and b:
            if a.val <= b.val:
                tail.next, a = a, a.next
            else:
                tail.next, b = b, b.next
            tail = tail.next
        tail.next = a if a else b
        return dummy.next
```

## Solution (Java)
```java
import java.util.Comparator;
import java.util.PriorityQueue;

class ListNode {
    int val;
    ListNode next;
    ListNode() {}
    ListNode(int val) { this.val = val; }
    ListNode(int val, ListNode next) { this.val = val; this.next = next; }
}

class Solution {
    // --- Min-heap approach ---
    public ListNode mergeKLists(ListNode[] lists) {
        PriorityQueue<ListNode> heap = new PriorityQueue<>(Comparator.comparingInt(node -> node.val));
        for (ListNode node : lists) {
            if (node != null) {
                heap.offer(node);
            }
        }

        ListNode dummy = new ListNode();
        ListNode tail = dummy;
        while (!heap.isEmpty()) {
            ListNode node = heap.poll();
            tail.next = node;
            tail = tail.next;
            if (node.next != null) {
                heap.offer(node.next);
            }
        }

        return dummy.next;
    }

    // --- Divide and conquer approach ---
    public ListNode mergeKListsDivideAndConquer(ListNode[] lists) {
        return divide(lists, 0, lists.length - 1);
    }

    private ListNode divide(ListNode[] lists, int lo, int hi) {
        if (lo > hi) return null;
        if (lo == hi) return lists[lo];

        int mid = lo + (hi - lo) / 2;
        ListNode left = divide(lists, lo, mid);
        ListNode right = divide(lists, mid + 1, hi);
        return mergeTwo(left, right);
    }

    private ListNode mergeTwo(ListNode a, ListNode b) {
        ListNode dummy = new ListNode();
        ListNode tail = dummy;
        while (a != null && b != null) {
            if (a.val <= b.val) {
                tail.next = a;
                a = a.next;
            } else {
                tail.next = b;
                b = b.next;
            }
            tail = tail.next;
        }
        tail.next = (a != null) ? a : b;
        return dummy.next;
    }
}
```

## Complexity Analysis
- **Time:** O(N log k) for both approaches, where N is the total number of nodes and k is the number of lists — each of the N nodes does O(log k) work (a heap push/pop, or being part of O(log k) merge levels).
- **Space:** O(k) for the heap approach (heap holds at most k elements); O(log k) recursion stack for divide and conquer. Neither counts the output list itself as extra space.

## Edge Cases
- `lists` is an empty array — return `None` immediately.
- `lists` contains only empty lists (e.g., `[[], []]`) — the heap ends up empty, result is `None`.
- A single list in `lists` — return it as-is.
- Duplicate values across different lists — the tiebreaker index in the heap tuple prevents comparing `ListNode` objects (which aren't orderable) when values tie.

## Common Mistakes
- Pushing raw `(node.val, node)` tuples into the heap without a tiebreaker — Python will attempt to compare the `ListNode` objects when values are equal, raising `TypeError`.
- Using the naive sequential merge (merge into accumulator one list at a time), which is correct but O(N*k) and will be flagged as suboptimal in an interview.
- Forgetting the dummy-head trick and fumbling the initial `tail` assignment.

## Related Problems
- [[K-Closest-Points-to-Origin]]
- [[Remove-Nth-Node-From-End-of-List]]
- [[Merge-Sorted-Array]]

## Follow-up / Variants
A natural follow-up: "what if the lists are too large to fit in memory / arrive as streams?" — the heap approach generalizes directly since it only ever holds `k` elements at once, making it the right answer for external or streaming merges, whereas divide-and-conquer assumes random access to all lists.
