---
title: VARIABLES & SIMPLE DATA TYPES
date: 2026-05-12
phase: phase-1
tags:
  - concept
  - python
links: []
status: learning
---
# VARIABLES AND SIMPLE DATA TYPES

## One-Line Summary
Variables are labelled containers that store data, and Python has several built-in data types — strings, integers, floats, and booleans — each with their own rules and operations.

---

## Core Concept
Think of a variable like a sticky note. You write a name on it (`message`) and stick it onto a value (`"Hello, Python!"`). Whenever you use that name later, Python goes and finds whatever is stuck to that note.

Unlike Java or C, you never declare *what kind* of data a variable will hold. Python figures that out automatically when you assign a value.

```python
message = "Hello, Python!"
print(message)           # Output: Hello, Python!
```

---

## How It Works

### Variables — The Basics

The assignment operator is `=` (not equality — that's `==`).

```python
message = "Hello, Python!"
message = "Hello, World!"   # You can reassign anytime
print(message)              # Output: Hello, World!
```

Python simply points the variable name to the new value. The old value is discarded (garbage collected).

---

### Variable Naming Rules

| Rule | Example |
|---|---|
| Letters, numbers, underscores only | `user_name`, `score1` ✅ |
| Cannot start with a number | `1name` ❌ |
| No spaces | `user name` ❌ |
| No special characters | `user@name` ❌ |
| Case-sensitive | `Name` ≠ `name` |
| Cannot be a Python keyword | `for`, `if`, `class` ❌ |

**Python convention** → use `snake_case` (lowercase + underscores):
```python
user_name = "shlok"      # ✅ Pythonic
userName = "shlok"       # ❌ That's Java/camelCase style
```

> ⚠️ **Common Mistake:** `NameError: name 'mesage' is not defined` — Python will tell you when a variable doesn't exist. Check your spelling first.

---

## Strings

### What Is a String?
A **string** is any sequence of characters wrapped in quotes. You can use single or double quotes — both work.

```python
name = "ada lovelace"
name = 'ada lovelace'    # identical
```

Use one type when the other appears inside the string:
```python
message = "I told my friend, 'Python is awesome!'"
```

---

### String Methods
Methods are actions you call *on* a string using dot notation: `string.method()`. They **do not modify the original** — they return a new string.

| Method | What It Does | Example |
|---|---|---|
| `.title()` | Capitalises each word | `"ada lovelace".title()` → `"Ada Lovelace"` |
| `.upper()` | All uppercase | `"hello".upper()` → `"HELLO"` |
| `.lower()` | All lowercase | `"HELLO".lower()` → `"hello"` |
| `.strip()` | Removes whitespace from both ends | `"  hello  ".strip()` → `"hello"` |
| `.lstrip()` | Removes whitespace from the left | `"  hello  ".lstrip()` → `"hello  "` |
| `.rstrip()` | Removes whitespace from the right | `"  hello  ".rstrip()` → `"  hello"` |
| `.removeprefix("x")` | Removes a specific prefix (Python 3.9+) | `"https://site.com".removeprefix("https://")` → `"site.com"` |
| `.removesuffix("x")` | Removes a specific suffix (Python 3.9+) | `"file.txt".removesuffix(".txt")` → `"file"` |

```python
name = "ada lovelace"
print(name.title())   # Ada Lovelace
print(name.upper())   # ADA LOVELACE
print(name.lower())   # ada lovelace
```

> 💡 `.lower()` is extremely useful when **storing user input** — you never know if someone types "ADA", "ada", or "Ada". Always normalise to lowercase before storing or comparing.

---

### f-Strings (String Interpolation)
The modern way to embed variables inside strings (Python 3.6+). Put `f` before the opening quote and wrap variable names in `{}`.

```python
first_name = "ada"
last_name = "lovelace"
full_name = f"{first_name} {last_name}"
print(full_name)                          # ada lovelace
print(f"Hello, {full_name.title()}!")     # Hello, Ada Lovelace!
```

You can put any expression inside `{}`:
```python
print(f"2 + 2 = {2 + 2}")    # 2 + 2 = 4
```

> 📝 **Note:** Before Python 3.6, people used `.format()`:
> ```python
> full_name = "{} {}".format(first_name, last_name)
> ```
> f-strings are cleaner and faster — always prefer them.

---

### Whitespace in Strings
**Whitespace** = spaces, tabs (`\t`), newlines (`\n`). Python lets you embed them directly in strings.

```python
print("Python")
print("\tPython")         # tab before Python
print("Languages:\nPython\nJava\nC")

Ouput:

Python
	Python
Languages:
Python
Java
C
```

**Stripping whitespace** is important for **user input** — extra spaces cause silent bugs:
```python
username = "  shlok  "
print(username == "shlok")         # False! (spaces matter)
print(username.strip() == "shlok") # True
```

> ⚠️ **Common Mistake:** Stripping only returns a new string — it does NOT change the original. Save it back:
> ```python
> username = username.strip()   # now the variable is clean
> ```

---

## Numbers

### Integers
Whole numbers, positive or negative, no decimal point.

```python
age = 20
print(age)       # 20
```

**Arithmetic operators:**
```python
2 + 3    # 5    — addition
5 - 2    # 3    — subtraction
3 * 4    # 12   — multiplication
10 / 2   # 5.0  — division (always returns float in Python 3)
10 // 3  # 3    — floor division (integer result, truncates)
10 % 3   # 1    — modulo (remainder)
2 ** 3   # 8    — exponent (2 to the power of 3)
```

> ⚠️ **Gotcha from Java/C:** In Python 3, `10 / 2` = `5.0` (a float), NOT `5` (an integer). Use `//` for integer division.

**Underscores in large numbers** — Python ignores them; they're just for readability:
```python
universe_age = 14_000_000_000     # same as 14000000000
print(universe_age)               # 14000000000
```

**Multiple assignment on one line:**
```python
x, y, z = 0, 0, 0    # assigns 0 to all three
```

---

### Floats
Numbers with a decimal point.

```python
price = 0.1
pi = 3.14159
```

**Floating point imprecision** — a known limitation of how computers store decimals in binary:
```python
print(0.1 + 0.1)    # 0.2       ✅
print(0.2 + 0.1)    # 0.30000000000000004  ⚠️ not exactly 0.3
```

This is not a Python bug — it's a fundamental property of IEEE 754 floating-point arithmetic used by almost every programming language. For financial calculations, use the `decimal` module.

---

### Integer + Float Mixed Math
When you mix an int and a float, Python always returns a float:
```python
print(1 + 2.0)    # 3.0
print(3 * 2.0)    # 6.0
```

---

## Booleans

A **Boolean** is a value that is either `True` or `False` — exactly two options. (Capital T and F — this differs from Java's lowercase.)

```python
is_active = True
is_banned = False
```

Booleans are typically produced by comparison operations:
```python
print(5 > 3)      # True
print(5 == 5)     # True
print(5 != 5)     # False
```

> ⚠️ **Common Mistake:** `True` and `False` must be capitalised. `true` and `false` are `NameError` in Python (unlike Java/C/JavaScript).

Booleans are actually a subtype of integers in Python:
```python
print(True + True)    # 2
print(False + 5)      # 5
```
You won't use this often, but it's good to know.

---

## Constants

Python has **no built-in constant keyword** (unlike Java's `final` or C's `const`). By convention, a variable written in ALL_CAPS signals "don't change this":

```python
MAX_CONNECTIONS = 5000
PI = 3.14159
```

This is purely a convention — Python won't stop you from changing it. Just don't.

---

## The `type()` Function
Check what type a value or variable is:

```python
print(type(42))         # <class 'int'>
print(type(3.14))       # <class 'float'>
print(type("hello"))    # <class 'str'>
print(type(True))       # <class 'bool'>
```

---

## Type Conversion (Casting)
Convert between types explicitly:

```python
# String → Integer
age_str = "20"
age = int(age_str)      # 20

# Integer → String
count = 42
label = str(count)      # "42"

# Integer → Float
x = float(5)            # 5.0

# Float → Integer (truncates, doesn't round)
y = int(3.9)            # 3  ← not 4!
```

> ⚠️ **Common Mistake:** You can't concatenate a string and an integer directly:
> ```python
> age = 20
> print("I am " + age + " years old.")    # TypeError!
> print("I am " + str(age) + " years old.")  # ✅
> print(f"I am {age} years old.")            # ✅ (better — use f-strings)
> ```

---

## Comments

Comments are ignored by Python — they're notes for humans reading the code.

```python
# This is a single-line comment
message = "Hello"   # inline comment — after the code

# Multi-line comments are just multiple # lines
# Python has no block comment syntax like Java's /* */
```

> 💡 **Good commenting habit:** Don't explain *what* the code does (that's readable from the code). Explain *why* you made a specific decision.
> ```python
> # Using .lower() because user input casing is unpredictable
> username = input("Username: ").lower()
> ```

---

## The Zen of Python — Why Readability Matters

```python
import this    # run this in any Python shell
```

Key principles that shape how Python handles variables and types:
- *Readability counts* — `snake_case` names, clear variable names
- *Explicit is better than implicit* — Python makes you convert types manually
- *Simple is better than complex* — no type declarations, no semicolons, no `{}` blocks

---

## Quick Reference — Type Comparison Table

| Concept | Python | Java | C |
|---|---|---|---|
| Integer | `x = 5` | `int x = 5;` | `int x = 5;` |
| Float | `x = 3.14` | `double x = 3.14;` | `double x = 3.14;` |
| String | `x = "hello"` | `String x = "hello";` | `char x[] = "hello";` |
| Boolean | `x = True` | `boolean x = true;` | `int x = 1;` (no bool) |
| Constant | `MAX = 100` (convention) | `final int MAX = 100;` | `const int MAX = 100;` |
| Type check | `type(x)` | `x.getClass()` | Not built-in |
| Type convert | `int("5")` | `Integer.parseInt("5")` | `atoi("5")` |
| String interpolate | `f"Hello {name}"` | `"Hello " + name` | `sprintf(buf, "Hello %s", name)` |
| Integer division | `10 // 3` → `3` | `10 / 3` → `3` | `10 / 3` → `3` |
| Exponent | `2 ** 3` → `8` | `Math.pow(2, 3)` | `pow(2, 3)` |

---

## Common Mistakes Summary

| Mistake | Error | Fix |
|---|---|---|
| `mesage = "hi"` then `print(message)` | `NameError` | Fix the typo |
| `"Age: " + 20` | `TypeError` | Use `str(20)` or f-string |
| `int("3.14")` | `ValueError` | Use `float("3.14")` first |
| Forgetting `.strip()` on user input | Silent bug — spaces included | Always strip input |
| `print(0.1 + 0.2 == 0.3)` | `False` | Floating point is imprecise |
| `true` instead of `True` | `NameError` | Capitalise: `True` / `False` |
| Relying on `//` giving the same result as Java's `/` for negatives | Subtle bug | `//` in Python floors (rounds toward −∞), Java truncates toward 0 |

---

## Questions I Still Have

*< Write your open questions here. Return later and answer them. >*

---

## Related Notes

- [[03-Programming/Python/PYTHON BASICS/INTRODUCTION]]
- 