---
title: STRINGS
date: 2026-05-16
phase: phase-1
tags:
  - concept
  - python
links: []
status: learning
---
# STRINGS

## One-Line Summary

A string is a **sequence** of characters accessed individually by index (0-based), traversed with loops, sliced with `[n:m]` notation, and immutable (unchanged after creation); master string methods like `.upper()`, `.find()`, `.strip()`, and `.replace()` for powerful text manipulation.

---

## Core Concept

Think of a string like a train with numbered cars. Each car holds one character. You can look at car 0 (first char), car 1 (second char), and so on. You can walk the train from front to back (traverse), grab a section (slice), check if a car has a specific item (search), but you can't change the car itself — if you want a different one, you build a new train (immutability).

Unlike Java, where strings are also immutable, Python's string operations feel more natural with methods: `s.upper()` instead of `String.toUpperCase(s)`.

---

## How It Works

### Strings Are Sequences — Zero-Indexed

**Index = offset from the beginning, starting at 0**

```python
# Python
fruit = 'banana'
letter = fruit[0]        # index 0 = 'b' (first character)
print(letter)
# Output: b

# NOT 'a' (unlike everyday language where "first" = 1)
```

**Diagram (conceptual):**

```
String: b  a  n  a  n  a
Index:  0  1  2  3  4  5
```

Key insight: index 1 is the **second** character ('a'), index 2 is **third** ('n'), etc.

---

### Getting the Length — `len()`

```python
fruit = 'banana'
length = len(fruit)      # Python
print(length)            # 6
```

**Problem:** To get the last character, you might try:

```python
last = fruit[length]     # Python — WRONG
# IndexError: string index out of range
```

Why? Because `fruit` has 6 characters (indices 0–5), so index 6 doesn't exist.

**Solution 1: Subtract 1**

```python
last = fruit[length - 1]  # Python — 'a' (index 5)
print(last)
# Output: a
```

**Solution 2: Negative Indices (Pythonic)**

```python
last = fruit[-1]          # Python — last character
second_last = fruit[-2]   # second-to-last
first = fruit[-6]         # equivalent to fruit[0]
```

Negative indices count **backward** from the end: `-1` is last, `-2` is second-to-last, etc.

---

### Traversing a String — Loop Through Each Character

#### Option 1: While Loop (Index-Based)

```python
index = 0
while index < len(fruit):     # Python
    letter = fruit[index]
    print(letter)
    index = index + 1
# Output: b a n a n a (each on separate line)
```

Loop continues while `index < len(fruit)`. When `index` reaches 6, condition is false, loop exits.

> **Exercise 1 (from PDF):** Write a while loop that starts at the **last** character and works backward, printing each letter on a separate line.

#### Option 2: For Loop (Pythonic)

```python
for char in fruit:              # Python
    print(char)
# Output: b a n a n a

# Each iteration, 'char' takes the next character
```

**Key difference:** For loop automatically iterates; no index management needed. `char` is just the loop variable name (could be any name).

---

### String Slices — Extract a Segment

A **slice** is a portion of a string. Use bracket notation `[start:end]` where `start` is inclusive, `end` is **exclusive** (stop before it).

```python
s = 'Monty Python'
print(s[0:5])         # Python → 'Monty' (indices 0,1,2,3,4; NOT 5)
print(s[6:12])        # → 'Python' (indices 6-11)
```

**Omitting indices:**

```python
fruit = 'banana'
print(fruit[:3])      # Python → 'ban' (start at 0, stop before 3)
print(fruit[3:])      # → 'ana' (start at 3, go to end)
print(fruit[:])       # → 'banana' (entire string)
```

**Empty string (when start ≥ end):**

```python
print(fruit[3:3])     # Python → '' (empty string)
print(fruit[5:3])     # → '' (start > end, result is empty)
```

> **Exercise 2 (from PDF):** Given `fruit = 'banana'`, what does `fruit[:]` mean? (Answer: entire string, a copy)

---

### Strings Are Immutable

**You cannot change a character in a string.**

```python
greeting = 'Hello, world!'
greeting[0] = 'J'                     # Python — TypeError
# TypeError: 'str' object does not support item assignment
```

**Why?** Strings are immutable — designed to be unchangeable. This makes them safe (can be used as dict keys, won't surprise you mid-use).

**Solution: Create a new string**

```python
greeting = 'Hello, world!'
new_greeting = 'J' + greeting[1:]     # Python
print(new_greeting)
# Output: Jello, world!
```

This **concatenates** a new first letter with a slice of the original (from index 1 onward), creating a new string. The original is unchanged.

---

### Looping and Counting — The Counter Pattern

**Typical pattern:** Initialize counter to 0, loop through string, increment when condition met.

```python
word = 'banana'
count = 0
for letter in word:                   # Python
    if letter == 'a':
        count = count + 1
print(count)
# Output: 3 (three 'a's in 'banana')
```

This is called a **counter** — a variable that tracks a count, incremented each time you find what you're looking for.

> **Exercise 3 (from PDF):** Encapsulate this code in a function `count(word, letter)` that counts occurrences of `letter` in `word`.

---

### The `in` Operator — Substring Search

`in` is a **boolean operator** (returns `True` or `False`) that checks if a substring appears in a string.

```python
'a' in 'banana'          # Python → True
'seed' in 'banana'       # → False
```

Works for substrings, not just single characters:

```python
'na' in 'banana'         # Python → True (appears twice, but result is just True)
```

---

### String Comparison — `==`, `<`, `>`

**Equality:**

```python
if word == 'banana':                  # Python
    print('All right, bananas.')
```

**Alphabetical ordering:**

```python
if word < 'banana':                   # Python
    print(f'Your word, {word}, comes before banana.')
elif word > 'banana':
    print(f'Your word, {word}, comes after banana.')
else:
    print('All right, bananas.')
```

**Gotcha: Case matters!**

```python
'Pineapple' < 'banana'               # Python → True
# Because uppercase letters come BEFORE lowercase in ASCII
# 'P' (80) < 'b' (98)
```

**Solution: Normalize case before comparing**

```python
if word.lower() < 'banana':           # Python
    # Now case doesn't matter
```

---

### String Methods — Built-In Functions

Strings are **objects** with **methods** — functions attached to the object, called with dot notation: `string.method()`.

```python
word = 'banana'
new_word = word.upper()               # Python
print(new_word)
# Output: BANANA
```

**Key methods:**

|Method|Example|Output|
|---|---|---|
|`.upper()`|`'banana'.upper()`|`'BANANA'`|
|`.lower()`|`'HELLO'.lower()`|`'hello'`|
|`.capitalize()`|`'hello'.capitalize()`|`'Hello'`|
|`.find(sub)`|`'banana'.find('a')`|`1` (first occurrence)|
|`.find(sub, start)`|`'banana'.find('a', 3)`|`3` (search starting from index 3)|
|`.strip()`|`' hello '.strip()`|`'hello'` (removes leading/trailing whitespace)|
|`.startswith(prefix)`|`'hello'.startswith('he')`|`True`|
|`.replace(old, new)`|`'hello'.replace('l', 'x')`|`'hexxo'`|
|`.split(sep)`|`'a,b,c'.split(',')`|`['a', 'b', 'c']` (returns a list)|
|`.count(sub)`|`'banana'.count('a')`|`3`|

**Discover methods with `dir()`:**

```python
stuff = 'Hello world'
dir(stuff)     # Lists all methods: capitalize, casefold, center, count, ...
help(str.capitalize)  # Get detailed help on a method
```

#### Method `.find(substring)`

Searches for substring, returns **index of first occurrence** or `-1` if not found:

```python
word = 'banana'
index = word.find('a')       # Python → 1
index = word.find('na')      # → 2
index = word.find('seed')    # → -1 (not found)
```

#### Method `.startswith(prefix)`

Returns boolean; case-sensitive:

```python
line = 'Have a nice day'
line.startswith('Have')      # Python → True
line.startswith('h')         # → False (wrong case)
line.lower().startswith('h') # → True (after normalizing)
```

#### Chaining Methods

You can call multiple methods in sequence. Each returns a string; the next method operates on that string:

```python
line = 'Have a nice day'
line.lower().startswith('h')              # Python → True
# Equivalent to:
# temp = line.lower()  # 'have a nice day'
# temp.startswith('h')  # True
```

#### Method `.count(substring)`

Counts occurrences of substring:

```python
'banana'.count('a')          # Python → 3
'banana'.count('na')         # → 2
```

> **Exercise 4 (from PDF):** Read the string methods documentation, then use `.count()` to count letter 'a' in 'banana'.

---

### Parsing Strings — Extract Information

**Real-world example:** Extract email domain from an email address.

```python
data = 'From stephen.marquardt@uct.ac.za Sat Jan 5 09:14:16 2008'

# Find position of '@' symbol
atpos = data.find('@')                # Python → 21

# Find position of first space AFTER the '@'
sppos = data.find(' ', atpos)         # Start search from atpos → 31

# Extract substring from @ to space (excluding space)
host = data[atpos+1:sppos]            # data[22:31] → 'uct.ac.za'
print(host)
# Output: uct.ac.za
```

**Breakdown:**

1. Find `@` at index 21
2. Find space starting from index 21 (finds it at 31)
3. Slice from index 22 (one past `@`) to 31 (before space)

---

### Formatted String Literals (F-Strings)

**F-strings** (formatted string literals) embed expressions in strings using `{}` curly braces and `f` prefix:

```python
camels = 42
f'{camels}'                  # Python → '42' (expression becomes string)
f'I have spotted {camels} camels.'  # → 'I have spotted 42 camels.'

years = 3
count = 0.1
species = 'camels'
f'In {years} years I have spotted {count} {species}.'
# → 'In 3 years I have spotted 0.1 camels.'
```

**Power:** Any expression works inside `{}`:

```python
f'{2 + 3}'               # Python → '5'
f'{len("hello")}'        # → '5'
f'{3.14159:.2f}'         # → '3.14' (format specifier :.2f = 2 decimal places)
```

---

### Debugging Strings

**Problem:** Empty string causes `IndexError` when accessed:

```python
while True:                              # Python
    line = input('> ')
    if line[0] == '#':                   # ← Crashes if input is empty!
        continue
    if line == 'done':
        break
    print(line)

# When user presses Enter with empty input:
# IndexError: string index out of range
```

**Solution 1: Use `.startswith()` (returns False for empty string)**

```python
if line.startswith('#'):                # Python — safe for empty strings
    continue
```

**Solution 2: Check length first (guardian pattern)**

```python
if len(line) > 0 and line[0] == '#':    # Python
    # Evaluate len(line) > 0 first; if False, short-circuits
    # line[0] is never evaluated when line is empty
    continue
```

Python uses **short-circuit evaluation**: `and` stops as soon as it finds a `False`, so `line[0]` is never reached if `len(line) == 0`.

---

## Java vs Python

|Java|Python|
|---|---|
|`string.charAt(0)`|`string[0]`|
|`string.substring(1, 5)`|`string[1:5]` (end exclusive)|
|`string.length()`|`len(string)`|
|`string.toUpperCase()`|`string.upper()`|
|`string.indexOf('a')`|`string.find('a')`|
|Strings are immutable|Strings are immutable (same philosophy)|
|`+` concatenates|`+` concatenates (same)|
|`String.format()`|f-strings (more intuitive)|
|No slice notation|`string[:]` for copies, slices|

---

## Quick Reference

```python
# Indexing (0-based)
s = 'banana'
s[0]        # 'b' (first)
s[-1]       # 'a' (last)
s[-2]       # 'n' (second-to-last)

# Length
len(s)      # 6

# Slicing (start:end, end exclusive)
s[0:3]      # 'ban'
s[1:]       # 'anana' (to end)
s[:3]       # 'ban' (from start)
s[:]        # 'banana' (entire, copy)

# Immutability — create new string
s = 'X' + s[1:]  # Replace first char

# Iteration
for char in s:                  # for loop (best)
    print(char)

index = 0                       # while loop (older style)
while index < len(s):
    print(s[index])
    index += 1

# Membership
'a' in s                        # True
'seed' in s                     # False

# Comparison
s == 'banana'                   # True
s < 'cherry'                    # True (alphabetical)
s.lower() < 'cherry'            # Normalize case first

# Methods
s.upper()           # 'BANANA'
s.lower()           # 'banana'
s.find('a')         # 1 (first occurrence)
s.find('a', 2)      # 3 (start search from index 2)
s.count('a')        # 3
s.strip()           # Remove leading/trailing whitespace
s.startswith('ban') # True
s.replace('a', 'o') # 'bonona'
s.split(',')        # Split into list by delimiter

# Formatted strings (f-strings)
name = 'Alice'
age = 30
f'{name} is {age} years old'  # 'Alice is 30 years old'
f'{3.14159:.2f}'              # '3.14' (2 decimal places)

# Counter pattern
count = 0
for char in s:
    if char == 'a':
        count += 1
# count = 3

# Guardian pattern (safe indexing)
if len(s) > 0 and s[0] == 'b':
    print("Starts with 'b'")
```

---

## Practice Exercises

- [ ] **Indexing** — Given `word = 'python'`, print `word[0]`, `word[-1]`, `word[3]`.
- [ ] **Length Trick** — Get the last character without knowing the length (use negative index).
- [ ] **Slicing** — Extract the middle 3 characters of a 7-character string.
- [ ] **While Loop Traversal** — Reverse Exercise 1: write a while loop that prints the string backward.
- [ ] **For Loop Traversal** — Print each character on a separate line using for loop.
- [ ] **Character Count** — Count occurrences of 'e' in "The quick brown fox".
- [ ] **In Operator** — Check if 'quick' appears in the above string.
- [ ] **Case-Insensitive Comparison** — Compare two strings ignoring case (`.lower()`).
- [ ] **Slice Parsing** — Extract domain from email using `.find()` and slicing.
- [ ] **Methods Chaining** — Convert string to lowercase, check if it starts with 'h' (one line).
- [ ] **String Replacement** — Replace all 'o' with '0' (zero) in a string.
- [ ] **F-String** — Create a formatted sentence: "I have 5 apples and 3 oranges." using variables.
- [ ] **Guardian Pattern** — Write code that safely checks the first character even if the string is empty.
- [ ] **Counter in Function** — Wrap the counter pattern in a function `count(word, letter)`.

---

## Questions I Still Have

_Write your open questions here. Return later and answer them._

- Why does Python use 0-based indexing instead of 1-based like everyday language?
- Can I modify a string with a more efficient method than concatenation?
- What's the performance difference between `.find()` and the `in` operator?
- How do I escape special characters in strings (like quotes or backslashes)?

---

## Related Notes

- [[ITERATIVE STATEMENTS]] (loops used to traverse strings)
- [[FUNCTIONS]] (string methods are functions attached to objects)
- [[CONDITIONAL STATEMENT]] (if/else for string comparisons)