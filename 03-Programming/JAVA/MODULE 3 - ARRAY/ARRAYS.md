---
title: ARRAYS
date: 2026-10-10
language: Java
phase: phase-1
tags:
links: []
status: learning
---
> **One-line summary:** Arrays are the simplest way to store data: fast to **insert** into when unordered (O(1)), fast to **search** when ordered (O(log N) with binary search), but **slow to delete** in both cases (O(N), because items must shift) and **fixed in size**. This chapter uses arrays to introduce the three core operations (insert / search / delete), how to wrap a structure in a class with a clean **interface** (abstraction), **binary search**, **logarithms**, and **Big O notation**. The limits of arrays are the reason the rest of the book exists.

## What it is

Chapter 2 of _Data Structures and Algorithms in Java_ (Lafore). The array is the most commonly used storage structure and is built into most languages, so it is a convenient launch pad for introducing data structures and for seeing how object-oriented programming and data structures relate.

The chapter is split into four linked notes:

| Note                                  | Covers                                                                           | Question it answers                                                   |
| ------------------------------------- | -------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| [[ARRAY BASICS AND OPERATIONS]]       | Array Workshop applet, duplicates, Java array syntax, `array.java`               | What do insert, search and delete actually do, and what do they cost? |
| [[CLASSES AND INTERFACES FOR ARRAYS]] | `LowArray`, class interfaces, `HighArray`, abstraction, storing `Person` objects | How do I wrap an array in a class with a good interface?              |
| [[ORDERED ARRAYS AND BINARY SEARCH]]  | Ordered applet, binary search, `OrdArray`                                        | How do I make searching fast, and what does it cost?                  |
| [[LOGARITHMS AND BIG O NOTATIONS]]    | Logarithms, powers of two, Big O, running-time table                             | How do I measure and compare algorithm efficiency?                    |

This overview note holds the chapter-level comparison, the "why not use arrays for everything?" discussion and the chapter summary.

## Structure

### 1. The storyline of the chapter

1. **Three operations** matter for every structure: insert, search, delete. ([[ARRAY BASICS AND OPERATIONS]])
2. A plain array is easy but awkward to use directly, so **wrap it in a class** and hide the indices behind `insert` / `find` / `delete`. That is **abstraction**. ([[CLASSES AND INTERFACES FOR ARRAYS]])
3. If you keep the array **sorted**, you can use **binary search**, which is far faster. Insertion becomes the price. ([[ORDERED ARRAYS AND BINARY SEARCH]])
4. To compare algorithms honestly you need **logarithms and Big O**, which ignore constants and describe growth. ([[LOGARITHMS AND BIG O NOTATIONS]])
5. Even the best array still has weaknesses. That motivates other structures.

### 2. Unordered vs. ordered at a glance

|Operation|Unordered array|Ordered array|
|---|---|---|
|Search|O(N): linear, ~N/2 average|**O(log N)**: binary search|
|Insert|**O(1)**: put at `a[nElems]`|O(N): find the spot, shift larger items up|
|Delete|O(N): find + shift down|O(N): find (binary) + shift down|
|Best for|Frequent inserts, rare searches|Frequent searches, rare inserts/deletes|
|Example|(simple log of arrivals)|Employee database|
|Poor fit|Frequent lookups|Retail inventory (constant inserts/deletes)|

Detailed averages (N items, comparisons / moves):

| |No duplicates|Duplicates OK|
|---|---|---|
|Search|N/2 comparisons|N comparisons|
|Insertion|no comparisons, one move|no comparisons, one move|
|Deletion|N/2 comparisons, N/2 moves|N comparisons, more than N/2 moves|

### 3. Why not use arrays for everything?

Arrays get the job done, so why not use them for all storage? Two reasons.

**Reason 1: some operation is always slow.**

- **Unordered array:** insertion is quick, **O(1)**, but searching is slow, **O(N)**.
- **Ordered array:** searching is quick, **O(log N)**, but insertion is slow, **O(N)**.
- **Both kinds:** deletion is **O(N)**, because on average half the items must be moved to fill the hole.

It would be nice to have a structure that does _everything_ (insertion, deletion and searching) quickly: ideally in **O(1)**, otherwise **O(log N)**. In the chapters ahead we'll see how closely this ideal can be approached, and **the price that must be paid in complexity**.

**Reason 2: the size is fixed at creation (`new`).**

- You rarely know in advance how many items will arrive, so you **guess** the size.
- Guess **too large**: wasted memory (cells never filled).
- Guess **too small**: the array **overflows**, which causes at best an error message to the user and at worst a crash.

**Two ways around the fixed size:**

- **Other structures are more flexible** and expand to hold however many items are inserted. The **linked list** (Chapter 5) is one.
- **Java's `Vector` class** acts much like an array but is **expandable**. The extra capability costs some efficiency.

### 4. Make your own expandable array (the book's suggestion)

You might build your own "vector" class. When `insert()` is about to overflow the internal array, it:

1. creates a **new, larger array**,
2. **copies** the old contents into it,
3. **inserts** the new item.

All of this is **invisible to the class user**. That's abstraction again: the interface stays `insert()`, while the internals change.

```java
// A sketch of the idea (beyond the chapter): grow by doubling when full
public void insert(long value) {
    if (nElems == a.length) {                       // full?
        long[] bigger = new long[a.length * 2];     // 1. create a larger array
        for (int j = 0; j < nElems; j++)            // 2. copy the old contents
            bigger[j] = a[j];
        a = bigger;                                 //    switch to the new array
    }
    a[nElems] = value;                              // 3. insert the new item
    nElems++;
}
```

_(Beyond the chapter: this is how `java.util.ArrayList` works, and doubling is what makes appends cheap on average, a topic called amortized time in [[VI - Big O]].)_

### 5. Chapter summary (key takeaways)

- Arrays in Java are **objects**, created with the `new` operator.
- **Unordered arrays** offer fast insertion but slow searching and deletion.
- **Wrapping an array in a class** protects it from being inadvertently altered.
- A **class interface** is composed of the methods (and occasionally fields) that the class user can access.
- A class interface can be designed to make things **simple for the class user**.
- A **binary search** can be applied to an ordered array.
- The **logarithm to the base B of a number A** is (roughly) the number of times you can divide A by B before the result is less than 1.
- **Linear searches** require time proportional to the number of items in an array.
- **Binary searches** require time proportional to the logarithm of the number of items.
- **Big O notation** provides a convenient way to compare the speed of algorithms: O(1) is excellent, O(log N) good, O(N) fair, O(N²) poor.

## Mechanics / reference

### Big O of everything in this chapter

|Algorithm|Running time|
|---|---|
|Linear search|O(N)|
|Binary search|O(log N)|
|Insertion in unordered array|O(1)|
|Insertion in ordered array|O(N)|
|Deletion in unordered array|O(N)|
|Deletion in ordered array|O(N)|

### Choosing between them

|If you mostly...|Prefer|
|---|---|
|Add items and rarely look them up|Unordered array (O(1) insert)|
|Look items up and rarely change the data|Ordered array (O(log N) search)|
|Add and remove constantly _and_ look up constantly|Neither array is great, a motivation for the structures in later chapters|
|Don't know how many items you'll have|A resizable structure (linked list, `Vector`-style class)|

### Array code at a glance (all four notes)

| Class            | Key idea                                                        | Where                                 |
| ---------------- | --------------------------------------------------------------- | ------------------------------------- |
| `ArrayApp`       | Procedural insert / find / delete in `main()`                   | [[ARRAY BASICS AND OPERATIONS]]       |
| `LowArray`       | Private array, `setElem` / `getElem` (low-level)                | [[CLASSES AND INTERFACES FOR ARRAYS]] |
| `HighArray`      | `insert` / `find` / `delete` / `display`, class tracks `nElems` | [[CLASSES AND INTERFACES FOR ARRAYS]] |
| `ClassDataArray` | Same, for `Person` objects (`equals()` on the key)              | [[CLASSES AND INTERFACES FOR ARRAYS]] |
| `OrdArray`       | Sorted array, binary-search `find()`, shifting `insert`         | [[ORDERED ARRAYS AND BINARY SEARCH]]  |

## Pitfalls

- **Picking the array type without thinking about the workload.** Frequent inserts favour unordered; frequent lookups favour ordered.
- **Expecting an ordered array to be fast overall.** Only search is fast. Insertion and deletion are still O(N).
- **Forgetting the fixed size.** An array can't grow; plan for overflow or resize manually.
- **Believing "deletion can be made fast" in either array.** Shifting is inherent to contiguous storage with no holes.
- **Comparing algorithms by exact counts.** Use Big O: it's about growth with N.
- **Exposing the array and its indices to users.** A good interface hides them (abstraction).
- **Ignoring duplicates when designing a structure.** They change the cost of search, insert and delete.

## Flashcards

- Why do arrays make a good starting point for studying data structures? :: They're familiar, built into most languages, and show how OOP and data structures relate #card
- What is the best-case (ideal) goal for a data structure's operations? :: O(1) for insertion, deletion and searching, or otherwise O(log N) #card
- Which operation is fast in an unordered array, and which are slow? :: Insertion is fast (O(1)); searching and deletion are slow (O(N)) #card
- Which operation is fast in an ordered array, and which are slow? :: Searching is fast (O(log N)); insertion and deletion are slow (O(N)) #card
- Why is deletion O(N) in both kinds of array? :: On average half the items must be shifted to fill the hole #card
- What is the main limitation of array size in Java? :: It's fixed at creation with `new` #card
- What are the consequences of guessing an array size too large or too small? :: Too large wastes memory; too small overflows (an error message or a crash) #card
- Which structure from a later chapter can expand to hold more items? :: The linked list (Chapter 5) #card
- Which built-in Java class acts like an expandable array? :: `Vector`, at the cost of some efficiency #card
- How would you make your own expandable array class? :: When insertion would overflow, create a larger array, copy the old contents, then insert the new item (invisible to the user) #card
- Why does wrapping an array in a class help? :: It protects the array from being inadvertently altered and hides how it works #card
- What is a class interface? :: The methods (and occasionally fields) of a class that its user can access #card
- When should you prefer an ordered array? :: When searches are frequent and insertions/deletions are rare (e.g., an employee database) #card
- When is an ordered array a poor choice? :: When data changes constantly, like store inventory #card
- How is the base-B logarithm of A described intuitively? :: Roughly the number of times you can divide A by B before the result is less than 1 #card
- Time order of linear vs. binary search? :: Linear: proportional to N. Binary: proportional to log N #card
- What does Big O give you? :: A convenient way to compare the speed of algorithms by how time grows with N #card

## Open questions

- [ ] Which structure gives fast insert **and** fast search? (Later chapters.)
- [ ] How do sorting algorithms put an unordered array into order, and at what cost? (Chapter 3.)
- [ ] How do stacks and queues use arrays internally? (Chapter 4.)
- [ ] How does a linked list avoid the shifting cost and the fixed size? (Chapter 5.)
- [ ] How does recursion express binary search? (Chapter 6.)
- [ ] When does `Vector`/`ArrayList` resizing hurt performance?

## Key terms

|Term|Definition|
|---|---|
|Array|Fixed-size block of consecutive cells accessed by index|
|Unordered array|Items in arbitrary order; fast insert, slow search|
|Ordered array|Items sorted by key; fast search (binary), slow insert|
|Class interface|The public methods through which a class is used|
|Abstraction|Separating what a class does from how it does it|
|Linear search|One-by-one scan, O(N)|
|Binary search|Halving search on sorted data, O(log N)|
|Logarithm|Inverse of exponentiation; how many halvings reach 1|
|Big O notation|Description of how running time grows with N|
|`Vector`|Java class acting like an expandable array|
|Linked list|A flexible, expandable structure (Chapter 5)|
