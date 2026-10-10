---
title: RECURSIVE RUNTIME
date: 2026-10-09
language: Java
phase: phase-1
tags:
links: []
status: learning
---
# 09 · Recursive Runtimes

> **Summary:** A function that makes several recursive calls usually runs in `O(branches^depth)`. Derive it by drawing the call tree rather than guessing from the number of calls. Recursion also uses stack space equal to its maximum depth, not the total number of calls.

## Notes

A tricky one. What is the runtime of this?

```java
int f(int n) {
    if (n <= 1) {
        return 1;
    }
    return f(n - 1) + f(n - 1);
}
```

Many people see two calls and jump to `O(N²)`. **That is completely incorrect.** Don't assume; derive it by walking through the code. Call `f(4)`:

```
                       f(4)
              /                  \
          f(3)                    f(3)
        /      \                /      \
     f(2)      f(2)          f(2)      f(2)
     /  \      /  \          /  \      /  \
  f(1) f(1) f(1) f(1)     f(1) f(1) f(1) f(1)
```

The tree has **depth `N`**, and each node (function call) has **two children**, so each level has twice as many calls as the one above:

|Level|# Nodes|As a power of 2|
|---|---|---|
|0|1|2⁰|
|1|2|2¹|
|2|4|2²|
|3|8|2³|
|4|16|2⁴|

Total nodes: `2⁰ + 2¹ + 2² + ... + 2ᴺ = 2ᴺ⁺¹ − 1`, so the runtime is **`O(2ᴺ)`**.

**Pattern to remember:** when a recursive function makes **multiple calls**, the runtime will often (but not always) look like

> **`O(branches^depth)`**

where `branches` is the number of times each call branches. Here, 2 branches and depth `N` give `O(2ᴺ)`.

### The base of an exponent does matter

Unlike the base of a log, the base of an exponent does matter. Compare `2ⁿ` and `8ⁿ`: `8ⁿ = (2³)ⁿ = 2³ⁿ = 2²ⁿ · 2ⁿ`. They differ by a factor of `2²ⁿ`, which is _very much not_ a constant factor.

### Space complexity of this function

Space complexity of this function is `O(N)`. There are `O(2ᴺ)` nodes in the tree in total, but only `O(N)` exist **at any given time** (one root-to-current-node path), so we only need `O(N)` memory.

## Pitfalls

- **Seeing two recursive calls and saying `O(N²)`.** It is `O(2ᴺ)`: derive it from the call tree.
- **Assuming the base of an exponent doesn't matter.** It does (`8ⁿ` vs `2ⁿ`), unlike the base of a log.

## Key terms

|Term|Definition|
|---|---|
|Branches / depth|In recursion: how many calls each call makes, and how deep the call tree goes; runtime is often `O(branches^depth)`|

## Flashcards

- What is the runtime of `f(n) { if (n<=1) return 1; return f(n-1) + f(n-1); }`? :: `O(2ᴺ)`: two branches per call, depth `N` #card
- What is the common pattern for recursive functions with multiple calls? :: `O(branches^depth)` (often but not always) #card
- Does the base of an exponent matter in big O? :: Yes. `8ⁿ = 2²ⁿ · 2ⁿ` differs from `2ⁿ` by a non-constant factor `2²ⁿ` #card
- What is the space complexity of the two-branch recursion `f`? :: `O(N)`: only one root-to-leaf path of calls is on the stack at a time, even though there are `O(2ᴺ)` nodes in total #card
- How many nodes are in a binary recursion tree of depth `N`? :: `2⁰ + 2¹ + ... + 2ᴺ = 2ᴺ⁺¹ − 1` #card


→ Next: [[WORKED EXAMPLES - LOOPS]]