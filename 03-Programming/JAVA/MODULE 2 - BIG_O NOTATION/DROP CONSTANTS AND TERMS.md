---
title: DROP CONSTANTS AND TERMS
date: 2026-10-09
language: Java
phase: phase-1
tags:
links: []
status: learning
---
# 05 · Dropping Constants and Non-Dominant Terms

> **Summary:** Big O keeps only the rate of growth. Drop constant factors (`O(2N)` → `O(N)`) and smaller terms (`O(N² + N)` → `O(N²)`), but keep a sum when the terms have no known relationship.

## Notes

### Drop the constants

It is entirely possible for `O(N)` code to run faster than `O(1)` code **for specific inputs**. Big O only describes the **rate of increase**. For that reason we **drop constants**: an algorithm you might call `O(2N)` is `O(N)`.

Many people resist this. They see two non-nested loops and keep "`O(2N)`" thinking it is more precise. It isn't.

```java
// Min and Max 1                         // Min and Max 2
int min = Integer.MAX_VALUE;             int min = Integer.MAX_VALUE;
int max = Integer.MIN_VALUE;             int max = Integer.MIN_VALUE;
for (int x : array) {                    for (int x : array) {
    if (x < min) min = x;                    if (x < min) min = x;
    if (x > max) max = x;                }
}                                        for (int x : array) {
                                             if (x > max) max = x;
                                         }
```

Which is faster? One does one loop with two lines inside; the other does two loops with one line each. To count _instructions_ you'd have to go down to assembly level (multiplication costs more than addition, the compiler optimizes, ...). That is horrendously complicated, so don't start down this road.

**Both are `O(N)`.** Big O lets us express _how the runtime scales_. We simply have to accept that `O(N)` is not always better than `O(N²)` for a given input, only eventually, as input grows.

### Drop the non-dominant terms

What about `O(N² + N)`? That second `N` isn't exactly a constant, but it isn't important either. We already said `O(N² + N²)` is `O(N²)`. If we don't care about a second `N²` term, why care about a smaller `N` term? We don't.

- `O(N² + N)` → `O(N²)`
- `O(N + log N)` → `O(N)`
- `O(5·2ᴺ + 1000N¹⁰⁰)` → `O(2ᴺ)`

**But a sum can stay a sum:** `O(B² + A)` cannot be reduced _without special knowledge of A and B_, because there's no known relationship between them.

### Growth order, slowest to fastest

From the book's graph:

```
O(1) < O(log x) < O(x) < O(x log x) < O(x²) < O(2ˣ) < O(x!)
```

`O(x²)` is much worse than `O(x)` but nowhere near as bad as `O(2ˣ)` or `O(x!)`. Even worse exist: `O(xˣ)`, `O(2ˣ · x!)`.

## Pitfalls

- **Keeping constants** ("it's `O(2N)`, more precise"). It isn't. Drop them.
- **Keeping a smaller term** (`O(N² + N)`). Drop the non-dominant term. But **don't** drop a term you can't compare (`O(N + M)`).

## Key terms

|Term|Definition|
|---|---|
|Dominant term|The fastest-growing term in a runtime expression, the one you keep|
|Drop constants|Rule: `O(2N)` → `O(N)`|
|Drop non-dominant terms|Rule: `O(N² + N)` → `O(N²)`|

## Flashcards

- Why do we drop constants? :: Big O describes rate of increase; counting exact instructions is hopelessly complicated and `O(2N)` and `O(N)` scale the same way #card
- Which is faster: one loop with two statements, or two loops with one statement each? :: Both are `O(N)`; big O doesn't distinguish them #card
- How do you simplify `O(N² + N)`, `O(N + log N)` and `O(5·2ᴺ + 1000N¹⁰⁰)`? :: `O(N²)`, `O(N)`, `O(2ᴺ)` (drop non-dominant terms) #card
- When can't a sum be simplified? :: When there's no known relationship between the terms, e.g. `O(B² + A)` or `O(N + M)` #card
- Rank these slowest-growing to fastest: `N²`, `log N`, `2ᴺ`, `N`, `N!`, `N log N` :: `log N`, `N`, `N log N`, `N²`, `2ᴺ`, `N!` #card



→ Next: [[ADD VS MULTIPLY]]