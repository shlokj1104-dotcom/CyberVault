---
title: BEST, WORST AND EXPECTED CASE
date: 2026-10-09
language: Java
phase: phase-1
tags:
links: []
status: learning
---
# 03 · Best, Worst and Expected Case

> **Summary:** An algorithm's runtime depends on which input you consider. Best, worst and expected cases pick the input scenario; O, Ω and Θ pick the kind of bound. The two families are unrelated.

## Notes

An algorithm's runtime can be described three ways, depending on **which input** you consider. This version uses **bubble sort** as the example.

_Bubble sort walks through the array repeatedly, swapping adjacent elements that are out of order. After each pass, the largest unsorted element has "bubbled" to the end. A common optimisation stops early if a full pass makes no swaps._

```
repeat:
    swapped = false
    for i from 0 to N-2:
        if a[i] > a[i+1]:
            swap a[i] and a[i+1]
            swapped = true
until swapped is false
```

|Case|Situation|Runtime|
|---|---|---|
|**Best**|Array is already sorted. One pass, zero swaps, then the early exit fires|`O(N)`|
|**Worst**|Array is sorted in reverse. Each pass moves only one element into its final place, so about `N-1` passes, each with up to `N` comparisons|`O(N²)`|
|**Expected**|Random order. About half of all pairs are out of order, giving roughly `N²/4` swaps and about `N` passes|`O(N²)`|

### Points to remember

- The best case of `O(N)` **depends on the early-exit optimisation**. Without it, even a sorted array needs `O(N²)` work, because the algorithm keeps making passes it doesn't need.
- We **rarely discuss the best case**: you could special-case almost any algorithm for one input and claim `O(1)`. It's not a useful concept.
- For **bubble sort, the worst case and expected case are the same** (`O(N²)`). This is the common situation. When they differ, state both.

### Relationship between best/worst/expected and O/Θ/Ω

There is none. Candidates mix these up because both families talk about "higher", "lower" and "exactly right".

- Best / worst / expected describe the runtime **for particular inputs or scenarios**.
- Big O / Ω / Θ describe the **upper / lower / tight bound** of the runtime (for whichever case you chose).

So you can say "the worst case is `O(N²)`" (a bound for the worst-case scenario).

## Pitfalls

- **Mixing up "best/worst/expected" with "O/Ω/Θ".** The first family picks the _input scenario_; the second picks the _kind of bound_.
- **Quoting best case as the algorithm's runtime.** Any algorithm can be special-cased to `O(1)` for one input.
- **Forgetting the early-exit flag when analysing best case.** Without it, bubble sort's best case is `O(N²)`, not `O(N)`.

## Key terms

|Term|Definition|
|---|---|
|Best / worst / expected case|The runtime for the most favorable input, the least favorable input, and a typical input|
|Early exit|An optimisation that stops the algorithm once a pass finds nothing left to do|

## Flashcards

- What are the three ways to describe an algorithm's runtime by scenario? :: Best case, worst case, and expected case #card
- What are bubble sort's best, worst and expected cases? :: Best `O(N)` (already sorted, with early exit), worst `O(N²)` (reverse sorted), expected `O(N²)` (random order) #card
- Why is bubble sort's best case `O(N)` and not `O(N²)`? :: With an early-exit flag, a sorted array needs only one pass with zero swaps, so the algorithm stops after `N-1` comparisons #card
- Why is bubble sort's worst case the same as its expected case? :: Random input still leaves about half the pairs out of order, so the number of passes and comparisons stays quadratic #card
- Why do we rarely discuss best-case time? :: Any algorithm can be special-cased for one input to get `O(1)`, so it isn't a useful concept #card
- What is the relationship between best/worst/expected and O/Θ/Ω? :: None. The first family describes input scenarios; the second describes upper/lower/tight bounds #card

## Open questions

- [ ] Does the choice of early-exit condition change the best case, or only the constant factor?
- [ ] How does binary search's best case (target at the middle, `O(1)`) compare with bubble sort's best case as an example of why best case is rarely useful?


→ Next: [[SPACE COMPEXITY]]