---
title: WORKING WITH LIST
date: 2026-11-05
language: Python
phase: phase-1
tags:
links: []
status: learning
---
> **One-line summary:** A **`for` loop** runs the same indented block once per item in a list, however long the list is. **`range()`** generates numbers (it stops _before_ the end value), **`list()`** turns them into a list, and `min()`, `max()`, `sum()` summarise them; a **list comprehension** builds a list in one line. A **slice** (`lst[a:b]`) takes part of a list, and `lst[:]` makes a real **copy** (plain `b = a` does not). A **tuple** is an **immutable** list written with parentheses. **Indentation** is how Python knows what is inside a loop, and **PEP 8** (four spaces per level) is the style guide.

> _Note: the pages provided end partway through "Styling Your Code" (after the Indentation section). Add the remaining style rules and the chapter summary when you have those pages._

## Structure

### 1. Looping through an entire list

A `for` loop does the same thing with every item in a list. Without one, you'd retrieve each item individually, which is repetitive and means **rewriting the code whenever the list's length changes**. The loop lets Python handle that.

```python
magicians = ['alice', 'david', 'carolina']
for magician in magicians:
    print(magician)
```

Output:

```
alice
david
carolina
```

Read it as: _"For every magician in the list of magicians, print the magician's name."_

### 2. A closer look at looping

Step by step:

1. `for magician in magicians:` pulls the **first** value (`'alice'`) from the list and stores it in `magician`.
2. `print(magician)` prints the current value.
3. Python goes back to the `for` line, takes the **next** value (`'david'`), and repeats.
4. After the last value (`'carolina'`), no values remain, so Python moves on to the next line **after** the loop (or the program ends).

Key points:

- The steps repeat **once per item**, whether the list has 3 items or a million.
- The **loop variable** (`magician`) is temporary and can have any name, but a meaningful **singular name for one item** is best:

```python
for cat in cats:
for dog in dogs:
for item in list_of_items:
```

Singular/plural naming tells you whether code is working with a **single element** or the **whole list**.

### 3. Doing more work inside a loop

Every **indented** line after the `for` line is **inside the loop** and runs once per item. You can have as many lines as you like.

```python
magicians = ['alice', 'david', 'carolina']
for magician in magicians:
    print(magician.title() + ", that was a great trick!")
    print("I can't wait to see your next trick, " + magician.title() + ".\n")
```

Output (one grouped block per person; the `\n` adds a blank line):

```
Alice, that was a great trick!
I can't wait to see your next trick, Alice.

David, that was a great trick!
I can't wait to see your next trick, David.

Carolina, that was a great trick!
I can't wait to see your next trick, Carolina.
```

### 4. Doing something after a loop

Any line **not indented** after the loop runs **once**, after the loop finishes.

```python
for magician in magicians:
    print(magician.title() + ", that was a great trick!")
    print("I can't wait to see your next trick, " + magician.title() + ".\n")

print("Thank you, everyone. That was a great magic show!")   # runs once, at the end
```

This is the usual way to **summarise** after processing a whole data set (e.g. draw every character in a game, then show a "Play Now" button).

### 5. Avoiding indentation errors

Python uses **indentation** to decide which lines belong together. It forces neatly structured code, but you must watch for four common mistakes.

**a) Forgetting to indent the line after `for`** → `IndentationError`

```python
for magician in magicians:
print(magician)            # not indented
```

```
IndentationError: expected an indented block
```

Fix: indent the line(s) immediately after the `for` statement.

**b) Forgetting to indent additional lines** → **logical error** (no error message!)

```python
for magician in magicians:
    print(magician.title() + ", that was a great trick!")
print("I can't wait to see your next trick, " + magician.title() + ".\n")   # not indented
```

Because at least one line is indented, Python reports nothing. The unindented line runs **once, after the loop**, when `magician` holds the **last** value (`'carolina'`), so only Carolina gets that message. The syntax is valid but the **logic** is wrong. Check whether you just need to indent a line or a group of lines.

**c) Indenting unnecessarily** → `IndentationError: unexpected indent`

```python
message = "Hello Python world!"
    print(message)         # doesn't belong to the line above
```

Only indent when you have a specific reason; at this stage, only the actions you want repeated in a `for` loop.

**d) Indenting unnecessarily after the loop** → logical error

```python
for magician in magicians:
    print(...)
    print(...)
    print("Thank you everyone, that was a great magic show!")   # wrongly indented
```

The "thank you" now prints **once per magician** instead of once. Unindent code that should run only once.

### 6. Forgetting the colon

The colon at the end of the `for` line tells Python the next line starts the loop body.

```python
for magician in magicians          # missing colon
    print(magician)
```

→ **`SyntaxError`**. Easy to fix, but not always easy to _find_: we often see what we expect to see. Single-character errors can eat a lot of time.

### 7. Making numerical lists with `range()`

Lists are ideal for sets of numbers (scores, temperatures, coordinates, populations).

**`range()`** generates a series of numbers. It starts at the **first value** and **stops before the second value**:

```python
for value in range(1, 5):
    print(value)
```

```
1
2
3
4
```

The end value (5) is **never produced**. This is another off-by-one behaviour. To get 1 to 5, use `range(1, 6)`. If the output isn't what you expect, **adjust the end value by 1**.

### 8. Turning `range()` into a list

Wrap it in **`list()`**:

```python
numbers = list(range(1, 6))
print(numbers)              # [1, 2, 3, 4, 5]
```

**Skip numbers with a third argument (the step):**

```python
even_numbers = list(range(2, 11, 2))
print(even_numbers)         # [2, 4, 6, 8, 10]
```

It starts at 2, adds 2 repeatedly, and stops when it reaches or passes 11.

**Squares of 1 to 10, the long way:**

```python
squares = []                          # 1. empty list
for value in range(1, 11):            # 2. loop 1..10
    square = value**2                 # 3. raise to the 2nd power
    squares.append(square)            # 4. add to the list
print(squares)                        # 5. [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

**More concise (no temporary variable):**

```python
squares = []
for value in range(1, 11):
    squares.append(value**2)
```

Either is fine. A temporary variable can make code clearer, or just longer. Write code you understand first, then look for more efficient forms.

### 9. Simple statistics on a list of numbers

```python
>>> digits = [1, 2, 3, 4, 5, 6, 7, 8, 9, 0]
>>> min(digits)
0
>>> max(digits)
9
>>> sum(digits)
45
```

These work just as well on a list with a million numbers.

### 10. List comprehensions

A **list comprehension** combines the `for` loop and the creation/appending of elements into **one line**. The book includes them because you'll see them as soon as you read other people's code.

```python
squares = [value**2 for value in range(1, 11)]
print(squares)      # [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

**How to build one:**

1. Descriptive list name (`squares`).
2. Open a square bracket.
3. Write the **expression** for the values you want stored (`value**2`).
4. Write the `for` loop that feeds values into the expression (`for value in range(1, 11)`).
5. Close the bracket.

Note: **no colon** at the end of the `for` part inside a comprehension. Consider writing one when you find yourself writing three or four lines to build a list and it feels repetitive.

### 11. Slicing a list

A **slice** is a specific group of items from a list. You give the first and last index, and Python **stops one item before the second index** (same rule as `range()`).

```python
players = ['charles', 'martina', 'michael', 'florence', 'eli']
print(players[0:3])    # ['charles', 'martina', 'michael']
print(players[1:4])    # ['martina', 'michael', 'florence']
```

A slice is itself a list (brackets and quotes are kept in the output).

**Omit an index:**

|Slice|Meaning|Result|
|---|---|---|
|`players[:4]`|From the start up to (not including) index 4|`['charles', 'martina', 'michael', 'florence']`|
|`players[2:]`|From index 2 to the end|`['michael', 'florence', 'eli']`|
|`players[-3:]`|The last three items|`['michael', 'florence', 'eli']`|

`[2:]` and `[-3:]` work **regardless of the list's length**.

### 12. Looping through a slice

```python
print("Here are the first three players on my team:")
for player in players[:3]:
    print(player.title())
```

```
Here are the first three players on my team:
Charles
Martina
Michael
```

Uses: top three scores (sort in descending order, take the first three), processing data in chunks of a set size, showing a web app's data a page at a time.

### 13. Copying a list

Make a slice with **no indexes** (`[:]`): it starts at the first item and ends at the last, producing a **copy**.

```python
my_foods = ['pizza', 'falafel', 'carrot cake']
friend_foods = my_foods[:]

my_foods.append('cannoli')
friend_foods.append('ice cream')

print(my_foods)       # ['pizza', 'falafel', 'carrot cake', 'cannoli']
print(friend_foods)   # ['pizza', 'falafel', 'carrot cake', 'ice cream']
```

Two **separate** lists: each keeps only its own addition.

**The wrong way:**

```python
friend_foods = my_foods      # NOT a copy
```

This tells Python to connect `friend_foods` to the list **already contained** in `my_foods`, so **both variables point to the same list**. Adding to one appears in both:

```
['pizza', 'falafel', 'carrot cake', 'cannoli', 'ice cream']   # both lists identical
```

Rule: if you see unexpected behaviour when working with a "copy", make sure you copied with a **slice**.

### 14. Tuples

Lists are for items that **change**. A **tuple** is for values that **cannot** change. Python calls unchangeable values **immutable**, and an immutable list is a **tuple**.

**Defining:** parentheses instead of square brackets. Access items by index just like a list.

```python
dimensions = (200, 50)
print(dimensions[0])    # 200
print(dimensions[1])    # 50
```

**Trying to change an item** raises an error (which is the point: you _want_ Python to stop code that alters the rectangle's size):

```python
dimensions[0] = 250
```

```
TypeError: 'tuple' object does not support item assignment
```

**Looping** works the same as with a list:

```python
for dimension in dimensions:
    print(dimension)      # 200, then 50
```

**Writing over a tuple:** you can't modify a tuple, but you **can assign a new tuple to the variable**. Overwriting a variable is valid.

```python
dimensions = (200, 50)
dimensions = (400, 100)   # fine: a whole new tuple
```

Tuples are simpler than lists; use them when a set of values **shouldn't change during the program**.

### 15. Styling your code (partial)

- Write code that is **easy to read**. Code is **read more often than it is written** (debugging, adding features, sharing). Given a choice between easier to write and easier to read, choose **easier to read**.
- **PEP 8** = Python Enhancement Proposal 8, the style guide. (A PEP is how someone proposes a change to Python; PEP 8 is one of the oldest.)
- **Indentation:** PEP 8 recommends **four spaces per indentation level**.
- Don't mix **tabs and spaces**: the interpreter gets confused and the problems are hard to diagnose. Use the TAB key, but set your editor to **insert spaces** instead of tab characters. Most editors can convert tabs in a file to spaces.

## Mechanics / reference

### `range()` forms

|Call|Produces|
|---|---|
|`range(5)`|0, 1, 2, 3, 4 (single-argument form is not covered in these pages)|
|`range(1, 5)`|1, 2, 3, 4|
|`range(1, 6)`|1, 2, 3, 4, 5|
|`range(2, 11, 2)`|2, 4, 6, 8, 10 (third argument = step)|
|`list(range(1, 6))`|`[1, 2, 3, 4, 5]`|

The end value is **never included**.

### Slice cheat sheet (`players` has 5 items, indexes 0 to 4)

|Slice|Meaning|
|---|---|
|`[0:3]`|Indexes 0, 1, 2|
|`[1:4]`|Indexes 1, 2, 3|
|`[:4]`|Start to index 3|
|`[2:]`|Index 2 to end|
|`[-3:]`|Last three items|
|`[:]`|Whole list (a copy)|

### List vs tuple

||List|Tuple|
|---|---|---|
|Brackets|`[ ]`|`( )`|
|Can change items|Yes|**No** (`TypeError`)|
|Index access|Yes|Yes|
|Loop with `for`|Yes|Yes|
|Reassign the variable|Yes|Yes (new tuple)|
|Use when|Items change during the program|Values must stay fixed|

### Copy vs alias

|Code|Result|
|---|---|
|`b = a[:]`|Two **separate** lists|
|`b = a`|One list, **two names** pointing at it|

### Errors in this chapter

|Error|Cause|Fix|
|---|---|---|
|`IndentationError: expected an indented block`|Nothing indented after `for ...:`|Indent the loop body|
|`IndentationError: unexpected indent`|Indented a line that doesn't belong to a block|Remove the indent|
|`SyntaxError`|Missing colon after `for ...`|Add `:`|
|`TypeError: 'tuple' object does not support item assignment`|Tried to change a tuple item|Reassign the whole tuple, or use a list|
|_(no error)_ wrong output|Line indented/unindented wrongly (**logical error**)|Check which lines should repeat|

### Common patterns

```python
# 1. Loop over a list
for item in items:
    print(item)

# 2. Build a list of numbers
nums = list(range(1, 11))

# 3. Build a list with a transformation (long and short)
squares = []
for v in range(1, 11):
    squares.append(v**2)
squares = [v**2 for v in range(1, 11)]

# 4. Summary stats
min(nums); max(nums); sum(nums)

# 5. Loop over part of a list
for p in players[:3]:
    print(p.title())

# 6. Copy a list safely
copy = original[:]

# 7. Fixed values
size = (200, 50)
```

## Pitfalls

- **Forgetting the colon** after `for ...` → `SyntaxError`.
- **Forgetting to indent** the loop body → `IndentationError`.
- **Indenting only the first line** of a multi-line body: no error, but the other lines run once after the loop (logical error).
- **Indenting after-loop code** so it repeats for every item.
- **Indenting a line for no reason** → `unexpected indent`.
- **Mixing tabs and spaces.** Hard to diagnose; set the editor to insert spaces.
- **Expecting `range(1, 5)` to include 5.** It stops at 4; adjust the end value by 1.
- **Expecting a slice `[0:3]` to include index 3.** It stops one item before.
- **Using the loop variable after the loop** and forgetting it holds the **last** value.
- **Naming the loop variable vaguely** (`x`, `i`) when a singular name (`cat` for `cats`) is clearer.
- **Copying a list with `b = a`.** Both names refer to the same list; use `b = a[:]`.
- **Trying to modify a tuple item.** Tuples are immutable; reassign the whole tuple.
- **Thinking a tuple "can't change" at all.** The _variable_ can be reassigned to a new tuple; the existing tuple object can't be edited.
- **Forgetting `list()`** around `range()` when you want an actual list (`print(range(1,6))` doesn't give you `[1, 2, 3, 4, 5]`).
- **Putting a colon in a list comprehension's `for`.** It has none.
- **Overusing comprehensions** before you're comfortable with ordinary loops. Clarity first.

## Flashcards

- What does a `for` loop do? :: Repeats an indented block once for each item in a list #card
- Why use a loop instead of retrieving items individually? :: Less repetition, and the code doesn't need changing when the list's length changes #card
- What does `for magician in magicians:` do on each pass? :: Stores the next list item in `magician` #card
- What is the best way to name a loop variable? :: A meaningful singular name for one item (`cat` in `cats`) #card
- How does Python know which lines are inside a loop? :: They are indented after the `for` line #card
- What happens to unindented lines after a loop? :: They run once, after the loop finishes #card
- What does `"\n"` at the end of a printed string do in a loop? :: Adds a blank line after each pass #card
- Error for forgetting to indent the line after `for`? :: `IndentationError: expected an indented block` #card
- What is a logical error? :: Valid syntax that doesn't produce the intended result (e.g. forgetting to indent a second line in a loop) #card
- Error for an unnecessary indent? :: `IndentationError: unexpected indent` #card
- What happens if the after-loop line is accidentally indented? :: It repeats once per item #card
- Forgetting the colon on a `for` line gives which error? :: `SyntaxError` #card
- What does `range(1, 5)` produce? :: 1, 2, 3, 4 (stops before the end value) #card
- How do you get 1 through 5 with `range()`? :: `range(1, 6)` #card
- How do you turn `range()` into a list? :: Wrap it in `list()`: `list(range(1, 6))` #card
- What does `range(2, 11, 2)` give? :: 2, 4, 6, 8, 10 #card
- What does the third argument of `range()` do? :: Sets the step between numbers #card
- How do you square a number in Python? :: `value**2` #card
- Which functions find min, max and total of a number list? :: `min()`, `max()`, `sum()` #card
- What is a list comprehension? :: A one-line way to build a list by combining the `for` loop and the element expression #card
- Write the squares of 1 to 10 as a comprehension. :: `[value**2 for value in range(1, 11)]` #card
- Does a list comprehension's `for` have a colon? :: No #card
- What is a slice? :: A specific group of items from a list, e.g. `players[0:3]` #card
- Where does a slice stop? :: One item before the second index #card
- What does `players[:4]` return? :: Items from the start up to index 3 #card
- What does `players[2:]` return? :: Items from index 2 to the end #card
- What does `players[-3:]` return? :: The last three items #card
- How do you loop over only the first three items? :: `for p in players[:3]:` #card
- How do you copy a list? :: Slice with no indexes: `new = old[:]` #card
- What does `friend_foods = my_foods` do? :: Makes both names refer to the same list (not a copy) #card
- What is a tuple? :: An immutable list, written with parentheses #card
- What does immutable mean? :: The values cannot be changed #card
- What happens if you assign to a tuple item? :: `TypeError: 'tuple' object does not support item assignment` #card
- Can you loop over a tuple? :: Yes, with a `for` loop like a list #card
- Can you reassign a variable holding a tuple? :: Yes, assigning a whole new tuple is valid #card
- When should you use a tuple instead of a list? :: When the values should not change during the program #card
- What is PEP 8? :: The Python style guide (Python Enhancement Proposal 8) #card
- How many spaces per indentation level does PEP 8 recommend? :: Four #card
- Why avoid mixing tabs and spaces? :: The interpreter gets confused and the problems are hard to diagnose #card
- Why prioritise readable code? :: Code is read more often than it's written #card

## Open questions

- [ ] What does `range(5)` (single argument) do, and when is it used?
- [ ] Are slices and `[:]` copies _shallow_? What if the list holds other lists?
- [ ] How do I make a tuple with a single element? (Probably needs a trailing comma.)
- [ ] What's the rest of PEP 8 (line length, blank lines, naming)? Continue from the pages not yet covered.
- [ ] Can a list comprehension include a condition (`if`)?
- [ ] Why does `friend_foods = my_foods` alias rather than copy? How do variables refer to objects in memory?
- [ ] Time complexity: how does slicing/copying scale on a list of a million items? (DSA link)

## Key terms

|Term|Definition|
|---|---|
|Loop|Code that repeats an action for each item|
|`for` loop|Runs an indented block once per item in a sequence|
|Loop variable|Temporary variable holding the current item|
|Indentation|Leading whitespace that groups lines into a block|
|`IndentationError`|Wrong or missing indentation|
|Logical error|Valid code that does the wrong thing|
|`range()`|Generates a series of numbers, stopping before the end value|
|Step|Third argument of `range()`; the increment between numbers|
|`list()`|Converts a sequence such as `range()` into a list|
|`min()` / `max()` / `sum()`|Smallest, largest, total of a list of numbers|
|List comprehension|One-line list builder: `[expression for item in sequence]`|
|Slice|Part of a list: `lst[start:stop]`|
|Copy (`[:]`)|New independent list with the same items|
|Immutable|Cannot be changed after creation|
|Tuple|Immutable list in parentheses|
|PEP 8|Python's official style guide|
|Four-space indent|PEP 8 standard for each indentation level|

## Try it yourself (chapter exercises)

- [ ] 4-1 Pizzas: list of 3+ pizzas; loop to print each; then "I like pepperoni pizza."-style sentences; closing line outside the loop ("I really love pizza!")
- [ ] 4-2 Animals: 3+ animals with something in common; loop; sentence per animal; closing line outside the loop
- [ ] 4-3 Counting to Twenty: `for` loop printing 1 to 20 inclusive
- [ ] 4-4 One Million: list 1 to 1,000,000, loop to print (stop with Ctrl-C if slow)
- [ ] 4-5 Summing a Million: `min()`, `max()`, `sum()` on that list
- [ ] 4-6 Odd Numbers: odd numbers 1 to 20 using `range()`'s third argument
- [ ] 4-7 Threes: multiples of 3 from 3 to 30
- [ ] 4-8 Cubes: first 10 cubes (`**3`) with a loop
- [ ] 4-9 Cube Comprehension: same list with a list comprehension
- [ ] 4-10 Slices: print first three, middle three, last three items of a list (with messages)
- [ ] 4-11 My Pizzas, Your Pizzas: copy the pizza list to `friend_pizzas`; add a different pizza to each; prove they're separate with loops
- [ ] 4-12 More Loops: write two `for` loops to print each foods list (from `foods.py`)
- [ ] 4-13 Buffet: tuple of five foods; loop to print; try to modify one (Python must reject it); rewrite the tuple with two changed items and print again

## Related

→ Next: [[IF STATEMENT]]
