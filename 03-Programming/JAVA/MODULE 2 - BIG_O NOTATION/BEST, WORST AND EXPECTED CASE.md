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

An algorithm's runtime can be described three ways, depending on **which input** you consider. The book uses **quick sort** as the example.

_Quick sort picks a "pivot" element, swaps values so smaller elements come before it and larger after (a "partial sort"), then recursively sorts the left and right sides._

|Case|Situation|Runtime|
|---|---|---|
|**Best**|All elements are equal; quick sort just traverses the array once (depends slightly on the implementation; some run fast on a sorted array)|`O(N)`|
|**Worst**|Pivot is repeatedly the biggest element (easy to trigger: pivot = first element and array sorted in reverse). Each recursion only shrinks the subarray by **one** element instead of halving it|`O(N²)`|
|**Expected**|Usually pivots are neither wonderful nor terrible; a bad pivot doesn't happen over and over|`O(N log N)`|

### Points to remember

- We **rarely discuss the best case**: you could special-case almost any algorithm for one input and claim `O(1)`. It's not a useful concept.
- For **most algorithms the worst case and expected case are the same**. When they differ, state both.

### Relationship between best/worst/expected and O/Θ/Ω

There is none. Candidates mix these up because both families talk about "higher", "lower" and "exactly right".

- Best / worst / expected describe the runtime **for particular inputs or scenarios**.
- Big O / Ω / Θ describe the **upper / lower / tight bound** of the runtime (for whichever case you chose).

So you can say "the worst case is `O(N²)`" (a bound for the worst-case scenario).

## Pitfalls

- **Mixing up "best/worst/expected" with "O/Ω/Θ".** The first family picks the _input scenario_; the second picks the _kind of bound_.
- **Quoting best case as the algorithm's runtime.** Any algorithm can be special-cased to `O(1)` for one input.

## Key terms

|Term|Definition|
|---|---|
|Best / worst / expected case|The runtime for the most favorable input, the least favorable input, and a typical input|

## Flashcards

- What are the three ways to describe an algorithm's runtime by scenario? :: Best case, worst case, and expected case #card
- What are quick sort's best, worst and expected cases? :: Best `O(N)` (e.g. all elements are equal), worst `O(N²)` (repeatedly bad pivot), expected `O(N log N)` #card
- When does quick sort hit its worst case? :: When the pivot is repeatedly the biggest (or smallest) element, e.g. pivot = first element on a reverse-sorted array #card
- Why do we rarely discuss best-case time? :: Any algorithm can be special-cased for one input to get `O(1)`, so it isn't a useful concept #card
- What is the relationship between best/worst/expected and O/Θ/Ω? :: None. The first family describes input scenarios; the second describes upper/lower/tight bounds #card

## Open questions

- [ ] How does quick sort's pivot choice (random, first, median-of-three) change how often the worst case happens?