---
title: INTRODUCTION TO LIST
date: 2026-11-05
language: Python
phase: phase-1
tags:
links: []
status: learning
---
> **One-line summary:** A **list** is an ordered collection of items written in square brackets (`['trek', 'redline']`). You reach items by **index** (starting at **0**, with **-1** for the last), change them with assignment, grow the list with **`append()`** and **`insert()`**, shrink it with **`del`**, **`pop()`** and **`remove()`**, and organise it with **`sort()`** / **`reverse()`** (permanent) or **`sorted()`** (temporary). **`len()`** gives the length. The classic error is **`IndexError`**, caused by off-by-one thinking or an empty list.

## Structure

### 1. What is a list?

A **list** is a collection of items in a **particular order**. It can hold anything (letters, digits, names), and the items need not be related. Lists let you store a set of information in one place, whether it has a few items or millions.

- Name lists in the **plural** (`letters`, `digits`, `names`) since they usually hold more than one element.
- **Square brackets `[]`** mark a list; **commas** separate the elements.

```python
bicycles = ['trek', 'cannondale', 'redline', 'specialized']
print(bicycles)
```

Printing a list shows Python's own representation, **brackets and quotes included**:

```
['trek', 'cannondale', 'redline', 'specialized']
```

That's not what you want users to see, so you access individual items instead.

### 2. Accessing elements

Lists are **ordered**, so you ask for an item by its **position (index)**: list name, then the index in square brackets.

```python
print(bicycles[0])            # trek
```

- A single element comes back **without brackets or quotes**: clean output.
- Chapter 2 string methods work on any element:

```python
print(bicycles[0].title())    # Trek
```

### 3. Index positions start at 0

The first item is at index **0**, not 1 (true of most programming languages). So:

- Second item → index `1`; fourth item → index `3`.
- Rule of thumb: **position − 1 = index**.
- Unexpected results? Suspect an **off-by-one error**.

```python
print(bicycles[1])    # cannondale
print(bicycles[3])    # specialized
```

**Negative indexes** count from the end:

|Index|Returns|
|---|---|
|`-1`|Last item|
|`-2`|Second from the end|
|`-3`|Third from the end|

```python
print(bicycles[-1])   # specialized
```

`-1` is useful because you often want the last item **without knowing how long the list is**.

### 4. Using individual values

Items from a list behave like any other variable, e.g. in concatenation:

```python
message = "My first bicycle was a " + bicycles[0].title() + "."
print(message)        # My first bicycle was a Trek.
```

### 5. Modifying elements

Use the list name and index, then assign the new value. Works at **any** index.

```python
motorcycles = ['honda', 'yamaha', 'suzuki']
motorcycles[0] = 'ducati'
print(motorcycles)    # ['ducati', 'yamaha', 'suzuki']
```

Only that item changes; the rest stay the same.

### 6. Adding elements

Most lists are **dynamic**: you build them, then add and remove items as the program runs (e.g. aliens in a game, new registered users).

**`append()`: add to the end**

```python
motorcycles = ['honda', 'yamaha', 'suzuki']
motorcycles.append('ducati')
# ['honda', 'yamaha', 'suzuki', 'ducati']
```

Other elements are unaffected. Because you often don't know the data until the program runs, a common pattern is to **start with an empty list** and append values as they arrive:

```python
motorcycles = []
motorcycles.append('honda')
motorcycles.append('yamaha')
motorcycles.append('suzuki')
# ['honda', 'yamaha', 'suzuki']
```

**`insert(index, value)`: add at any position**

```python
motorcycles = ['honda', 'yamaha', 'suzuki']
motorcycles.insert(0, 'ducati')
# ['ducati', 'honda', 'yamaha', 'suzuki']
```

`insert()` opens a space at that position, stores the value there, and **shifts every later item one position to the right**.

### 7. Removing elements

You can remove an item **by position** or **by value**.

**a) `del`: by position, value is gone**

```python
motorcycles = ['honda', 'yamaha', 'suzuki']
del motorcycles[0]      # ['yamaha', 'suzuki']
```

Works at any index (`del motorcycles[1]` removes the second item). You can't access the removed value afterwards.

**b) `pop()`: by position, but you get the value back**

`pop()` removes the **last** item and returns it so you can use it. The name comes from a **stack**: you pop one item off the top, and the top of the stack is the end of the list.

```python
motorcycles = ['honda', 'yamaha', 'suzuki']
popped_motorcycle = motorcycles.pop()
print(motorcycles)          # ['honda', 'yamaha']
print(popped_motorcycle)    # suzuki
```

Use case: "The last motorcycle I owned was a Suzuki."

```python
last_owned = motorcycles.pop()
print("The last motorcycle I owned was a " + last_owned.title() + ".")
```

**Pop from any position** by giving an index:

```python
first_owned = motorcycles.pop(0)
print('The first motorcycle I owned was a ' + first_owned.title() + '.')
```

Each `pop()` removes the item from the list.

**`del` or `pop()`?** If you want to delete an item and **not use it**, use `del`. If you want to **use it as you remove it**, use `pop()`.

**c) `remove()`: by value**

Use it when you know the value but not the position.

```python
motorcycles = ['honda', 'yamaha', 'suzuki', 'ducati']
motorcycles.remove('ducati')
# ['honda', 'yamaha', 'suzuki']
```

You can store the value in a variable first, which also lets you keep using it after removal:

```python
too_expensive = 'ducati'
motorcycles.remove(too_expensive)
print("\nA " + too_expensive.title() + " is too expensive for me.")
```

> **Note:** `remove()` deletes **only the first occurrence** of the value. If it might appear more than once, you need a loop to check all occurrences are removed (covered in Chapter 7).

### 8. Organising a list

**`sort()`: permanent alphabetical sort**

```python
cars = ['bmw', 'audi', 'toyota', 'subaru']
cars.sort()
print(cars)     # ['audi', 'bmw', 'subaru', 'toyota']
```

The change is **permanent**: you can't get back the original order. Reverse alphabetical with `reverse=True`:

```python
cars.sort(reverse=True)    # ['toyota', 'subaru', 'bmw', 'audi']
```

**`sorted()`: temporary sort (function)**

Shows a sorted version **without changing** the real list.

```python
cars = ['bmw', 'audi', 'toyota', 'subaru']
print(sorted(cars))        # ['audi', 'bmw', 'subaru', 'toyota']
print(cars)                # still ['bmw', 'audi', 'toyota', 'subaru']
```

`sorted()` also accepts `reverse=True`.

> **Note:** Sorting is more complicated when values aren't all lowercase. Capital letters are handled in several possible ways; the chapter keeps everything lowercase for simplicity.

**`reverse()`: flip the order**

```python
cars = ['bmw', 'audi', 'toyota', 'subaru']
cars.reverse()
print(cars)     # ['subaru', 'toyota', 'audi', 'bmw']
```

It does **not** sort backwards alphabetically; it just reverses the current order. It is permanent, but you can undo it by calling `reverse()` a second time.

### 9. Length of a list

```python
>>> cars = ['bmw', 'audi', 'toyota', 'subaru']
>>> len(cars)
4
```

Useful for counting aliens left, data to visualise, registered users, etc.

> **Note:** `len()` counts items **starting at 1**, so no off-by-one issue when finding length. (Indexes start at 0, but the count does not.)

### 10. Avoiding index errors

```python
motorcycles = ['honda', 'yamaha', 'suzuki']
print(motorcycles[3])
```

```
IndexError: list index out of range
```

Three items means valid indexes are **0, 1, 2**. People expect the third item to be number 3, but in Python it is index **2**. An index error means Python can't find the index you asked for.

- **Fix:** adjust the index by one and rerun.
- **`-1` always works** for the last item, even if the list size changes, **except on an empty list**:

```python
motorcycles = []
print(motorcycles[-1])      # IndexError: list index out of range
```

- **Debugging tip:** if you can't figure out the error, **print the list or `len(list)`**. A list managed dynamically may look very different from what you assumed.

## Mechanics / reference

### List operations at a glance

|Task|Syntax|Changes the list?|Returns|
|---|---|---|---|
|Access item|`lst[i]`|No|The item|
|Last item|`lst[-1]`|No|The item|
|Modify item|`lst[i] = value`|Yes|n/a|
|Add to end|`lst.append(v)`|Yes|n/a|
|Insert at position|`lst.insert(i, v)`|Yes (shifts right)|n/a|
|Delete by index|`del lst[i]`|Yes|n/a|
|Pop last|`lst.pop()`|Yes|The removed item|
|Pop at index|`lst.pop(i)`|Yes|The removed item|
|Remove by value|`lst.remove(v)`|Yes (first match only)|n/a|
|Sort permanently|`lst.sort()`|Yes|n/a|
|Sort temporarily|`sorted(lst)`|**No**|A new sorted list|
|Reverse|`lst.reverse()`|Yes|n/a|
|Length|`len(lst)`|No|Number of items|

### Removing: which one?

|You know…|You want the value afterwards?|Use|
|---|---|---|
|Position|No|`del lst[i]`|
|Position|Yes|`lst.pop(i)` (or `pop()` for last)|
|Value only|Either|`lst.remove(value)`|

### Permanent vs temporary

|Permanent (changes the list)|Temporary (list unchanged)|
|---|---|
|`sort()`, `sort(reverse=True)`, `reverse()`|`sorted()`, `sorted(reverse=True)`|

### Index map (list of 4)

|Item|`'bmw'`|`'audi'`|`'toyota'`|`'subaru'`|
|---|---|---|---|---|
|Positive index|0|1|2|3|
|Negative index|-4|-3|-2|-1|

### Common patterns

```python
# 1. Build a list dynamically
users = []
users.append(name)

# 2. Use and remove the last item
latest = items.pop()

# 3. Show sorted without damaging the original
print(sorted(items))

# 4. Remove by value, keep the value for a message
unwanted = 'ducati'
motorcycles.remove(unwanted)

# 5. Safe-ish "last item" access
last = items[-1]        # fails only if items is empty

# 6. Debug an IndexError
print(items)
print(len(items))
```

## Pitfalls

- **Counting from 1.** The first item is index `0`; the nth item is index `n - 1`.
- **`IndexError` on `lst[len(lst)]`.** The largest valid index is `len(lst) - 1`.
- **Using `-1` on an empty list.** Still an `IndexError`.
- **Printing a whole list for users.** You get brackets and quotes; access items individually or format them.
- **Expecting `sorted()` to change the list.** It doesn't; use `sort()` for permanent change.
- **Expecting `sort()` to be reversible.** The original order is lost for good.
- **Thinking `reverse()` sorts backwards.** It only flips the current order.
- **Confusing `sort(reverse=True)` with `reverse()`.** The first sorts then reverses alphabetical order; the second just flips whatever order exists.
- **Using `del` when you still need the value.** Use `pop()`.
- **Forgetting `pop()` removes the item.** After using it, the item is no longer in the list.
- **Assuming `remove()` removes every match.** It deletes only the **first** occurrence.
- **Using `remove()` with a position.** It takes a **value**; `del` and `pop()` take an index.
- **Forgetting `insert()` shifts items right.** Indexes of later items change.
- **Sorting mixed-case values.** Capitals complicate the order; keep values lowercase for simple sorting.
- **Calling methods on the wrong thing.** `cars.sort()` (method) vs `sorted(cars)` (function), and `len(cars)` is a function, not `cars.len()`.
- **Forgetting that item removal changes later indexes.** After `del lst[0]`, the old second item is now at index 0.

## Flashcards

- What is a list? :: A collection of items in a particular order #card
- What marks a list in Python? :: Square brackets, with items separated by commas #card
- Why name lists in the plural? :: A list usually holds more than one element (`names`, `digits`) #card
- What does `print(bicycles)` show? :: The whole list as Python represents it, including brackets and quotes #card
- How do you access an item in a list? :: `list_name[index]` #card
- What index does the first item have? :: `0` #card
- How do you get the 4th item? :: Index `3` (position minus one) #card
- What does index `-1` return? :: The last item in the list #card
- What does index `-2` return? :: The second item from the end #card
- Does `bicycles[0]` include brackets or quotes when printed? :: No, just the element's value #card
- Can you use string methods on list items? :: Yes, e.g. `bicycles[0].title()` #card
- How do you change an item in a list? :: Assign to its index: `motorcycles[0] = 'ducati'` #card
- What does `append()` do? :: Adds an item to the end of the list #card
- How do you build a list dynamically? :: Start with an empty list `[]` and `append()` values #card
- What does `insert(0, 'ducati')` do? :: Adds the item at index 0 and shifts all others one position right #card
- What does `del lst[1]` do? :: Removes the second item; you can't use its value afterwards #card
- What does `pop()` do? :: Removes the last item and returns it so you can use it #card
- Why is it called `pop`? :: A list is like a stack; pop takes the item off the top, which is the end of the list #card
- How do you pop an item from a specific position? :: `lst.pop(index)` #card
- When use `del` vs `pop()`? :: `del` to discard the item; `pop()` to use it as you remove it #card
- How do you remove an item when you only know its value? :: `lst.remove(value)` #card
- How many matches does `remove()` delete? :: Only the first occurrence #card
- What does `sort()` do? :: Sorts the list alphabetically and permanently #card
- How do you sort in reverse alphabetical order? :: `lst.sort(reverse=True)` #card
- What's the difference between `sort()` and `sorted()`? :: `sort()` permanently changes the list; `sorted()` returns a sorted copy and leaves the list unchanged #card
- What does `reverse()` do? :: Reverses the current order of the list (not a reverse alphabetical sort) #card
- How do you undo `reverse()`? :: Call `reverse()` again #card
- How do you find the number of items in a list? :: `len(lst)` #card
- Does `len()` have off-by-one problems? :: No, it counts from 1 #card
- What causes an `IndexError`? :: Asking for an index that doesn't exist in the list #card
- Valid indexes for a 3-item list? :: `0`, `1`, `2` (and `-1`, `-2`, `-3`) #card
- When does `lst[-1]` raise an `IndexError`? :: When the list is empty #card
- How to debug an index error you can't figure out? :: Print the list or its length to see what it really contains #card
- What is a "dynamic" list? :: One you build, then add and remove items from as the program runs #card

## Open questions

- [ ] What is slicing (`lst[1:3]`), and how does it relate to indexes? (Probably Chapter 4.)
- [ ] How do I loop through a list instead of printing item by item? (Chapter 4.)
- [ ] Are lists mutable vs tuples? What changes if I use a tuple?
- [ ] How does `sort()` handle mixed-case or non-string values (e.g. `key=str.lower`)?
- [ ] What happens if `remove()` is called with a value not in the list? (Likely `ValueError`.)
- [ ] How are Python lists implemented internally, and why does `insert(0, x)` cost more than `append(x)`? (Useful for DSA.)

## Key terms

|Term|Definition|
|---|---|
|List|Ordered collection of items in square brackets|
|Element / item|A single value in a list|
|Index|Position of an element, starting at 0|
|Negative index|Counts from the end; `-1` is the last item|
|Off-by-one error|Being one position out when counting indexes|
|`IndexError`|Requested index doesn't exist in the list|
|Dynamic list|List that grows and shrinks while the program runs|
|Empty list|`[]`; a list with no items|
|`append()`|Adds an item to the end|
|`insert()`|Adds an item at a chosen index, shifting others right|
|`del`|Statement that removes an item by index|
|`pop()`|Removes an item (last by default) and returns it|
|`remove()`|Removes the first item matching a value|
|`sort()`|Permanently sorts the list|
|`sorted()`|Function returning a temporarily sorted copy|
|`reverse()`|Permanently flips the order of the list|
|`len()`|Function returning the number of items|
|Stack|Collection where the last item added is the first removed (the idea behind `pop`)|

## Try it yourself (chapter exercises)

- [ ] 3-1 Names: list of friends' names; print each by index
- [ ] 3-2 Greetings: same list, print a personalised message to each
- [ ] 3-3 Your Own List: list of transportation modes; print statements like "I would like to own a Honda motorcycle."
- [ ] 3-4 Guest List: at least three dinner guests; print an invitation to each
- [ ] 3-5 Changing Guest List: one guest can't come; print their name, replace them, print new invitations
- [ ] 3-6 More Guests: bigger table; add guests using `insert()` (beginning, middle) and `append()` (end); reprint invitations
- [ ] 3-7 Shrinking Guest List: only two guests fit; `pop()` guests one at a time with an apology message, message the remaining two, then `del` the last two and print the empty list
- [ ] 3-8 Seeing the World: five places, non-alphabetical; use `sorted()`, `reverse()`, `sort()` and print after each step
- [ ] 3-9 Dinner Guests: use `len()` to print how many people are invited
- [ ] 3-10 Every Function: build a list and use every function/method from this chapter at least once
- [ ] 3-11 Intentional Error: deliberately cause an `IndexError`, then fix it

## Related

→ Next: [[WORKING WITH LIST]]