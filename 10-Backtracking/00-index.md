---
title: "Backtracking — Pattern Primer"
tags: [moc, pattern-primer, backtracking]
---

# Backtracking

Backtracking is exhaustive search over a decision tree, with **pruning**: at each node you make a choice, recurse, and then undo the choice (backtrack) before trying the next option. It's the right tool whenever a problem asks for "all" solutions/combinations/arrangements satisfying some constraint, not just a single optimal count (that's more often DP).

## When to reach for this pattern
- Keywords: "all combinations," "all subsets," "all permutations," "all valid arrangements," "generate all ways to..."
- The search space is a tree of choices, and invalid branches can be detected and abandoned early (pruned) before exploring further.
- Contrast with DP: if the problem only wants a count or an optimal value (not the actual list of solutions), and overlapping subproblems exist, DP is usually more efficient. Backtracking is required when you need to *enumerate* the actual solutions.

## Generic template
```python
def backtrack(state, choices_so_far):
    if is_solution(state):
        record(choices_so_far)
        return
    for choice in get_candidates(state):
        if not is_valid(choice, state):
            continue
        make_choice(choice, state)
        backtrack(new_state, choices_so_far + [choice])
        undo_choice(choice, state)  # the "backtrack" step
```

## Key technique: controlling reuse and duplicates with a start index
When elements can be reused unlimited times, recurse with the same start index (`backtrack(i, ...)`). When each element can be used at most once, advance the start index (`backtrack(i+1, ...)`). This single index-passing decision is what separates several near-identical-looking problems (Combination Sum vs. Combination Sum II vs. Subsets). See [[Combination-Sum]] for the "unlimited reuse" version, worked in detail.

## Problems
- [[Combination-Sum]] — Medium

## Study tip
Always copy the current path (`path[:]` or `list(path)`) when recording a solution — appending the live mutable list reference is the single most common backtracking bug, since subsequent `pop()` calls silently corrupt previously recorded answers.
