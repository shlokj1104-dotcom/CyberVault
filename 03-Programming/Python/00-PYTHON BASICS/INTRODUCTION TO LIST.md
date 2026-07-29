---
title: INTRODUCTION TO LIST
date: 2026-07-29
language: Python
phase: Phase 0 · Setup
tags:
  - python
links: []
status: learning
---
> **One-line summary:** A list is Python's ordered, mutable container — built with square brackets, indexed from zero, and equipped with a rich set of methods for changing, adding, removing, and organizing its contents.

## What it is

A **list** is an ordered collection that can hold any number of items — letters, numbers, names, or a mixture of types — without requiring the items to share any particular (specific, singled out from others) relationship. Because a list usually holds more than one value, convention (a common, agreed-upon way of doing something) favors a plural name (`bikes`, `digits`, `names`). Square brackets `[]` define a list literal, and commas separate its elements. Printing a list directly shows Python's raw representation, brackets and all, which is rarely what you want a user to see — retrieving individual elements produces cleaner output.

## Structure

```
index:   0         1         2
+---------+---------+---------+
|'trek'   |'yamaha' |'suzuki' |
+---------+---------+---------+
-index:  -3        -2        -1
```

Every element has two addresses: a positive index counted from the front, starting at 0, and a negative index counted from the back, ending at -1 for the last element. The negative scheme is handy (useful and convenient) because it lets you reach the tail of a list without knowing its length.

## Mechanics / reference

| Operation                 | Syntax                                  | Effect                                                               |
| ------------------------- | --------------------------------------- | -------------------------------------------------------------------- |
| Read by position          | `lst[i]`                                | Returns the element at index `i`; no brackets/quotes in the result   |
| Read from the end         | `lst[-1]`                               | Returns the last element, regardless of list length                  |
| Overwrite an element      | `lst[i] = value`                        | Replaces the element at index `i` in place                           |
| Add to the end            | `lst.append(value)`                     | Grows the list by one, at the tail                                   |
| Add at a position         | `lst.insert(i, value)`                  | Shifts everything at and after `i` one slot right, then inserts      |
| Delete by position        | `del lst[i]`                            | Removes the element at `i`; that value is gone for good              |
| Remove and keep the value | `lst.pop()` / `lst.pop(i)`              | Removes the last element (or the one at `i`) and returns it          |
| Delete by value           | `lst.remove(value)`                     | Finds the first match and deletes it; later duplicates are untouched |
| Sort permanently          | `lst.sort()` / `lst.sort(reverse=True)` | Reorders the list itself, ascending or descending                    |
| Sort temporarily          | `sorted(lst)`                           | Returns a new sorted list; the original keeps its order              |
| Flip order                | `lst.reverse()`                         | Reverses the current order in place (not alphabetical)               |
| Count elements            | `len(lst)`                              | Returns how many items the list holds                                |

## When to use it

Reach for a list whenever the data is a dynamic, ordered set of items that will grow or shrink as a program runs — a game's pool of on-screen enemies, a form's collected user entries, a queue of tasks still to process. Its strength is exactly this elasticity (the ability to stretch or shrink easily as needed): start empty and build it up with `append()`, or trim it down with `pop()`/`remove()` as circumstances (the conditions or situation surrounding something) change.

## Why it matters for security

| Concept                          | Attacker's perspective                                                                                                                                                             | Defender's perspective                                                                                                                                              |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Unchecked indexing               | Feeding malformed or short input can trigger an out-of-range read, crashing the process (a denial-of-service) or leaking implementation detail through an unhandled traceback      | Validate length before indexing, or wrap access in a try/except, and avoid surfacing (bringing something up into view or attention) raw traceback text to end users |
| Mutability and shared references | If a function hands back a direct reference to an internal list (say, an access list), calling code can silently mutate the original, potentially altering security-relevant state | Return a copy (`list(original)` or `original[:]`) when exposing an internal collection so the source stays protected                                                |

## Worked example

A small playlist queue shows several operations acting on the same list in sequence:

```python
queue = []
queue.append('Blinding Lights')
queue.append('Levitating')
queue.insert(0, 'As It Was')
print(queue)
# ['As It Was', 'Blinding Lights', 'Levitating']

now_playing = queue.pop(0)
print(now_playing)   # 'As It Was'
print(queue)          # ['Blinding Lights', 'Levitating']

queue.sort()
print(queue)          # ['Blinding Lights', 'Levitating']  -- alphabetical, permanent
```

Two empty-list append calls build the queue up from nothing, `insert(0, ...)` pushes a track to the front, `pop(0)` removes and hands back that same track for playback, and `sort()` permanently reorders whatever remains.

## Java vs Python: list operations

|Aspect|Java (`ArrayList<String>`)|Python (`list`)|
|---|---|---|
|Create with values|`new ArrayList<>(List.of("trek","cannondale"))`|`['trek', 'cannondale']`|
|Read by index|`bikes.get(0)`|`bikes[0]`|
|Add to end|`bikes.add("ducati")`|`bikes.append('ducati')`|
|Insert at position|`bikes.add(0, "ducati")`|`bikes.insert(0, 'ducati')`|
|Remove by index|`bikes.remove(0)` (int overload)|`del bikes[0]` or `bikes.pop(0)`|
|Remove by value|`bikes.remove("ducati")` (Object overload — autoboxing caveats (warnings about limitations to keep in mind) with numbers)|`bikes.remove('ducati')`|
|Sort in place|`Collections.sort(bikes)`|`bikes.sort()`|
|Sort without mutating|`new ArrayList<>(bikes)` then sort the copy|`sorted(bikes)`|
|Reverse order|`Collections.reverse(bikes)`|`bikes.reverse()`|
|Length|`bikes.size()`|`len(bikes)`|
|Out-of-bounds access|Throws `IndexOutOfBoundsException`|Raises `IndexError`|

## Pitfalls

- Off-by-one confusion: the _n_th item sits at index _n − 1_, since counting starts at 0
- `sort()` mutates the list permanently and cannot be undone; `sorted()` leaves the original untouched — mixing the two up loses data
- `remove()` deletes only the first matching value; a list with duplicates needs a loop to clear every occurrence
- Even `lst[-1]` raises an `IndexError` if the list is empty — negative indexing doesn't protect against a genuinely (truly, without exaggeration) empty list

## Flashcards

- Python's first list index is __ #card
- Method that adds an element to the end of a list: __ #card
- Method that adds an element at a chosen position, shifting the rest right: __ #card
- Statement used to delete by index when you don't need the removed value: __ #card
- Method that removes an element and returns it for further use: __ #card
- Method that deletes the first element matching a given value: __ #card
- Function that reports how many items a list holds: __ #card
- Method that sorts a list in place, changing it permanently: __ #card
- Function that returns a new sorted list without touching the original: __ #card

## Open questions

- [ ] How does `sort()` behave on a list mixing uppercase and lowercase strings — where do capitals land relative to lowercase letters?
- [ ] When a list has several duplicate values, what's the cleanest loop-based pattern for removing every occurrence with `remove()`?
- [ ] In a real application, what's the right way to catch an `IndexError` so the user sees a friendly message instead of a raw traceback?

## Key terms

|Term|Definition|
|---|---|
|List|An ordered, mutable collection of items written with square brackets|
|Index|The position of an element in a list, counted from 0|
|Negative indexing|Counting positions backward from the end, starting at -1|
|Mutable|Capable of being changed after creation, as a list is|
|In-place|An operation that modifies the original object rather than returning a new one|
|IndexError|The exception Python raises when an index falls outside a list's valid range|

## Vocabulary

- Dynamically (changing while the program runs, rather than fixed in advance)
- Concatenation (joining pieces of text together into one string)
- Mutability (the property of being changeable after creation)
- Aliasing (two variables secretly pointing at the same underlying data)
- Encapsulation (keeping an object's internal data protected from outside interference)
- Bounds (the valid lower and upper limits of a list's indices)
- Traceback (Python's printed report of where and why an error occurred)

## Related

---

→ Next: [[Python Looping Through Lists]]