---
title: ARRAY BASICS AND OPERATIONS
date: 2026-10-10
language: Java
phase: phase-1
tags:
links: []
status: learning
---
> **One-line summary:** An array stores items in consecutive cells reached by an **index**. Every structure in this book is judged by three operations: **insert**, **search**, **delete**. In an **unordered** array, insertion is one step (drop the item at position `nElems`), but searching averages **N/2** steps and deletion costs about **N** steps, because everything above the hole must **shift down** so no gaps ("holes") are left. In Java, an array is an **object**: created with `new`, 0-indexed, **fixed size**, with a `length` field, and auto-initialized to 0 / `null`.

## What it is

An **array** is the most common data storage structure, and it is built into almost every language. That makes it the natural starting point: you already know how it behaves, so you can focus on _how to judge a data structure_ (how many steps do insert, search and delete take?) and _how to wrap it in a class_ (next note).

|Topic|Question it answers|
|---|---|
|The three fundamental operations|What must every data structure be able to do?|
|Array Workshop (insert / find / delete)|What does each algorithm actually do, step by step?|
|Duplicates|How does allowing equal keys change the cost of each operation?|
|Arrays in Java|How do I create, index, size and initialize an array?|
|`array.java`|What does the whole thing look like as plain procedural code?|

This is **part 1 of 4** of Chapter 2. See [[DSA 2 - Arrays]] for the chapter overview.

## Structure

### 1. The three fundamental operations

Imagine a kids' baseball coach tracking who is at practice. The program needs to:

1. **Insert** a player when they arrive.
2. **Search** for a player (is number 678 here?).
3. **Delete** a player when they go home.

These three operations are the basics of almost every data storage structure in this book. Whenever a new structure appears, the questions are always the same: _how fast is insert, how fast is search, how fast is delete?_

> **How "time" is measured here:** by the **number of steps** an algorithm takes, not by seconds. More steps means longer. (In the book's Workshop applets, one button press = one step.)

### 2. The Array Workshop: what each algorithm does

Picture an array of 20 cells with 10 filled. The filled cells are always **contiguous from index 0** (no gaps). A variable (`nElems`) tracks how many are filled, so the **next free cell is always `a[nElems]`**.

#### Insertion: 1 step

The new item goes into the first empty cell. The algorithm knows where that is because it knows how many items there are.

```
index:       0     1   ...     9    10    11
before:     80   181   ...   205     .     .     nElems = 10
after:      80   181   ...   205   678     .     nElems = 11
                                     ^
                       678 goes into a[10]
```

No searching, no moving. The time does **not depend on N**.

#### Searching: N/2 steps on average

Start at cell 0 and examine cells one by one until the key matches. This is a **linear search**.

- Item near the front: found quickly. Item near the back: found late.
- **Average** (no duplicates): about **N/2** comparisons.
- **Worst case:** the item is in the last cell, so **N** comparisons.
- **Key not in the array:** every occupied cell is examined, so **N** comparisons before giving up.

#### Deletion: about N steps

You must **find** the item first (about N/2 comparisons), and then you must **close the hole**.

**No holes allowed.** A hole is an empty cell with filled cells above it. If holes were allowed, every algorithm would need to check "is this cell empty?" before reading it, and would waste time walking over empty cells. So the filled cells must stay contiguous.

So after removing the item, every item at a higher index shifts **down one cell**:

```
index:      0    1    2    3    4    5    6    7    8    9
before:    84   61   15   73   26 [38]   11   49   53   32
                                     ^    <    <    <    <
        delete 38, shift the 4 items above it down
after:     84   61   15   73   26   11   49   53   32
        nElems goes from 10 to 9
```

On average, half the items are above the deleted one, so about **N/2 moves**. Total: **N/2 comparisons + N/2 moves ≈ N steps**.

**Why searching and deleting are "not so fast":** apart from insertion, all the algorithms walk through some or all of the cells. Much faster (but more complex) structures exist, and the rest of the book is about them.

### 3. The duplicates issue

When you design a storage structure you must decide: **can two items have the same key?**

- Key = employee number: duplicates make no sense.
- Key = last name: duplicates are very likely, so they must be allowed.
- Baseball players' shirt numbers: duplicates should **not** be allowed (you couldn't tell players apart).

The decision changes the cost of every operation.

**Searching with duplicates.** Even after finding a match you may need to keep going to the last occupied cell to find all other matches. Whether you must depends on the question:

- "Find **me everyone** with blue eyes" → scan everything, always **N** steps.
- "Find **me someone** with blue eyes" → stop at the first match.

**Insertion with duplicates.** Still one step. But if duplicates are **not** allowed and the user might type the same key twice, a careful program has to check **all N existing items first**, which turns insertion from 1 step into **N** steps. The Workshop applet skips that check for speed and trusts the user.

**Deletion with duplicates.** It depends on what "delete" means:

- Delete only the **first** item with that key: same as before, N/2 comparisons and N/2 moves.
- Delete **every** item with that key: you must check all N cells and (probably) move more than N/2 cells. It is also messier because each time one item is removed, the items above it must shift **one further** than before. If 3 items are deleted, items beyond the last one shift 3 places. (The applet uses this second meaning.)

**Table 2.1: Duplicates OK vs. no duplicates (average cost, N items)**

| |No duplicates|Duplicates OK|
|---|---|---|
|Search|N/2 comparisons|N comparisons|
|Insertion|No comparisons, one move|No comparisons, one move|
|Deletion|N/2 comparisons, N/2 moves|N comparisons, more than N/2 moves|

(Inserting one item counts as one move.) The difference between N and N/2 rarely matters. What matters far more is whether an operation takes **1** step, **log N** steps, **N** steps, or **N²** steps. That is the idea behind Big O, covered in [[LOGARITHMS AND BIG O NOTATIONS]].

### 4. Arrays in Java

Java arrays use C/C++-like syntax, but with a few differences.

#### Creating an array

In many languages arrays are primitive types. **In Java, arrays are objects**, so you must use `new`:

```java
// declares a REFERENCE to an array (no array exists yet)
int[] intArray;
// creates the array, and sets intArray to refer to it
intArray = new int[100];

int[] intArray = new int[100];   // the same, in one statement
// alternative syntax ([] after the name)
int intArray[] = new int[100];
```

- The `[]` after the **type** (`int[]`) is preferred, because it makes clear the brackets are **part of the type**, not the name.
- Because an array is an object, **`intArray` is a reference** (an address). It is **not** the array itself. The array lives elsewhere in memory, and `intArray` just holds its address.

```
intArray ---> [ 0 | 0 | 0 | ... | 0 ]
(reference)   100 ints, stored elsewhere in memory
```

- Every array has a **`length`** field (note: a field, not a method):

```java
int arrayLength = intArray.length;   // 100
```

- **Size is fixed** once created. You cannot grow or shrink an array.

#### Accessing elements

Use an index in square brackets. The first element is **0**, so an array of 10 elements has indices **0 to 9**.

```java
temp = intArray[3];     // reads the FOURTH element
intArray[7] = 66;       // writes into the EIGHTH cell
```

An index below 0 or above `length - 1` throws an **Array Index Out of Bounds** runtime error.

_(Beyond the chapter: the actual exception is `ArrayIndexOutOfBoundsException`.)_

#### Initialization

- A newly created array of integers is **automatically filled with 0**. (Unlike C++, this is true even for arrays created inside a method.)
- An array of **objects** starts with every element `null`:

```java
// 4000 references, all null
autoData[] carArray = new autoData[4000];
```

Until you assign real objects, each element holds `null`. Using a `null` element (calling a method on it) causes a runtime error, so **always assign an element before using it**.

_(Beyond the chapter: that error is a `NullPointerException`. Also, only the 4000 **references** exist at first. No `autoData` objects have been created yet.)_

- To start with values other than 0, use an **initialization list**:

```java
int[] intArray = { 0, 3, 6, 9, 12, 15, 18, 21, 24, 27 };
```

This one statement replaces both the declaration and the `new`. **The size is set by the number of values in the list** (here, 10).

### 5. `array.java`: everything in one method

First the old-fashioned procedural version: a single class with a single `main()`. It stores 10 numbers, searches for 66, deletes 55, and prints.

```java
class ArrayApp {
    public static void main(String[] args) {
        long[] arr;                 // reference to array
        arr = new long[100];        // make array
        int nElems = 0;             // number of items
        int j;                      // loop counter
        long searchKey;             // key of item to search for

        // insert 10 items
        arr[0] = 77;  arr[1] = 99;  arr[2] = 44;
        arr[3] = 55;  arr[4] = 22;
        arr[5] = 88;  arr[6] = 11;  arr[7] = 00;
        arr[8] = 66;  arr[9] = 33;
        nElems = 10;                // now 10 items in array

        // display items
        for (j = 0; j < nElems; j++)
            System.out.print(arr[j] + " ");
        System.out.println("");

        // find item with key 66
        searchKey = 66;
        // for each element,
        for (j = 0; j < nElems; j++)
            //   found item?
            if (arr[j] == searchKey)
                //   yes, exit before end
                break;
        // at the end?
        if (j == nElems)
            // yes
            System.out.println("Can't find " + searchKey);
        else
            // no
            System.out.println("Found " + searchKey);

        // delete item with key 55
        searchKey = 55;
        // look for it
        for (j = 0; j < nElems; j++)
            if (arr[j] == searchKey)
                break;
        // move higher ones down
        for (int k = j; k < nElems - 1; k++)
            arr[k] = arr[k + 1];
        // decrement size
        nElems--;

        // display items
        for (j = 0; j < nElems; j++)
            System.out.print(arr[j] + " ");
        System.out.println("");
    }
}
```

Output:

```
77 99 44 55 22 88 11 0 66 33
Found 66
77 99 44 22 88 11 0 66 33
```

**How each operation maps to code:**

|Operation|Code idea|
|---|---|
|Insert|`arr[0] = 77;` normal array syntax, plus keep `nElems` up to date|
|Search|Loop `j` over `0..nElems-1` comparing `arr[j]` with `searchKey`. If the loop **ends with `j == nElems`**, nothing matched.|
|Delete|Search first. Then copy each higher element one cell down (`arr[k] = arr[k+1]`), then `nElems--`|
|Display|Loop over `0..nElems-1` and print `arr[j]`|

**Notes on the design choices:**

- Data type is **`long`** to make clear it is _data_, while **`int`** is used for _indices_.
- A primitive type keeps the code simple. In real programs the items usually have several fields, so they are **objects** (see [[CLASSES AND INTERFACES FOR ARRAYS]]).
- The delete code **assumes the item is present** (the book admits this is rash). A real program would handle "not found".
- **Program organization is poor:** one class, one method, everything inline. The next note fixes that with classes.

## Mechanics / reference

### Array facts at a glance

|Fact|Detail|
|---|---|
|Type|An **object**; the variable holds a **reference**|
|Creation|`new type[size]`, or an initialization list `{ ... }`|
|Size|`arr.length` (field). **Fixed** after creation|
|Indices|`0` to `length - 1`; anything else → runtime error|
|Default contents|Numbers: `0`. Object arrays: `null`|
|`length` vs `nElems`|`length` = **capacity** (cells that exist). `nElems` = **count** (cells actually in use). You track `nElems` yourself.|

### Cost of each operation (unordered array, N items)

|Operation|What happens|Steps|
|---|---|---|
|Insert|Put at `a[nElems]`, `nElems++`|**1** (constant)|
|Search (found)|Linear scan|**N/2** on average, N worst case|
|Search (not found)|Scan all occupied cells|**N**|
|Delete|Find it, then shift higher items down|**N/2** comparisons + **N/2** moves ≈ **N**|

### Common patterns

```java
// 1. Linear search; j == nElems means "not found"
int j;
for (j = 0; j < nElems; j++)
    if (arr[j] == searchKey) break;
boolean found = (j != nElems);

// 2. Insert at the end (unordered): O(1)
arr[nElems] = value;
nElems++;

// 3. Delete at index j: shift everything above it down by one
for (int k = j; k < nElems - 1; k++)
    arr[k] = arr[k + 1];
nElems--;

// 4. Display
for (int i = 0; i < nElems; i++)
    System.out.print(arr[i] + " ");
```

_(Beyond the chapter: the shifting loop can also be done with `System.arraycopy(arr, j + 1, arr, j, nElems - j - 1)`. Same idea, one call.)_

## Pitfalls

- **Looping to `length` instead of `nElems`.** You would process empty cells (zeros). Loop to the **count**, not the capacity.
- **Off-by-one on indices.** Valid indices are `0..length-1`. `arr[arr.length]` is out of bounds.
- **Deleting without shifting.** This leaves a **hole**, and every other algorithm would then have to check for empty cells. Always shift and decrement `nElems`.
- **Deleting a key that isn't there.** In `array.java`, if the key is missing, `j` ends at `nElems`, no shifting happens, but `nElems--` still runs, so the **last item is silently dropped**. Always check `j == nElems` before deleting.
- **Inserting into a full array.** `arr[nElems] = value` when `nElems == arr.length` is out of bounds. The book's code doesn't guard against it.
- **Declaring but not creating.** `int[] a;` makes only a reference. You need `new` before using it.
- **Using `null` elements.** In an object array every cell starts `null`. Calling a method on one fails at run time.
- **Thinking `b = a` copies an array.** It copies the **reference**, so both names point to the same array. _(Beyond the chapter.)_
- **Trying to resize an array.** Impossible. You must create a bigger array and copy.
- **Forgetting that duplicates change the algorithm.** "Find one" can stop early; "find all" must scan to the end.

## Flashcards

- What are the three fundamental operations of most data structures? :: Insertion, searching, and deletion #card
- How is the "time" of an algorithm measured in this book? :: By the number of steps it takes (more steps = longer) #card
- Where does insertion put a new item in the unordered array? :: In the first empty cell, `a[nElems]` #card
- Why is insertion in an unordered array only one step? :: The algorithm knows where the first empty cell is (from `nElems`), so no search or shifting is needed #card
- How many comparisons does a linear search take on average (no duplicates)? :: About N/2 #card
- What is the worst case for a linear search? :: N comparisons (item in the last cell, or not present at all) #card
- What happens in a linear search for a key that isn't in the array? :: Every occupied cell is examined (N comparisons) before it reports "not found" #card
- Why does deletion cost about N steps? :: About N/2 comparisons to find the item, plus about N/2 moves to shift higher items down #card
- What is a "hole" in an array? :: One or more empty cells that have filled cells above them (at higher indices) #card
- Why are holes not allowed? :: Every algorithm would have to check for empty cells and waste time on them, so occupied cells are kept contiguous #card
- After deleting the item at index 5, what happens to the item at index 6? :: It shifts down into index 5 (and so on up to the last occupied cell) #card
- How does allowing duplicates affect search? :: Search must continue to the end to find all matches, so it always takes N steps (unless you only want one match) #card
- How does disallowing duplicates affect insertion (if you guard against mistakes)? :: You would have to check all N items before inserting, turning a 1-step insert into N steps #card
- What does "delete" mean when duplicates are allowed? :: Either delete only the first match (N/2 comparisons and moves) or every match (N comparisons, more than N/2 moves) #card
- Are arrays primitive types or objects in Java? :: Objects, so they are created with `new` #card
- What does `int[] a;` actually create? :: Only a reference to an array. No array exists until you use `new` #card
- What does the array variable hold? :: The address (reference) of the array, not the array itself #card
- How do you get the size of an array? :: The `length` field: `arr.length` #card
- Can you change the size of a Java array after creating it? :: No, the size is fixed #card
- What is the index range for an array of 10 elements? :: 0 to 9 #card
- What happens if you use an index below 0 or above `length - 1`? :: An Array Index Out of Bounds runtime error #card
- What is an array of ints initialized to by default? :: 0 #card
- What are the elements of an array of objects initially? :: `null` #card
- What happens when you use an element that is still `null`? :: A runtime error (null pointer) #card
- What is an initialization list, and how is the size decided? :: `int[] a = {0, 3, 6};` The size equals the number of values in the list #card
- In `array.java`, how do you detect that a search failed? :: After the loop, `j == nElems` #card
- Why does the book use `long` for data and `int` for indices? :: To make it clear which is the data and which is an index #card
- What is the difference between `arr.length` and `nElems`? :: `length` is the array's capacity; `nElems` is how many cells are actually filled #card

## Open questions

- [ ] How is a binary search faster than a linear search, and what must be true of the array for it to work? 
- [ ] How do I wrap the array in a class so the user never touches indices?
- [ ] What exactly does "O(N)" mean, and why can I ignore the 2 in N/2?
- [ ] How should insert behave when the array is full?
- [ ] Which structures avoid having to shift items on deletion? (Later chapters.)

## Key terms

|Term|Definition|
|---|---|
|Array|A fixed-size block of consecutive cells accessed by index|
|Index|The 0-based position of a cell|
|`length`|Field giving the array's capacity|
|`nElems`|Variable (kept by the programmer) counting the items actually stored|
|Insertion|Adding a new item|
|Search|Finding an item with a given key|
|Deletion|Removing an item and closing the gap|
|Linear search|Checking cells one after another from the start|
|Key|The field used to identify/search for an item|
|Hole|An empty cell below filled cells (not allowed)|
|Duplicates|Items that share the same key|
|Initialization list|`{ ... }` syntax that creates and fills an array in one statement|
|Reference|A variable holding the address of an object (an array variable is one)|
|Procedural program|A program organized as plain code in one method rather than around classes|

## Related

→ Next: [[CLASSES AND INTERFACES FOR ARRAYS]]