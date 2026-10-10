---
title: INTRODUCTION
date: 2026-10-09
language: Java
phase: phase-1
tags:
links: []
status: learning
---
# 01 · What Big O Is

> **Summary:** Big O is the language for describing how an algorithm's time (or memory) grows as its input grows. It describes the rate of increase, not the number of seconds.

## Notes

**Big O time** is the metric we use to describe the efficiency of algorithms. It answers one question: _when the input gets bigger, how much slower (or hungrier) does the algorithm get?_

Why it matters:

1. In an interview you are expected to describe the runtime of what you write, and to judge it harshly yourself.
2. Without it you can't tell whether a change made your algorithm better or worse.
3. It is the shared vocabulary behind every algorithm and data structure.

|Topic|The question it answers|
|---|---|
|Time complexity|How does the runtime scale with input size?|
|Big O / Θ / Ω|What do "upper", "lower" and "tight" bound mean, and what does industry mean by "big O"?|
|Best / worst / expected case|For _which input_ are we describing the runtime?|
|Space complexity|How does memory use scale (including the recursion stack)?|
|Drop constants|Why is `O(2N)` just `O(N)`?|
|Drop non-dominant terms|Why is `O(N² + N)` just `O(N²)`?|
|Add vs. multiply|How do I combine the runtimes of a multi-part algorithm?|
|Amortized time|How do I describe an operation that is _usually_ cheap but _occasionally_ expensive?|
|Log N runtimes|Where does `log` come from?|
|Recursive runtimes|How do I analyze a function that calls itself more than once?|

Each of these has its own file in this set. This file covers the foundation.

### An analogy: sending a file

You must get a file to a friend across the country as fast as possible. Email/FTP is the first thought. But if the file is huge (say 1 TB, which can take more than a day to upload), **physically flying the drive there is faster**. Even driving it would be faster for a big enough file.

So which method is "faster"? It depends on the file size, which is exactly what Big O captures:

|Method|Runtime|Meaning|
|---|---|---|
|Electronic transfer|`O(s)`, where `s` is the file size|Time grows **linearly** with the size of the file|
|Airplane transfer|`O(1)` with respect to file size|**Constant**: a bigger file doesn't make the flight longer|

(Yes, this is a simplification, but it is fine for the purpose.)

**The key lesson:** _no matter how big the constant is and how slow the linear increase is, linear will eventually surpass constant._ A flight that takes 10 hours always loses to a slow upload for small files, but wins for large enough ones. Big O is about what happens as the input grows.

### Time complexity

**Asymptotic runtime** (big O time) describes how the runtime grows with the input size. `O(1)` is a flat line; `O(s)` is a rising line that must eventually cross it.

Common runtimes you will see: `O(log N)`, `O(N log N)`, `O(N)`, `O(N²)`, `O(2ᴺ)`. **There is no fixed list.** Anything can show up (`O(√N)`, `O(N!)`, ...).

**Multiple variables are allowed.** Painting a fence `w` meters wide and `h` meters high takes `O(wh)`. With `p` layers of paint: `O(whp)`.

## Pitfalls

- **Treating Big O as "seconds".** It describes how runtime _scales_. `O(N)` code can beat `O(1)` code for small inputs.

## Key terms

|Term|Definition|
|---|---|
|Big O|The rate of increase of runtime (or space) as input grows; in industry, the tightest bound|
|Asymptotic runtime|Runtime described as the input size grows large; another name for big O time|
|Time complexity|How runtime scales with input size|

## Flashcards

- What does Big O measure? :: How the runtime (or memory) of an algorithm scales as the input size grows, i.e. its rate of increase #card
- In the file-transfer analogy, what are the runtimes of electronic transfer and airplane transfer? :: Electronic is `O(s)` (linear in file size `s`); airplane is `O(1)` (constant) #card
- What is the key lesson of the file-transfer analogy? :: Linear growth will eventually exceed constant, however big the constant or small the linear slope #card
- Is there a fixed list of possible runtimes? :: No. Common ones are `O(log N)`, `O(N log N)`, `O(N)`, `O(N²)`, `O(2ᴺ)`, but any expression can occur #card
- Can a runtime have multiple variables? :: Yes. e.g. painting a fence `w` wide, `h` high, `p` layers is `O(whp)` #card



→ Next: [[BIG O, BIG THETA, BIG OMEGA]]