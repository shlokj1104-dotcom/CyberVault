---
title: DICTIONARIES, TUPLES & SETS
date: 2026-05-17
phase: phase-1
tags:
  - concept
  - python
links: []
status: learning
---
# DICTIONARIES, TUPLES & SETS

## One-Line Summary

A **dictionary** stores _key-value pairs_ in `{}` — look things up by name, not by position. A **tuple** is an _immutable, ordered sequence_ in `()` — like a frozen list. A **set** stores _unique, unordered items_ in `{}` — great for deduplication and membership testing.

---

## PART 1 — DICTIONARIES

### Core Concept

Think of a dictionary like a **real physical dictionary**: you look up a _word_ (key) to get its _definition_ (value). You don't need to know what page it's on — you just use the word directly. This is fundamentally different from a list, where you need to know the position (index) of what you want.

Unlike Java's `HashMap<String, Integer>`, Python dicts can hold **any type** as a value — strings, numbers, lists, even other dicts. Since Python 3.7+, dictionaries also **preserve insertion order**.

---

### Creating a Dictionary

Use curly braces `{}` with `key: value` pairs separated by commas.

```python
# From values — simple one-liner
alien_0 = {'color': 'green', 'points': 5}   # Python

# Multiline format — preferred for readability when there are many pairs
favorite_languages = {
    'jen': 'python',
    'sarah': 'c',
    'edward': 'ruby',
    'phil': 'python',
}

# Empty dictionary — fill later
alien_0 = {}

# Build using dict() constructor
eng2sp = dict()
eng2sp['one'] = 'uno'
eng2sp['two'] = 'dos'
# Result: {'one': 'uno', 'two': 'dos'}

# ⚠️ Never name a variable 'dict' — it shadows the built-in!
```

> **Tip:** Keep a trailing comma after the last pair when writing multiline dicts — it makes adding new pairs easier and keeps version diffs clean.

---

### Accessing Values

Use the key inside square brackets `[]`. If the key doesn't exist, Python raises a `KeyError`.

```python
alien_0 = {'color': 'green', 'points': 5}   # Python

# Basic access
print(alien_0['color'])    # green
print(alien_0['points'])   # 5

# Use the value in an expression
new_points = alien_0['points']
print("You just earned " + str(new_points) + " points!")
# You just earned 5 points!

# Access a non-existent key → KeyError
# print(alien_0['speed'])   # KeyError: 'speed'

# Safe access with .get() — returns None (or a default) instead of crashing
print(alien_0.get('speed'))           # None
print(alien_0.get('speed', 'slow'))   # slow  ← custom default
```

> **`get()` vs `[]`:** Use `[]` when you _know_ the key exists. Use `.get()` when the key _might not_ be there, to avoid crashes.

---

### Adding and Modifying Key-Value Pairs

Dictionaries are **mutable** — you can add new pairs or overwrite existing ones at any time.

```python
alien_0 = {'color': 'green', 'points': 5}   # Python

# Add new key-value pairs
alien_0['x_position'] = 0
alien_0['y_position'] = 25
print(alien_0)
# {'color': 'green', 'points': 5, 'x_position': 0, 'y_position': 25}

# Modify an existing value — just reassign
alien_0['color'] = 'yellow'
print(alien_0['color'])   # yellow

# Real-world example: alien changing speed
alien_0 = {'x_position': 0, 'y_position': 25, 'speed': 'medium'}
if alien_0['speed'] == 'slow':
    x_increment = 1
elif alien_0['speed'] == 'medium':
    x_increment = 2
else:
    x_increment = 3
alien_0['x_position'] = alien_0['x_position'] + x_increment
print(alien_0['x_position'])   # 2
```

---

### Removing Key-Value Pairs

```python
alien_0 = {'color': 'green', 'points': 5}   # Python

# del — permanently removes the key AND its value
del alien_0['points']
print(alien_0)   # {'color': 'green'}

# ⚠️ The deleted pair is gone permanently — no undo!

# pop(key) — removes AND returns the value (like list.pop())
color = alien_0.pop('color')
print(color)     # green
print(alien_0)   # {}

# pop(key, default) — safe if key might not exist
val = alien_0.pop('speed', 'not found')
print(val)   # not found  ← no KeyError!
```

---

### Checking Membership

The `in` operator checks **keys only** (not values). This uses a hash table under the hood — it's O(1) regardless of dict size, unlike a list's O(n) linear search.

```python
favorite_languages = {'jen': 'python', 'sarah': 'c'}   # Python

# Check if a KEY exists
print('jen' in favorite_languages)       # True
print('erin' not in favorite_languages)  # True

# Check if a VALUE exists
vals = list(favorite_languages.values())
print('python' in vals)   # True

# Practical use — add key only if it doesn't exist
if 'erin' not in favorite_languages:
    print("Erin, please take our poll!")
```

---

### Looping Through a Dictionary

#### Loop Through All Key-Value Pairs — `.items()`

```python
user_0 = {   # Python
    'username': 'efermi',
    'first': 'enrico',
    'last': 'fermi',
}

for key, value in user_0.items():
    print(f"\nKey: {key}")
    print(f"Value: {value}")

# You can name the loop variables anything meaningful:
for name, language in favorite_languages.items():
    print(f"{name.title()}'s favorite language is {language.title()}.")
```

#### Loop Through Keys Only — `.keys()`

```python
# Explicit keys() call
for name in favorite_languages.keys():
    print(name.title())

# Implicit — looping directly through a dict loops through keys
for name in favorite_languages:   # same as .keys()
    print(name.title())

# Loop through keys IN ORDER
for name in sorted(favorite_languages.keys()):
    print(f"{name.title()}, thank you for taking the poll.")
```

#### Loop Through Values Only — `.values()`

```python
# All values (may include duplicates)
for language in favorite_languages.values():
    print(language.title())

# Unique values only — wrap in set()
for language in set(favorite_languages.values()):
    print(language.title())
# Output: Python, C, Ruby  ← no duplicates!
```

> **Order note:** Since Python 3.7, dictionaries maintain insertion order. When you need alphabetical order, use `sorted()`.

---

### Dictionary as a Counter — Classic Pattern

A very common and powerful pattern: use a dict where keys are things you've seen, and values are the count.

```python
word = 'brontosaurus'   # Python
d = dict()
for c in word:
    if c not in d:
        d[c] = 1        # first time we've seen this letter
    else:
        d[c] = d[c] + 1  # seen it before, increment count
print(d)
# {'b': 1, 'r': 2, 'o': 2, 'n': 1, 't': 2, 's': 2, 'a': 1, 'u': 2}
```

> **Shortcut:** Python's `collections.Counter` does this automatically: `Counter('brontosaurus')`.

---

### Nesting

Nesting = storing complex structures inside dictionaries.

#### A List of Dictionaries

Use when many _objects_ share the same structure (e.g., a fleet of aliens, a list of users):

```python
alien_0 = {'color': 'green', 'points': 5}   # Python
alien_1 = {'color': 'yellow', 'points': 10}
alien_2 = {'color': 'red', 'points': 15}

aliens = [alien_0, alien_1, alien_2]

for alien in aliens:
    print(alien)

# Generate 30 identical aliens programmatically
aliens = []
for alien_number in range(30):
    new_alien = {'color': 'green', 'points': 5, 'speed': 'slow'}
    aliens.append(new_alien)

# Modify the first 3
for alien in aliens[:3]:
    if alien['color'] == 'green':
        alien['color'] = 'yellow'
        alien['speed'] = 'medium'
        alien['points'] = 10
```

#### A List Inside a Dictionary

Use when one key has _multiple values_ (e.g., a person with multiple favourite languages):

```python
pizza = {   # Python
    'crust': 'thick',
    'toppings': ['mushrooms', 'extra cheese'],
}

print(f"You ordered a {pizza['crust']}-crust pizza with the following toppings:")
for topping in pizza['toppings']:
    print(f"\t{topping}")

# Multiple favourite languages per person
favorite_languages = {
    'jen': ['python', 'ruby'],
    'sarah': ['c'],
    'edward': ['ruby', 'go'],
    'phil': ['python', 'haskell'],
}

for name, languages in favorite_languages.items():
    print(f"\n{name.title()}'s favorite languages are:")
    for language in languages:
        print(f"\t{language.title()}")
```

#### A Dictionary Inside a Dictionary

Use when each key maps to a rich object of its own (e.g., users on a website):

```python
users = {   # Python
    'aeinstein': {
        'first': 'albert',
        'last': 'einstein',
        'location': 'princeton',
    },
    'mcurie': {
        'first': 'marie',
        'last': 'curie',
        'location': 'paris',
    },
}

for username, user_info in users.items():
    print(f"\nUsername: {username}")
    full_name = user_info['first'] + " " + user_info['last']
    location = user_info['location']
    print(f"\tFull name: {full_name.title()}")
    print(f"\tLocation: {location.title()}")
```

> ⚠️ **Don't nest too deeply.** If you find yourself nesting more than 2 levels, there's usually a simpler design.

---

---

## PART 2 — TUPLES

### Core Concept

Think of a tuple like a **row in a database table** or the **coordinates of a point** — it's a fixed bundle of values that travel together and _never change_. Once a tuple is created, its contents are locked. This is what _immutable_ means.

Unlike lists (which are like train carriages you can add/remove), a tuple is like **a sealed envelope** — you can look at what's inside, but you can't add, remove, or swap the contents.

Tuples are also **hashable** (can be used as dictionary keys) and **comparable** (can be sorted). Lists cannot be used as dict keys.

---

### Creating a Tuple

```python
# With parentheses (recommended for clarity)
t = ('a', 'b', 'c', 'd', 'e')   # Python

# Without parentheses — technically valid (comma makes it a tuple, not parens!)
t = 'a', 'b', 'c', 'd', 'e'

# SINGLE ELEMENT — must have a trailing comma!
t1 = ('a',)            # ✅ type: tuple
t2 = ('a')             # ❌ type: str — parens without comma = just grouping!
print(type(t1))   # <class 'tuple'>
print(type(t2))   # <class 'str'>

# Empty tuple
t = tuple()
print(t)   # ()

# Convert a sequence to a tuple
t = tuple('lupins')
print(t)   # ('l', 'u', 'p', 'i', 'n', 's')

t = tuple([1, 2, 3])
print(t)   # (1, 2, 3)
```

---

### Accessing Elements

Tuples support indexing and slicing exactly like lists:

```python
t = ('a', 'b', 'c', 'd', 'e')   # Python

# Indexing
print(t[0])    # a
print(t[-1])   # e   ← negative indexing works too

# Slicing — returns a new tuple
print(t[1:3])   # ('b', 'c')
print(t[:2])    # ('a', 'b')
print(t[2:])    # ('c', 'd', 'e')
```

---

### Tuples are Immutable

You cannot change a tuple after creation. Any attempt raises a `TypeError`:

```python
t = ('a', 'b', 'c', 'd', 'e')   # Python

# WRONG — raises TypeError: object doesn't support item assignment
# t[0] = 'A'

# CORRECT — create a new tuple instead
t = ('A',) + t[1:]
print(t)   # ('A', 'b', 'c', 'd', 'e')

# Tuple methods — only 2 (no sort/append/remove):
t = (1, 2, 2, 3, 2)
print(t.count(2))   # 3  ← how many times 2 appears
print(t.index(3))   # 3  ← index of first occurrence of 3
```

> **Why immutable?** Immutability makes tuples safe to use as dictionary keys, safe to pass between functions (no aliasing bugs), and slightly faster to create than lists.

---

### Tuple Assignment — Python Superpower

One of Python's most elegant features: assign multiple variables at once by placing a tuple on the left side of `=`:

```python
# Basic tuple unpacking
m = ('have', 'fun')   # Python
x, y = m
print(x)   # have
print(y)   # fun

# Stylistically, parens on the left are usually omitted:
x, y = 'have', 'fun'   # equivalent

# SWAP two variables — no temp variable needed!
a, b = 5, 10
a, b = b, a   # elegant Python swap
print(a, b)   # 10 5

# The right side can be any sequence (list, string, tuple)
addr = 'monty@python.org'
uname, domain = addr.split('@')
print(uname)    # monty
print(domain)   # python.org

# Must match counts — mismatched raises ValueError
# a, b = 1, 2, 3   # ValueError: too many values to unpack
```

---

### Comparing Tuples

Python compares tuples element-by-element, left to right — just like alphabetical ordering, but generalised to any type:

```python
print((0, 1, 2) < (0, 3, 4))         # True  — first element equal, compare 2nd
print((0, 1, 2000000) < (0, 3, 4))   # True  — 2000000 is irrelevant once 1 < 3

# sort() uses this element-by-element comparison
t = [(3, 'b'), (1, 'z'), (2, 'a')]
t.sort()
print(t)   # [(1, 'z'), (2, 'a'), (3, 'b')]  ← sorted by first element
```

---

### DSU Pattern — Decorate, Sort, Undecorate

A powerful pattern using tuple comparison to sort by a computed key:

```python
# Sort words from longest to shortest   # Python
txt = 'but soft what light in yonder window breaks'
words = txt.split()

# 1. DECORATE — pair each word with its length
t = []
for word in words:
    t.append((len(word), word))   # (length, word)

# 2. SORT — sorts by first element (length), breaks ties by second (word)
t.sort(reverse=True)

# 3. UNDECORATE — extract words in sorted order
result = []
for length, word in t:
    result.append(word)

print(result)
# ['yonder', 'window', 'breaks', 'light', 'what', 'soft', 'but', 'in']
```

---

### Dictionaries and Tuples Together

`dict.items()` returns a list of `(key, value)` tuples — enabling powerful sorting tricks:

```python
d = {'b': 1, 'a': 10, 'c': 22}   # Python

# Convert dict to list of tuples — then sort by key
t = list(d.items())
t.sort()
print(t)   # [('a', 10), ('b', 1), ('c', 22)]

# Loop using tuple assignment
for key, val in d.items():
    print(val, key)
# 10 a, 1 b, 22 c

# Sort dictionary contents BY VALUE (not key)
lst = []
for key, val in d.items():
    lst.append((val, key))   # (value, key) — value first!
lst.sort(reverse=True)

for val, key in lst:
    print(key, val)
# c 22, a 10, b 1   ← sorted by value, descending
```

#### Top-N Pattern — Find Most Common

```python
# Count word frequencies then find top 10
import string
fhand = open('romeo-full.txt')   # Python
counts = dict()
for line in fhand:
    line = line.translate(str.maketrans('', '', string.punctuation))
    line = line.lower()
    words = line.split()
    for word in words:
        counts[word] = counts.get(word, 0) + 1

# Build (value, key) list and sort descending
lst = []
for key, val in counts.items():
    lst.append((val, key))

lst.sort(reverse=True)
for val, key in lst[:10]:   # top 10
    print(key, val)
# i 61, and 42, romeo 40, ...
```

---

### Tuples as Dictionary Keys

Because tuples are **hashable**, they can be used as dictionary keys. Lists cannot.

```python
# Telephone directory: map (last_name, first_name) → phone number
directory = {}   # Python
directory['Doe', 'John'] = '555-1234'   # the key IS a tuple
directory['Smith', 'Jane'] = '555-5678'

# Access
print(directory['Doe', 'John'])   # 555-1234

# Loop through composite keys using tuple assignment
for last, first in directory:
    print(first, last, directory[last, first])
```

---

### When to Use Tuples vs Lists

|Situation|Use|
|---|---|
|Data that must not change (coordinates, RGB colours)|Tuple|
|Need to use as a dictionary key|Tuple|
|Returning multiple values from a function|Tuple|
|Passing to a function to prevent aliasing bugs|Tuple|
|Data that needs to grow/shrink (shopping cart)|List|
|Need `sort()`, `append()`, `remove()`|List|

---

### List Comprehension — Bonus

A concise way to build a list from another sequence:

```python
# Traditional loop
list_of_ints = []   # Python
for x in ['42', '65', '12']:
    list_of_ints.append(int(x))

# List comprehension — same thing in one line
list_of_ints = [int(x) for x in ['42', '65', '12']]
print(sum(list_of_ints))   # 119
```

---

---

## PART 3 — SETS

### Core Concept

Think of a set like a **bag of unique marbles** — you can throw marbles in and check if a marble is in the bag, but each marble appears only once (no duplicates), and the marbles have no particular order.

Sets are the right tool when you care about **membership** and **uniqueness**, not position or counts. They're also the tool for classic mathematical set operations: union, intersection, difference.

Like dictionary keys (because they use the same hash table), **set elements must be hashable** — so numbers, strings, and tuples can go in a set, but lists and dicts cannot.

---

### Creating a Set

```python
# Literal syntax — curly braces with values (NOT key:value pairs!)
fruits = {'apple', 'banana', 'cherry'}   # Python

# ⚠️ Empty set — CANNOT use {} (that creates an empty dict!)
empty_set = set()   # correct
empty_dict = {}     # this is a dict, not a set!

# Convert from list (removes duplicates!)
languages = ['python', 'c', 'ruby', 'python', 'c']
unique_languages = set(languages)
print(unique_languages)   # {'python', 'c', 'ruby'}  — order may vary

# Convert from string (each character becomes an element)
vowels = set('aeiou')
print(vowels)   # {'a', 'e', 'i', 'o', 'u'}

# Set comprehension
squares = {x**2 for x in range(1, 6)}
print(squares)   # {1, 4, 9, 16, 25}
```

---

### Adding and Removing Elements

```python
fruits = {'apple', 'banana'}   # Python

# add() — add one element
fruits.add('cherry')
print(fruits)   # {'apple', 'banana', 'cherry'}

# Adding a duplicate — silently ignored (no error, no change)
fruits.add('apple')
print(fruits)   # {'apple', 'banana', 'cherry'}  ← unchanged

# update() — add multiple elements from an iterable
fruits.update(['mango', 'papaya'])

# remove() — removes element, raises KeyError if not found
fruits.remove('banana')

# discard() — removes element, does NOTHING if not found (safe!)
fruits.discard('not_in_set')   # no error

# pop() — removes and returns a random element (since sets are unordered)
removed = fruits.pop()

# clear() — empties the entire set
fruits.clear()
print(fruits)   # set()
```

---

### Membership Testing

Sets excel at `in` checks — O(1) speed regardless of size:

```python
fruits = {'apple', 'banana', 'cherry'}   # Python

print('apple' in fruits)       # True
print('mango' in fruits)       # False
print('mango' not in fruits)   # True

# Practical use from the PDFs — unique values from dict.values()
favorite_languages = {
    'jen': 'python',
    'sarah': 'c',
    'edward': 'ruby',
    'phil': 'python',
}
for language in set(favorite_languages.values()):
    print(language.title())
# Output: Python, C, Ruby  ← no "Python" twice!
```

---

### Set Operations — The Mathematical Power

```python
a = {1, 2, 3, 4, 5}   # Python
b = {3, 4, 5, 6, 7}

# Union — all elements from BOTH sets (no duplicates)
print(a | b)          # {1, 2, 3, 4, 5, 6, 7}
print(a.union(b))     # same

# Intersection — elements in BOTH sets
print(a & b)              # {3, 4, 5}
print(a.intersection(b))  # same

# Difference — elements in a but NOT in b
print(a - b)              # {1, 2}
print(a.difference(b))    # same

# Symmetric Difference — elements in ONE but NOT BOTH
print(a ^ b)                         # {1, 2, 6, 7}
print(a.symmetric_difference(b))     # same
```

---

### Set Relations

```python
a = {1, 2, 3}   # Python
b = {1, 2, 3, 4, 5}
c = {6, 7}

# Subset — all of a's elements are in b
print(a.issubset(b))    # True
print(a <= b)           # True (same)

# Superset — b contains all of a
print(b.issuperset(a))  # True
print(b >= a)           # True (same)

# Disjoint — no elements in common
print(a.isdisjoint(c))  # True
print(a.isdisjoint(b))  # False
```

---

### Frozen Sets

A `frozenset` is an **immutable** set — can be used as a dictionary key or inside another set:

```python
fs = frozenset([1, 2, 3])   # Python
print(fs)   # frozenset({1, 2, 3})

# Can be a dictionary key
lookup = {frozenset({1, 2}): 'pair', frozenset({3}): 'single'}
print(lookup[frozenset({1, 2})])   # pair

# Cannot modify a frozenset
# fs.add(4)   # AttributeError: 'frozenset' object has no attribute 'add'
```

---

### Sets vs Lists vs Dicts for Membership

```python
# Practical comparison — finding unique words in a file
words_list = []   # Python
words_set = set()

with open('some_file.txt') as f:
    for line in f:
        for word in line.split():
            words_list.append(word)
            words_set.add(word)

# words_set has no duplicates — instantly
# 'python' in words_list  ← O(n) — scans entire list
# 'python' in words_set   ← O(1) — hash lookup, near instant
```

---

---

## Glossary

|Term|Definition|
|---|---|
|**dictionary**|A collection of key-value pairs where keys must be unique and hashable|
|**key**|The index used to look up a value in a dictionary — must be immutable/hashable|
|**value**|The data associated with a key — can be any type|
|**key-value pair**|A single entry in a dictionary — the key and its associated value together|
|**item**|Another word for a key-value pair in a dictionary|
|**hashable**|An object that has a hash value which never changes during its lifetime — immutable types (int, str, tuple) are hashable; mutable types (list, dict) are not|
|**hash table**|The internal data structure Python uses for dicts and sets — enables O(1) lookup|
|**tuple**|An immutable, ordered sequence of values — like a frozen list|
|**immutable**|Cannot be changed after creation — tuples and strings are immutable; lists and dicts are not|
|**tuple assignment**|Assigning multiple variables at once using a sequence on the right: `x, y = 1, 2`|
|**DSU**|Decorate-Sort-Undecorate — a pattern using tuples to sort by a computed key|
|**comparable**|A type that supports `<`, `>`, `==` comparisons — enables sorting|
|**scatter**|Treating a sequence as individual arguments (unpacking)|
|**gather**|Collecting multiple arguments into a tuple (using `*args`)|
|**set**|An unordered collection of unique, hashable elements|
|**frozenset**|An immutable version of a set — can be used as a dict key|
|**union**|All elements from two sets combined — `a|
|**intersection**|Only elements present in both sets — `a & b`|
|**difference**|Elements in one set but not the other — `a - b`|
|**nesting**|Storing collections inside other collections — lists of dicts, dicts of lists, etc.|
|**shape error**|A bug caused by a data structure having the wrong type, size, or composition|

---

## Java vs Python

|Concept|Java|Python|
|---|---|---|
|Create dict/map|`HashMap<String,Integer> map = new HashMap<>()`|`my_dict = {}`|
|Create from values|`Map.of("a", 1, "b", 2)`|`{'a': 1, 'b': 2}`|
|Add/update entry|`map.put("key", value)`|`my_dict['key'] = value`|
|Get value|`map.get("key")`|`my_dict['key']`|
|Safe get|`map.getOrDefault("key", 0)`|`my_dict.get('key', 0)`|
|Remove entry|`map.remove("key")`|`del my_dict['key']` or `my_dict.pop('key')`|
|Check key exists|`map.containsKey("key")`|`'key' in my_dict`|
|Check value exists|`map.containsValue(val)`|`val in my_dict.values()`|
|Loop key-value|`for (Map.Entry<K,V> e : map.entrySet())`|`for k, v in my_dict.items()`|
|Loop keys only|`for (String k : map.keySet())`|`for k in my_dict.keys()`|
|Loop values only|`for (Integer v : map.values())`|`for v in my_dict.values()`|
|Count pairs|`map.size()`|`len(my_dict)`|
|Create tuple|No direct equivalent (use array or record)|`t = (1, 2, 3)`|
|Tuple unpacking|No equivalent|`x, y = (1, 2)`|
|Variable swap|`int tmp = a; a = b; b = tmp;`|`a, b = b, a`|
|Create set|`HashSet<String> set = new HashSet<>()`|`my_set = {'a', 'b'}`|
|Add to set|`set.add("x")`|`my_set.add('x')`|
|Remove from set|`set.remove("x")`|`my_set.discard('x')` (safe)|
|Set union|`set1.addAll(set2)` (modifies)|`set1 \| set2` (new set)|
|Set intersection|`set1.retainAll(set2)` (modifies)|`set1 & set2` (new set)|
|Set difference|`set1.removeAll(set2)` (modifies)|`set1 - set2` (new set)|
|Membership test|`set.contains("x")`|`'x' in my_set` — O(1)!|
|Immutable set|`Collections.unmodifiableSet(set)`|`frozenset(...)`|

---

## Quick Reference

```python
# ══ DICTIONARIES ══════════════════════════════════════════════════   # Python

# Creating
d = {}                                   # empty
d = {'a': 1, 'b': 2}                    # from values
d = dict(a=1, b=2)                      # using dict()

# Accessing
d['a']                                   # 1 (KeyError if missing)
d.get('a')                               # 1 (None if missing)
d.get('x', 0)                            # 0 (custom default if missing)

# Modifying
d['c'] = 3                               # add/update
del d['a']                               # remove by key
val = d.pop('b')                         # remove and return value
val = d.pop('x', None)                   # safe pop with default

# Checking
'a' in d                                 # True — checks KEYS
'a' not in d                             # False
len(d)                                   # number of key-value pairs

# Looping
for k in d:                              # loop through keys (default)
for k in d.keys():                       # explicit keys
for v in d.values():                     # loop through values
for k, v in d.items():                   # loop through key-value pairs
for k in sorted(d.keys()):              # keys in alphabetical order

# Useful patterns
d[key] = d.get(key, 0) + 1              # counter pattern
items_sorted_by_value = sorted(d.items(), key=lambda kv: kv[1])


# ══ TUPLES ════════════════════════════════════════════════════════

# Creating
t = (1, 2, 3)                            # with parens
t = 1, 2, 3                              # without parens (also valid)
t = (42,)                                # single element — needs trailing comma!
t = tuple()                              # empty
t = tuple('abc')                         # from string → ('a', 'b', 'c')

# Accessing
t[0]                                     # first element
t[-1]                                    # last element
t[1:3]                                   # slice → new tuple

# Tuple assignment
x, y = 10, 20                            # unpack
a, b = b, a                              # swap — elegant Python idiom
uname, domain = 'monty@python.org'.split('@')

# Sorting
sorted(t)                                # returns new sorted LIST
t.count(val)                             # count occurrences
t.index(val)                             # find index of first occurrence

# With dicts
list(d.items())                          # list of (key, value) tuples
for k, v in d.items()                    # tuple assignment in a for loop


# ══ SETS ══════════════════════════════════════════════════════════

# Creating
s = {1, 2, 3}                            # literal (note: {} alone = empty DICT!)
s = set()                                # empty set
s = set([1, 2, 2, 3])                   # from list → {1, 2, 3} (deduped)
s = {x**2 for x in range(5)}            # set comprehension

# Modifying
s.add(4)                                 # add one element
s.update([5, 6])                         # add multiple elements
s.remove(3)                              # remove (KeyError if absent)
s.discard(99)                            # remove (silent if absent — safer)
s.pop()                                  # remove+return random element
s.clear()                                # empty the set

# Checking
3 in s                                   # True
3 not in s                               # False
len(s)                                   # count of elements

# Set operations
a | b    # a.union(b)            → all elements in a OR b
a & b    # a.intersection(b)     → elements in BOTH a AND b
a - b    # a.difference(b)       → elements in a but NOT b
a ^ b    # a.symmetric_difference(b) → elements in exactly ONE

# Relations
a <= b   # a.issubset(b)         → all of a's elements are in b
a >= b   # a.issuperset(b)       → a contains all of b
a.isdisjoint(b)                  → no elements in common

# Immutable
fs = frozenset({1, 2, 3})        → can be dict key, can't be modified
```

---

## Debugging Common Bugs

**1. `KeyError` — accessing a missing key**

```python
d = {'a': 1}
print(d['b'])           # ❌ KeyError!
print(d.get('b', 0))   # ✅ returns 0
```

**2. Using `{}` for an empty set**

```python
empty = {}              # ❌ This is an empty DICT, not a set!
empty = set()           # ✅ Correct empty set
```

**3. Tuple single-element missing comma**

```python
t = (42)    # ❌ This is just an int in parens!
t = (42,)   # ✅ Trailing comma makes it a tuple
```

**4. Trying to modify a tuple**

```python
t = (1, 2, 3)
t[0] = 99              # ❌ TypeError — tuples are immutable!
t = (99,) + t[1:]      # ✅ Create a new tuple instead
```

**5. Looping through a dict and modifying it**

```python
for k in d:             # ❌ RuntimeError if you add/remove keys during loop!
    del d[k]

for k in list(d.keys()):   # ✅ Loop over a copy of keys
    del d[k]
```

**6. Mutable default in function argument**

```python
def add(item, t=[]):   # ❌ list is shared across ALL calls!
    t.append(item)
    return t

def add(item, t=None):   # ✅ create fresh list each call
    if t is None:
        t = []
    t.append(item)
    return t
```

---

## Practice Exercises

**Dictionaries:**

- [ ] **Person** — Store `first_name`, `last_name`, `age`, `city` about yourself. Print each value with a label.
- [ ] **Glossary** — Create a dict with 5 programming terms as keys and their definitions as values. Loop through and print each term neatly. Add 5 more terms and loop again.
- [ ] **Rivers** — Dict of `{'nile': 'egypt', 'amazon': 'brazil', 'yangtze': 'china'}`. Loop `.items()` to print "The Nile runs through Egypt." etc.
- [ ] **Polling** — Store favorite languages dict. Make a list of people who should vote. Loop: if they're already in the dict, thank them; if not, invite them.
- [ ] **Alien Fleet** — Generate 30 identical green aliens using `range()`. Change the first 5 to yellow medium-speed. Print the first 5 to confirm.
- [ ] **Pizza Order** — Dict with `crust` and `toppings` (a list). Print a readable order summary.
- [ ] **Cities** — Dict of dicts: each city key maps to a sub-dict with `country`, `population`, `fact`. Loop and print all info neatly.
- [ ] **Counter** — Count how many times each word appears in a user-typed sentence.

**Tuples:**

- [ ] **Dimensions** — Create a tuple of screen dimensions `(1920, 1080)`. Print the values with labels. Then try to assign a new value — observe the error.
- [ ] **Swap** — Swap two variables using tuple assignment in one line. Print before and after.
- [ ] **DSU sort** — Take a list of words, sort them by length (longest first). Use DSU.
- [ ] **Email splitter** — Use tuple assignment with `.split('@')` to separate username and domain.
- [ ] **Dict sorted by value** — Build a word frequency counter, then print the top 5 words by count using the (value, key) tuple trick.
- [ ] **Tuple as dict key** — Simulate a phone book using `(last, first)` tuples as keys.

**Sets:**

- [ ] **Unique words** — Read a multi-line string, split into words, find unique ones using a set. Compare performance idea vs using a list.
- [ ] **Common languages** — Two students each list their favorite programming languages. Find languages they both like (intersection), languages only one likes (symmetric difference), all languages together (union).
- [ ] **Deduplication** — Take a list with many duplicates and convert to a set to remove them. Convert back to a sorted list.
- [ ] **Fast lookup** — Take a list of 1 million numbers (use `range()`). Compare `5 in list(range(1000000))` vs `5 in set(range(1000000))` using `time` module.

---

## Questions I Still Have

_Write your open questions here. Return later and answer them._

---

## Related Notes

- [[LIST]]
- [[ITERATIVE STATEMENTS]]
- [[CONDITIONAL STATEMENT]]
- [[FUNCTIONS]]