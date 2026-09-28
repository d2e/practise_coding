---
title: "Process Stock Transactions to Generate Portfolio"
aliases: []
tags: [real-world, hashing, simulation, medium]
source: "Glassdoor (Sr SWE), reported Nov 2025"
leetcode_number: unknown
leetcode_difficulty: "Medium (real interview, not on LeetCode)"
pattern: "Stateful Stream Processing / Hashing"
status: not-started
---

# Process Stock Transactions to Generate Portfolio

> [!warning] Reconstructed from a crowd-sourced interview report
> This problem comes from a one-line Glassdoor interview summary, not an official problem bank. The exact input format, transaction types, and edge-case rules the real interviewer used are unknown. The version below is a reasonable, self-consistent reconstruction that exercises the same skill (stateful processing of a transaction stream into a hashmap). If you actually get this question, spend the first two minutes nailing down the real schema and rules with the interviewer instead of assuming this version.

## Problem Statement
You're given an ordered list of transactions for a single brokerage account, applied in the order given. Each transaction is one of:
- `("BUY", ticker, quantity, price)` — purchase `quantity` shares of `ticker` at `price` per share; cash decreases by `quantity * price`, holdings for `ticker` increase by `quantity`.
- `("SELL", ticker, quantity, price)` — sell `quantity` shares of `ticker` at `price` per share; cash increases by `quantity * price`, holdings for `ticker` decrease by `quantity`.
- `("CASH", amount)` — a deposit (`amount > 0`) or withdrawal (`amount < 0`) with no effect on holdings.

Starting from an empty portfolio (`cash = 0`, no holdings), process every transaction in order and return the final state: the cash balance, and a mapping of ticker → shares held for every ticker with a non-zero position.

Constraints (assumed): transaction count up to ~10^5; quantities and prices are non-negative; a `SELL` is never given for more shares than currently held (garbage-in-garbage-out — assume the feed is valid, but see *Edge Cases* for what to do if you can't assume that).

## Examples
```
Input:
[
  ("CASH", 10000),
  ("BUY", "GOOG", 10, 150.0),
  ("BUY", "AAPL", 5, 190.0),
  ("SELL", "GOOG", 4, 160.0),
  ("CASH", -500),
]

Output:
cash = 10000 - 1500 - 950 + 640 - 500 = 7690.0
holdings = {"GOOG": 6, "AAPL": 5}
```

## Intuition & Approach
This isn't a search/optimization problem — it's a simulation. There is exactly one correct way to process each transaction, in order, and the entire "algorithm" is: pick the right update rule per transaction type and apply it to two pieces of running state (a scalar `cash` and a `ticker -> quantity` hashmap). The only design decisions that matter are (1) how you dispatch on transaction type cleanly, and (2) how you keep the output tidy — specifically, dropping tickers whose position nets to exactly zero, since "0 shares of GOOG" is not a real holding.

The naive trap is to treat this as more complex than it is (e.g. reaching for a class hierarchy or an event-sourcing framework) when a single pass with a dict and a running total is the entire solution. The one genuine subtlety is deciding what to do when a ticker's quantity hits zero after a sell (remove the key vs. leave it at 0) and what to do with malformed/adversarial input (oversell, negative price) if you can't assume a clean feed.

## Pattern Recognition
> [!tip] Pattern: Stateful Stream Processing / Hashing
> Whenever a problem says "process these events **in order**, maintain running state, return the final state," reach for a single linear pass with a hashmap (or a few scalars) as the accumulator — no sorting, no recursion, no extra data structure is needed beyond what holds the current state.

## Algorithm Steps
1. Initialize `cash = 0.0` and `holdings = {}` (ticker → quantity).
2. For each transaction, dispatch on its type:
   - `BUY ticker qty price`: `cash -= qty * price`; `holdings[ticker] = holdings.get(ticker, 0) + qty`.
   - `SELL ticker qty price`: `cash += qty * price`; `holdings[ticker] = holdings.get(ticker, 0) - qty`.
   - `CASH amount`: `cash += amount`.
3. After processing all transactions, remove any ticker from `holdings` whose quantity is exactly `0` (or negative, if you're defending against a bad feed — see Edge Cases).
4. Return `(cash, holdings)`.

## Solution (Python)
```python
from typing import List, Tuple, Dict, Union

Transaction = Union[
    Tuple[str, str, int, float],  # ("BUY"/"SELL", ticker, qty, price)
    Tuple[str, float],            # ("CASH", amount)
]

def process_transactions(transactions: List[Transaction]) -> Tuple[float, Dict[str, int]]:
    cash = 0.0
    holdings: Dict[str, int] = {}

    for txn in transactions:
        kind = txn[0]
        if kind == "BUY":
            _, ticker, qty, price = txn
            cash -= qty * price
            holdings[ticker] = holdings.get(ticker, 0) + qty
        elif kind == "SELL":
            _, ticker, qty, price = txn
            cash += qty * price
            holdings[ticker] = holdings.get(ticker, 0) - qty
        elif kind == "CASH":
            _, amount = txn
            cash += amount
        else:
            raise ValueError(f"Unknown transaction type: {kind}")

    # Drop zeroed-out (or invalid negative) positions — they aren't real holdings.
    holdings = {t: q for t, q in holdings.items() if q > 0}
    return cash, holdings
```

## Solution (Java)
```java
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;

class PortfolioProcessor {

    static class Result {
        double cash;
        Map<String, Integer> holdings;
        Result(double cash, Map<String, Integer> holdings) {
            this.cash = cash;
            this.holdings = holdings;
        }
    }

    // Each transaction is represented as Object[]:
    //   {"BUY"/"SELL", ticker (String), qty (Integer), price (Double)}
    //   {"CASH", amount (Double)}
    Result processTransactions(List<Object[]> transactions) {
        double cash = 0.0;
        Map<String, Integer> holdings = new LinkedHashMap<>();

        for (Object[] txn : transactions) {
            String kind = (String) txn[0];
            switch (kind) {
                case "BUY": {
                    String ticker = (String) txn[1];
                    int qty = (Integer) txn[2];
                    double price = (Double) txn[3];
                    cash -= qty * price;
                    holdings.merge(ticker, qty, Integer::sum);
                    break;
                }
                case "SELL": {
                    String ticker = (String) txn[1];
                    int qty = (Integer) txn[2];
                    double price = (Double) txn[3];
                    cash += qty * price;
                    holdings.merge(ticker, -qty, Integer::sum);
                    break;
                }
                case "CASH": {
                    double amount = (Double) txn[1];
                    cash += amount;
                    break;
                }
                default:
                    throw new IllegalArgumentException("Unknown transaction type: " + kind);
            }
        }

        // Drop zeroed-out (or invalid negative) positions.
        holdings.values().removeIf(q -> q <= 0);
        return new Result(cash, holdings);
    }
}
```

## Complexity Analysis
- **Time:** O(n) — one constant-time hashmap operation per transaction, n transactions total.
- **Space:** O(u) — where u is the number of distinct tickers ever bought, for the holdings map.

## Edge Cases
- A `SELL` that would drive a ticker's quantity negative (oversell) — decide up front whether to trust the feed (assume it can't happen), clamp at zero, or raise/flag it; state your assumption explicitly to the interviewer.
- A ticker bought and fully sold back to exactly 0 shares — must be excluded from the final holdings output, not returned as `{"GOOG": 0}`.
- An empty transaction list — should return `(0.0, {})`.
- `CASH` withdrawals that would drive cash negative (overdraft) — same "state your assumption" treatment as oversell.
- Floating-point price arithmetic accumulating rounding error over many transactions — in a real system you'd use fixed-point/decimal arithmetic (e.g. integer cents) rather than raw floats; worth mentioning even if you code the example with floats.

## Common Mistakes
- Leaving zero-quantity tickers in the output map instead of pruning them.
- Recomputing holdings from scratch per transaction (e.g. re-scanning the whole list) instead of maintaining running state in one pass — turns an O(n) problem into O(n²).
- Mutating a shared/passed-in dict across calls instead of starting from a fresh empty state, if this is wrapped in a class with a `reset()` expectation.

## Related Problems
- [[Portfolio-Data-Parsing-From-Strings]]
- [[Stock-Ledger-Reconciliation]]
- [[Ransom-Note]]
- [[Ways-to-Make-a-Fair-Array]]

## Follow-up / Variants
A common follow-up: support querying the portfolio's state *as of* an arbitrary point in the transaction history (not just the final state) — this turns the problem into building a prefix-state index (snapshot every k transactions, or store cumulative deltas per ticker) so a query doesn't require re-processing from the start. Another variant: transactions arrive out of order with timestamps and must be sorted/replayed correctly, which turns this into a sort-then-simulate problem.
