---
title: ORDERED ARRAYS AND BINARY SEARCH
date: 2026-10-10
language:
phase: phase-1
tags:
links: []
status: learning
---
> **One-line summary:** An **ordered array** keeps its items sorted by key (smallest at index 0). That order lets you use **binary search**: look at the middle cell, throw away the half that cannot contain the key, repeat. Each step **halves** the range, so searching needs only about **log₂ N** comparisons instead of N/2. The price: **insertion becomes slow** (find the spot, then shift all larger items up), and **deletion** is still slow (shift down), same as the unordered array. Ordered arrays suit data that is searched often but changed rarely.

## What it is

An ordered array has the same interface as `HighArray` (`insert`, `find`, `delete`, `display`), but internally it keeps the data in ascending order. The reward is a much faster `find()`.

|Topic|Question it answers|
|---|---|
|Ordered array|What changes when the items are kept sorted?|
|Linear search (ordered)|How does order help even a simple scan?|
|Binary search|How can I find a key in about log N steps?|
|`find()` code|How exactly do `lowerBound`, `upperBound` and `curIn` work?|
|`OrdArray` class|How do insert and delete change when the array is sorted?|
|Pros and cons|When is an ordered array a good (or bad) choice?|

This is **part 3 of 4** of Chapter 2. See [[DSA 2 - Arrays]] for the overview.

## Structure

### 1. What is an ordered array?

Items are arranged in **ascending key order**: the smallest value is at index 0, and each cell holds a value **larger** than the cell below it.

```
index:  0    1    2    3    4    5    6    7    8    9
       [13] [34] [248][287][311][386][427][585][885][915]     ← always sorted
```

- **Inserting** now requires **finding the right location** (just above a smaller value, just below a larger one), and then **moving all the larger values up** to make room.
- In the book's ordered array, **duplicates are not allowed**. As seen in [[ARRAY BASICS AND OPERATIONS]], that speeds up searching a little but slows down insertion.
- **Why bother ordering?** Because it enables **binary search**, which is dramatically faster.

### 2. Linear search in an ordered array

Linear search works like before (check cell 0, 1, 2, ...), with one improvement: **it can quit early.**

> If you reach an item **larger** than the key, the key can't be further along (everything beyond is even larger), so stop and report "not found".

Example: searching for 400 in the array above. The search stops when it reaches **427**, the first value bigger than 400. Compare with the unordered array, where a missing key forces a scan of **all** N cells.

Deletion also quits partway through when the key isn't found (same reason), and otherwise shifts higher items down like before.

### 3. Binary search

The big payoff of order. It is "much faster than linear search, especially for large arrays."

#### The guess-a-number game

A friend thinks of a number from 1 to 100. After each guess she says **too high**, **too low** or **correct**. The best strategy is always to guess the **middle** of the remaining range:

- Guess 50. If "too low", the number is in 51–100, so guess 75.
- If "too high", the number is in 1–49, so guess 25.
- Each guess **halves** the range of possible values, until the range has one number.

A linear approach (1, then 2, then 3...) would take about **50** guesses on average. Halving takes at most **7**.

**Table 2.2: Guessing the number 33**

|Step|Guess|Result|Range of possible values|
|---|---|---|---|
|0|||1–100|
|1|50|Too high|1–49|
|2|25|Too low|26–49|
|3|37|Too high|26–36|
|4|31|Too low|32–36|
|5|34|Too high|32–33|
|6|32|Too low|33–33|
|7|33|Correct||

Seven guesses is the **maximum**. You might get lucky and hit the number before the range shrinks to one (e.g., if the number were 50 or 34).

#### Binary search in the Ordered applet

In the Ordered Workshop applet, a binary search shows a red arrow at the current guess and a vertical line marking the **range** still under consideration.

- Step 1: range is the whole array (cells 0–29 in the example), guess = middle (index 14, value 506).
- Step 2: the key (559) is larger, so the range becomes 15–29 and the next guess is index 22.
- Each press of **Find** halves the range. Even with 60 items, about half a dozen presses locate any item.
- When binary search is selected, **insert and delete also use it** to find the insertion point or the item to delete. Duplicates aren't permitted.

### 4. The `find()` method (the heart of `OrdArray`)

```java
public int find(long searchKey) {
    int lowerBound = 0;
    int upperBound = nElems - 1;
    int curIn;

    while (true) {
        curIn = (lowerBound + upperBound) / 2;
        if (a[curIn] == searchKey)
            return curIn;                        // found it
        else if (lowerBound > upperBound)
            return nElems;                       // can't find it
        else {                                   // divide range
            if (a[curIn] < searchKey)
                lowerBound = curIn + 1;          // it's in upper half
            else
                upperBound = curIn - 1;          // it's in lower half
        }
    }
}
```

**The three variables:**

|Variable|Meaning|
|---|---|
|`lowerBound`|Lowest index still possibly holding the key (starts at `0`)|
|`upperBound`|Highest index still possibly holding the key (starts at `nElems - 1`, the last **occupied** cell)|
|`curIn`|The "current index", the middle of the range: `(lowerBound + upperBound) / 2`|

**One pass through the loop:**

1. Set `curIn` to the middle of the range.
2. **Lucky?** If `a[curIn] == searchKey`, return `curIn`.
3. **Range gone?** If `lowerBound > upperBound`, the range no longer exists. The key isn't there, so return `nElems`. (When `lowerBound == upperBound`, the range has **one** cell and needs one more pass through the loop.)
4. Otherwise **halve the range**:
    - `a[curIn] < searchKey` → key is in the **upper** half → `lowerBound = curIn + 1`
    - `a[curIn] > searchKey` → key is in the **lower** half → `upperBound = curIn - 1`

```
 lowerBound              curIn                     upperBound
     ↓                     ↓                           ↓
    [ . . . . . . . . . . [M] . . . . . . . . . . . . . ]

 searchKey < a[curIn]  →  new range = [lowerBound .. curIn-1]
 searchKey > a[curIn]  →  new range = [curIn+1   .. upperBound]
```

Why `curIn + 1` and `curIn - 1`, not just `curIn`? Because `a[curIn]` was **already checked** at the top of the loop, so there is no reason to keep it in the range.

**Why return `nElems` for "not found"?** It is **not a valid index** (the last filled cell is `nElems - 1`), so the caller can interpret it as failure. The user finds out `nElems` via the class's `size()` method.

#### Trace: key found

Array: `[0, 11, 22, 33, 44, 55, 66, 77, 88, 99]`, `nElems = 10`, `find(55)`.

|Pass|lower|upper|curIn|`a[curIn]`|Decision|
|---|---|---|---|---|---|
|1|0|9|4|44|44 < 55, so `lowerBound = 5`|
|2|5|9|7|77|77 > 55, so `upperBound = 6`|
|3|5|6|5|55|**found**, return 5|

#### Trace: key not found

Same array, `find(35)`.

|Pass|lower|upper|curIn|`a[curIn]`|Decision|
|---|---|---|---|---|---|
|1|0|9|4|44|44 > 35, so `upperBound = 3`|
|2|0|3|1|11|11 < 35, so `lowerBound = 2`|
|3|2|3|2|22|22 < 35, so `lowerBound = 3`|
|4|3|3|3|33|33 < 35, so `lowerBound = 4`|
|5|4|3|3|33|not equal, and `lower > upper`, so return `nElems` (10)|

### 5. The `OrdArray` class

Same shape as `HighArray`, with differences in `find()`, `insert()`, `delete()`, plus a new `size()`.

```java
class OrdArray {
    private long[] a;                            // ref to array a
    private int nElems;                          // number of data items

    public OrdArray(int max) {                   // constructor
        a = new long[max];                       // create array
        nElems = 0;
    }

    public int size() { return nElems; }

    public int find(long searchKey) { ... }      // binary search, shown above

    public void insert(long value) {             // put element into array
        int j;
        for (j = 0; j < nElems; j++)             // find where it goes
            if (a[j] > value)                    // (linear search)
                break;
        for (int k = nElems; k > j; k--)         // move bigger ones up
            a[k] = a[k - 1];
        a[j] = value;                            // insert it
        nElems++;                                // increment size
    }

    public boolean delete(long value) {
        int j = find(value);                     // uses binary search
        if (j == nElems)                         // can't find it
            return false;
        else {                                   // found it
            for (int k = j; k < nElems; k++)     // move bigger ones down
                a[k] = a[k + 1];
            nElems--;                            // decrement size
            return true;
        }
    }

    public void display() {                      // displays array contents
        for (int j = 0; j < nElems; j++)         // for each element,
            System.out.print(a[j] + " ");        //   display it
        System.out.println("");
    }
}
```

```java
class OrderedApp {
    public static void main(String[] args) {
        int maxSize = 100;                       // array size
        OrdArray arr;                            // reference to array
        arr = new OrdArray(maxSize);             // create the array

        arr.insert(77);  arr.insert(99);  arr.insert(44);  arr.insert(55);  arr.insert(22);   // insert 10 items
        arr.insert(88);  arr.insert(11);  arr.insert(00);  arr.insert(66);  arr.insert(33);

        int searchKey = 55;                      // search for item
        if (arr.find(searchKey) != arr.size())
            System.out.println("Found " + searchKey);
        else
            System.out.println("Can't find " + searchKey);

        arr.display();                           // display items

        arr.delete(00);                          // delete 3 items
        arr.delete(55);
        arr.delete(99);

        arr.display();                           // display items again
    }
}
```

Worked out by hand, this prints:

```
Found 55
0 11 22 33 44 55 66 77 88 99
11 22 33 44 66 77 88
```

(Even though the numbers were inserted in a scrambled order, the array always holds them sorted.)

#### `insert()` step by step

Insert **33** into `[11, 22, 44, 55]` (`nElems = 4`):

1. **Find the spot (linear scan):** `a[0]=11`, `a[1]=22`, `a[2]=44 > 33`, so break with `j = 2`.
2. **Shift larger items up**, starting from the end: `k=4`: `a[4]=a[3]` (55); `k=3`: `a[3]=a[2]` (44). Stop when `k > j` fails.
3. **Place it:** `a[2] = 33`. Now `[11, 22, 33, 44, 55]`.
4. `nElems++` (now 5).

If no element is larger, the scan ends with `j == nElems`, the shift loop does nothing, and the value goes at the end.

**Why the book keeps a linear search in `insert()`:** a binary search could locate the spot, but on average **half the items must be moved anyway**, so insertion is not fast regardless. (For the last ounce of speed you could use a binary-search variant, as the Workshop applet does.)

**`delete()`** calls `find()` to locate the item (binary search), then shifts everything above it down by one, exactly like before.

**`size()`** exists so the caller can compare `find()`'s return value with the item count to detect failure.

### 6. Advantages (and disadvantages) of ordered arrays

|Operation|Unordered array|Ordered array|
|---|---|---|
|Search|Slow: O(N)|**Fast: O(log N)** (binary search)|
|Insert|**Fast: O(1)**|Slow: O(N) (find spot + shift up)|
|Delete|Slow: O(N)|Slow: O(N) (shift down)|

- **Major advantage:** searches are much faster.
- **Disadvantage:** insertion is slower, since all items with a higher key must be moved up.
- **Deletion** is slow in **both** kinds, because items must be moved down to fill the hole.

**Use an ordered array when searches are frequent but insertions and deletions are not.**

- Good fit: a **database of company employees**. Hiring and laying off are infrequent compared to looking up a record or updating salary/address.
- Poor fit: a **retail store inventory**, with frequent insertions and deletions as items arrive and are sold. They would run slowly.

## Mechanics / reference

### Ordered vs. unordered array (average case, N items)

|Operation|Unordered|Ordered (linear)|Ordered (binary)|
|---|---|---|---|
|Search|N/2 comparisons|N/2 (can stop early on a miss)|~log₂ N comparisons|
|Insert|1 step|find spot + N/2 moves|find spot (binary: log N) + N/2 moves|
|Delete|N/2 comps + N/2 moves|N/2 comps + N/2 moves|log N comps + N/2 moves|

### Binary search: rules of the loop

|Situation|Action|
|---|---|
|`a[curIn] == searchKey`|Return `curIn`|
|`lowerBound > upperBound`|Range is empty, return `nElems` (not found)|
|`a[curIn] < searchKey`|`lowerBound = curIn + 1`|
|`a[curIn] > searchKey`|`upperBound = curIn - 1`|

### Common patterns

```java
// 1. Binary search template (iterative), returns index or "not found" sentinel
int lo = 0, hi = nElems - 1;
while (lo <= hi) {
    int mid = (lo + hi) / 2;
    if (a[mid] == key)      return mid;
    else if (a[mid] < key)  lo = mid + 1;
    else                    hi = mid - 1;
}
return nElems;              // not found (or -1 in most other codebases)

// 2. Insert into a sorted array
int j;
for (j = 0; j < nElems; j++) if (a[j] > value) break;     // find spot
for (int k = nElems; k > j; k--) a[k] = a[k - 1];         // shift up (from the end!)
a[j] = value;
nElems++;
```

_(Beyond the chapter, interview staples:_

- _`(lo + hi) / 2` can **overflow** for very large indices. The safe form is `lo + (hi - lo) / 2`._
- _Most codebases return **-1** for "not found". The book returns `nElems`. `java.util.Arrays.binarySearch` returns `-(insertion point) - 1` for a miss._
- _Binary search only works if the data is **sorted**. On unsorted data it returns wrong answers without any error.)_

## Pitfalls

- **Binary search on an unsorted array.** Gives wrong results silently. The order is a precondition.
- **Setting `lowerBound = curIn` / `upperBound = curIn` (no ±1).** `curIn` is already checked, and with this mistake the loop can cycle forever.
- **Searching up to `a.length - 1` instead of `nElems - 1`.** You would search empty cells (zeros) that are not part of the data.
- **Shifting up from the front** in `insert()`. The shift must start at the **end** (`k = nElems` going down) or you overwrite items.
- **Forgetting `nElems++` / `nElems--`.**
- **Treating the returned `nElems` as a real index.** It means "not found". Compare it with `size()`.
- **Assuming insertion is fast because search is.** Insertion is **O(N)** because of shifting, even with a binary search to find the spot.
- **Assuming duplicates are handled.** This `OrdArray` doesn't allow duplicates, and `insert()` doesn't check for them. If duplicates exist, `find()` returns **some** matching index, not necessarily the first.
- **Using an ordered array for frequently changing data** (like store inventory). Inserts and deletes keep paying the shifting cost.
- **Confusing "range of size one" with "range gone".** At `lowerBound == upperBound` you still need one more check. The range is gone only when `lowerBound > upperBound`.

## Flashcards

- What is an ordered array? :: An array whose items are kept in ascending (or descending) key order, with the smallest at index 0 #card
- What must insertion do in an ordered array? :: Find the correct location, then move all larger items up to make room #card
- How can a linear search stop early in an ordered array? :: Once it reaches an item larger than the key, the key can't be further along, so it stops #card
- What is the main reason to keep an array ordered? :: It allows binary search, which is much faster than linear search #card
- How does binary search work? :: Check the middle of the range; discard the half that can't contain the key; repeat until found or the range is empty #card
- In the guess-a-number game (1–100), what is the maximum number of guesses with binary search? :: 7 #card
- What is the best first guess in the 1–100 game and why? :: 50, because it halves the range of possibilities #card
- On average, how many guesses would a linear approach need for 1–100? :: About 50 #card
- What are `lowerBound` and `upperBound` in `find()`? :: The lowest and highest indices that could still contain the key #card
- What are their initial values? :: `0` and `nElems - 1` #card
- How is `curIn` computed? :: `(lowerBound + upperBound) / 2`, the middle of the range #card
- If `a[curIn] < searchKey`, what do you do? :: Search the upper half: `lowerBound = curIn + 1` #card
- If `a[curIn] > searchKey`, what do you do? :: Search the lower half: `upperBound = curIn - 1` #card
- Why `curIn + 1` rather than `curIn`? :: `a[curIn]` has already been checked, so it can be excluded from the range #card
- When does binary search conclude the key is absent? :: When `lowerBound > upperBound` (the range no longer exists) #card
- What does `OrdArray.find()` return when the key isn't found? :: `nElems`, which is not a valid index #card
- Why is `nElems` a good "not found" value? :: Valid indices are `0` to `nElems - 1`, so `nElems` can't be confused with a real position #card
- How does the user detect an unsuccessful `find()` in `OrderedApp`? :: `arr.find(key) == arr.size()` means not found #card
- What is the time complexity of binary search? :: O(log N) #card
- Why does `OrdArray.insert()` still use a linear search? :: Simplicity: half the items must be shifted anyway, so insertion is slow regardless #card
- In `insert()`, from which end do you shift elements? :: From the end (the highest index), moving each item up by one #card
- Which `OrdArray` method uses `find()` (binary search)? :: `delete()` #card
- Why is deletion slow in ordered arrays? :: Items above the deleted one must be shifted down to fill the hole #card
- What is the main drawback of an ordered array? :: Slow insertion (and still slow deletion) #card
- When is an ordered array a good choice? :: When searches are frequent and insertions/deletions are rare (e.g., an employee database) #card
- When is an ordered array a poor choice? :: When data changes constantly (e.g., retail inventory) #card
- What does `size()` return in `OrdArray`? :: `nElems`, the number of items currently stored #card
- What precondition does binary search need? :: The data must be sorted #card

## Open questions

- [ ] Could `insert()` use binary search to find its spot, and would it change the overall O(N)?
- [ ] How do I make `find()` return the **first** of several duplicate keys?
- [ ] How would binary search look written recursively? (Recursion chapter.)
- [ ] Can any structure give fast search **and** fast insert/delete? (Later chapters.)
- [ ] How do I avoid the `(lo + hi) / 2` overflow problem?
- [ ] How would I binary search for an **insertion point** (the first element greater than the key)?

## Key terms

|Term|Definition|
|---|---|
|Ordered array|An array kept sorted by key|
|Linear search|Checking items one by one from the start|
|Binary search|Repeatedly halving the range of possible locations by comparing with the middle item|
|`lowerBound`|Low end of the current search range|
|`upperBound`|High end of the current search range|
|`curIn`|Current index, the middle of the range|
|Range|The set of indices where the key might still be|
|`OrdArray`|Array class that stores data in sorted order and uses binary search in `find()`|
|`size()`|Method returning the number of stored items|
|Guess-a-number game|Everyday analogy for binary search|

## Related

← Previous: [[DSA 2.2 - Classes and Interfaces for Arrays]] → Next: [[DSA 2.4 - Logarithms and Big O Notation]] ↑ Up: [[DSA 2 - Arrays]]