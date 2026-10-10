---
title: WORKED EXAMPLES -  NAMING, TREES AND PRIMES
date: 2026-10-09
language: Java
phase: phase-1
tags:
links: []
status: learning
---
# 11 · Worked Examples: Naming, Trees and Primes (Examples 8–10)

> **Summary:** Name each input with its own meaningful variable, so a runtime is not built on one `N` that means two things. A BST traversal is `O(N)` even though the tree is balanced, and a primality check only needs to go up to `√n`.

## Notes

### Example 8

An algorithm takes an array of strings, **sorts each string**, then **sorts the full array**. What's the runtime?

**The common (wrong) reasoning:** sorting each string is `O(N log N)`, and we do this for each string, so `O(N · N log N)`. Then sorting the array is another `O(N log N)`. Total `O(N² log N + N log N)` = `O(N² log N)`.

**This is completely incorrect.** The error: we used `N` in **two different ways**. In one case it's the length of a string (which string?), in the other it's the length of the array.

How to prevent this in interviews: **don't use the variable "N" at all**, or only use it when there's no ambiguity about what it represents. The book wouldn't even use `a` and `b` or `m` and `n` here: it's too easy to forget which is which. And `O(a²)` is a completely different runtime from `O(a·b)`. **Use logical names:**

- Let **`s`** = the length of the longest string.
- Let **`a`** = the length of the array.

Now work through it in parts:

1. Sorting each string: `O(s log s)`.
2. We do this for every string (there are `a` of them): `O(a · s log s)`.
3. Now sort all the strings. Most candidates say `O(a log a)`. But you must also account for **comparing strings**: each comparison takes `O(s)` time, and there are `O(a log a)` comparisons → `O(a · s · log a)`.

Add the two parts:

```
O(a·s log s + a·s log a) = O(a·s(log a + log s))
```

**This is it. There's no way to reduce it further.**

### Example 9

Sum the values of all the nodes in a **balanced binary search tree**:

```java
int sum(Node node) {
    if (node == null) {
        return 0;
    }
    return sum(node.left) + node.value + sum(node.right);
}
```

> _Just because it's a binary search tree doesn't mean there's a log in it!_

Two ways to see it.

**What it means (the straightforward way):** the code touches **each node once** and does constant work per touch (excluding the recursive calls). So the runtime is linear in the number of nodes: **`O(N)`**, where `N` is the number of nodes.

**Recursive pattern:** the runtime of a multi-branch recursion is typically `O(branches^depth)`. Two branches → `O(2^depth)`. Many people think "something went wrong, we've created an exponential algorithm (yikes!)". The second statement is correct (it _is_ exponential), but **consider what variable it is exponential with respect to.**

The tree is a balanced BST, so with `N` total nodes, **depth ≈ `log N`**. So we get `O(2^(log N))`. Simplify using the definition `2ᴾ = Q → log₂Q = P`:

```
Let P = 2^(log N)
  → log₂ P = log₂ N      (taking log₂ of both sides, and log₂(2^x) = x)
  → P = N
  → 2^(log N) = N
```

So the runtime is **`O(N)`**, where `N` is the number of nodes. Both ways agree.

### Example 10

Checks whether a number is prime by testing divisibility. It only needs to go up to **`√n`**, because if `n` is divisible by a number greater than its square root, it's also divisible by something smaller. For example, `33` is divisible by 11 (greater than `√33`), but the "counterpart" to 11 is 3 (`3 × 11 = 33`), so 33 was already eliminated by 3.

```java
boolean isPrime(int n) {
    for (int x = 2; x * x <= n; x++) {
        if (n % x == 0) {
            return false;
        }
    }
    return true;
}
```

_Many people get this wrong. If you're careful about your logic, it's fairly easy._ The work inside the loop is constant, so we just need to know the number of iterations in the worst case. The loop starts at `x = 2` and ends when `x * x = n`, i.e. when `x = √n`. So it's really:

```java
for (int x = 2; x <= sqrt(n); x++) { ... }
```

→ **`O(√n)`**.

## Pitfalls

- **Using `N` for two different things** (Example 8). Rename: `a` for the array length, `s` for the string length.
- **Assuming a binary search tree always means a `log`** (Example 9). Visiting every node is `O(N)`.

## Flashcards

- What went wrong in "sort each string `O(N log N)` × `N` strings, then sort the array"? :: `N` was used for two different things (string length and array length). Use distinct names like `s` and `a` #card
- What is the correct runtime for sorting each string then sorting the array of strings? :: `O(a·s(log a + log s))`, where `a` is the array length and `s` the longest string length #card
- Why does sorting `a` strings cost `O(a·s log a)` and not `O(a log a)`? :: Each of the `O(a log a)` comparisons compares two strings, which takes `O(s)` #card
- What is the runtime of summing all nodes in a balanced BST recursively? :: `O(N)`: each node is touched once. (`2^(log N) = N` confirms it.) #card
- Why is `2^(log₂ N) = N`? :: By definition of log: if `P = 2^(log N)` then `log₂P = log₂N`, so `P = N` #card
- What is the runtime of `isPrime` with loop condition `x*x <= n`? :: `O(√n)` #


→ Next: [[WORKED EXAMPLES - RECURSION]]