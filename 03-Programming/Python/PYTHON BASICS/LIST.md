---
title: LIST
date: 2026-05-17
phase: phase-1
tags:
  - concept
  - python
links: []
status: learning
---
# LISTS

## One-Line Summary

A list stores an **ordered, mutable collection of items** in a single variable using square brackets `[]` — you can access items by index, and freely add, remove, change, sort, and slice them at any time.

---

## Core Concept

Think of a list like a **numbered train with carriages**. Each carriage holds one item, and each carriage has a number (index) starting from **0**, not 1. You can add new carriages, remove old ones, swap cargo between them, or rearrange the whole train — the train is _mutable_, meaning it changes. This is different from a string, which is _immutable_ (you can't change individual characters in place).

Unlike Java where you must declare `ArrayList<String>` with a type, Python lists can hold **anything** — strings, ints, floats, even other lists — all mixed together.

Unlike Java's `ArrayList`, Python lists have a shorthand negative index trick: `my_list[-1]` always gives you the last item, no matter the size.

---

## How It Works

### Creating a List

Square brackets `[]` define a list. Items are separated by commas. Name your list **plural** — it's a convention that makes code readable.

```python
# From values
bicycles = ['trek', 'cannondale', 'redline', 'specialized']   # Python

# Empty list — fill it later with append()
motorcycles = []

# Mixed types (valid, but uncommon in practice)
mixed = ['spam', 2.0, 5, [10, 20]]   # nested list inside!

# A nested list counts as ONE element for len() purposes
nested = ['spam', 1, ['Brie', 'Roquefort'], [1, 2, 3]]
print(len(nested))   # 4  ← the inner lists each count as 1 item

# Printing a raw list shows brackets — usually not what you want to show users
print(bicycles)   # ['trek', 'cannondale', 'redline', 'specialized']
```

---

### Accessing Elements — Indexing

Use the index number in square brackets. Indexing starts at **0**.

```python
bicycles = ['trek', 'cannondale', 'redline', 'specialized']   # Python

# Positive indexing — left to right
print(bicycles[0])    # trek         ← first item
print(bicycles[1])    # cannondale   ← second item
print(bicycles[3])    # specialized  ← fourth (last) item

# Negative indexing — right to left (Python superpower — no Java equivalent!)
print(bicycles[-1])   # specialized  ← always the last item
print(bicycles[-2])   # redline      ← second-to-last

# Chaining string methods directly on a list element
print(bicycles[0].title())   # Trek   (capitalised)

# Using list values in f-strings
message = f"My first bicycle was a {bicycles[0].title()}."
print(message)   # My first bicycle was a Trek.
```

> **Index position rule:** if a list has `n` items, valid positive indices are `0` to `n-1`. The last item is always `my_list[-1]`.

---

### Using Individual Values from a List

You can use a list element anywhere you'd use a regular variable — in expressions, concatenation, f-strings, or as function arguments.

```python
bicycles = ['trek', 'cannondale', 'redline', 'specialized']   # Python

# Old-style string concatenation (still valid)
message = "My first bicycle was a " + bicycles[0].title() + "."
print(message)   # My first bicycle was a Trek.

# Modern f-string style (preferred)
message = f"My first bicycle was a {bicycles[0].title()}."
print(message)   # My first bicycle was a Trek.
```

---

### Lists are Mutable

Unlike strings, you can **change** any item in a list by assigning to its index. This is what _mutable_ means — the object itself changes in place.

```python
numbers = [17, 123]   # Python
numbers[1] = 5
print(numbers)   # [17, 5]   ← 123 is gone, replaced by 5

# The 'in' operator works on lists just like on strings
cheeses = ['Cheddar', 'Edam', 'Gouda']
print('Edam' in cheeses)    # True
print('Brie' in cheeses)    # False
```

---

### Adding Elements

#### `append()` — Add to the End

The most common way. New item lands at the very end. Modifies the list in place, returns `None`.

```python
motorcycles = ['honda', 'yamaha', 'suzuki']   # Python
motorcycles.append('ducati')
print(motorcycles)   # ['honda', 'yamaha', 'suzuki', 'ducati']

# Building a list from scratch — the standard Python pattern:
motorcycles = []
motorcycles.append('honda')
motorcycles.append('yamaha')
motorcycles.append('suzuki')
# Result: ['honda', 'yamaha', 'suzuki']
```

#### `insert()` — Add at Any Position

Specify the index where the new item should go. Everything at that index and after shifts right by one.

```python
motorcycles = ['honda', 'yamaha', 'suzuki']   # Python

motorcycles.insert(0, 'ducati')   # Add at beginning
print(motorcycles)   # ['ducati', 'honda', 'yamaha', 'suzuki']

motorcycles.insert(2, 'kawasaki')   # Add in the middle
```

#### `extend()` — Merge Another List In

Takes a list as argument and appends all its elements. The original second list is unmodified.

```python
t1 = ['a', 'b', 'c']   # Python
t2 = ['d', 'e']
t1.extend(t2)
print(t1)   # ['a', 'b', 'c', 'd', 'e']
# t2 is unchanged: ['d', 'e']
```

#### `+` Operator — Concatenate (Creates a NEW list)

Unlike `append()` which modifies in place, `+` creates a brand new list. The original lists are untouched.

```python
a = [1, 2, 3]   # Python
b = [4, 5, 6]
c = a + b
print(c)   # [1, 2, 3, 4, 5, 6]
# a and b are unchanged
```

#### `*` Operator — Repeat a List

Creates a new list by repeating the original `n` times.

```python
print([0] * 4)      # [0, 0, 0, 0]   # Python
print([1, 2, 3] * 3)   # [1, 2, 3, 1, 2, 3, 1, 2, 3]
```

---

### Removing Elements

Three ways — each has a specific use case.

#### `del` — Remove by Index (Permanent, No Return)

Use when you know the position and **don't need the value** afterward.

```python
motorcycles = ['honda', 'yamaha', 'suzuki']   # Python
del motorcycles[0]
print(motorcycles)   # ['yamaha', 'suzuki']

# del with a slice — removes a range at once
t = ['a', 'b', 'c', 'd', 'e', 'f']
del t[1:5]
print(t)   # ['a', 'f']
```

#### `pop()` — Remove AND Return the Value

Removes an item and gives it back to you so you can still use it. Think of "popping" a can off a stack — the can is in your hand after.

```python
motorcycles = ['honda', 'yamaha', 'suzuki']   # Python

# No argument → removes the LAST item
popped = motorcycles.pop()
print(motorcycles)   # ['honda', 'yamaha']
print(popped)        # suzuki  ← still accessible!

# pop(index) → removes from any position
first_owned = motorcycles.pop(0)   # removes honda
print(f"First bike: {first_owned.title()}")   # First bike: Honda
```

> **del vs pop():** If you want to delete and **never use** the value → `del`. If you want to delete and **still use** the value → `pop()`.

#### `remove()` — Remove by VALUE (Index Unknown)

Use when you know what value to remove but not where it sits. Removes only the **first** occurrence. Returns `None`.

```python
motorcycles = ['honda', 'yamaha', 'suzuki', 'ducati']   # Python
motorcycles.remove('ducati')
print(motorcycles)   # ['honda', 'yamaha', 'suzuki']

# Save the value to use it after removal:
too_expensive = 'ducati'
motorcycles.remove(too_expensive)
print(f"{too_expensive.title()} is too expensive for me.")
# Output: Ducati is too expensive for me.
```

> ⚠️ `remove()` deletes only the **first match**. To remove all occurrences, use a `while` loop (see [[ITERATIVE STATEMENTS]]).

---

### List Slices

The slice operator `[start:stop]` extracts a portion of the list. Works exactly like string slicing.

```python
t = ['a', 'b', 'c', 'd', 'e', 'f']   # Python

print(t[1:3])    # ['b', 'c']   ← index 1 up to (not including) 3
print(t[:4])     # ['a', 'b', 'c', 'd']   ← from beginning to index 3
print(t[3:])     # ['d', 'e', 'f']   ← from index 3 to the end
print(t[:])      # ['a', 'b', 'c', 'd', 'e', 'f']   ← full copy

# Slice assignment — update multiple elements at once
t[1:3] = ['x', 'y']
print(t)   # ['a', 'x', 'y', 'd', 'e', 'f']
```

> **Copying a list safely:** `copy = my_list[:]` makes a fresh independent copy. This avoids _aliasing_ problems (see below).

---

### Organising a List

#### `sort()` — Permanent Sort

Permanently changes the list order. The original order is gone forever. Returns `None`.

```python
cars = ['bmw', 'audi', 'toyota', 'subaru']   # Python

cars.sort()                 # A → Z
print(cars)   # ['audi', 'bmw', 'subaru', 'toyota']

cars.sort(reverse=True)    # Z → A
print(cars)   # ['toyota', 'subaru', 'bmw', 'audi']
```

> ⚠️ **Common trap:** `t = t.sort()` sets `t` to `None` because `sort()` returns `None`. Never assign the result of `sort()`.

#### `sorted()` — Temporary Sorted View

Returns a **new** sorted list without touching the original.

```python
cars = ['bmw', 'audi', 'toyota', 'subaru']   # Python

print(sorted(cars))              # ['audi', 'bmw', 'subaru', 'toyota']
print(cars)                      # ['bmw', 'audi', 'toyota', 'subaru'] ← unchanged

print(sorted(cars, reverse=True))   # reverse sorted view
```

#### `reverse()` — Flip the Order

Permanently reverses the list. Not alphabetical — just flips whatever order already exists. Call it again to restore.

```python
cars = ['bmw', 'audi', 'toyota', 'subaru']   # Python
cars.reverse()
print(cars)   # ['subaru', 'toyota', 'audi', 'bmw']

cars.reverse()   # call again to undo
```

#### `len()` — Count Items

Returns how many items are in the list. Python counts from 1 for length, but indices start at 0 — so the last valid index is always `len(list) - 1`.

```python
cars = ['bmw', 'audi', 'toyota', 'subaru']   # Python
print(len(cars))   # 4
```

---

### Lists and Built-in Functions

These built-in functions let you process a whole list without writing loops:

```python
nums = [3, 41, 12, 9, 74, 15]   # Python

print(len(nums))         # 6    ← count of items
print(max(nums))         # 74   ← largest value
print(min(nums))         # 3    ← smallest value
print(sum(nums))         # 154  ← total (numbers only)
print(sum(nums)/len(nums))  # 25.666...  ← average
```

> `sum()` only works on numeric lists. `max()`, `min()`, `len()` work on any comparable type.

---

### Traversing a List

The most common way to loop through every element is `for`:

```python
cheeses = ['Cheddar', 'Edam', 'Gouda']   # Python
for cheese in cheeses:
    print(cheese)
# Cheddar
# Edam
# Gouda

# Looping over an empty list simply executes zero times — no error.
```

When you need the index (to update elements), combine `range()` and `len()`:

```python
numbers = [1, 2, 3, 4]   # Python
for i in range(len(numbers)):
    numbers[i] = numbers[i] * 2
print(numbers)   # [2, 4, 6, 8]
```

---

### Lists and Strings

A string and a list are both sequences, but they are **not the same type**. Use these to convert between them:

```python
# String → List of characters
s = 'spam'   # Python
t = list(s)
print(t)   # ['s', 'p', 'a', 'm']

# String → List of words (split on spaces by default)
s = 'pining for the fjords'
t = s.split()
print(t)        # ['pining', 'for', 'the', 'fjords']
print(t[2])     # the

# Split on a custom delimiter
s = 'spam-spam-spam'
delimiter = '-'
print(s.split(delimiter))   # ['spam', 'spam', 'spam']

# List of strings → single string (join — inverse of split)
t = ['pining', 'for', 'the', 'fjords']
delimiter = ' '
print(delimiter.join(t))   # 'pining for the fjords'

# Join with no space
print(''.join(t))   # 'piningforthefjords'
```

> ⚠️ Don't use `list` as a variable name — it shadows the built-in function!

---

### Objects, Values, and Aliasing

This is a common source of bugs — **crucial to understand**.

```python
# Two variables, same VALUE but DIFFERENT objects
a = [1, 2, 3]   # Python
b = [1, 2, 3]
print(a is b)    # False  ← equivalent but NOT identical
print(a == b)    # True   ← same values

# ALIASING — two variables pointing to the SAME object
a = [1, 2, 3]
b = a            # b is now an ALIAS for a
print(b is a)    # True

b[0] = 17        # modifying b ALSO modifies a!
print(a)         # [17, 2, 3]  ← a changed!

# Safe copy using slice — creates a new independent object
orig = t[:]
t.sort()
# orig is unchanged, t is sorted
```

> **Rule:** `b = a` does NOT copy the list — it creates a second name for the same object. Any change through `b` also affects `a`. Use `b = a[:]` to make a real copy.

---

### Lists and Functions — Passing Lists In

When you pass a list to a function, the function gets a **reference** to the same list. Modifying it inside the function changes the original.

```python
def delete_head(t):   # Python
    del t[0]

letters = ['a', 'b', 'c']
delete_head(letters)
print(letters)   # ['b', 'c']  ← original was modified!
```

If you want the function to create a new list without touching the original, return a new list:

```python
def tail(t):   # Python
    return t[1:]   # returns a new list, original untouched

letters = ['a', 'b', 'c']
rest = tail(letters)
print(rest)     # ['b', 'c']
print(letters)  # ['a', 'b', 'c']  ← unchanged
```

> **Crucial distinction:** `append()` and `sort()` modify the list in place and return `None`. The `+` operator and `sorted()` return new lists. Mixing these up is a classic bug.

---

### Objects and Values — `==` vs `is`

Every value in Python is an **object** — it has a type and a value. Two variables can hold the **same value** (equivalent) without being the **same object** (identical). This distinction matters a lot with lists.

```python
# Strings — Python reuses the same object for identical string literals
a = 'banana'   # Python
b = 'banana'
print(a == b)    # True  ← same value (equivalent)
print(a is b)    # True  ← also same object (Python optimises this)

# Lists — Python ALWAYS creates a new object, even with identical contents
a = [1, 2, 3]
b = [1, 2, 3]
print(a == b)    # True   ← equivalent (same values)
print(a is b)    # False  ← NOT identical (two separate objects in memory)
```

> **Equivalent** = same values. **Identical** = literally the same object in memory. If two objects are identical, they are also equivalent — but not vice versa.

A **reference** is the association between a variable name and the object it points to. When you write `a = [1, 2, 3]`, `a` is a reference to a list object.

---

### Parsing Lines — `split()` in Real Programs

`split()` is powerful when reading files line by line. A common pattern: read a file, split each line into words, then process specific words by index.

```python
# Find the day of the week from email "From" lines   # Python
fhand = open('mbox-short.txt')
for line in fhand:
    line = line.rstrip()                  # remove trailing newline
    if not line.startswith('From '):      # skip non-From lines
        continue
    words = line.split()                  # split on whitespace
    print(words[2])                       # third word = day of week
# Output: Sat, Fri, Fri, Fri...

# Safer version — guard against blank lines (empty split → empty list)
fhand = open('mbox-short.txt')
for line in fhand:
    words = line.split()
    if len(words) == 0: continue          # GUARD — skip blank lines
    if words[0] != 'From': continue       # skip non-From lines
    print(words[2])
```

> **Why the guard matters:** `words[0]` on an empty list causes `IndexError: list index out of range`. Always check `len(words) == 0` before indexing when reading files.

---

### Avoiding Index Errors

```python
motorcycles = ['honda', 'yamaha', 'suzuki']   # Python

# WRONG — causes IndexError: list index out of range
# print(motorcycles[3])   ← no index 3 in a 3-item list!

# CORRECT — use -1 to always safely get the last item
print(motorcycles[-1])   # suzuki

# Special edge case: empty list causes IndexError even with -1
motorcycles = []
# print(motorcycles[-1])   # IndexError!

# Debug tip: when an IndexError confuses you, print the list and its length
print(motorcycles)
print(len(motorcycles))
```

---

### Debugging List Code — 4 Common Pitfalls

1. **`sort()` returns `None`** — most list methods modify in place and return `None`. This is opposite to string methods which return new strings. Never do `t = t.sort()`.
    
2. **Pick one idiom for adding/removing** — there are many ways to do the same thing. Stick to consistent patterns:
    
    ```python
    t.append(x)      # ✅ correct — modifies in place   # Python
    t = t + [x]      # ✅ correct — creates new list
    
    t.append([x])    # ❌ wrong — appends a nested list!
    t = t.append(x)  # ❌ wrong — t becomes None!
    t + [x]          # ❌ wrong — result is discarded!
    t = t + x        # ❌ wrong — TypeError if x is not a list!
    ```
    
3. **Make copies before destructive operations** — if you need the original after sorting:
    
    ```python
    orig = t[:]   # Python
    t.sort()      # t is sorted, orig is safe
    ```
    
4. **Guard against empty lists when splitting/indexing** — blank lines cause `words[0]` to crash:
    
    ```python
    words = line.split()   # Python
    if len(words) == 0: continue      # guard: skip blank lines
    if words[0] != 'From': continue   # then check the word
    print(words[2])
    ```
    

---

---

## Glossary

|Term|Definition|
|---|---|
|**list**|A sequence of values — the values can be any type|
|**element**|One of the values in a list (also called an _item_)|
|**index**|An integer value that indicates the position of an element in a list|
|**nested list**|A list that is an element of another list — counts as ONE element for `len()`|
|**mutable**|Can be changed in place — lists are mutable, strings are not|
|**list traversal**|Sequentially accessing each element in a list, usually with a `for` loop|
|**object**|Something a variable can refer to — has a type and a value|
|**equivalent**|Having the same value (`a == b` is `True`)|
|**identical**|Being the exact same object in memory (`a is b` is `True`)|
|**reference**|The association between a variable name and the object it points to|
|**aliasing**|When two or more variables refer to the same object — changes through one affect the other|
|**delimiter**|A character or string used to indicate where a string should be split (used in `split()` and `join()`)|

---

## Java vs Python

|Concept|Java|Python|
|---|---|---|
|Create list|`ArrayList<String> list = new ArrayList<>()`|`my_list = []`|
|Create from values|`Arrays.asList("a","b","c")`|`['a', 'b', 'c']`|
|Add to end|`list.add("x")`|`my_list.append('x')`|
|Add at position|`list.add(0, "x")`|`my_list.insert(0, 'x')`|
|Merge two lists|`list1.addAll(list2)`|`list1.extend(list2)` or `list1 + list2`|
|Replace item|`list.set(0, "x")`|`my_list[0] = 'x'`|
|Delete by index|`list.remove(0)`|`del my_list[0]`|
|Delete and return|`list.remove(0)`|`my_list.pop(0)`|
|Delete by value|`list.remove("x")`|`my_list.remove('x')`|
|Get item|`list.get(0)`|`my_list[0]`|
|Get last item|`list.get(list.size()-1)`|`my_list[-1]` ← Python shortcut!|
|Size / length|`list.size()`|`len(my_list)`|
|Sort (permanent)|`Collections.sort(list)`|`my_list.sort()`|
|Sort (copy)|— (extra code needed)|`sorted(my_list)`|
|Reverse|`Collections.reverse(list)`|`my_list.reverse()`|
|Contains|`list.contains("x")`|`'x' in my_list`|
|Max / Min / Sum|manual loop|`max()` / `min()` / `sum()`|
|Slice (sublist)|`list.subList(1, 3)`|`my_list[1:3]`|
|Copy list safely|`new ArrayList<>(list)`|`my_list[:]`|
|Is same object?|`list1 == list2` (reference equality)|`list1 is list2`|
|Same values?|`list1.equals(list2)`|`list1 == list2`|
|`null` / `None`|`null`|`None`|
|List methods return|the list (chainable in some cases)|`None` — methods modify in place!|

---

## Quick Reference

```python
# ── Creating ────────────────────────────────────────────────────   # Python
my_list = ['a', 'b', 'c']   # from values
my_list = []                  # empty

# ── Accessing ────────────────────────────────────────────────────
my_list[0]     # first item
my_list[-1]    # last item (Python shortcut!)
my_list[1:3]   # slice: index 1 and 2 (not 3)
my_list[:]     # full copy

# ── Modifying ────────────────────────────────────────────────────
my_list[0] = 'new'           # replace by index
my_list[1:3] = ['x', 'y']   # slice assignment

# ── Adding ───────────────────────────────────────────────────────
my_list.append('d')          # add to END — in place, returns None
my_list.insert(0, 'z')       # add at position — in place, returns None
my_list.extend(other_list)   # merge in other list — in place
new = my_list + other_list   # concatenate — returns NEW list

# ── Removing ─────────────────────────────────────────────────────
del my_list[0]               # delete by index — gone forever
popped = my_list.pop()       # remove last + return it
popped = my_list.pop(1)      # remove at index 1 + return it
my_list.remove('b')          # remove first occurrence of VALUE, returns None

# ── Organising ───────────────────────────────────────────────────
my_list.sort()               # permanent A→Z, returns None
my_list.sort(reverse=True)   # permanent Z→A, returns None
sorted(my_list)              # temporary sorted copy — original untouched
my_list.reverse()            # flip order permanently, returns None
len(my_list)                 # count of items

# ── Built-in Functions ───────────────────────────────────────────
len(my_list)    # count
max(my_list)    # largest (comparable items)
min(my_list)    # smallest
sum(my_list)    # total (numbers only)

# ── Checking Membership ──────────────────────────────────────────
'x' in my_list       # True if 'x' is in the list
'x' not in my_list   # True if 'x' is NOT in the list

# ── Strings ↔ Lists ──────────────────────────────────────────────
list('spam')              # ['s', 'p', 'a', 'm']
'a b c'.split()           # ['a', 'b', 'c']
'a-b-c'.split('-')        # ['a', 'b', 'c']
' '.join(['a','b','c'])   # 'a b c'

# ── Copying Safely (avoid aliasing) ─────────────────────────────
copy = my_list[:]         # new independent list

# ── Traversing ───────────────────────────────────────────────────
for item in my_list:
    print(item)

for i in range(len(my_list)):   # when you need the index
    my_list[i] = my_list[i] * 2
```

---

## Practice Exercises

- [ ] **Names** — Store 3 friend names in a list called `names`. Print each one by accessing it by index.
- [ ] **Greetings** — Use the same `names` list. Print `"Hello, [name]!"` for each using an f-string.
- [ ] **Your Own List** — Create a `motorcycles` list. Print it using `title()`. Modify the first item. Try all three remove methods.
- [ ] **Guest List** — Invite 3 people to dinner. One can't make it — use `remove()`. Add a replacement with `insert()`. Print new invitations.
- [ ] **More Guests** — Find a bigger table. Use `insert()` to add 3 people (start, middle, end). Print total count with `len()`.
- [ ] **Shrinking Guest List** — Table cancelled. Use `pop()` in a loop until 2 remain. Print apology for each removed. Use `del` to clear the last 2.
- [ ] **Sort Explorer** — Create a `places` list. Print original → `sorted()` view → `sorted(reverse=True)` → `sort()` permanently → `reverse()`. Observe what changes.
- [ ] **Accumulator** — Use `sum()` and `len()` to find the average of a numeric list.
- [ ] **String ↔ List** — Take `"pining for the fjords"`, split it into words, access the third word, then join back with `-` as delimiter.
- [ ] **Aliasing Trap** — Create a list `a`, assign `b = a`, modify through `b`, print `a`. Then try `b = a[:]` and repeat — observe the difference.
- [ ] **Intentional IndexError** — Deliberately cause an `IndexError` in a program, read the traceback, then fix it using `-1` indexing.
- [ ] **Every Function** — Make a list of 5 items. Use every method from this note at least once.
- [ ] **Seeing the World** — Store 5 places you'd like to visit. Print original → `sorted()` view → `sorted(reverse=True)` view → show original is unchanged → `reverse()` → `sort()` → `sort(reverse=True)`.
- [ ] **Dinner Guests** — Use `len()` on your guest list from the Guest List exercise to print how many people you're inviting.
- [ ] **Find Unique Words** — Open a text file, read line by line, split each line into words. For each word, if it is not already in a `unique_words` list, append it. At the end, sort and print the unique words list.
- [ ] **Average with a List** — Collect numbers from the user in a loop (stop on `'done'`), store them in a list, then use `sum()` and `len()` to compute and print the average.
- [ ] **Chop function** — Write a function `chop(t)` that takes a list, removes the first AND last elements in place, and returns `None`. Then write `middle(t)` that returns a NEW list with all but the first and last elements (original unchanged).

---

## Questions I Still Have

_Write your open questions here. Return later and answer them._

---

## Related Notes

- [[ITERATIVE STATEMENTS]]
- [[CONDITIONAL STATEMENT]]