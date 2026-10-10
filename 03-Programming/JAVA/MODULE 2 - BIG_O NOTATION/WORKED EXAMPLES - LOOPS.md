---
title: WORD EXAMPLES - LOOPS
date: 2026-10-09
language: Java
phase: phase-1
tags:
links: []
status: learning
---
# 10 · Worked Examples: Loops

> **Summary:** Loop-based examples show how to count work: single passes are `O(N)`, nested loops over the same data are `O(N²)`, and nested loops over two different inputs are `O(ab)`. Constants and smaller terms do not change the result.

## Notes

Big O is hard at first; once it "clicks" it gets fairly easy. _The same patterns come up again and again, and the rest you can derive._ This file covers Examples 1 to 7. For the full list at a glance, see the README.

### Example 1

```java
void foo(int[] array) {
    int sum = 0;
    int product = 1;
    for (int i = 0; i < array.length; i++) {
        sum += array[i];
    }
    for (int i = 0; i < array.length; i++) {
        product *= array[i];
    }
    System.out.println(sum + ", " + product);
}
```

**`O(N)`.** Iterating through the array twice doesn't matter (drop constants: `O(2N)` → `O(N)`).

### Example 2

```java
void printPairs(int[] array) {
    for (int i = 0; i < array.length; i++) {
        for (int j = 0; j < array.length; j++) {
            System.out.println(array[i] + "," + array[j]);
        }
    }
}
```

The inner loop has `O(N)` iterations and is run `N` times → **`O(N²)`**.

Another way: look at the **meaning** of the code. It prints all pairs (two-element sequences). There are `O(N²)` pairs, so the runtime is `O(N²)`.

### Example 3

Very similar, but the inner loop starts at `i + 1`:

```java
void printUnorderedPairs(int[] array) {
    for (int i = 0; i < array.length; i++) {
        for (int j = i + 1; j < array.length; j++) {
            System.out.println(array[i] + "," + array[j]);
        }
    }
}
```

_This pattern of for loop is very common. You must know the runtime and **deeply understand it**; memorizing common runtimes isn't enough._ There are several ways to derive it:

**(a) Counting the iterations.** First time through, `j` runs `N-1` steps; then `N-2`, then `N-3`, ... down to 2, 1. Total:

```
(N-1) + (N-2) + (N-3) + ... + 2 + 1
= 1 + 2 + 3 + ... + (N-1)
= sum of 1 through N-1
= N(N-1)/2
```

→ `O(N²)`.

**(b) What it means.** It iterates through each pair `(i, j)` where `j > i`. There are `N²` total pairs; roughly half have `i < j` and half have `i > j`. So the code goes through about `N²/2` pairs → `O(N²)`.

**(c) Visualizing what it does** (`N = 8`):

```
(0,1) (0,2) (0,3) (0,4) (0,5) (0,6) (0,7)
      (1,2) (1,3) (1,4) (1,5) (1,6) (1,7)
            (2,3) (2,4) (2,5) (2,6) (2,7)
                  (3,4) (3,5) (3,6) (3,7)
                        (4,5) (4,6) (4,7)
                              (5,6) (5,7)
                                    (6,7)
```

It looks like **half of an N×N matrix** (size about `N²/2`) → `O(N²)`.

**(d) Average work.** The outer loop runs `N` times. How much work does the inner loop do on average? The values `1, 2, 3, ..., N` average `N/2` (like `1..10` averaging about 5). So total work is `N × N/2 = N²/2` → `O(N²)`.

### Example 4

Now two different arrays:

```java
void printUnorderedPairs(int[] arrayA, int[] arrayB) {
    for (int i = 0; i < arrayA.length; i++) {
        for (int j = 0; j < arrayB.length; j++) {
            if (arrayA[i] < arrayB[j]) {
                System.out.println(arrayA[i] + "," + arrayB[j]);
            }
        }
    }
}
```

Break it up: the `if` inside the `j` loop is `O(1)` (a sequence of constant-time statements). So it's:

```java
for (int i = 0; i < arrayA.length; i++) {
    for (int j = 0; j < arrayB.length; j++) {
        /* O(1) work */
    }
}
```

For each element of `arrayA`, the inner loop does `b = arrayB.length` iterations. With `a = arrayA.length`, the runtime is **`O(ab)`**.

> **If you said `O(N²)`, remember your mistake for the future.** It is not `O(N²)` because there are **two different inputs**, and both matter. _This is an extremely common mistake._

### Example 5

```java
void printUnorderedPairs(int[] arrayA, int[] arrayB) {
    for (int i = 0; i < arrayA.length; i++) {
        for (int j = 0; j < arrayB.length; j++) {
            for (int k = 0; k < 100000; k++) {
                System.out.println(arrayA[i] + "," + arrayB[j]);
            }
        }
    }
}
```

Nothing really changed. 100,000 units of work is still **constant**, so the runtime is **`O(ab)`**.

### Example 6

```java
void reverse(int[] array) {
    for (int i = 0; i < array.length / 2; i++) {
        int other = array.length - i - 1;
        int temp = array[i];
        array[i] = array[other];
        array[other] = temp;
    }
}
```

**`O(N)`.** It only goes through half the array (in terms of iterations), but that does not impact the big O time. It swaps from both ends toward the middle.

### Example 7

Which of these are equivalent to `O(N)`? Why?

- `O(N + P)`, where `P < N/2`
- `O(2N)`
- `O(N + log N)`
- `O(N + M)`

Go through them:

- If `P < N/2`, then `N` is the dominant term and we drop `O(P)` → `O(N)`.
- `O(2N)` is `O(N)` since we drop constants.
- `O(N)` dominates `O(log N)`, so we drop the `log N` → `O(N)`.
- There is **no established relationship between `N` and `M`**, so we must keep both variables.

All but the last are equivalent to `O(N)`.

## Pitfalls

- **Using one variable for two different inputs** (Example 4). Two arrays → `O(a·b)`, not `O(N²)`.

## Flashcards

- What is the runtime of two non-nested loops over an array? :: `O(N)` (drop the constant 2) #card
- What is the runtime of nested loops where the inner starts at `j = i + 1`? :: `O(N²)`: about `N(N-1)/2` iterations, i.e. roughly half an `N × N` matrix #card
- Why is nested looping over two different arrays `O(ab)` and not `O(N²)`? :: There are two inputs with independent sizes, and both matter #card
- Does a constant inner loop (e.g. 100,000 iterations) change the big O? :: No, 100,000 units of work is still constant #card
- Why does `reverse(array)` that loops to `length/2` run in `O(N)`? :: Half of `N` is still `N`; constants are dropped #card


→ Next: [[WORKED EXAMPLES -  NAMING, TREES AND PRIMES]]