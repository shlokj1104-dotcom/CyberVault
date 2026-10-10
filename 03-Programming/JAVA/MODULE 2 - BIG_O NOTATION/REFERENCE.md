---
title: REFERENCE
date: 2026-10-09
language: Java
phase: phase-1
tags:
links: []
status: learning
---
# 14 · Reference

> **Summary:** A quick-lookup sheet: growth rates from slowest to fastest, rules for simplifying expressions, a step-by-step analysis checklist, the identities used in the chapter, and common code shapes with their runtimes.

## Notes

### Growth rates, slowest to fastest

|Runtime|Name|Typical source|
|---|---|---|
|`O(1)`|Constant|Index into an array, arithmetic|
|`O(log N)`|Logarithmic|Halving the problem each step (binary search, balanced BST lookup)|
|`O(√N)`|Square root|Loop that stops at `x·x ≤ n` (primality)|
|`O(N)`|Linear|One pass through the data|
|`O(N log N)`|Linearithmic|Efficient sorting (e.g. quick sort expected case)|
|`O(N²)`|Quadratic|Nested loops over the same data|
|`O(2ᴺ)`|Exponential|Recursion with 2 branches and depth `N`|
|`O(N!)`|Factorial|Generating all permutations|

### Rules for simplifying

|Rule|Example|
|---|---|
|Drop constants|`O(2N)` → `O(N)`; `O(100000)` → `O(1)`|
|Drop non-dominant terms|`O(N² + N)` → `O(N²)`; `O(N + log N)` → `O(N)`|
|Different inputs → different variables|two arrays → `O(a·b)`, never `O(N²)`|
|Sum stays if no known relationship|`O(B² + A)`, `O(N + M)` can't be reduced|
|Do A **then** B|**add**: `O(A + B)`|
|Do B **for each** A|**multiply**: `O(A · B)`|
|Log base doesn't matter|`O(log₂ N)` = `O(log₁₀ N)`|
|Exponent base **does** matter|`O(2ⁿ)` ≠ `O(8ⁿ)`|

### How to analyze a piece of code (checklist from the chapter)

1. **Decide what the inputs are**, and give each its own, _meaningful_ variable name.
2. **Loops:** iterations × work per iteration. Nested loops multiply. Check whether the inner loop's bounds depend on the outer one (`j = i + 1` → still `N²/2`).
3. **Sequential blocks:** add, then keep the dominant term.
4. **Recursion:** draw the call tree. Work out _branches_ and _depth_, then `O(branches^depth)` (but check against a "what does it mean" count, as in Examples 9 and 16).
5. **Space:** count the longest chain of calls alive at once (the stack depth) plus any data you allocate.
6. **Sanity-check three ways when stuck:** _count the work_, _ask what the code means_, _ask how the runtime changes when the input doubles_.

### Useful identities used in the chapter

|Identity|Where it shows up|
|---|---|
|`1 + 2 + 3 + ... + (N-1) = N(N-1)/2`|Nested loop with `j = i + 1` (file 10, Example 3)|
|`2⁰ + 2¹ + ... + 2ᴺ = 2ᴺ⁺¹ − 1`|Recursion tree node count; `allFib` total (files 09, 12)|
|`1 + 2 + 4 + ... + X ≈ 2X`|Amortized `ArrayList` doubling (file 07)|
|`2ᵏ = N ⇔ k = log₂ N`|Halving → logarithm (file 08)|
|`2^(log₂ N) = N`|Balanced BST traversal (file 11, Example 9)|
|`n! = n × (n-1) × ... × 1`|Number of permutations (file 12)|

### Common patterns

```java
// 1. Single pass                          → O(N)
for (int x : array) { /* O(1) */ }

// 2. All pairs                            → O(N²)
for (int i = 0; i < n; i++)
    for (int j = 0; j < n; j++) { }

// 3. Unordered pairs                      → O(N²)  (about N²/2)
for (int i = 0; i < n; i++)
    for (int j = i + 1; j < n; j++) { }

// 4. Two inputs, nested                   → O(a·b)
for (int x : arrA)
    for (int y : arrB) { }

// 5. Halving                              → O(log N)
while (n > 1) { n /= 2; }

// 6. Two-branch recursion                 → O(2^N)
int f(int n) { if (n <= 1) return 1; return f(n-1) + f(n-1); }

// 7. Memoize it                           → O(N)
if (memo[n] > 0) return memo[n];
```

## Open questions

- [ ] What are the runtimes of the standard operations on common data structures (array, linked list, hash table, balanced tree)?
- [ ] What is the full derivation of the sums `1 + 2 + ... + N` and `2⁰ + ... + 2ᴺ`? (Deferred to the book's math appendix.)