---
title: SPACE COMPEXITY
date: 2026-10-09
language: Java
phase: phase-1
tags:
links: []
status: learning
---
# 04 · Space Complexity

> **Summary:** Space complexity measures memory use as input grows, just like time. Recursive calls take stack space while they are alive, but only the calls that exist at the same time count toward it.

## Notes

Time is not the only thing that matters. We also care about **memory (space)**. Space complexity is a parallel concept to time complexity:

- An array of size `n` → `O(n)` space.
- An `n × n` two-dimensional array → `O(n²)` space.

### Stack space in recursive calls counts too

```java
int sum(int n) {          // Ex 1
    if (n <= 0) {
        return 0;
    }
    return n + sum(n - 1);
}
```

Each call adds a level to the **call stack**, and all of them are alive at the same time:

```
sum(4)
   -> sum(3)
      -> sum(2)
         -> sum(1)
            -> sum(0)
```

That is `O(n)` time **and** `O(n)` space, because every pending call takes up real memory.

**But `n` calls total does not mean `O(n)` space.** What counts is how many are on the stack **simultaneously**.

```java
int pairSumSequence(int n) {          // Ex 2
    int sum = 0;
    for (int i = 0; i < n; i++) {
        sum += pairSum(i, i + 1);
    }
    return sum;
}

int pairSum(int a, int b) {
    return a + b;
}
```

There are `O(n)` calls to `pairSum`, but each one **finishes before the next starts**, so they never coexist on the stack. Space is `O(1)`.

## Pitfalls

- **Counting stack frames that don't coexist.** `n` calls ≠ `O(n)` space if each finishes before the next starts (`pairSumSequence` is `O(1)` space).
- **Forgetting recursion's stack space.** `sum(n)` is `O(n)` space even though it allocates nothing.

## Key terms

|Term|Definition|
|---|---|
|Space complexity|How memory use scales with input size (including call-stack depth)|
|Call stack|Memory holding each active function call; recursion depth determines its size|

## Flashcards

- What is space complexity? :: The amount of memory an algorithm needs, which scales with the input just like time #card
- What space does an array of size `n` take? An `n × n` 2D array? :: `O(n)` and `O(n²)` #card
- Does recursion stack space count toward space complexity? :: Yes. `sum(n)` that recurses `n` deep uses `O(n)` space #card
- Why is `pairSumSequence(n)` only `O(1)` space despite `O(n)` calls to `pairSum`? :: Each call finishes before the next starts, so they never coexist on the call stack #card

## Open questions

- [ ] How do I analyze space for algorithms that allocate new arrays or strings inside recursion?


→ Next: [[DROP CONSTANTS AND TERMS]]