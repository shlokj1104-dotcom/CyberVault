---
title: IF STATEMENT
date: 2026-11-05
language: Python
phase: phase-1
tags:
links: []
status: learning
---
> **One-line summary:** A **conditional test** is an expression that evaluates to `True` or `False`; an **`if` statement** runs its indented block only when the test is `True`. Compare with `==`, `!=`, `<`, `<=`, `>`, `>=`, combine tests with `and` / `or`, and check list membership with `in` / `not in`. Use **`if-else`** for two outcomes, **`if-elif-else`** when **only one** block should run, and a **series of separate `if`s** when **every** true condition should run. An **empty list is `False`** in an `if`, so you can check a list isn't empty before looping.

## Structure

### 1. A simple example

Loop through cars and print each in title case, except `'bmw'`, which goes in uppercase:

```python
cars = ['audi', 'bmw', 'subaru', 'toyota']

for car in cars:
    if car == 'bmw':
        print(car.upper())
    else:
        print(car.title())
```

```
Audi
BMW
Subaru
Toyota
```

### 2. Conditional tests

At the heart of every `if` is a **conditional test**: an expression that evaluates to **`True`** or **`False`**.

- `True` → Python executes the code after the `if`.
- `False` → Python **ignores** that code.

### 3. Checking for equality (`==`)

```python
>>> car = 'bmw'          # single = is a STATEMENT: "set car to 'bmw'"
>>> car == 'bmw'         # double == is a QUESTION: "is car equal to 'bmw'?"
True
>>> car = 'audi'
>>> car == 'bmw'
False
```

|Symbol|Role|
|---|---|
|`=`|Assignment (sets a value)|
|`==`|Equality operator (asks a question, returns `True`/`False`)|

### 4. Ignoring case when checking equality

Equality is **case sensitive**:

```python
>>> car = 'Audi'
>>> car == 'audi'
False
```

If case doesn't matter, lowercase first. `lower()` doesn't change the stored value:

```python
>>> car = 'Audi'
>>> car.lower() == 'audi'
True
>>> car
'Audi'             # unchanged
```

Real use: a site checks that a new username isn't just a different capitalisation of an existing one. Convert the new name to lowercase and compare with lowercase versions of existing names, so `'John'` is rejected if `'john'` exists.

### 5. Checking for inequality (`!=`)

`!` means _not_. The test is `True` when the values **don't** match.

```python
requested_topping = 'mushrooms'
if requested_topping != 'anchovies':
    print("Hold the anchovies!")
```

Most tests check equality, but sometimes inequality is more efficient.

```python
answer = 17
if answer != 42:
    print("That is not the correct answer. Please try again!")
```

### 6. Numerical comparisons

```python
>>> age = 18
>>> age == 18
True
>>> age = 19
>>> age < 21
True
>>> age <= 21
True
>>> age > 21
False
>>> age >= 21
False
```

### 7. Checking multiple conditions

**`and`: both must be `True`.** If either (or both) fails, the whole expression is `False`.

```python
>>> age_0 = 22
>>> age_1 = 18
>>> age_0 >= 21 and age_1 >= 21
False
>>> age_1 = 22
>>> age_0 >= 21 and age_1 >= 21
True
```

Parentheses improve readability (not required): `(age_0 >= 21) and (age_1 >= 21)`.

**`or`: at least one must be `True`.** It fails only when **both** fail.

```python
>>> age_0 = 22
>>> age_1 = 18
>>> age_0 >= 21 or age_1 >= 21
True
>>> age_0 = 18
>>> age_0 >= 21 or age_1 >= 21
False
```

|Operator|Passes when|Fails when|
|---|---|---|
|`and`|Both tests pass|Either (or both) fails|
|`or`|Either (or both) passes|Both fail|

### 8. Checking whether a value is in a list (`in`)

```python
>>> requested_toppings = ['mushrooms', 'onions', 'pineapple']
>>> 'mushrooms' in requested_toppings
True
>>> 'pepperoni' in requested_toppings
False
```

Powerful: keep a list of essential values and check whether the value you're testing matches one of them.

### 9. Checking whether a value is not in a list (`not in`)

```python
banned_users = ['andrew', 'carolina', 'david']
user = 'marie'

if user not in banned_users:
    print(user.title() + ", you can post a response if you wish.")
```

Reads clearly: if `user` is **not in** `banned_users`, the test is `True`.

### 10. Boolean expressions

A **Boolean expression** is just another name for a conditional test. A **Boolean value** is `True` or `False`. They're handy for tracking program state:

```python
game_active = True
can_edit = False
```

### 11. Simple `if` statements

One test, one action.

```python
if conditional_test:
    do something
```

```python
age = 19
if age >= 18:
    print("You are old enough to vote!")
    print("Have you registered to vote yet?")
```

- Indentation works as in `for` loops: **all indented lines run if the test passes**, and the whole block is skipped if it doesn't.
- If `age` were under 18, this program would produce **no output**.

### 12. `if-else` statements

Take one action when the test passes and **a different action in all other cases**.

```python
age = 17
if age >= 18:
    print("You are old enough to vote!")
    print("Have you registered to vote yet?")
else:
    print("Sorry, you are too young to vote.")
    print("Please register to vote as soon as you turn 18!")
```

In a simple `if-else`, **exactly one** of the two blocks always runs. Good when there are only two possible situations.

### 13. The `if-elif-else` chain

For **more than two** situations. Python runs each test **in order** until one passes, executes that block, and **skips the rest**. **Only one block runs.**

Amusement park: under 4 free, 4 to under 18 is $5, 18 or older is $10.

```python
age = 12

if age < 4:
    print("Your admission cost is $0.")
elif age < 18:
    print("Your admission cost is $5.")
else:
    print("Your admission cost is $10.")
```

Output: `Your admission cost is $5.`

- `elif` is really another `if` that runs **only if the previous test failed**. By the time Python reaches it, it already knows `age >= 4`.
- If every `if`/`elif` fails, the `else` block runs.

**Better structure: set a variable in the chain, print once after it:**

```python
if age < 4:
    price = 0
elif age < 18:
    price = 5
else:
    price = 10

print("Your admission cost is $" + str(price) + ".")
```

Same output, but more efficient and easier to modify (change one `print`, not three). Note `str(price)`, from Chapter 2.

### 14. Using multiple `elif` blocks

Use as many as you need. Seniors (65+) pay $5:

```python
if age < 4:
    price = 0
elif age < 18:
    price = 5
elif age < 65:
    price = 10
else:
    price = 5          # only 65+ reaches here, so this must be 5
```

### 15. Omitting the `else` block

`else` isn't required. Sometimes a final `elif` is clearer:

```python
elif age >= 65:
    price = 5
```

**Why prefer it:** `else` is a **catchall**: it matches anything the earlier tests didn't, which can include **invalid or even malicious data**. If you have a specific final condition, use a final `elif` and omit `else`, so each block runs only under the correct conditions.

### 16. Testing multiple conditions (independent `if`s)

`if-elif-else` is only right when **just one** test needs to pass. When more than one condition could be `True` and you want to act on **each**, use a **series of simple `if` statements** (no `elif`/`else`).

```python
requested_toppings = ['mushrooms', 'extra cheese']

if 'mushrooms' in requested_toppings:
    print("Adding mushrooms.")
if 'pepperoni' in requested_toppings:
    print("Adding pepperoni.")
if 'extra cheese' in requested_toppings:
    print("Adding extra cheese.")

print("\nFinished making your pizza!")
```

```
Adding mushrooms.
Adding extra cheese.

Finished making your pizza!
```

Each `if` runs **every time**. With `elif` instead, Python would stop after the first pass and only mushrooms would be added.

> **Rule:** only one block should run → `if-elif-else`. More than one block may need to run → independent `if`s.

### 17. Using `if` with lists: special items

Handle a special value differently inside a `for` loop:

```python
requested_toppings = ['mushrooms', 'green peppers', 'extra cheese']

for requested_topping in requested_toppings:
    if requested_topping == 'green peppers':
        print("Sorry, we are out of green peppers right now.")
    else:
        print("Adding " + requested_topping + ".")

print("\nFinished making your pizza!")
```

```
Adding mushrooms.
Sorry, we are out of green peppers right now.
Adding extra cheese.

Finished making your pizza!
```

### 18. Checking that a list is not empty

Until now we've assumed lists have at least one item. Once users supply the data, that's no longer safe. **A list name used in an `if` is `True` if it has at least one item and `False` if empty.**

```python
requested_toppings = []

if requested_toppings:
    for requested_topping in requested_toppings:
        print("Adding " + requested_topping + ".")
    print("\nFinished making your pizza!")
else:
    print("Are you sure you want a plain pizza?")
```

Output (list is empty): `Are you sure you want a plain pizza?`

### 19. Using multiple lists

Check requests against a list of what's **available** before acting:

```python
available_toppings = ['mushrooms', 'olives', 'green peppers',
                      'pepperoni', 'pineapple', 'extra cheese']
requested_toppings = ['mushrooms', 'french fries', 'extra cheese']

for requested_topping in requested_toppings:
    if requested_topping in available_toppings:
        print("Adding " + requested_topping + ".")
    else:
        print("Sorry, we don't have " + requested_topping + ".")

print("\nFinished making your pizza!")
```

```
Adding mushrooms.
Sorry, we don't have french fries.
Adding extra cheese.

Finished making your pizza!
```

If the pizzeria's selection is stable, `available_toppings` could be a **tuple**.

### 20. Styling `if` statements

PEP 8's only recommendation for conditional tests: use **a single space around comparison operators** (`==`, `>=`, `<=`, etc.).

```python
if age < 4:      # better
if age<4:        # worse
```

It doesn't affect how Python reads the code; it just makes it easier for people to read.

## Mechanics / reference

### Comparison operators

|Operator|Meaning|Example (`age = 19`)|Result|
|---|---|---|---|
|`==`|Equal|`age == 18`|`False`|
|`!=`|Not equal|`age != 18`|`True`|
|`<`|Less than|`age < 21`|`True`|
|`<=`|Less than or equal|`age <= 21`|`True`|
|`>`|Greater than|`age > 21`|`False`|
|`>=`|Greater than or equal|`age >= 21`|`False`|

### Logical and membership operators

|Operator|Meaning|
|---|---|
|`and`|Both tests must be `True`|
|`or`|At least one test must be `True`|
|`in`|Value is in the list|
|`not in`|Value is not in the list|

### Which structure to use?

|Situation|Use|
|---|---|
|One test, one action|`if`|
|Two outcomes; one must always run|`if-else`|
|Three or more outcomes; only one should run|`if-elif-else` (or `if-elif` with no `else`)|
|Several conditions may be true and each needs handling|Separate `if` statements|
|Specific final condition, want to avoid a catchall|Final `elif`, omit `else`|
|Need to know a list has items first|`if my_list:`|

### `if-elif-else` vs independent `ifs`

||`if-elif-else`|Independent `if`s|
|---|---|---|
|Tests run|Until the **first** pass, then stop|**All** of them, every time|
|Blocks that can run|At most one|Zero, one, or many|
|Good for|Mutually exclusive cases (age bands)|Independent conditions (toppings)|

### Truthiness of lists

|Expression|Value|
|---|---|
|`if []:`|`False` (empty list)|
|`if ['a']:`|`True` (at least one item)|

### Common patterns

```python
# 1. Case-insensitive comparison
if name.lower() == 'admin':

# 2. Special case inside a loop
for item in items:
    if item == special:
        ...
    else:
        ...

# 3. Membership check against allowed values
if choice in allowed:

# 4. Banned / not-present check
if user not in banned_users:

# 5. Set a value in a chain, then use it once
if age < 4:
    price = 0
elif age < 18:
    price = 5
else:
    price = 10
print("Cost: $" + str(price))

# 6. Guard against an empty list
if items:
    for item in items:
        ...
else:
    print("Nothing to process.")
```

## Pitfalls

- **Using `=` instead of `==`** in a test. Single `=` assigns, double `==` compares.
- **Forgetting the colon** after `if`, `elif` or `else`.
- **Forgetting to indent** the block under an `if` (same indentation rules as `for`).
- **Case-sensitive comparisons.** `'Audi' == 'audi'` is `False`; use `.lower()` when case shouldn't matter.
- **Thinking `lower()` changes the variable.** It doesn't; the stored value stays the same.
- **Using `elif` when every condition must be checked.** Only the first passing test runs; use independent `if`s.
- **Using separate `if`s when exactly one outcome is intended.** More than one block may run.
- **Relying on `else` as a catchall** when you have a specific final condition. It can match invalid or malicious data; use a final `elif`.
- **Forgetting to adjust the `else` value** when you add an `elif` (e.g. the senior price in the `else` block).
- **Writing `age_0 >= 21 and >= 21`.** Each side of `and`/`or` needs a full test: `age_0 >= 21 and age_1 >= 21`.
- **Assuming a list is never empty.** Check with `if my_list:` before looping when users supply the data.
- **Forgetting `str()`** when putting a number in a message (`"$" + str(price)`).
- **No spaces around comparison operators** (`age<4`). Works, but against PEP 8.
- **Confusing `in` with a loop.** `'x' in lst` is a single `True`/`False` test.
- **Expecting output when no branch runs.** A simple `if` with a failing test (and no `else`) prints nothing.

## Flashcards

- What is a conditional test? :: An expression that evaluates to `True` or `False` #card
- What does Python do if the test in an `if` is `True`? :: Executes the indented code after it #card
- What does it do if the test is `False`? :: Ignores the indented block #card
- What's the difference between `=` and `==`? :: `=` assigns a value; `==` asks whether two values are equal #card
- Is string equality case sensitive? :: Yes; `'Audi' == 'audi'` is `False` #card
- How do you compare ignoring case? :: Convert with `lower()` first: `car.lower() == 'audi'` #card
- Does `car.lower() == 'audi'` change `car`? :: No, the stored value is unchanged #card
- What does `!=` test? :: That two values are not equal #card
- Name the numerical comparison operators. :: `<`, `<=`, `>`, `>=` (plus `==` and `!=`) #card
- What does `and` require? :: Both conditions must be `True` #card
- What does `or` require? :: At least one condition must be `True` #card
- When does an `or` expression fail? :: Only when both tests fail #card
- How do you check a value is in a list? :: `value in my_list` #card
- How do you check a value is not in a list? :: `value not in my_list` #card
- What is a Boolean expression? :: Another name for a conditional test #card
- What is a Boolean value? :: `True` or `False` #card
- What does a simple `if` do when the test fails and there's no `else`? :: Nothing; there's no output #card
- When use `if-else`? :: When exactly one of two actions should always run #card
- How does `if-elif-else` run? :: Tests in order until one passes, runs that block, skips the rest #card
- How many blocks of an `if-elif-else` chain can run? :: Only one #card
- What is `elif`? :: Another `if` test that runs only if the previous tests failed #card
- Can a chain have multiple `elif` blocks? :: Yes, as many as you like #card
- Is the final `else` required? :: No, it can be omitted #card
- Why might a final `elif` be better than `else`? :: `else` is a catchall that can match invalid or malicious data; `elif` runs only under a specific condition #card
- Why set `price` inside the chain and print once after? :: More efficient and easier to modify (one `print` instead of several) #card
- When use a series of independent `if` statements? :: When more than one condition could be true and you want to act on each #card
- What goes wrong using `elif` for independent toppings? :: Only the first passing test runs, so other toppings are missed #card
- What does an empty list evaluate to in an `if`? :: `False` #card
- What does a list with at least one item evaluate to? :: `True` #card
- Why check a list isn't empty before looping? :: Users may supply the data, so you can't assume it has items #card
- How do you validate requests against available items? :: Use `in` against a list of available values #card
- What does PEP 8 say about conditional tests? :: Use a single space around comparison operators (`age < 4`) #card

## Open questions

- [ ] Does Python short-circuit `and` / `or` (stop evaluating early)? How does that matter in practice?
- [ ] What's the precedence of `not`, `and`, `or` when mixed without parentheses?
- [ ] Which other values count as `False` besides `False` and an empty list (e.g. `0`, `''`, `None`)?
- [ ] Can I chain comparisons like `4 <= age < 18`?
- [ ] How does `in` perform on a list vs a set or dictionary? (DSA link: O(n) vs O(1) average.)
- [ ] How do I read user input to feed these tests? (Later chapter.)

## Key terms

|Term|Definition|
|---|---|
|Conditional test|Expression that evaluates to `True` or `False`|
|`if` statement|Runs its block only if the test is `True`|
|Equality operator (`==`)|Returns `True` if both sides match|
|Inequality operator (`!=`)|Returns `True` if the sides don't match|
|Case sensitive|Capitalisation matters in the comparison|
|`and`|Combines tests; all must pass|
|`or`|Combines tests; at least one must pass|
|`in` / `not in`|Membership tests on a list|
|Boolean expression|Another name for a conditional test|
|Boolean value|`True` or `False`|
|`if-else`|Two-way choice; one block always runs|
|`elif`|"Else if": a further test in a chain|
|`if-elif-else` chain|Series of tests where only the first passing block runs|
|Catchall|`else` block that matches anything not matched earlier|
|Independent `if` statements|Separate tests that all run every time|
|PEP 8 spacing|One space around comparison operators|

## Try it yourself (chapter exercises)

- [ ] 5-1 Conditional Tests: 10+ tests (5 `True`, 5 `False`), printing a prediction and the result for each
- [ ] 5-2 More Conditional Tests: at least one `True` and one `False` each for string equality/inequality, `lower()`, numeric comparisons, `and`, `or`, in a list, not in a list
- [ ] 5-3 Alien Colors #1: `alien_color` of green/yellow/red; `if` green → "earned 5 points"; one version that passes, one that fails
- [ ] 5-4 Alien Colors #2: `if-else` (green → 5 points, otherwise 10); one version for each block
- [ ] 5-5 Alien Colors #3: `if-elif-else` (green 5, yellow 10, red 15); three versions
- [ ] 5-6 Stages of Life: baby (<2), toddler (2 to <4), kid (4 to <13), teenager (13 to <20), adult (20 to <65), elder (65+)
- [ ] 5-7 Favorite Fruit: `favorite_fruits` list; five independent `if`s checking for fruits ("You really like bananas!")
- [ ] 5-8 Hello Admin: 5+ usernames including `'admin'`; special greeting for admin, generic for others
- [ ] 5-9 No Users: add a list-not-empty check ("We need to find some users!"); test by emptying the list
- [ ] 5-10 Checking Usernames: `current_users` vs `new_users` (some overlap); report taken/available, **case-insensitive**
- [ ] 5-11 Ordinal Numbers: list 1 to 9; `if-elif-else` to print 1st, 2nd, 3rd, 4th ... 9th, one per line
- [ ] 5-12 Styling `if` statements: check spacing around comparison operators in this chapter's programs
- [ ] 5-13 Your Ideas: record ideas for games, data sets and web apps you might build

## Related

→ Next: [[DICTIONARIES]]