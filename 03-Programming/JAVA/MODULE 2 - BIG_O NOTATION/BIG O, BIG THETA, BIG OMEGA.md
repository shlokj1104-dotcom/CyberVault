---
title: BIG O, BIG THETA, BIG OMEGA
date: 2026-10-09
language: Java
phase: phase-1
tags:
links: []
status: learning
---
# 02 · Big O, Big Theta, Big Omega

> **Summary:** Academics use three bounds: O (upper), Ω (lower) and Θ (tight). In interviews and industry, "big O" means the tightest bound, which academics call Θ.

## Notes

_(The book says: if you've never met big O in an academic setting you can skip this, but it clears up wording confusion, and interviewers do like it.)_

Academics use three symbols:

|Symbol|Name|Meaning|Analogy|
|---|---|---|---|
|`O`|Big O|**Upper bound**: "at least as fast as this"|`x ≤ 130`|
|`Ω`|Big Omega|**Lower bound**: "no faster than this"|`x ≥ ...`|
|`Θ`|Big Theta|**Tight bound**: both `O` **and** `Ω`|`x = ...` (squeezed from both sides)|

Take "print every value in an array":

- It is `O(N)`, but also technically `O(N²)`, `O(N³)`, `O(2ᴺ)`... because it is at least as fast as each of those. These are all valid but not useful upper bounds (like saying "Bob is ≤ 1,000,000 years old").
- It is `Ω(N)`, and also `Ω(log N)` and `Ω(1)`, because it can't be faster than those.
- It is `Θ(N)` because it is both `O(N)` and `Ω(N)`.

### What industry (and therefore interviews) means

People have merged Θ and O into one idea. In an interview, "big O" means what academics call **Θ**: the **tightest** description. Saying that printing an array is `O(N²)` would be seen as incorrect. You just say `O(N)`. **This book, and your interviews, use big O as "the tightest bound".**

## Pitfalls

- **Quoting an academic-style loose bound in an interview.** Saying "printing an array is `O(N²)`" is technically true but considered wrong in industry. Give the tightest bound.

## Key terms

|Term|Definition|
|---|---|
|Upper bound (`O`)|"At least as fast as" this runtime|
|Lower bound (`Ω`)|"No faster than" this runtime|
|Tight bound (`Θ`)|Both an upper and a lower bound|

## Flashcards

- What does big O mean in academia? :: An **upper bound** on the runtime (like `≤`); printing an array is `O(N)` and also `O(N²)` #card
- What do Ω and Θ mean in academia? :: Ω is a **lower bound**; Θ is a **tight bound** (both `O` and `Ω`) #card
- What does "big O" mean in industry and interviews? :: Essentially the academic Θ: the tightest description of the runtime #card
- Is "printing an array is `O(N²)`" acceptable in an interview? :: Technically true but considered incorrect in industry; say `O(N)` #card
