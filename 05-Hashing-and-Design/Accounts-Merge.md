---
title: "Accounts Merge"
aliases: []
tags: [leetcode, union-find, graph, hashing, medium]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: 721
leetcode_difficulty: Medium
pattern: "Union-Find (Disjoint Set Union)"
status: not-started
---

# Accounts Merge

## Problem Statement
Given a list of accounts, where each account is a list `[name, email_1, email_2, ...]` — the first element is the account owner's name, and the rest are emails on that account — merge accounts that belong to the same person. Two accounts belong to the same person if they share at least one common email. Note the same name can belong to different people (name alone doesn't identify a person; shared email does).

Return the merged accounts in the format `[name, sorted_email_1, sorted_email_2, ...]`, where each merged account's emails are sorted lexicographically. The order of the returned list of accounts doesn't matter.

## Examples
```
Input:
accounts = [
  ["John", "johnsmith@mail.com", "john_newyork@mail.com"],
  ["John", "johnsmith@mail.com", "john00@mail.com"],
  ["Mary", "mary@mail.com"],
  ["John", "johnnybravo@mail.com"]
]
Output: [
  ["John", "john00@mail.com", "john_newyork@mail.com", "johnsmith@mail.com"],
  ["Mary", "mary@mail.com"],
  ["John", "johnnybravo@mail.com"]
]
```
The first and second "John" accounts share `johnsmith@mail.com`, so they merge. The third "John" (`johnnybravo@mail.com`) shares no email with the others, so it stays separate — same name, different person.

## Intuition & Approach
Think of every email as a node in a graph, and every account as declaring an edge (or a clique) between all the emails it lists. Two emails end up in the same merged account exactly when they're in the same connected component of this graph. This is the textbook signature of **Union-Find (Disjoint Set Union)**: we don't care about the shape of the connections, only which things end up grouped together, and we need to do this efficiently as we incrementally discover more connections (one account at a time).

An alternative is building an explicit adjacency graph (email -> set of emails it directly connects to) and running DFS/BFS from each unvisited email to find connected components. This works too, but Union-Find is more natural here because "merge these two groups" is exactly the `union` operation, and we never need to explicitly enumerate edges.

The steps:
1. For each account, union all of its emails together (e.g., union email[1] with email[2], email[1] with email[3], etc. — you only need to chain them, not connect every pair).
2. Keep a map from email -> account owner's name (any account containing that email will have the same name attached in the final answer, per problem's guarantee that if emails match, the name is consistent for that person).
3. After processing all accounts, group emails by their Union-Find root.
4. For each root/group, sort its emails and prepend the owner's name.

## Pattern Recognition
> [!tip] Pattern: Union-Find
> Whenever a problem describes "these items are connected if they share X" and asks you to group items into their connected components — especially when connections are discovered incrementally — reach for Union-Find with path compression and union by rank/size. It's near-O(1) amortized per operation and avoids building an explicit graph.

## Algorithm Steps
1. Initialize a `parent` map (or array, mapping email -> parent email) and a `email_to_name` map.
2. For each account, treat the first email as an "anchor." For every other email in the account, `union(anchor, email)` and record `email_to_name[email] = name`.
3. After all unions, build a map from `find(email)` (the root) to a list of all emails that share that root.
4. For each root, sort its email list, look up the owner name via `email_to_name[root]` (or any email in the group), and prepend it.
5. Return the list of `[name] + sorted_emails`.

## Solution (Python)
```python
from typing import List
from collections import defaultdict

class DSU:
    def __init__(self):
        self.parent = {}

    def find(self, x: str) -> str:
        if x not in self.parent:
            self.parent[x] = x
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])  # path compression
        return self.parent[x]

    def union(self, x: str, y: str) -> None:
        root_x, root_y = self.find(x), self.find(y)
        if root_x != root_y:
            self.parent[root_x] = root_y

class Solution:
    def accountsMerge(self, accounts: List[List[str]]) -> List[List[str]]:
        dsu = DSU()
        email_to_name = {}

        for account in accounts:
            name = account[0]
            emails = account[1:]
            first_email = emails[0]
            for email in emails:
                email_to_name[email] = name
                dsu.union(first_email, email)

        groups = defaultdict(list)
        for email in email_to_name:
            root = dsu.find(email)
            groups[root].append(email)

        result = []
        for root, emails in groups.items():
            name = email_to_name[root]
            result.append([name] + sorted(emails))

        return result
```

## Solution (Java)
```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

class Solution {
    private int[] parent;
    private int[] rank;

    private int find(int x) {
        if (parent[x] != x) {
            parent[x] = find(parent[x]); // path compression
        }
        return parent[x];
    }

    private void union(int x, int y) {
        int rootX = find(x), rootY = find(y);
        if (rootX == rootY) return;
        if (rank[rootX] < rank[rootY]) {
            parent[rootX] = rootY;
        } else if (rank[rootX] > rank[rootY]) {
            parent[rootY] = rootX;
        } else {
            parent[rootY] = rootX;
            rank[rootX]++;
        }
    }

    public List<List<String>> accountsMerge(List<List<String>> accounts) {
        Map<String, Integer> emailToId = new HashMap<>();
        Map<String, String> emailToName = new HashMap<>();
        int nextId = 0;

        for (List<String> account : accounts) {
            String name = account.get(0);
            for (int i = 1; i < account.size(); i++) {
                String email = account.get(i);
                emailToName.put(email, name);
                if (!emailToId.containsKey(email)) {
                    emailToId.put(email, nextId++);
                }
            }
        }

        parent = new int[nextId];
        rank = new int[nextId];
        for (int i = 0; i < nextId; i++) {
            parent[i] = i;
        }

        for (List<String> account : accounts) {
            int firstId = emailToId.get(account.get(1));
            for (int i = 2; i < account.size(); i++) {
                union(firstId, emailToId.get(account.get(i)));
            }
        }

        Map<Integer, List<String>> groups = new HashMap<>();
        for (String email : emailToId.keySet()) {
            int root = find(emailToId.get(email));
            groups.computeIfAbsent(root, k -> new ArrayList<>()).add(email);
        }

        List<List<String>> result = new ArrayList<>();
        for (List<String> emails : groups.values()) {
            Collections.sort(emails);
            String name = emailToName.get(emails.get(0));
            List<String> merged = new ArrayList<>();
            merged.add(name);
            merged.addAll(emails);
            result.add(merged);
        }

        return result;
    }
}
```

## Complexity Analysis
- **Time:** O(N * K * α(N*K)) where N is the number of accounts and K is the average number of emails per account; α is the inverse Ackermann function (effectively constant) from path compression + union by attaching roots. Sorting the emails within each group adds O(E log E) total across all groups, where E is the total number of distinct emails.
- **Space:** O(N * K) for the parent map, email-to-name map, and grouping structure.

## Edge Cases
- An account with only one email (no emails to union with each other) — still needs to appear in the final output.
- Two people with the exact same name but disjoint emails — must remain separate accounts (the problem explicitly calls this out).
- An email appearing in more than two accounts (chains of merges, e.g., account A shares with B, B shares with C — all three must merge into one group).

## Common Mistakes
- Using the account's declared `name` field to decide whether to merge (wrong — merging is based purely on shared emails, name is just a display label).
- Forgetting path compression, which degrades `find` to O(N) in the worst case for skewed union chains.
- Unioning every pair of emails within an account (O(K²) per account) instead of just chaining each email to the first one (O(K) per account) — unnecessary work, though not incorrect.

## Related Problems
- [[Number-of-Islands]]
- [[Word-Ladder]]
- [[Shortest-Word-Distance-II]]

## Follow-up / Variants
A common follow-up is "what if accounts stream in one at a time and you need the merged result after each insertion?" — Union-Find handles this naturally since it's an online algorithm; you'd just re-derive the grouped output after each union rather than rebuilding from scratch.
