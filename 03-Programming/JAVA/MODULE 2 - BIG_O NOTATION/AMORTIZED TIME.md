---
title: AMORTIZED TIME
date: 2026-10-09
language: Java
phase: phase-1
tags:
links: []
status: learning
---
# 07 · Amortized Time

> **Summary:** Amortized time is the average cost per operation over a long sequence. A dynamic array's insertion is usually `O(1)` and occasionally `O(N)`, but it is amortized `O(1)` because the expensive resizes are rare.

## Notes

An **`ArrayList`** (a dynamically resizing array) gives you the benefits of an array with flexible size. It is implemented with an array. When the array hits capacity, the class creates a **new array of double the capacity** and copies all elements over.

**What is the runtime of an insertion?** A tricky question.

- If the array is **full** and contains `N` elements, inserting takes `O(N)`: create a size-`2N` array and copy `N` elements.
- But this doesn't happen often. **Most insertions are `O(1)`.**

We need a concept that accounts for both. That is **amortized time**: yes, the worst case happens every once in a while, but once it happens it won't happen again for so long that the cost is "amortized" (spread out).

### Deriving the amortized time

1. We double the capacity when the size is a power of 2. After `X` elements, we've doubled at sizes 1, 2, 4, 8, 16, ..., `X`.
2. Those doublings cost, respectively, 1, 2, 4, 8, 16, ..., `X` copies.
3. Sum: `1 + 2 + 4 + 8 + ... + X`. Read right to left: `X + X/2 + X/4 + X/8 + ... + 1`, which is roughly **`2X`**.
4. So `X` insertions take `O(2X)` = `O(X)` total, and the **amortized time per insertion is `O(1)`**.

_(Beyond the chapter: Java's `ArrayList` really grows by about 1.5× rather than 2×. The argument works for any constant growth factor greater than 1, so the conclusion is unchanged. Growing by a fixed **+k** each time would **not** give `O(1)` amortized.)_

## Pitfalls

- **Assuming amortized `O(1)` means every operation is `O(1)`.** An individual insertion can still cost `O(N)`; only the average over many is `O(1)`.

## Key terms

|Term|Definition|
|---|---|
|Amortized time|Average cost per operation over a long sequence, spreading out rare expensive operations|
|Dynamic array (`ArrayList`)|Array that doubles its capacity (and copies) when full|

## Flashcards

- Why is insertion into an `ArrayList` tricky to describe? :: Usually `O(1)`, but when the array is full a resize copies `N` elements, which is `O(N)` #card
- What is amortized time? :: The average cost per operation over many operations when a rare expensive operation is "spread out"; `ArrayList` insertion is amortized `O(1)` #card
- Why is `ArrayList` insertion amortized `O(1)`? :: Doubling copies 1 + 2 + 4 + ... + X ≈ 2X elements in total for X insertions, so about 2 copies per insertion #card

## Open questions

- [ ] What are the runtimes of the standard operations on common data structures (array, linked list, hash table, balanced tree)? (Tracked in 14-reference.md.)