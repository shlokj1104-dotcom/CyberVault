---
title: LOGARITHM RUNTIME
date: 2026-10-09
language: Java
phase: phase-1
tags:
links: []
status: learning
---
# 08 · Logarithmic Runtimes

> **Summary:** `O(log N)` appears whenever the problem space is halved at each step. The number of halvings needed to reach 1 is `log₂ N`, and the base of the log does not matter for big O.

## Notes

`O(log N)` appears constantly. Where does it come from? Take **binary search**: find `x` in a sorted `N`-element array. Compare `x` to the midpoint; if smaller, search the left half; if bigger, search the right half.

```
search 9 within {1, 5, 8, 9, 11, 13, 15, 19, 21}
    compare 9 to 11 -> smaller.
    search 9 within {1, 5, 8, 9, 11}
        compare 9 to 8 -> bigger
        search 9 within {9, 11}
            compare 9 to 9
            return
```

We start with `N` elements, after one step `N/2`, then `N/4`, ... until we find the value or reach 1 element. **The runtime is how many times we can divide `N` by 2 until it becomes 1.**

```
N = 16
N = 8     /* divide by 2 */
N = 4     /* divide by 2 */
N = 2     /* divide by 2 */
N = 1     /* divide by 2 */
```

Run it backwards: how many times can we multiply 1 by 2 until we reach `N`? That is the `k` in **`2ᵏ = N`**, and that is exactly what a logarithm is:

```
2⁴ = 16   →   log₂ 16 = 4
log₂ N = k   →   2ᵏ = N
```

**Takeaway to carry with you:** when you see a problem where the **number of elements in the problem space gets halved each time**, it will likely be `O(log N)`.

The same reason explains why **finding an element in a balanced binary search tree is `O(log N)`**: with each comparison we go left or right, and half the nodes are on each side, so the problem space halves.

### What's the base of the log?

It doesn't matter for big O. Logs of different bases differ only by a constant factor, and we drop constants. (The book has a "Bases of Logs" section later for the proof.)

## Key terms

|Term|Definition|
|---|---|
|Logarithm (`log₂ N`)|The number of times you can halve `N` to reach 1; the `k` in `2ᵏ = N`|

## Flashcards

- Where does `O(log N)` come from? :: Repeatedly halving the problem: the number of times you can divide `N` by 2 to reach 1 is `log₂ N` #card
- What does `log₂ N = k` mean? :: `2ᵏ = N` #card
- When should you suspect `O(log N)`? :: When the number of elements in the problem space is halved each step #card
- Why is lookup in a balanced BST `O(log N)`? :: Each comparison sends you left or right, discarding half the remaining nodes #card
- Does the base of a log matter in big O? :: No. Logs of different bases differ only by a constant factor #card

## Open questions

- [ ] What is the full proof that the base of a log doesn't matter? (The book points to "Bases of Logs" later in the book.)


→ Next: [[RECURSIVE RUNTIME]]