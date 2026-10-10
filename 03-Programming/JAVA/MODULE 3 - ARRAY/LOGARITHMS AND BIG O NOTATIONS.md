---
title: LOGARITHMS AND BIG O NOTATIONS
date: 2026-10-10
language:
phase: phase-1
tags:
links: []
status: learning
---
> **One-line summary:** A binary search on N items needs about **log₂ N** comparisons, the number of times you can **halve** N before you reach 1. Every ×10 in size adds only about **3.3** steps. **Big O notation** captures this kind of growth: it says how an algorithm's running time changes with the number of items N, and it **drops the constant factor K** (and the log base) because what matters is the _shape_ of the growth, not the machine. Linear search is **O(N)**, binary search **O(log N)**, unordered insertion **O(1)**, and ordered insertion and both deletions **O(N)**.

## What it is

This part answers the question hanging over the whole chapter: _how much faster is binary search, and how do we talk about "faster" in a way that's meaningful?_

|Topic|Question it answers|
|---|---|
|Comparisons in binary search|How many steps for 10, 100, a million items?|
|Powers of two|How do I compute that without simulating the search?|
|Logarithm|What is log₂ N, intuitively?|
|Big O notation|How do I describe an algorithm's efficiency independently of the machine?|
|Running times|What is the Big O of each operation so far?|

This is **part 4 of 4** of Chapter 2. See [[DSA 2 - Arrays]] for the overview. For a deeper treatment of Big O rules (non-dominant terms, add vs. multiply, amortized time, recursion), see [[VI - Big O]].

## Structure

### 1. How many comparisons does binary search need?

Binary search finds a record in an array of 100 in **at most 7** comparisons. For other sizes:

**Table 2.3: Comparisons needed in binary search**

|Range (N)|Comparisons needed|
|---|---|
|10|4|
|100|7|
|1,000|10|
|10,000|14|
|100,000|17|
|1,000,000|20|
|10,000,000|24|
|100,000,000|27|
|1,000,000,000|30|

**Compare with linear search** (average N/2 comparisons):

|N|Linear (avg)|Binary (max)|
|---|---|---|
|10|5|4|
|100|50|7|
|1,000|500|10|
|1,000,000|500,000|20|

For very small N the difference isn't dramatic. But **the more items there are, the bigger the gap**. For all but very small arrays, binary search is greatly superior.

> **Memorize this pattern:** every time you multiply the number of items by **10**, binary search adds only **3 or 4** steps (actually 3.322, then rounded). That's because as a number grows, its logarithm grows far more slowly. A billion items need only 30 comparisons.

### 2. The equation: doubling and powers of two

Verify Table 2.3 by repeatedly **halving** the range until it can't be divided further. The count of halvings = comparisons.

Equivalently, flip it around: start from a range of **1** and **double** it each step, to see the largest range you can cover with `s` steps.

**Table 2.4: Powers of two**

|Step s (= log₂ r)|Range r|As a power of 2 (2ˢ)|
|---|---|---|
|0|1|2⁰|
|1|2|2¹|
|2|4|2²|
|3|8|2³|
|4|16|2⁴|
|5|32|2⁵|
|6|64|2⁶|
|7|128|2⁷|
|8|256|2⁸|
|9|512|2⁹|
|10|1024|2¹⁰|

Check against the original problem: for a range of **100**, 6 steps give only 64 (not enough) but 7 steps give 128 (enough). So **7** comparisons are correct, and for a range of **1000**, **10** steps (1024) are correct.

**The formula (steps → range):**

> **r = 2ˢ**

where `s` = number of steps (times you double), `r` = range. For `s = 6`, `r = 2⁶ = 64`.

### 3. The opposite: logarithms

The real question is the reverse: **given the range r, how many steps s?**

The inverse of raising something to a power is a **logarithm**:

> **s = log₂(r)**

- The **base-2 logarithm** of a number `r` is the number of times you must multiply 2 by itself to get `r`.
- In Table 2.4, the first column `s` is exactly `log₂(r)`.
- Intuitive version: **log₂ N is (roughly) the number of times you can divide N by 2 before the result is less than 1.** Example for 100: 100 → 50 → 25 → 12.5 → 6.25 → 3.125 → 1.5625 → 0.78 is **7 divisions**.

**Computing it:** calculators and most languages have a `log` function, usually base 10. Convert to base 2 by multiplying by **3.322**:

- log₁₀(100) = 2, so log₂(100) = 2 × 3.322 = **6.644**.
- Round **up** to the whole number **7** (you can't do 0.644 of a comparison). That's the 7 in the tables.

The point isn't to calculate logarithms by hand. What matters is the **relationship**: as N grows, log N grows very slowly.

_(Beyond the chapter: in Java, `Math.log(n) / Math.log(2)` gives log₂ n, and `32 - Integer.numberOfLeadingZeros(n - 1)` gives ⌈log₂ n⌉ for n ≥ 2.)_

### 4. Big O notation

A shorthand for how efficient an algorithm is. It's like classifying cars as "compact" or "midsize" without quoting exact dimensions.

**Why not just say "A is twice as fast as B"?** Because the proportion can change radically as N changes. At 50% more items A might be three times as fast; with half as many items they might be equal. What you need is a comparison that tells how an algorithm's speed **relates to the number of items**.

Let `T` = running time and `N` = number of items. Look at each algorithm so far.

#### Insertion in an unordered array: constant

It doesn't depend on N. The item always goes to `a[nElems]`.

```
T = K
```

`K` is a constant that bundles up everything machine-specific (processor speed, compiler efficiency, ...). To find `K` in reality you would time one insertion.

#### Linear search: proportional to N

On average, half the items are compared:

```
T = K * N / 2
```

Lump the `2` into `K` (new K = old K / 2):

```
T = K * N
```

The average linear-search time is **proportional** to the array size: twice as big means twice as long.

#### Binary search: proportional to log N

```
T = K * log₂(N)
```

Any logarithm is related to any other by a constant (3.322 to go from base 2 to base 10), so fold it into `K` too. Now **the base doesn't need to be specified**:

```
T = K * log(N)
```

#### Don't need the constant

Big O notation looks like those formulas but **drops the constant K**. When comparing algorithms you don't care about the particular chip or compiler. You want to know **how T changes as N changes**, not the actual numbers.

- The uppercase letter **O** means "**order of**".
- Linear search takes **O(N)** time.
- Binary search takes **O(log N)** time.
- Insertion into an unordered array takes **O(1)**, or **constant time** (the numeral 1 in the parentheses).

**Table 2.5: Running times in Big O**

|Algorithm|Running time|
|---|---|
|Linear search|O(N)|
|Binary search|O(log N)|
|Insertion in unordered array|O(1)|
|Insertion in ordered array|O(N)|
|Deletion in unordered array|O(N)|
|Deletion in ordered array|O(N)|

#### How good is each?

Figure 2.9 graphs number of steps against number of items. A (very subjective) rating:

|Big O|Name|Rating|Where you've seen it|
|---|---|---|---|
|O(1)|Constant|**Excellent**|Insert into unordered array|
|O(log N)|Logarithmic|**Good**|Binary search|
|O(N)|Linear|**Fair**|Linear search, delete, ordered insert|
|O(N²)|Quadratic|**Poor**|Bubble sort (next chapter), certain graph algorithms|

The graph shows the O(N²) curve shooting off the chart almost immediately, O(N) rising steadily, O(log N) flattening out, and O(1) staying flat.

> The idea of Big O isn't to give actual running times. It's to convey **how the running time is affected by the number of items**. That is the most meaningful way to compare algorithms (short of measuring running times in a real installation).

## Mechanics / reference

### Growth at a glance

Steps for various N (ignoring the constant K; log₂ values rounded up):

|N|O(1)|O(log N)|O(N)|O(N²)|
|---|---|---|---|---|
|10|1|4|10|100|
|100|1|7|100|10,000|
|1,000|1|10|1,000|1,000,000|
|1,000,000|1|20|1,000,000|10¹²|

### How the formulas become Big O

|Raw formula|Simplify (fold constants into K)|Big O|
|---|---|---|
|`T = K`|already constant|**O(1)**|
|`T = K * N / 2`|`K * N`|**O(N)**|
|`T = K * log₂(N)`|`K * log(N)` (base is a constant)|**O(log N)**|

### Quick rules of thumb for these algorithms

|If the algorithm...|It's probably...|
|---|---|
|Does a fixed number of steps regardless of N|O(1)|
|Walks through (up to) all N items once|O(N)|
|Repeatedly **halves** the remaining range|O(log N)|
|Shifts (on average) half the items|O(N)|

### Common patterns

```java
// How many halvings until the range is gone? (≈ binary search comparisons)
int steps = 0;
for (long r = n; r >= 1; r /= 2) steps++;
// n = 100 → 7, n = 1000 → 10, n = 1_000_000 → 20

// Exact formula (base conversion): ceil(log2(n))
int steps2 = (int) Math.ceil(Math.log(n) / Math.log(2));
```

_(Beyond the chapter: the version above gives `floor(log₂ n) + 1`, which is the exact worst-case count of comparisons. The two agree except when `n` is a power of two.)_

## Pitfalls

- **Saying "A is twice as fast as B".** The ratio changes with N. Compare growth rates (Big O) instead.
- **Keeping constants in Big O.** O(N/2) or O(2N) is just O(N). O(log₂ N) and O(log₁₀ N) are both O(log N).
- **Treating O(N) as "exactly N steps".** Big O describes how time scales, not exact counts. Linear search averages N/2 but is still O(N).
- **Assuming Big O predicts actual seconds.** For small N, an O(N) algorithm with a tiny constant can beat an O(log N) one. Big O matters as N grows.
- **Forgetting to round up** when counting comparisons. 6.644 comparisons means **7**.
- **Mixing up O(1) with "one step".** O(1) means the time doesn't grow with N (it could be 5 steps or 50, as long as it's fixed).
- **Thinking log N grows like N.** It grows _extremely_ slowly: a billion items needs only about 30 steps.
- **Confusing "ordered insertion is O(N)" with "ordered search is O(N)".** Search is O(log N) with binary search. Insertion pays for the **shifting**.
- **Ignoring that deletion is O(N) in both array types.** Neither array makes deletion fast.

## Flashcards

- Why is binary search better than linear search for large arrays? :: Each comparison halves the range, so it needs only about log₂ N comparisons instead of N/2 #card
- How many comparisons does binary search need for 100 items? :: 7 #card
- For 1,000 items? :: 10 #card
- For 1,000,000 items? :: 20 #card
- For 1,000,000,000 items? :: 30 #card
- How many steps does multiplying N by 10 add to binary search? :: About 3 or 4 (3.322) #card
- What is the formula relating steps `s` and range `r`? :: `r = 2^s` #card
- What is the inverse formula? :: `s = log₂(r)` #card
- What is a logarithm (base 2) in simple terms? :: The number of times you must multiply 2 by itself to get r (or roughly how many times you can halve r before reaching below 1) #card
- How do you convert a base-10 logarithm to base 2? :: Multiply by 3.322 #card
- What is log₂(100)? :: About 6.644, rounded up to 7 comparisons #card
- Why round the number of comparisons up? :: You can't make a fraction of a comparison #card
- Why do 6 steps fail and 7 succeed for a range of 100? :: 2⁶ = 64 < 100, while 2⁷ = 128 ≥ 100 #card
- What is the worst-case number of binary search steps for 1000 items? :: 10, since 2¹⁰ = 1024 ≥ 1000 #card
- Why is "A is twice as fast as B" a weak statement? :: The ratio changes as the number of items changes #card
- What does Big O notation tell you? :: How an algorithm's running time scales with the number of items N #card
- What does the "O" stand for? :: "Order of" #card
- What does Big O do with the constant K? :: Drops it; it only reflects machine/compiler factors, not growth #card
- What is the Big O of linear search? :: O(N) #card
- Why is linear search O(N) even though it averages N/2? :: The factor 1/2 is a constant and is folded into K #card
- What is the Big O of binary search? :: O(log N) #card
- Why doesn't the log base matter in O(log N)? :: Any two logarithm bases differ by a constant factor, which is absorbed into K #card
- What is the Big O of insertion in an unordered array? :: O(1), constant time #card
- What is the Big O of insertion in an ordered array? :: O(N) (items must be shifted) #card
- What is the Big O of deletion in unordered and ordered arrays? :: O(N) for both #card
- Rank O(1), O(log N), O(N), O(N²) from best to worst. :: O(1) excellent, O(log N) good, O(N) fair, O(N²) poor #card
- Which algorithm in the next chapter is O(N²)? :: Bubble sort #card
- What is the point of Big O? :: To show how running time is affected by the number of items, not to give exact times #card

## Open questions

- [ ] What is the Big O of the sorting algorithms in the next chapter?
- [ ] How do add vs. multiply and dropping non-dominant terms work in Big O? (See [[VI - Big O]].)
- [ ] What is the difference between average-case and worst-case Big O?
- [ ] Which data structures achieve O(log N) for search, insert and delete together? (Later chapters.)
- [ ] Is there any structure with O(1) search? (Hash tables, later.)
- [ ] How does recursion relate to the "halving" idea? (Recursion chapter.)

## Key terms

|Term|Definition|
|---|---|
|Logarithm (base 2)|The inverse of raising 2 to a power; how many times you must double (or can halve) to reach r|
|`r = 2^s`|Range covered by `s` doublings|
|`s = log₂(r)`|Number of steps needed for range `r`|
|Big O notation|A measure of how an algorithm's time grows with the number of items N|
|Constant `K`|Machine- and implementation-specific factor, dropped in Big O|
|O(1)|Constant time (independent of N)|
|O(log N)|Logarithmic time (binary search)|
|O(N)|Linear time (linear search, shifting)|
|O(N²)|Quadratic time (e.g., bubble sort)|
|Proportional|Changing N by a factor changes T by the same factor (`T = K * N`)|

## Related

← Previous: [[DSA 2.3 - Ordered Arrays and Binary Search]] ↑ Up: [[DSA 2 - Arrays]] See also: [[VI - Big O]]