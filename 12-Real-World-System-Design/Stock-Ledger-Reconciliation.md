---
title: "Stock Ledger Reconciliation"
aliases: []
tags: [real-world, hashing, diffing, hard]
source: "Naukri/Code360 campus hiring, HackerRank collab round, 60 min, reported Aug 2025"
leetcode_number: unknown
leetcode_difficulty: "Hard (real interview, not on LeetCode)"
pattern: "Multiset Diffing / Reconciliation"
status: not-started
---

# Stock Ledger Reconciliation

> [!warning] Reconstructed from a crowd-sourced interview report
> The source only says "Stock Ledger Reconciliation, HackerRank collab, 60 min, Hard" — no problem statement survived. The version below reconstructs a realistic finance-domain reconciliation task (comparing two independently-recorded transaction ledgers, e.g. your firm's internal ledger vs. a custodian/broker's ledger) because that's the standard meaning of "reconciliation" in this domain, and it's a natural "Hard, 60-minute, collaborative" problem shape. Confirm the actual schema and matching rules with the interviewer if you encounter this for real.

## Problem Statement
You maintain an internal ledger of stock trades. Once a day, your custodian bank sends their own independent record of the same trades. The two ledgers should agree, but due to timing, manual entry, or system bugs, they sometimes don't. Each ledger is a list of transaction records with fields: `trade_id` (may be missing/unreliable — don't assume it's always populated or consistent across systems), `date`, `ticker`, `side` (`BUY`/`SELL`), `quantity`, `price`.

Given the internal ledger and the custodian ledger, produce a reconciliation report identifying:
1. **Matched** — trades that appear in both ledgers with identical `(date, ticker, side, quantity, price)`.
2. **Only in internal** — trades in the internal ledger with no matching counterpart in the custodian ledger.
3. **Only in custodian** — trades in the custodian ledger with no matching counterpart in the internal ledger.
4. **Mismatched** — trades that clearly refer to the same underlying trade (same `date`, `ticker`, `side`) but disagree on `quantity` and/or `price` — these are the ones that most need a human's attention, and should be reported separately from "only in X", since they represent a data-entry error rather than a missing trade.

Duplicate identical trades on the same day for the same ticker/side are legitimate (e.g. buying GOOG twice in one day) — the reconciliation must be multiset-aware, not just set-aware, or it will silently under- or over-report discrepancies.

## Examples
```
Internal:
  [(date=1, ticker=GOOG, side=BUY, qty=10, price=150.0),
   (date=1, ticker=GOOG, side=BUY, qty=10, price=150.0),   # bought twice, legitimately
   (date=1, ticker=AAPL, side=SELL, qty=5,  price=190.0)]

Custodian:
  [(date=1, ticker=GOOG, side=BUY, qty=10, price=150.0),   # only one copy recorded — a real discrepancy
   (date=1, ticker=AAPL, side=SELL, qty=5,  price=195.0)]  # price mismatch

Report:
  matched:          [(1, GOOG, BUY, 10, 150.0)]            # one of the two GOOG buys matched
  only_in_internal:  [(1, GOOG, BUY, 10, 150.0)]            # the second, unmatched GOOG buy
  only_in_custodian: []
  mismatched:        [ (internal=(1,AAPL,SELL,5,190.0), custodian=(1,AAPL,SELL,5,195.0)) ]
```

## Intuition & Approach
The brute-force approach — compare every internal record against every custodian record — is O(n·m) and doesn't even solve the "duplicates" problem cleanly. The key realization is that this is a **multiset comparison** problem wearing a finance costume: it's the same shape as "given two arrays, find what's common, what's only in A, what's only in B" — except elements can legitimately repeat, and some non-identical elements should still be paired together as "the same trade with wrong details" rather than reported as two separate anomalies.

Split it into two passes:
1. **Exact matching first.** Build a `Counter` (multiset) of custodian records keyed by the full tuple `(date, ticker, side, quantity, price)`. Walk the internal records; for each one, if the exact tuple has remaining count in the custodian counter, it's a match — decrement and record it. Otherwise, it's unmatched so far.
2. **Fuzzy matching second, on what's left.** Among the internal records that didn't exact-match and the custodian records that didn't exact-match, group both sides by the "identity" key `(date, ticker, side)` (deliberately excluding `quantity`/`price`, since those are exactly the fields that might be wrong). Any internal/custodian pair sharing an identity key is a **mismatch** (pair them off, e.g. greedily or by remaining count); whatever's left over after that on each side is genuinely **only in internal** / **only in custodian**.

This two-pass structure is the crux of the problem: without splitting the identity key from the "must-match" fields, you can't distinguish a *missing* trade from a *corrupted* trade, and that distinction is the entire point of a reconciliation report.

## Pattern Recognition
> [!tip] Pattern: Multiset Diffing with a partial-key fallback
> When you need to diff two collections that may have duplicates, and you also need to distinguish "genuinely missing" from "present but wrong," don't just do a single hashmap pass on the full record. Do an exact pass keyed on everything, then a second pass on the leftovers keyed on a *subset* of fields (the "identity") to catch corrupted-but-present records.

## Algorithm Steps
1. Build `custodian_counter = Counter(custodian_records)` (exact full-tuple counts).
2. For each internal record: if `custodian_counter[record] > 0`, it's a **match** — decrement the counter and add to `matched`; else add it to `unmatched_internal`.
3. Whatever remains with count `> 0` in `custodian_counter` (expanded back into individual records) becomes `unmatched_custodian`.
4. Group `unmatched_internal` and `unmatched_custodian` by identity key `(date, ticker, side)` into two dicts of lists.
5. For each identity key present in both groups, pair records off (e.g. one-to-one in list order) as **mismatched**; any leftover records after pairing (identity key only on one side, or more records on one side than the other) become the final `only_in_internal` / `only_in_custodian`.
6. Return `{matched, only_in_internal, only_in_custodian, mismatched}`.

## Solution (Python)
```python
from collections import Counter, defaultdict
from typing import List, NamedTuple, Dict, Tuple

class Trade(NamedTuple):
    date: int
    ticker: str
    side: str
    quantity: int
    price: float

def reconcile(internal: List[Trade], custodian: List[Trade]) -> Dict:
    custodian_counter = Counter(custodian)
    matched = []
    unmatched_internal = []

    for trade in internal:
        if custodian_counter[trade] > 0:
            custodian_counter[trade] -= 1
            matched.append(trade)
        else:
            unmatched_internal.append(trade)

    unmatched_custodian = list(custodian_counter.elements())

    def by_identity(trades: List[Trade]) -> Dict[Tuple, List[Trade]]:
        groups = defaultdict(list)
        for t in trades:
            groups[(t.date, t.ticker, t.side)].append(t)
        return groups

    internal_groups = by_identity(unmatched_internal)
    custodian_groups = by_identity(unmatched_custodian)

    mismatched = []
    only_in_internal = []
    only_in_custodian = []

    all_identities = set(internal_groups) | set(custodian_groups)
    for identity in all_identities:
        i_list = internal_groups.get(identity, [])
        c_list = custodian_groups.get(identity, [])
        paired = min(len(i_list), len(c_list))
        for k in range(paired):
            mismatched.append({"internal": i_list[k], "custodian": c_list[k]})
        only_in_internal.extend(i_list[paired:])
        only_in_custodian.extend(c_list[paired:])

    return {
        "matched": matched,
        "only_in_internal": only_in_internal,
        "only_in_custodian": only_in_custodian,
        "mismatched": mismatched,
    }
```

## Solution (Java)
```java
import java.util.*;

class LedgerReconciler {

    record Trade(int date, String ticker, String side, int quantity, double price) {}
    record IdentityKey(int date, String ticker, String side) {}

    static class Report {
        List<Trade> matched = new ArrayList<>();
        List<Trade> onlyInInternal = new ArrayList<>();
        List<Trade> onlyInCustodian = new ArrayList<>();
        List<Trade[]> mismatched = new ArrayList<>(); // {internal, custodian}
    }

    Report reconcile(List<Trade> internal, List<Trade> custodian) {
        Map<Trade, Integer> custodianCounts = new HashMap<>();
        for (Trade t : custodian) custodianCounts.merge(t, 1, Integer::sum);

        Report report = new Report();
        List<Trade> unmatchedInternal = new ArrayList<>();

        for (Trade t : internal) {
            int count = custodianCounts.getOrDefault(t, 0);
            if (count > 0) {
                custodianCounts.put(t, count - 1);
                report.matched.add(t);
            } else {
                unmatchedInternal.add(t);
            }
        }

        List<Trade> unmatchedCustodian = new ArrayList<>();
        for (Map.Entry<Trade, Integer> e : custodianCounts.entrySet()) {
            for (int i = 0; i < e.getValue(); i++) unmatchedCustodian.add(e.getKey());
        }

        Map<IdentityKey, List<Trade>> internalGroups = groupByIdentity(unmatchedInternal);
        Map<IdentityKey, List<Trade>> custodianGroups = groupByIdentity(unmatchedCustodian);

        Set<IdentityKey> allIdentities = new HashSet<>();
        allIdentities.addAll(internalGroups.keySet());
        allIdentities.addAll(custodianGroups.keySet());

        for (IdentityKey identity : allIdentities) {
            List<Trade> iList = internalGroups.getOrDefault(identity, List.of());
            List<Trade> cList = custodianGroups.getOrDefault(identity, List.of());
            int paired = Math.min(iList.size(), cList.size());
            for (int k = 0; k < paired; k++) {
                report.mismatched.add(new Trade[]{iList.get(k), cList.get(k)});
            }
            report.onlyInInternal.addAll(iList.subList(paired, iList.size()));
            report.onlyInCustodian.addAll(cList.subList(paired, cList.size()));
        }

        return report;
    }

    private Map<IdentityKey, List<Trade>> groupByIdentity(List<Trade> trades) {
        Map<IdentityKey, List<Trade>> groups = new HashMap<>();
        for (Trade t : trades) {
            IdentityKey key = new IdentityKey(t.date(), t.ticker(), t.side());
            groups.computeIfAbsent(key, k -> new ArrayList<>()).add(t);
        }
        return groups;
    }
}
```

## Complexity Analysis
- **Time:** O(n + m) — where n and m are the internal and custodian ledger sizes: one pass to build the counter, one pass to match, and a final pass over the (typically much smaller) unmatched leftovers to group and pair by identity.
- **Space:** O(n + m) — for the counter and the grouped-by-identity maps.

## Edge Cases
- Both ledgers empty — should return an all-empty report, not error.
- A trade that appears 3 times in internal but only 2 times in custodian (partial duplicate mismatch) — the exact-match pass correctly matches 2 and leaves 1 as unmatched-internal, which then correctly falls through to `only_in_internal` if no identity-mismatched counterpart exists.
- Multiple distinct mismatches sharing the same identity key (e.g. two different GOOG BUYs on the same day, both with wrong prices) — the pairing step must not double-count or cross-pair them incorrectly; pairing in list order is a defensible tie-break, but call this assumption out explicitly since the "right" pairing is inherently ambiguous without a `trade_id`.
- Records with a reliable `trade_id` present — prefer keying on `trade_id` directly instead of the identity heuristic when it's available and trustworthy; the heuristic above is specifically for when it's not.

## Common Mistakes
- Treating this as a plain set difference (`set(internal) - set(custodian)`) and silently losing legitimate duplicate trades.
- Reporting every field-level mismatch as "only in internal" + "only in custodian" instead of recognizing it's the *same* trade with wrong data — this is the difference between a report that's actionable (fix this trade) and one that's noisy (two false anomalies for one real problem).
- Forgetting that a trade with a matching identity key but only, say, a `quantity` discrepancy (price agrees) is still a mismatch and should not be silently treated as fully matched — the exact-match pass already handles this correctly since it require the full tuple to match, but it's easy to break this while refactoring.

## Related Problems
- [[Process-Stock-Transactions-to-Generate-Portfolio]]
- [[Accounts-Merge]]
- [[Ransom-Note]]
- [[Merge-Sorted-Array]]

## Follow-up / Variants
A natural follow-up: instead of a one-time batch reconciliation, support **streaming** reconciliation where trades from both ledgers arrive continuously and discrepancies must be flagged with low latency — this pushes you toward maintaining the same counter/group structures incrementally rather than rebuilding them from scratch. Another common follow-up: tolerate small floating-point price differences (e.g. `abs(price_diff) < 0.01`) as "matched" rather than "mismatched," which requires switching the exact-match key away from raw float equality.
