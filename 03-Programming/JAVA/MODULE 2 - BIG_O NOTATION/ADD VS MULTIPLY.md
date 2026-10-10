---
title: ADD VS MULTIPLY
date: 2026-10-09
language: Java
phase: phase-1
tags:
links: []
status: learning
---
# 06 · Add vs. Multiply

> **Summary:** For a multi-part algorithm, "do this, then do that" adds the runtimes, and "do this for each time you do that" multiplies them. This is a very common source of interview mistakes.

## Notes

An algorithm has two steps. Do you add or multiply the runtimes? _A very common source of confusion._

```java
// ADD:  O(A + B)                      // MULTIPLY:  O(A * B)
for (int a : arrA) {                   for (int a : arrA) {
    print(a);                              for (int b : arrB) {
}                                              print(a + "," + b);
for (int b : arrB) {                       }
    print(b);                          }
}
```

- **Left:** `A` chunks of work, **then** `B` chunks of work → `O(A + B)`.
- **Right:** `B` chunks of work **for each** element of `A` → `O(A * B)`.

### The rule of thumb

- Form **"do this, then, when you're all done, do that"** → **add** the runtimes.
- Form **"do this _for each time_ you do that"** → **multiply** the runtimes.

It's very easy to mess this up in an interview, so be careful.

## Pitfalls

- **Adding when you should multiply, or the reverse.** "Then" → add. "For each" → multiply.

## Flashcards

- When do you add the runtimes of two parts? :: When the form is "do this, then when you're all done, do that" → `O(A + B)` #card
- When do you multiply the runtimes? :: When the form is "do this for each time you do that" → `O(A * B)` #card
