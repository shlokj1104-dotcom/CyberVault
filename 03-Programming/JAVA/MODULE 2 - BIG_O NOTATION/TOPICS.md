---
title: TOPICS
date: 2026-10-09
phase: phase-1
tags:
links: []
status: learning
language: Java
---
# Big O Notes: Index

> **Summary:** Big O describes how an algorithm's time (or memory) grows as the input grows, not how many seconds it takes. Drop constants, drop non-dominant terms, use separate variables for separate inputs, add runtimes for "do this, then that", and multiply them for "do this for each time you do that".

## Notes

These notes are split into 14 topic files. Read them in order the first time. After that, jump to whichever topic you need.

| File                                                                         | Covers                                                                          |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| [[03-Programming/JAVA/MODULE 2 - BIG_O NOTATION/INTRODUCTION\|INTRODUCTION]] | What Big O measures, the file-transfer analogy, time complexity                 |
| [[BIG O, BIG THETA, BIG OMEGA]]                                              | Upper, lower and tight bounds; what interviews mean by "big O"                  |
| [[BEST, WORST AND EXPECTED CASE]]                                            | Describing runtime by input scenario (quick sort)                               |
| [[SPACE COMPEXITY]]                                                          | Memory use, including the recursion call stack                                  |
| [[DROP CONSTANTS AND TERMS]]                                                 | Dropping constants and non-dominant terms; growth order                         |
| [[ADD VS MULTIPLY]]                                                          | Combining the runtimes of sequential and nested work                            |
| [[AMORTIZED TIME]]                                                           | Amortized time and dynamic arrays (ArrayList)                                   |
| [[LOGARITHM RUNTIME]]                                                        | Where O(log N) comes from                                                       |
| [[RECURSIVE RUNTIME]]                                                        | Recursion trees and O(branches^depth)                                           |
| [[WORKED EXAMPLES - LOOPS]]                                                  | Examples 1 to 7: loops and nested loops                                         |
| [[WORKED EXAMPLES -  NAMING, TREES AND PRIMES]]                              | Examples 8 to 10: naming variables, BST sum, primality                          |
| [[WORKED EXAMPLES - RECURSION]]                                              | Examples 11 to 16: factorial, permutations, Fibonacci, memoization, powers of 2 |
| [[REFERENCE]]                                                                | Growth-rate table, simplification rules, checklist, identities, code patterns   |

### Worked examples at a glance

|#|What the code does|Runtime|Lesson|File|
|---|---|---|---|---|
|1|Two separate loops over an array|`O(N)`|Doing it twice doesn't matter|10|
|2|Nested loops over the same array|`O(N²)`|Inner `O(N)`, run `N` times|10|
|3|Nested loops, inner starts at `i + 1`|`O(N²)`|`N²/2` pairs; drop the constant|10|
|4|Nested loops over two different arrays|`O(ab)`|Different inputs → different variables|10|
|5|Same, plus a constant 100,000 loop|`O(ab)`|Constant work is still constant|10|
|6|Reverse an array (half the iterations)|`O(N)`|Half of `N` is still `N`|10|
|7|Which expressions equal `O(N)`?|all but `O(N + M)`|Drop constants and dominated terms|10|
|8|Sort each string, then sort the array|`O(a·s(log a + log s))`|Don't reuse `N` for two meanings|11|
|9|Sum nodes of a balanced BST|`O(N)`|A BST doesn't mean there's a `log`|11|
|10|Primality test up to `√n`|`O(√n)`|Loop stops at `x·x ≤ n`|11|
|11|Recursive factorial|`O(n)`|One call per level|12|
|12|All permutations of a string|`O(n²·n!)` (upper bound)|`n·n!` calls × `O(n)` each|12|
|13|Naive recursive Fibonacci|`O(2ᴺ)`|Two branches, depth `N`|12|
|14|Print `fib(0..n)` using naive `fib`|`O(2ⁿ)`|`n` isn't constant; sum the work|12|
|15|Same, with memoization|`O(n)`|Cache turns exponential into linear|12|
|16|Print powers of 2 up to `n`|`O(log n)`|Halving each call|12|
