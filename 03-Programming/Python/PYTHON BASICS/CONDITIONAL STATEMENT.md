---
title: CONDITIONAL STATEMENT
date: 2026-05-12
phase: phase-1
tags:
  - concept
  - python
links: []
status: learning
---
# IF STATEMENTS

## One-Line Summary
`if` statements let your program examine the current state of a program and respond appropriately — running a block of code only when a condition is `True`.

---

## Core Concept
Think of a traffic light. Your code looks at the light colour and decides what to do: green → go, red → stop, yellow → slow down. That's an `if` statement — it checks a condition and picks the right action.

Unlike Java or C, there are no curly braces `{}` to wrap blocks. Python uses **indentation** to define what belongs inside an `if`. Get the indentation wrong and the logic breaks.

```python
cars = ['audi', 'bmw', 'subaru', 'toyota']

for car in cars:
    if car == 'bmw':         # Python
        print(car.upper())   # Output: BMW
    else:
        print(car.title())   # Output: Audi, Subaru, Toyota
```

---

## How It Works

### Conditional Tests

Every `if` statement is built around a **conditional test** — an expression that evaluates to `True` or `False`. These are also called **Boolean expressions**.

**Checking for equality** — uses `==` (double equals):
```python
car = 'bmw'       # Python
car == 'bmw'      # True
car == 'audi'     # False
```
`=` assigns a value. `==` asks a question: *"are these the same?"*

**Case-insensitive equality** — convert to lowercase before comparing. The original variable is not modified:
```python
car = 'Audi'             # Python
car.lower() == 'audi'    # True  (car still stores 'Audi')
```

**Checking for inequality** — uses `!=`:
```python
requested_topping = 'mushrooms'    # Python

if requested_topping != 'anchovies':
    print("Hold the anchovies!")   # Output: Hold the anchovies!
```

**Numerical comparisons:**
```python
age = 19         # Python
age == 19        # True
age != 18        # True
age < 21         # True
age <= 21        # True
age > 21         # False
age >= 21        # False
```

**Multiple conditions:**
```python
age_0, age_1 = 22, 18                          # Python

age_0 >= 21 and age_1 >= 21    # False (both must pass)
age_0 >= 21 or  age_1 >= 21    # True  (one is enough)
```
You can wrap individual tests in parentheses for readability: `(age_0 >= 21) and (age_1 >= 21)`.

**Membership in a list — `in` and `not in`:**
```python
requested_toppings = ['mushrooms', 'onions', 'pineapple']    # Python

'mushrooms' in requested_toppings     # True
'pepperoni' in requested_toppings     # False

banned_users = ['andrew', 'carolina', 'david']
user = 'marie'
user not in banned_users              # True → allowed to post
```

**Boolean values** — variables that store `True` or `False` directly:
```python
game_active = True     # Python
can_edit    = False
```

---

### The Four Forms of if Statements

**1. Simple if** — one test, one action. If the test fails, nothing happens:
```python
age = 19                                   # Python
if age >= 18:
    print("You are old enough to vote!")
    print("Have you registered to vote yet?")
# Output:
# You are old enough to vote!
# Have you registered to vote yet?
```

**2. if-else** — one action when `True`, a different action when `False`:
```python
age = 17                                              # Python
if age >= 18:
    print("You are old enough to vote!")
else:
    print("Sorry, you are too young to vote.")
    print("Please register as soon as you turn 18!")
# Output:
# Sorry, you are too young to vote.
# Please register as soon as you turn 18!
```

**3. if-elif-else chain** — multiple conditions tested in order. Python runs **only the first passing block** and skips the rest:
```python
age = 12                                               # Python

if age < 4:
    price = 0
elif age < 18:
    price = 25
elif age < 65:
    price = 40
else:
    price = 20    # seniors 65+

print(f"Your admission cost is ${price}.")
# Output: Your admission cost is $25.
```
The `else` block is a catchall. If you want every condition to be explicit and intentional, use a final `elif` and omit `else` entirely.

**4. Series of independent if statements** — when multiple conditions can be `True` at the same time and you need to act on all of them:
```python
requested_toppings = ['mushrooms', 'extra cheese']    # Python

if 'mushrooms'    in requested_toppings: print("Adding mushrooms.")
if 'pepperoni'    in requested_toppings: print("Adding pepperoni.")
if 'extra cheese' in requested_toppings: print("Adding extra cheese.")
# Output:
# Adding mushrooms.
# Adding extra cheese.
```
If you used `elif` here, Python would stop after the first match and never add extra cheese. **Use `if-elif-else` when exactly one block should run. Use separate `if` statements when several blocks might need to run.**

---

### Using if Statements with Lists

**Handle special items while looping:**
```python
requested_toppings = ['mushrooms', 'green peppers', 'extra cheese']    # Python

for topping in requested_toppings:
    if topping == 'green peppers':
        print("Sorry, we are out of green peppers right now.")
    else:
        print(f"Adding {topping}.")
print("\nFinished making your pizza!")
# Output:
# Adding mushrooms.
# Sorry, we are out of green peppers right now.
# Adding extra cheese.
```

**Check that a list is not empty before looping** — an empty list evaluates to `False`:
```python
requested_toppings = []    # Python

if requested_toppings:
    for topping in requested_toppings:
        print(f"Adding {topping}.")
else:
    print("Are you sure you want a plain pizza?")
# Output: Are you sure you want a plain pizza?
```

**Cross-check two lists:**
```python
available = ['mushrooms', 'olives', 'green peppers', 'pepperoni', 'extra cheese']    # Python
requested = ['mushrooms', 'french fries', 'extra cheese']

for topping in requested:
    if topping in available:
        print(f"Adding {topping}.")
    else:
        print(f"Sorry, we don't have {topping}.")
# Output:
# Adding mushrooms.
# Sorry, we don't have french fries.
# Adding extra cheese.
```

---

### Styling if Statements (PEP 8)

Use a single space around all comparison operators:

| Write this | Not this |
|---|---|
| `if age < 4:` | `if age<4:` |
| `if x == y:` | `if x==y:` |
| `if count >= 10:` | `if count>=10:` |

Spacing does not affect how Python runs the code — it just makes it easier to read.

---

## Java vs Python

| Java | Python |
|---|---|
| `if (age >= 18) { }` | `if age >= 18:` — no parentheses, no braces, use indentation |
| `else if` | `elif` |
| `&&` | `and` |
| `\|\|` | `or` |
| `!condition` | `not condition` |
| `true` / `false` | `True` / `False` — capital first letter |
| No direct equivalent | `in` / `not in` for list membership |

---

## Practice Exercises

- [ ] **5-1 / 5-2** — Write 10 conditional tests (5 → True, 5 → False). Cover: equality, `.lower()`, inequality, numbers, `and`, `or`, `in`, `not in`.
- [ ] **5-3 / 5-4 / 5-5** — Alien Colors: write `if`, then `if-else`, then `if-elif-else` for green / yellow / red aliens awarding 5 / 10 / 15 points.
- [ ] **5-6** — Stages of Life: `if-elif-else` chain for baby / toddler / kid / teenager / adult / elder.
- [ ] **5-7** — Favourite Fruit: list of 3 fruits + 5 `if` statements checking membership.
- [ ] **5-8 / 5-9** — Hello Admin: loop through usernames, greet admin specially, handle empty list.
- [ ] **5-10** — Checking Usernames: case-insensitive uniqueness check across two lists.
- [ ] **5-11** — Ordinal Numbers: `if-elif-else` inside a loop to print 1st, 2nd, 3rd … 9th.

---

## Questions I Still Have

*Write your open questions here. Return later and answer them.*

---

## Related Notes

- [[VARIABLES &  SIMPLE DATA TYPES]]
- [[DICTIONARIES, TUPLES & SETS]]