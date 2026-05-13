---
title: ITERATIVE STATEMENTS
date: 2026-05-12
phase: phase-1
tags:
  - concept
  - PYTHON
links: []
status: learning
---
# ITERATIVE STATEMENTS

## One-Line Summary
Iteration means repeating a block of code — `while` keeps running as long as a condition is `True`, and `for` runs once for each item in a collection — combined with `input()` they let you build fully interactive programs.

---

## Core Concept
Think of an ATM. It keeps asking "what do you want to do?" after every action — until you press *Exit*. Inside, it's a `while` loop: show a prompt, read input, react, repeat. The `input()` function is the "read input" step. The `for` loop is like a conveyor belt — it processes each item exactly once and then stops on its own.

Unlike Java or C, Python's `for` loop never requires you to manage an index manually unless you want to. And there is no `++` operator — Python uses `+= 1` instead.

---

## How It Works

### Updating Variables

Before you can update a variable inside a loop, you must **initialise** it first. Python evaluates the right side before assigning, so using an uninitialised variable causes a `NameError`:

```python
>>> x = x + 1    # Python
NameError: name 'x' is not defined

>>> x = 0        # initialise first
>>> x = x + 1    # works fine now
```

Adding 1 to a variable is called an **increment**. Subtracting 1 is a **decrement**. Python shorthand:

```python
x += 1    # same as x = x + 1    # Python
x -= 1    # same as x = x - 1
x *= 2    # same as x = x * 2
```

---

### The while Loop

Runs its body repeatedly as long as the condition is `True`. Python checks the condition **before** every iteration.

**Flow of execution:**
1. Evaluate the condition → `True` or `False`
2. If `False`, exit the loop and continue after it
3. If `True`, run the body, then go back to step 1

```python
n = 5              # Python
while n > 0:
    print(n)
    n = n - 1
print('Blastoff!')
# Output: 5 4 3 2 1 Blastoff!
```

The variable `n` is the **iteration variable** — it changes each pass and eventually makes the condition `False`. If there is no iteration variable, the loop runs forever: an **infinite loop**.

A simpler counting example:
```python
current_number = 1              # Python
while current_number <= 5:
    print(current_number)
    current_number += 1
# Output: 1 2 3 4 5
```

---

### Getting User Input with input()

`input()` pauses the program, displays a prompt, and returns whatever the user types as a **string**:

```python
message = input("Tell me something: ")    # Python
print(message)
```

Always add a trailing space to your prompt so the cursor doesn't butt up against the colon.

**Multi-line prompt** — build it in a variable first:
```python
prompt = "If you tell us who you are, we can personalise your experience."    # Python
prompt += "\nWhat is your first name? "
name = input(prompt)
print(f"\nHello, {name}!")
```

**Numerical input** — `input()` always returns a string. Wrap with `int()` before doing any maths or comparisons:

```python
age = input("How old are you? ")    # Python
age = int(age)                      # convert string → integer
age >= 18                           # True or False — works now
```

In one line:
```python
height = int(input("How tall are you, in inches? "))    # Python
```

---

### The Modulo Operator %

`%` divides two numbers and returns the **remainder** — not the quotient:

```python
4 % 3    # 1    # Python
6 % 3    # 0
7 % 3    # 1
```

Classic use — even/odd check. If `number % 2 == 0` there is no remainder, so it divides evenly → even:

```python
number = int(input("Enter a number: "))    # Python
if number % 2 == 0:
    print(f"The number {number} is even.")
else:
    print(f"The number {number} is odd.")
```

---

### Letting the User Choose When to Quit

Define a **quit value** and loop until the user enters it. Initialise the variable to an empty string so the `while` condition has something to compare against on the very first check:

```python
prompt = "\nTell me something (enter 'quit' to exit): "    # Python

message = ""
while message != 'quit':
    message = input(prompt)
    if message != 'quit':
        print(message)
```

---

### Using a Flag

For programs where many different events could stop the loop, use a **flag** — a Boolean variable that signals whether the program should keep running. The `while` condition stays simple; all exit logic lives elsewhere:

```python
active = True                       # Python
while active:
    message = input(prompt)
    if message == 'quit':
        active = False
    else:
        print(message)
```

Any event anywhere in the program can set `active = False` to stop the loop — useful in games or menus with many possible exits.

---

### Exiting with break

`break` exits the loop immediately, regardless of the condition. Useful when the exit condition is discovered **inside** the loop body:

```python
while True:                             # Python
    city = input("\nCity you've visited (or 'quit'): ")
    if city == 'quit':
        break
    else:
        print(f"I'd love to go to {city.title()}!")
```

`break` works in both `while` and `for` loops.

---

### Skipping an Iteration with continue

`continue` skips the rest of the current loop body and jumps straight back to the condition check — the loop does not exit:

```python
current_number = 0                  # Python
while current_number < 10:
    current_number += 1
    if current_number % 2 == 0:
        continue                    # skip even numbers
    print(current_number)
# Output: 1 3 5 7 9
```

With a `while True` loop:
```python
while True:                         # Python
    line = input('> ')
    if line[0] == '#':
        continue                    # skip comment lines
    if line == 'done':
        break
    print(line)
```

---

### Avoiding Infinite Loops

Every `while` loop must have a way to eventually make its condition `False` or hit a `break`. Forgetting the update is the most common mistake:

```python
# Infinite loop — x never changes    # Python
x = 1
while x <= 5:
    print(x)
    # forgot x += 1
```

If your program gets stuck, press **CTRL-C** to stop it. To prevent it: after writing any `while` loop, ask yourself — "is there at least one path that makes this condition `False` or hits `break`?"

---

### The for Loop (Definite Loop)

Use `for` when you know the collection to loop through. It runs the body **exactly once per item** and stops automatically — no condition to manage:

```python
friends = ['Joseph', 'Glenn', 'Sally']    # Python
for friend in friends:
    print('Happy New Year:', friend)
print('Done!')
# Output:
# Happy New Year: Joseph
# Happy New Year: Glenn
# Happy New Year: Sally
# Done!
```

`friend` is the **iteration variable** — it takes the value of each item one at a time. `for` and `in` are reserved keywords.

`while` = **indefinite** loop (runs until a condition is False)
`for` = **definite** loop (runs for each item in a known set)

---

### Loop Patterns

These three patterns cover the vast majority of real `for` loop use cases. Always initialise the variable **before** the loop starts.

**Counting loop** — count how many items:
```python
count = 0                             # Python
for item in [3, 41, 12, 9, 74, 15]:
    count = count + 1
print('Count:', count)    # Output: Count: 6
# Built-in shortcut: len([3, 41, 12, 9, 74, 15]) → 6
```

**Summing loop (accumulator)** — add up all values:
```python
total = 0                             # Python
for item in [3, 41, 12, 9, 74, 15]:
    total = total + item
print('Total:', total)    # Output: Total: 154
# Built-in shortcut: sum([3, 41, 12, 9, 74, 15]) → 154
```

**Maximum / minimum loop** — track the largest or smallest seen so far. Use `None` to mark "not yet set":
```python
largest = None                              # Python
for item in [3, 41, 12, 9, 74, 15]:
    if largest is None or item > largest:
        largest = item
print('Largest:', largest)    # Output: Largest: 74
# Built-in shortcut: max([3, 41, 12, 9, 74, 15]) → 74

smallest = None
for item in [3, 41, 12, 9, 74, 15]:
    matching = None or item < smallest:
        smallest = item
print('Smallest:', smallest)    # Output: Smallest: 3
# Built-in shortcut: min([3, 41, 12, 9, 74, 15]) → 3
```

---

### Using while Loops with Lists

**Don't modify a list inside a `for` loop** — Python loses track of items. Use `while` instead.

**Moving items from one list to another** — `pop()` removes and returns the last item:
```python
unconfirmed = ['alice', 'brian', 'candace']    # Python
confirmed = []

while unconfirmed:
    user = unconfirmed.pop()
    print(f"Verifying: {user.title()}")
    confirmed.append(user)

print("\nConfirmed users:")
for user in confirmed:
    print(user.title())
# Verifying: Candace → Brian → Alice
```

`while unconfirmed:` evaluates to `True` while the list has items, `False` when it's empty.

**Removing all instances of a value** — `remove()` deletes only the first match, so loop until none remain:
```python
pets = ['dog', 'cat', 'dog', 'goldfish', 'cat', 'rabbit', 'cat']    # Python
while 'cat' in pets:
    pets.remove('cat')
print(pets)    # ['dog', 'dog', 'goldfish', 'rabbit']
```

**Filling a dictionary with user input:**
```python
responses = {}                                                              # Python
polling_active = True

while polling_active:
    name = input("\nWhat is your name? ")
    response = input("Which mountain would you like to climb someday? ")
    responses[name] = response
    repeat = input("Would you like another person to respond? (yes/no) ")
    if repeat == 'no':
        polling_active = False

print("\n--- Poll Results ---")
for name, response in responses.items():
    print(f"{name} would like to climb {response}.")
```

---

### Debugging with Bisection

When a loop has a bug in 100 lines, don't check line by line. **Bisect**: add a `print()` check in the middle. If that value is correct, the bug is in the second half. If not, it's in the first half. Repeat. Each check halves the search space — most bugs found in 6–7 steps instead of 100.

---

## Java vs Python

| Java | Python |
|---|---|
| `Scanner sc = new Scanner(System.in)` then `sc.nextLine()` | `input("prompt ")` — one line |
| Always typed: `int age = sc.nextInt()` | Always a string: `age = int(input(...))` |
| `i++` / `i--` | `i += 1` / `i -= 1` — no `++` in Python |
| `for (String s : list)` | `for s in list:` |
| `for (int i = 0; i < n; i++)` | `for i in range(n):` |
| `while (condition) { }` | `while condition:` |
| `break` / `continue` | same — `break` / `continue` |
| `null` | `None` — used to mark "not yet set" |
| No direct equivalent | flag variable: `active = True` |
| `x % 2 == 0` → even | same |

---

## Quick Reference

```python
# User input                               # Python
name = input("Enter your name: ")
age  = int(input("Enter your age: "))

# Modulo
x % y      # remainder of x / y
x % 2      # 0 if even, 1 if odd

# while — basic
while condition:
    body
    update_variable   # don't forget!

# while — quit value
message = ""
while message != 'quit':
    message = input(prompt)

# while — flag
active = True
while active:
    ...
    if exit_event:
        active = False

# while True — break
while True:
    ...
    if exit_event:
        break

# continue — skip rest of current iteration
while condition:
    if skip_condition:
        continue
    body

# for loop
for item in collection:
    body

# range-based for
for i in range(5):       # 0 1 2 3 4
    print(i)

for i in range(2, 8):    # 2 3 4 5 6 7
    print(i)

# Remove all instances of a value from a list
while value in my_list:
    my_list.remove(value)
```

---

## Practice Exercises

- [ ] **Counting** — Write a `while` loop counting from 1 to 5. Rewrite with `for` and `range()`.
- [ ] **User quit** — Loop asking for input until the user types `'quit'`. Don't print `'quit'` itself.
- [ ] **Flag version** — Rewrite the above using an `active` flag instead of checking `!= 'quit'`.
- [ ] **break version** — Rewrite using `while True` and `break`.
- [ ] **Modulo** — Ask for a number; print whether it is even or odd. Then check if it is a multiple of 10.
- [ ] **Pizza Toppings** — Loop asking for toppings until `'quit'`; print each as it is entered.
- [ ] **Movie Tickets** — Loop asking ages; print price (under 3 → free, 3–12 → $10, over 12 → $15).
- [ ] **Three Exits** — Solve Movie Tickets three ways: conditional test, flag, `break`.
- [ ] **Accumulator** — Sum a list of numbers with a loop, then verify with `sum()`.
- [ ] **Max/Min** — Find max and min with a loop, then verify with `max()` / `min()`.
- [ ] **Move list** — Use `while` + `pop()` to move items from one list to another.
- [ ] **Remove all** — Use `while 'cat' in pets` to remove all instances of `'cat'`.
- [ ] **Poll** — Build a `while` loop that collects name + answer pairs into a dictionary, then prints the results.

---

## Questions I Still Have

*Write your open questions here. Return later and answer them.*

---

## Related Notes

- [[CONDITIONAL STATEMENT]]
- 