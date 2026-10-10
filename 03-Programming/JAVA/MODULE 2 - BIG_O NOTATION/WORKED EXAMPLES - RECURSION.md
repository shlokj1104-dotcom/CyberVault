---
title: WORKED EXAMPLES - RECURSION
date: 2026-10-09
language: Java
phase: phase-1
tags:
links: []
status: learning
---
# 12 · Worked Examples: Recursion (Examples 11–16)

> **Summary:** Recursive examples are analyzed by counting calls and the work per call. Exponential recursion can become linear with memoization, and a loop count that changes with the input must be summed rather than multiplied.

## Notes

### Example 11

```java
int factorial(int n) {
    if (n < 0) {
        return -1;
    } else if (n == 0) {
        return 1;
    } else {
        return n * factorial(n - 1);
    }
}
```

A straight recursion from `n` to `n-1` to `n-2` down to 1. **`O(n)`** time (one call per level, only one branch).

### Example 12

Counts all permutations of a string:

```java
void permutation(String str) {
    permutation(str, "");
}

void permutation(String str, String prefix) {
    if (str.length() == 0) {
        System.out.println(prefix);
    } else {
        for (int i = 0; i < str.length(); i++) {
            String rem = str.substring(0, i) + str.substring(i + 1);
            permutation(rem, prefix + str.charAt(i));
        }
    }
}
```

A (very!) tricky one. Think about **how many times `permutation` gets called** and **how long each call takes**, aiming for the tightest upper bound.

**How many times is it called in its base case?** To build a permutation we pick a character for each "slot". With 7 characters: 7 choices for the first slot, then 6 for the next (6 choices _for each_ of the 7), then 5, and so on: `7 × 6 × 5 × 4 × 3 × 2 × 1 = 7!`. So there are `n!` permutations, and `permutation` is called **`n!` times in its base case** (when `prefix` is the full permutation).

**How many times before the base case?** We must also count the calls that reach the loop body (lines with `substring`/recursive call). Picture the call tree: there are `n!` leaves, and each leaf is attached to a path of length `n`. So there are **no more than `n · n!` nodes** (function calls) in the tree.

**How long does each call take?**

- Printing `prefix` (the base-case line) is `O(n)`: every character must be printed.
- The `substring` concatenation and the `prefix + str.charAt(i)` together are `O(n)`: the lengths of `rem`, `prefix` and the char always sum to `n`.

So each node in the call tree corresponds to `O(n)` work.

**Total runtime:** `O(n · n!)` calls, each taking `O(n)` → at most **`O(n² · n!)`**.

A tighter bound exists through more complex mathematics (though not necessarily as a nice closed-form expression). That would almost certainly be beyond the scope of any normal interview.

### Example 13

```java
int fib(int n) {
    if (n <= 0) return 0;
    else if (n == 1) return 1;
    return fib(n - 1) + fib(n - 2);
}
```

Use the recursion pattern `O(branches^depth)`: 2 branches per call, depth `N` → **`O(2ᴺ)`**.

> **Tighter bound (bonus points):** through very complicated math, the real runtime is closer to **`O(1.6ᴺ)`**. It is exponential, but not exactly `2ᴺ`, because at the bottom of the call stack there is sometimes only one call, and a lot of the nodes are at the bottom (as in most trees), so single-vs-double call actually makes a big difference. Saying `O(2ᴺ)` suffices for an interview and is still technically correct (see the note on big theta in file 02). You may get "bonus points" if you recognize it's actually less than that.

**General rule:** when you see an algorithm with **multiple recursive calls**, you're probably looking at an **exponential** runtime.

### Example 14

Prints all Fibonacci numbers from 0 to `n`:

```java
void allFib(int n) {
    for (int i = 0; i < n; i++) {
        System.out.println(i + ": " + fib(i));
    }
}

int fib(int n) {
    if (n <= 0) return 0;
    else if (n == 1) return 1;
    return fib(n - 1) + fib(n - 2);
}
```

Many people rush to conclude: `fib(n)` takes `O(2ⁿ)` and it's called `n` times, so `O(n · 2ⁿ)`.

**Not so fast. Can you find the error?** The `n` is **changing**. Yes, `fib(n)` takes `O(2ⁿ)`, but it matters _what value of n is_. Walk through each call:

```
fib(1) -> 2¹ steps
fib(2) -> 2² steps
fib(3) -> 2³ steps
fib(4) -> 2⁴ steps
...
fib(n) -> 2ⁿ steps
```

Total work: `2¹ + 2² + 2³ + ... + 2ⁿ = 2ⁿ⁺¹ − 2`, which is `O(2ⁿ)`. So computing the first `n` Fibonacci numbers this terrible way is still **`O(2ⁿ)`**, not `O(n · 2ⁿ)`.

### Example 15

Same output, but this time it **caches** previously computed values in an array:

```java
void allFib(int n) {
    int[] memo = new int[n + 1];
    for (int i = 0; i < n; i++) {
        System.out.println(i + ": " + fib(i, memo));
    }
}

int fib(int n, int[] memo) {
    if (n <= 0) return 0;
    else if (n == 1) return 1;
    else if (memo[n] > 0) return memo[n];

    memo[n] = fib(n - 1, memo) + fib(n - 2, memo);
    return memo[n];
}
```

Walk through what happens:

```
fib(1) -> return 1
fib(2)
    fib(1) -> return 1
    fib(0) -> return 0
    store 1 at memo[2]
fib(3)
    fib(2) -> lookup memo[2] -> return 1
    fib(1) -> return 1
    store 2 at memo[3]
fib(4)
    fib(3) -> lookup memo[3] -> return 2
    fib(2) -> lookup memo[2] -> return 1
    store 3 at memo[4]
fib(5)
    fib(4) -> lookup memo[4] -> return 3
    fib(3) -> lookup memo[3] -> return 2
    store 5 at memo[5]
...
```

At each call to `fib(i)`, the values `fib(i-1)` and `fib(i-2)` are **already computed and stored**. We just look them up, add them, store the result, and return: **constant time**. A constant amount of work `N` times → **`O(n)`**.

This technique is called **memoization** and is a very common way to optimize exponential-time recursive algorithms.

### Example 16

Prints the powers of 2 from 1 through `n` inclusive (for `n = 4`: 1, 2, 4):

```java
int powersOf2(int n) {
    if (n < 1) {
        return 0;
    } else if (n == 1) {
        System.out.println(1);
        return 1;
    } else {
        int prev = powersOf2(n / 2);
        int curr = prev * 2;
        System.out.println(curr);
        return curr;
    }
}
```

Several ways to compute the runtime:

**What it does:** walk through `powersOf2(50)`:

```
powersOf2(50)
    -> powersOf2(25)
        -> powersOf2(12)
            -> powersOf2(6)
                -> powersOf2(3)
                    -> powersOf2(1)
                        -> print & return 1
                    print & return 2
                print & return 4
            print & return 8
        print & return 16
    print & return 32
```

The runtime is the number of times we can divide `50` (or `n`) by 2 until we reach the base case (1). From file 08, the number of times we can halve `n` until we get 1 is **`O(log n)`**.

**What it means:** the code computes the powers of 2 from 1 through `n`. Each call prints and returns **exactly one** number (excluding recursive calls). So if it prints 13 values, `powersOf2` was called 13 times. It prints all the powers of 2 between 1 and `n`, and there are `log n` of them → `O(log n)`.

**Rate of increase:** think about how the runtime changes as `n` grows (this is exactly what big O means). If `N` goes from `P` to `P+1`, the number of calls might not change at all. **When does it increase? It increases by 1 each time `n` doubles in size.** So the number of calls is the number of times you can double 1 until you get `n`, i.e. the `x` in `2ˣ = n`, which is `x = log n`. → **`O(log n)`**.

## Pitfalls

- **Multiplying by the loop count when the loop variable changes** (Example 14). `fib(i)` costs `2ⁱ`, so sum them instead of doing `n × 2ⁿ`.
- **Forgetting that string concatenation costs `O(length)`** (Example 12): that `O(n)` per call is what turns `n·n!` into `n²·n!`.

## Key terms

|Term|Definition|
|---|---|
|Memoization|Caching results of a function so repeated calls are looked up rather than recomputed|
|Permutation|An ordering of all elements; a string of length `n` has `n!` of them|
|Factorial (`n!`)|`n × (n-1) × ... × 1`|

## Flashcards

- What is the runtime of recursive factorial? :: `O(n)`: a straight line of `n` calls with constant work each #card
- How many calls does generating all permutations of a string of length `n` make (upper bound)? :: At most `n · n!` calls (n! leaves, each on a path of length n) #card
- What is the total runtime of the permutation generator? :: At most `O(n² · n!)`: `n·n!` calls × `O(n)` work each (printing and string concatenation) #card
- What is the runtime of naive recursive `fib(n)`? :: `O(2ᴺ)`; the tighter bound is about `O(1.6ᴺ)` #card
- Why isn't `allFib(n)` (naive fib called for 0..n) `O(n · 2ⁿ)`? :: `fib(i)` costs `2ⁱ`, not `2ⁿ`; the sum `2¹ + ... + 2ⁿ` is `O(2ⁿ)` #card
- What is memoization and what does it do to the Fibonacci runtime? :: Caching previously computed results so each is computed once; it turns exponential `O(2ⁿ)` into `O(n)` #card
- What is the runtime of `powersOf2(n)` (recursing on `n/2`)? :: `O(log n)`: each call halves `n`; it prints about `log n` powers of 2 #card
- What are three ways to analyze `powersOf2`? :: Trace what it does (halve until 1), reason about what it means (one print per power of 2 ≤ n), and look at the rate of increase (+1 call each time `n` doubles) #card

## Open questions

- [ ] How exactly does the `O(1.6ᴺ)` bound for Fibonacci arise?
- [ ] What is the tighter bound for the permutation generator, beyond `O(n² · n!)`?
- [ ] How do I recognize when an exponential recursion can be fixed with memoization or dynamic programming?


→ Next: [[REFERENCE]]