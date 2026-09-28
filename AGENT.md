# AGENT.md

Guidance for AI agents (and humans) working in this repository.

## What this repo is

An Obsidian vault of LeetCode-style tutorial notes for the 33 problems tagged
"Addepar" on [Interview Solver](https://interviewsolver.com/interview-questions/addepar).
It is a learning resource, not a buildable project — there is no build step,
package manifest, or test suite. "Correctness" here means: problem statements
match the real LeetCode problem, solutions are algorithmically correct and
match their stated complexity, and Obsidian wikilinks resolve.

Entry points: [[00-MOC]] (map of content), [[Patterns-Cheatsheet]], [[Study-Tracker]].

## Structure

- Root: `00-MOC.md`, `Patterns-Cheatsheet.md`, `Study-Tracker.md` — vault-wide navigation, not per-problem content.
- `NN-Pattern-Name/` — one folder per algorithmic pattern (e.g. `03-Sliding-Window/`). Each contains:
  - `00-index.md` — a primer on the pattern: when to recognize it, generic technique, links to every problem note in that folder.
  - One `.md` file per problem, named after the problem title in kebab-case (e.g. `Daily-Temperatures.md`).
- `.obsidian/` — vault config (`app.json`, `appearance.json`, `core-plugins.json` are tracked; `workspace.json` and similar per-machine state are gitignored).

A problem can only belong to one pattern folder even if it touches multiple
techniques (e.g. `Merge-k-Sorted-Lists` lives under Linked Lists but is
heap-heavy) — use `Related Problems` wikilinks for cross-pattern connections
rather than duplicating the file.

## Problem note template

Every problem note follows this exact section order — preserve it when
editing or adding notes:

```
---
title: "<Problem Title>"
aliases: []
tags: [leetcode, <pattern-tag-kebab-case>, <difficulty-lowercase>]
source: "https://interviewsolver.com/interview-questions/addepar"
leetcode_number: <number or "unknown">
leetcode_difficulty: <Easy|Medium|Hard>
pattern: "<Pattern Name>"
status: not-started
---

# <Problem Title>

## Problem Statement
## Examples
## Intuition & Approach
## Pattern Recognition        (Obsidian callout: > [!tip] Pattern: ...)
## Algorithm Steps
## Solution (Python)
## Solution (Java)
## Complexity Analysis
## Edge Cases
## Common Mistakes
## Related Problems           (wikilinks only, see below)
## Follow-up / Variants
```

Some notes legitimately show two approaches (e.g. `Longest-Increasing-Subsequence`
has O(n²) DP and O(n log n) patience sorting; `Merge-k-Sorted-Lists` has heap
and divide-and-conquer). In those cases use `###` subheadings inside
`Solution (Python)` / `Solution (Java)` rather than duplicating top-level
sections, and keep the Python and Java sections showing the same set of
approaches in the same order.

Java solutions must implement the *identical* algorithm/complexity as the
Python solution in the same note — they are a second language, not a second
approach. Use standard LeetCode Java conventions (`class Solution { ... }`,
official method signatures where known, `ListNode`/`TreeNode`/problem-specific
`Node` boilerplate classes defined above the `Solution` class when needed).

## Cross-linking rules

- `Related Problems` entries must be `[[Wikilink]]`s using the exact filename
  (no `.md`) of another note that actually exists in this vault — never
  invent a link to a problem outside the 33.
- Link based on genuinely shared technique or theme, not proximity. 2-4 links
  per note is normal; don't force irrelevant ones.
- Wikilinks inside fenced code blocks or inline code (e.g. example I/O like
  `[[1,3]]` for a list-of-lists) are not real links — don't "fix" these.

## Verifying changes

There's no CI. Before considering an edit done, sanity-check it by hand:

```bash
# every problem note has exactly one Java section
grep -L "## Solution (Java)" **/*.md

# code fences balanced (even count per file)
for f in $(find . -name "*.md"); do c=$(grep -c '^```' "$f"); [ $((c % 2)) -ne 0 ] && echo "$f: $c (BROKEN)"; done

# wikilinks resolve to real files (see conversation history for the fuller
# Python checker that also strips code blocks before scanning)
```

## Known caveats already flagged in-vault

- `Walls-and-Gates` and `Alien-Dictionary` are premium/locked LeetCode
  problems — noted in `09-Graphs-and-BFS-DFS/00-index.md` and their own notes.
- `Digit-Operations-to-Make-Two-Integers-Equal` is a newer/less-standard
  problem; its note explicitly flags uncertainty about exact constraint
  wording rather than asserting unverified specifics, and teaches the general
  Dijkstra/BFS-over-implicit-state-graph technique instead. Preserve that
  caveat if you touch this file — don't quietly firm up details that haven't
  been verified against the current LeetCode listing.

## Adding a new problem

1. Pick (or create) the right pattern folder.
2. Copy the section order from the template above — don't skip sections.
3. Add it to that folder's `00-index.md` list, to `00-MOC.md`'s full list and
   folder link, and to `Study-Tracker.md`'s checklist.
4. Add 2-4 genuine `Related Problems` links, and consider whether any
   *existing* notes should now link back to the new one.
