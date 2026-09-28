---
title: "Real-World & System Design — Index"
aliases: []
tags: [moc, real-world, system-design]
---

# Real-World & System Design

This folder is different from the other 11 pattern folders. Those hold clean, well-specified problems sourced from an established problem bank (LeetCode, via Interview Solver's Addepar tag). This folder holds **crowd-sourced interview reports** (Glassdoor, Naukri/Code360) that describe a *topic* rather than a precise problem statement — often just a title and a source/date.

That has two consequences, spelled out here once instead of repeated in every note:

1. **Problem statements here are reconstructions**, clearly marked with a `> [!warning] Reconstructed from a crowd-sourced interview report` callout at the top of each note. They're built to match the one-line description as closely and plausibly as possible, but the real interviewer's actual schema/rules may differ. Treat the note as *practice for the shape of the question*, not a verbatim transcript.
2. **Not every note follows the standard algorithmic template.** Coding-flavored items ([[Process-Stock-Transactions-to-Generate-Portfolio]], [[Stock-Ledger-Reconciliation]], [[Portfolio-Data-Parsing-From-Strings]]) use the same section order as the rest of the vault (Problem Statement → Examples → Intuition → Pattern → Algorithm Steps → Solution (Python) → Solution (Java) → Complexity → Edge Cases → Common Mistakes → Related Problems → Follow-up). System-design items ([[Design-Messenger-Chat-App]]) use a **different template** entirely — no code solution, no Big-O — structured instead as: Problem Prompt → Requirements Gathering → Clarifying Questions to Ask → API Design → Data Model → High-Level Architecture → Real-Time Delivery & Scaling → Trade-offs & Follow-ups → Related Problems. See `AGENT.md` at the vault root for the full template spec of both flavors.

## What these problems have in common

Despite the domain flavor (portfolios, ledgers, chat apps), the underlying techniques are all things covered elsewhere in this vault:

- **Hashing/aggregation** ([[Process-Stock-Transactions-to-Generate-Portfolio]], [[Portfolio-Data-Parsing-From-Strings]]) — same `dict.get(key, default)` accumulator pattern as [[Ransom-Note]].
- **Multiset diffing** ([[Stock-Ledger-Reconciliation]]) — a `Counter`-based diff, structurally close to how [[Accounts-Merge]] groups records by a shared key.
- **API/data-model design** ([[Design-Messenger-Chat-App]]) — a different skill entirely (no algorithm to optimize), but the "ordering" and "fan-out" concerns echo the merge-multiple-streams logic in [[Merge-k-Sorted-Lists]].

The real skill this folder tests, more than any of the other 11, is **turning a vague prompt into a precise spec out loud** — ask clarifying questions before coding, the same way you'd have to before trusting any of the reconstructions above.

## Problems in this folder

- [[Process-Stock-Transactions-to-Generate-Portfolio]] — hashing/simulation, Medium
- [[Design-Messenger-Chat-App]] — system design, Hard
- [[Stock-Ledger-Reconciliation]] — multiset diffing, Hard
- [[Portfolio-Data-Parsing-From-Strings]] — string parsing + hashing, Medium

One additional reported item was **not** given a tutorial note: *"Unnamed technical task, tight time limit"* (Glassdoor, SWE, Edinburgh, Mar 2026) — the source gives no content to reconstruct from, so there's nothing to teach. It's logged here only so the source list isn't silently incomplete.
