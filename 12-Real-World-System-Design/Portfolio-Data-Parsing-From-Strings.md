---
title: "Portfolio Data as Space-Separated Strings"
aliases: []
tags: [real-world, string-parsing, hashing, medium]
source: "Glassdoor (SWE), reported Jun 2026"
leetcode_number: unknown
leetcode_difficulty: "Medium (real interview, not on LeetCode)"
pattern: "Parsing + Hashing + Aggregation"
status: not-started
---

# Portfolio Data as Space-Separated Strings

> [!warning] Reconstructed from a crowd-sourced interview report
> The source description is just "Portfolio data as space-separated strings ('GOOG 10') — parse + hashmap + math." No exact problem statement survived. The version below reconstructs a plausible task matching that one-line description (parse ticker/quantity pairs, aggregate duplicates, then compute value against a price sheet), since that's the natural thing to build from raw "TICKER QUANTITY" strings. Confirm the real schema and required output with the interviewer if you encounter this for real.

## Problem Statement
You're given a portfolio as a list of raw strings, each formatted as `"<TICKER> <QUANTITY>"`, e.g. `"GOOG 10"`. The same ticker may appear more than once across the list (representing multiple lots bought at different times) and should be combined into a single aggregated position. You're also given a price sheet mapping ticker → current price per share.

Parse the raw strings, aggregate quantities per ticker, and compute:
1. `holdings`: a map of ticker → total aggregated quantity.
2. `value_by_ticker`: a map of ticker → `quantity * price` for each held ticker.
3. `total_value`: the sum of all `value_by_ticker` values — the total portfolio value.

A ticker present in `holdings` but missing from the price sheet should be treated as a data problem to flag, not silently ignored or crashed on (see Edge Cases).

## Examples
```
Input:
  lines = ["GOOG 10", "AAPL 5", "GOOG 3", "MSFT 0"]
  prices = {"GOOG": 150.0, "AAPL": 190.0, "MSFT": 300.0}

Output:
  holdings = {"GOOG": 13, "AAPL": 5, "MSFT": 0}
  value_by_ticker = {"GOOG": 1950.0, "AAPL": 950.0, "MSFT": 0.0}
  total_value = 2900.0
```

## Intuition & Approach
Strip away the finance flavor and this is a two-stage pipeline: **parse-and-aggregate**, then **join-and-reduce**. Stage one is standard string parsing — split each line on whitespace, convert the second token to an integer, and accumulate into a hashmap keyed by the first token (ticker). Stage two is a hashmap join: for every aggregated ticker, look up its price and multiply, then sum across all tickers for the total.

The interesting design decisions aren't algorithmic (this is all O(n) hashmap work) — they're about **defensive parsing**: what happens with malformed lines (extra whitespace, missing quantity, non-numeric quantity), and what happens when the two data sources (holdings vs. price sheet) don't perfectly line up (a ticker with no price, or a price sheet entry never held). A solution that only handles the happy path looks fine on the example but falls over on real, messy input — which is exactly what this kind of "parse real-world data" question is testing.

## Pattern Recognition
> [!tip] Pattern: Parse → Aggregate → Join
> Whenever the prompt is "parse these raw records, combine duplicates, then compute something against a second reference dataset," structure your solution as three explicit, separately-testable steps (parse, aggregate via hashmap, join via hashmap lookup) rather than one tangled loop — it makes the defensive-parsing edge cases much easier to reason about and test in isolation.

## Algorithm Steps
1. Initialize `holdings = {}` (ticker → quantity).
2. For each raw line: split on whitespace, validate you got exactly two tokens, parse the second as an integer; on failure, record/skip per your error-handling policy (state it explicitly). Otherwise `holdings[ticker] = holdings.get(ticker, 0) + quantity`.
3. Initialize `value_by_ticker = {}` and `total_value = 0.0`.
4. For each `(ticker, qty)` in `holdings`: look up `price = prices.get(ticker)`; if missing, flag it (see Edge Cases) and skip from value calculations; otherwise `value_by_ticker[ticker] = qty * price` and add it to `total_value`.
5. Return `(holdings, value_by_ticker, total_value)`.

## Solution (Python)
```python
from typing import List, Dict, Tuple

def parse_portfolio(
    lines: List[str], prices: Dict[str, float]
) -> Tuple[Dict[str, int], Dict[str, float], float]:
    holdings: Dict[str, int] = {}

    for line in lines:
        parts = line.split()
        if len(parts) != 2:
            continue  # malformed line — policy: skip rather than crash
        ticker, qty_str = parts
        try:
            qty = int(qty_str)
        except ValueError:
            continue  # non-numeric quantity — same skip policy
        holdings[ticker] = holdings.get(ticker, 0) + qty

    value_by_ticker: Dict[str, float] = {}
    total_value = 0.0
    missing_prices = []

    for ticker, qty in holdings.items():
        if ticker not in prices:
            missing_prices.append(ticker)
            continue
        value = qty * prices[ticker]
        value_by_ticker[ticker] = value
        total_value += value

    if missing_prices:
        # In a real system, log/raise on this rather than silently drop it —
        # a missing price means total_value is understated.
        pass

    return holdings, value_by_ticker, total_value
```

## Solution (Java)
```java
import java.util.HashMap;
import java.util.List;
import java.util.Map;

class PortfolioParser {

    record ParseResult(
        Map<String, Integer> holdings,
        Map<String, Double> valueByTicker,
        double totalValue,
        List<String> missingPrices
    ) {}

    ParseResult parsePortfolio(List<String> lines, Map<String, Double> prices) {
        Map<String, Integer> holdings = new HashMap<>();

        for (String line : lines) {
            String[] parts = line.trim().split("\\s+");
            if (parts.length != 2) continue; // malformed line — skip
            String ticker = parts[0];
            int qty;
            try {
                qty = Integer.parseInt(parts[1]);
            } catch (NumberFormatException e) {
                continue; // non-numeric quantity — skip
            }
            holdings.merge(ticker, qty, Integer::sum);
        }

        Map<String, Double> valueByTicker = new HashMap<>();
        double totalValue = 0.0;
        java.util.List<String> missingPrices = new java.util.ArrayList<>();

        for (Map.Entry<String, Integer> entry : holdings.entrySet()) {
            String ticker = entry.getKey();
            Double price = prices.get(ticker);
            if (price == null) {
                missingPrices.add(ticker);
                continue;
            }
            double value = entry.getValue() * price;
            valueByTicker.put(ticker, value);
            totalValue += value;
        }

        return new ParseResult(holdings, valueByTicker, totalValue, missingPrices);
    }
}
```

## Complexity Analysis
- **Time:** O(n + u) — n lines to parse (O(1) work each), u distinct tickers to join against the price sheet.
- **Space:** O(u) — the holdings and value maps, sized by distinct ticker count.

## Edge Cases
- A line with extra/irregular whitespace (`"GOOG  10"`, trailing spaces) — `split()` on whitespace with no argument (Python) or `split("\\s+")` after `trim()` (Java) handles this; a naive single-space split would not.
- A ticker aggregating to exactly `0` shares (fully sold, or a `"MSFT 0"` line) — should still appear in `holdings` with value `0.0`, not be silently dropped (contrast with [[Process-Stock-Transactions-to-Generate-Portfolio]], where a *derived* zero position is dropped — here the zero is directly stated input data, so keep it).
- A ticker in `holdings` with no entry in `prices` — must not crash the whole computation (a `KeyError`/`NullPointerException` on one bad lookup shouldn't take down the report); decide and state whether you skip it, default its price to 0, or fail loudly.
- A malformed line (wrong token count, non-numeric quantity) — decide and state a policy (skip-and-continue vs. collect-and-report-errors vs. fail-fast) rather than letting an exception propagate unhandled.
- Negative quantities in the raw input (representing a short position, or a data error) — clarify with the interviewer whether these are valid.

## Common Mistakes
- Splitting on a literal single space (`line.split(" ")`) instead of arbitrary whitespace, which breaks on double-spaced or tab-separated input.
- Letting a missing price or malformed line throw an unhandled exception that aborts processing of the entire (otherwise-valid) remaining input.
- Conflating "ticker not in `holdings`" with "ticker held with zero shares" — they should be handled the same in the final report (both mean no value contribution), but the *aggregation* step must still distinguish "not yet seen" (start at 0 and add) from "explicitly zero" only in the sense of not crashing on first encounter — using `.get(ticker, 0)` / `merge(..., Integer::sum)` naturally avoids this bug.

## Related Problems
- [[Process-Stock-Transactions-to-Generate-Portfolio]]
- [[Ransom-Note]]
- [[Valid-Number]]
- [[Reverse-Words-in-a-String]]

## Follow-up / Variants
A natural follow-up: the price sheet itself is also given as raw strings (`"GOOG 150.00"`) that need parsing with the same defensive rules, turning this into two parse-and-validate pipelines feeding one join. Another: support a "cost basis" input (average purchase price per lot) alongside quantity, and compute unrealized gain/loss (`(current_price - cost_basis) * quantity`) per ticker in addition to raw value.
