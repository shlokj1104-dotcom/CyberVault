---
title: DICTIONARIES
date: 2026-11-06
language: Python
phase: phase-1
tags:
links: []
status: learning
---
> **One-line summary:** A **dictionary** is a collection of **key-value pairs** in braces (`{'color': 'green', 'points': 5}`). You look up a value by its **key** (`alien['color']`), add or change pairs by assignment, remove them with **`del`**, and loop with **`items()`** (pairs), **`keys()`** (keys) or **`values()`** (values). Wrap `sorted()` around the keys for ordered output and `set()` around values to drop duplicates. **Nesting** lets you put dictionaries in a list, a list in a dictionary, or a dictionary in a dictionary, but don't nest too deeply.

## Structure

### 1. A simple dictionary

A game with aliens of different colours and point values:

```python
alien_0 = {'color': 'green', 'points': 5}

print(alien_0['color'])     # green
print(alien_0['points'])    # 5
```

### 2. Working with dictionaries

A **dictionary** is a collection of **key-value pairs**.

- Each **key** is connected to a **value**; give Python a key and it returns the associated value.
- A value can be a **number, string, list, or even another dictionary**. In fact, any object you can create in Python can be a value.
- Written in **braces `{}`**; each key is joined to its value by a **colon**, and pairs are separated by **commas**.
- You can store as many pairs as you want. The simplest dictionary has exactly one:

```python
alien_0 = {'color': 'green'}      # key 'color', value 'green'
```

### 3. Accessing values

Dictionary name, then the key in square brackets:

```python
alien_0 = {'color': 'green', 'points': 5}

new_points = alien_0['points']
print("You just earned " + str(new_points) + " points!")
# You just earned 5 points!
```

Each time an alien is shot down, you can look up how many points it's worth.

### 4. Adding new key-value pairs

Dictionaries are **dynamic**: add pairs any time by naming the dictionary, the **new key** in brackets, and assigning a value.

```python
alien_0 = {'color': 'green', 'points': 5}

alien_0['x_position'] = 0
alien_0['y_position'] = 25
print(alien_0)
# {'color': 'green', 'points': 5, 'y_position': 25, 'x_position': 0}
```

Notice the order of the pairs doesn't match the order you added them. Python cares only about the **connection between each key and its value**.

### 5. Starting with an empty dictionary

Define it with empty braces, then add each pair on its own line.

```python
alien_0 = {}

alien_0['color'] = 'green'
alien_0['points'] = 5

print(alien_0)      # {'color': 'green', 'points': 5}
```

Typically used when storing **user-supplied data** or when code **generates many pairs automatically**.

### 6. Modifying values

Name the dictionary, the key in brackets, then the new value:

```python
alien_0 = {'color': 'green'}
alien_0['color'] = 'yellow'
print("The alien is now " + alien_0['color'] + ".")    # The alien is now yellow.
```

**Example: change behaviour by changing one value.**

```python
alien_0 = {'x_position': 0, 'y_position': 25, 'speed': 'medium'}
print("Original x-position: " + str(alien_0['x_position']))

# How far to move depends on speed
if alien_0['speed'] == 'slow':
    x_increment = 1
elif alien_0['speed'] == 'medium':
    x_increment = 2
else:
    # This must be a fast alien.
    x_increment = 3

alien_0['x_position'] = alien_0['x_position'] + x_increment
print("New x-position: " + str(alien_0['x_position']))
```

Output: `Original x-position: 0` then `New x-position: 2`.

Changing `alien_0['speed']` to `'fast'` (a string, in quotes) makes the same code add a larger increment next run.

### 7. Removing key-value pairs

Use **`del`** with the dictionary name and the key. The key **and its value** are removed.

```python
alien_0 = {'color': 'green', 'points': 5}
del alien_0['points']
print(alien_0)      # {'color': 'green'}
```

> **Note:** the deleted pair is removed **permanently**.

### 8. A dictionary of similar objects

A dictionary can also store **one kind of information about many objects**, e.g. a poll:

```python
favorite_languages = {
    'jen': 'python',
    'sarah': 'c',
    'edward': 'ruby',
    'phil': 'python',
    }
```

Each key is a person's name; each value is their language.

**Formatting a multi-line dictionary:**

1. Press ENTER after the opening brace.
2. Indent the next line one level (four spaces) and write the first pair, followed by a comma.
3. Your editor should auto-indent the following pairs.
4. Put the closing brace on a new line, indented to align with the keys.
5. It's good practice to include a **comma after the last pair** so adding another is easy.

(Other formatting styles exist; you may see slight differences in editors or other sources.)

```python
print("Sarah's favorite language is " +
    favorite_languages['sarah'].title() +
    ".")
# Sarah's favorite language is C.
```

**Breaking a long `print`:** put the first part right after the opening parenthesis, break at a sensible point with a `+` at the end of the line, indent the continuation lines one level, and put the closing parenthesis on the last line.

### 9. Looping through all key-value pairs: `items()`

```python
user_0 = {
    'username': 'efermi',
    'first': 'enrico',
    'last': 'fermi',
    }

for key, value in user_0.items():
    print("\nKey: " + key)
    print("Value: " + value)
```

- `items()` returns the key-value pairs; the loop stores each pair in **two variables** you name (`key, value` or `k, v`, any names work).
- Pairs come back **not necessarily in the order stored**; Python tracks only the key-value connections.
- Use **descriptive variable names** when the data has a clear meaning:

```python
for name, language in favorite_languages.items():
    print(name.title() + "'s favorite language is " +
        language.title() + ".")
```

```
Jen's favorite language is Python.
Sarah's favorite language is C.
Phil's favorite language is Python.
Edward's favorite language is Ruby.
```

This works the same for a thousand or a million entries.

### 10. Looping through the keys: `keys()`

```python
for name in favorite_languages.keys():
    print(name.title())
```

- **Looping over keys is the default**, so `for name in favorite_languages:` gives exactly the same result. Use `keys()` explicitly if it makes the code easier to read.
- Inside the loop you can use the current key to get its value:

```python
friends = ['phil', 'sarah']
for name in favorite_languages.keys():
    print(name.title())
    if name in friends:
        print("  Hi " + name.title() +
            ", I see your favorite language is " +
            favorite_languages[name].title() + "!")
```

- `keys()` isn't just for loops; use it to check whether a key exists:

```python
if 'erin' not in favorite_languages.keys():
    print("Erin, please take our poll!")
```

### 11. Looping through keys in order: `sorted()`

Dictionaries don't give you items in a predictable order. To get keys in order, wrap `sorted()` around `keys()`:

```python
for name in sorted(favorite_languages.keys()):
    print(name.title() + ", thank you for taking the poll.")
```

Output: Edward, Jen, Phil, Sarah (alphabetical).

### 12. Looping through the values: `values()`

For when you only care about the values:

```python
print("The following languages have been mentioned:")
for language in favorite_languages.values():
    print(language.title())
```

```
Python
C
Python
Ruby
```

No check for repeats (`Python` appears twice). To get each value **once**, wrap it in **`set()`**: a set is like a list in which **each item must be unique**.

```python
for language in set(favorite_languages.values()):
    print(language.title())
```

Output: Python, C, Ruby (no repeats).

### 13. Nesting: a list of dictionaries

**Nesting** = storing dictionaries in a list, a list in a dictionary, or a dictionary in a dictionary.

One dictionary holds one alien; for many aliens, make a **list where each item is a dictionary**:

```python
alien_0 = {'color': 'green', 'points': 5}
alien_1 = {'color': 'yellow', 'points': 10}
alien_2 = {'color': 'red', 'points': 15}

aliens = [alien_0, alien_1, alien_2]

for alien in aliens:
    print(alien)
```

**Generate a fleet automatically:**

```python
aliens = []

for alien_number in range(30):
    new_alien = {'color': 'green', 'points': 5, 'speed': 'slow'}
    aliens.append(new_alien)

for alien in aliens[:5]:        # slice: show the first five
    print(alien)
print("...")

print("Total number of aliens: " + str(len(aliens)))    # 30
```

`range()` just says how many times to repeat. Each alien is a **separate object**, so you can modify each one individually.

**Modify some aliens** (first three: green → yellow, medium speed, 10 points):

```python
for alien in aliens[0:3]:
    if alien['color'] == 'green':
        alien['color'] = 'yellow'
        alien['speed'] = 'medium'
        alien['points'] = 10
    elif alien['color'] == 'yellow':
        alien['color'] = 'red'
        alien['speed'] = 'fast'
        alien['points'] = 15
```

It's common to store many dictionaries in a list when each describes one object (e.g. a `users` list). **All dictionaries in the list should have an identical structure** so you can loop through them the same way.

### 14. Nesting: a list in a dictionary

Use a list as a value when you want **more than one value per key**.

```python
pizza = {
    'crust': 'thick',
    'toppings': ['mushrooms', 'extra cheese'],
    }

print("You ordered a " + pizza['crust'] + "-crust pizza " +
    "with the following toppings:")

for topping in pizza['toppings']:
    print("\t" + topping)
```

```
You ordered a thick-crust pizza with the following toppings:
    mushrooms
    extra cheese
```

**Multiple favourite languages per person**, with a loop inside a loop:

```python
favorite_languages = {
    'jen': ['python', 'ruby'],
    'sarah': ['c'],
    'edward': ['ruby', 'go'],
    'phil': ['python', 'haskell'],
    }

for name, languages in favorite_languages.items():
    print("\n" + name.title() + "'s favorite languages are:")
    for language in languages:
        print("\t" + language.title())
```

Use a plural name (`languages`) for the loop variable holding each list. Refinement: check `len(languages)` and change the wording when a person has just one favourite ("Sarah's favorite language is C.").

> **Note:** don't nest lists and dictionaries too deeply. If you're nesting much deeper than these examples, or working with someone else's heavily nested code, a simpler solution probably exists.

### 15. Nesting: a dictionary in a dictionary

Usernames as keys, a dictionary of info as each value:

```python
users = {
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
    print("\nUsername: " + username)
    full_name = user_info['first'] + " " + user_info['last']
    location = user_info['location']

    print("\tFull name: " + full_name.title())
    print("\tLocation: " + location.title())
```

```
Username: aeinstein
    Full name: Albert Einstein
    Location: Princeton

Username: mcurie
    Full name: Marie Curie
    Location: Paris
```

The outer loop gives `username` and `user_info` (the inner dictionary); then you use the inner keys. Each user's dictionary has the **same keys**. Not required by Python, but different keys would make the loop much more complicated.

## Mechanics / reference

### Dictionary operations at a glance

|Task|Syntax|Notes|
|---|---|---|
|Create|`d = {'k': 'v', 'k2': 'v2'}`|Colon between key and value|
|Create empty|`d = {}`|Add pairs afterwards|
|Get value|`d['k']`|Key in square brackets|
|Add pair|`d['new'] = value`|Dynamic; any time|
|Modify value|`d['k'] = new_value`|Same syntax as adding|
|Delete pair|`del d['k']`|Permanent|
|Loop over pairs|`for k, v in d.items():`|Two loop variables|
|Loop over keys|`for k in d.keys():` or `for k in d:`|Keys is the default|
|Loop over values|`for v in d.values():`|May contain repeats|
|Unique values|`set(d.values())`|Set = no duplicates|
|Keys in order|`sorted(d.keys())`|Original dictionary unchanged|
|Key present?|`'k' in d.keys()` / `'k' not in d.keys()`|Membership test|

### What you're looping over

|Method|Gives you|Typical loop|
|---|---|---|
|`items()`|Key-value pairs|`for name, language in d.items():`|
|`keys()`|Keys only|`for name in d.keys():`|
|`values()`|Values only|`for language in d.values():`|

### List vs dictionary

| |List|Dictionary|
|---|---|---|
|Brackets|`[ ]`|`{ }`|
|Access by|Index (position)|Key|
|Holds|Ordered items|Key-value pairs|
|Add|`append()` / `insert()`|`d[key] = value`|
|Remove|`del lst[i]`, `pop()`, `remove()`|`del d[key]`|
|Use when|A sequence of items|Connecting related information|

### Nesting patterns

|Pattern|Example|Use when|
|---|---|---|
|List of dictionaries|`aliens = [{...}, {...}]`|Many objects, each with several attributes|
|List in a dictionary|`pizza = {'toppings': [...]}`|More than one value for a key|
|Dictionary in a dictionary|`users = {'id': {...}}`|Records looked up by a unique key|

### Common patterns

```python
# 1. Build from scratch
d = {}
d['name'] = 'ada'

# 2. Loop with descriptive names
for name, language in favorite_languages.items():
    ...

# 3. Sorted keys
for name in sorted(favorite_languages.keys()):
    ...

# 4. Unique values
for language in set(favorite_languages.values()):
    ...

# 5. Check key before use
if 'erin' not in favorite_languages.keys():
    print("Erin, please take our poll!")

# 6. Generate many objects
objs = []
for n in range(30):
    objs.append({'color': 'green', 'points': 5})

# 7. Loop in a loop (list as value)
for name, languages in d.items():
    for language in languages:
        ...
```

## Pitfalls

- **Forgetting quotes around a string value** (`alien_0['speed'] = fast` instead of `'fast'`). Python looks for a variable named `fast` and raises a `NameError`.
- **Using the wrong brackets.** Dictionary literals use `{}`; access uses `[]` with the key.
- **Forgetting the colon** between a key and its value, or the comma between pairs.
- **Expecting items in the order you added them.** Dictionaries track connections, not order; use `sorted()` for ordered output.
- **Expecting `del` to be undoable.** The key-value pair is removed permanently.
- **Looping with one variable over `items()`.** You need two (`for k, v in d.items()`); a single variable gets a (key, value) pair together.
- **Using `values()` and expecting unique results.** Repeats come through; wrap in `set()`.
- **Confusing `set()` with a list.** A set contains each item only once.
- **Mixing `str` and numbers in output.** Values like `5` need `str()` before concatenating: `str(alien_0['points'])`.
- **Giving dictionaries in a list different structures.** Loops become complicated; keep keys identical.
- **Over-nesting.** If you're going deeper than the chapter's examples, look for a simpler design.
- **Modifying only a copy.** In the alien examples each dictionary in the list is a separate object, so changes affect just that alien, which is usually what you want.
- **Forgetting the key isn't present.** Asking for a missing key is an error (the book hasn't covered handling this yet); check with `in` first.
- **Misaligned closing brace / missing trailing comma** in multi-line dictionaries. Not an error, but untidy and makes adding pairs error-prone.

## Flashcards

- What is a dictionary? :: A collection of key-value pairs #card
- What brackets wrap a dictionary? :: Braces `{}` #card
- How are a key and its value joined? :: With a colon (`'color': 'green'`) #card
- How are key-value pairs separated? :: By commas #card
- What can a dictionary value be? :: Any Python object: number, string, list, another dictionary #card
- How do you get a value from a dictionary? :: `dictionary['key']` #card
- How do you add a new key-value pair? :: `dictionary['new_key'] = value` #card
- Does the order of pairs match the order you added them? :: Not necessarily; Python tracks only key-value connections #card
- How do you start an empty dictionary? :: `my_dict = {}` #card
- When are empty dictionaries typically used? :: When storing user-supplied data or generating many pairs automatically #card
- How do you change a value? :: Assign a new value to the key: `d['color'] = 'yellow'` #card
- How do you remove a key-value pair? :: `del d['key']` #card
- Is deleting a pair reversible? :: No, it's permanent #card
- How do you break a long dictionary over several lines? :: ENTER after the opening brace, indent pairs one level, closing brace on its own line aligned with the keys #card
- Why put a comma after the last pair? :: So you're ready to add another pair on the next line #card
- What does `items()` return? :: The dictionary's key-value pairs #card
- How do you loop over all key-value pairs? :: `for key, value in d.items():` #card
- What does `keys()` give you? :: The dictionary's keys #card
- What's the default when you loop over a dictionary? :: Looping over the keys #card
- How do you check whether a key exists? :: `'erin' in d.keys()` or `'erin' not in d.keys()` #card
- How do you loop over keys in alphabetical order? :: `for k in sorted(d.keys()):` #card
- What does `values()` give you? :: The dictionary's values, without keys #card
- How do you get unique values only? :: Wrap in `set()`: `set(d.values())` #card
- What is a set? :: Like a list, but every item must be unique #card
- What is nesting? :: Storing dictionaries in a list, a list in a dictionary, or a dictionary in another dictionary #card
- What is a list of dictionaries good for? :: Storing many objects that each have several attributes (e.g. a fleet of aliens) #card
- How do you generate 30 similar dictionaries? :: Loop with `range(30)`, build a dictionary each time and `append()` it to a list #card
- Why should dictionaries in a list share the same structure? :: So you can loop through and process each one the same way #card
- When nest a list in a dictionary? :: When a key needs more than one value (e.g. several toppings) #card
- How do you loop through a list stored as a dictionary value? :: A nested `for` loop: outer over `items()`, inner over the list #card
- What does a dictionary in a dictionary look like? :: `users = {'username': {'first': ..., 'last': ...}}` #card
- Advice on depth of nesting? :: Don't nest too deeply; a simpler solution likely exists #card

## Open questions

- [ ] What happens when I access a key that doesn't exist (`KeyError`), and how does `get()` avoid it?
- [ ] Are dictionaries ordered in modern Python? (The book says order isn't predictable; newer versions keep insertion order. Check what changed and when.)
- [ ] What types can be dictionary keys? Why can't a list be a key (hashability)?
- [ ] How does a dictionary look up a key so fast? (Hash tables; DSA link: average O(1).)
- [ ] What else can I do with sets (union, intersection, membership)?
- [ ] What does `keys()` actually return (a view or a list)?
- [ ] How do I copy a nested structure without sharing inner objects (shallow vs deep copy)?

## Key terms

|Term|Definition|
|---|---|
|Dictionary|Collection of key-value pairs in braces|
|Key|Label used to look up a value|
|Value|Data associated with a key|
|Key-value pair|A key linked to its value by a colon|
|`del`|Removes a key-value pair permanently|
|`items()`|Method returning the key-value pairs for looping|
|`keys()`|Method returning the keys|
|`values()`|Method returning the values|
|`sorted()`|Function returning a sorted copy (e.g. of the keys)|
|`set()`|Builds a collection of unique items|
|Set|Unordered collection in which each item is unique|
|Nesting|Storing one structure inside another|
|List of dictionaries|List whose items are dictionaries|
|List in a dictionary|A value that is a list|
|Dictionary in a dictionary|A value that is itself a dictionary|

## Try it yourself (chapter exercises)

- [ ] 6-1 Person: dictionary with `first_name`, `last_name`, `age`, `city`; print each piece
- [ ] 6-2 Favorite Numbers: five names as keys, favourite numbers as values; print each
- [ ] 6-3 Glossary: five programming words and meanings; print neatly, blank line between entries (`\n`)
- [ ] 6-4 Glossary 2: replace the `print`s with a loop over keys and values; add five more terms
- [ ] 6-5 Rivers: three rivers and their countries; loops to print a sentence ("The Nile runs through Egypt."), then the rivers, then the countries
- [ ] 6-6 Polling: list of people to poll; thank those already in `favorite_languages`, invite the rest
- [ ] 6-7 People: start from 6-1; make two more person dictionaries; store all three in a list `people`; loop and print everything about each
- [ ] _(Exercises after 6-7 weren't in the pages provided.)_

## Related

→ Next: [[7 - User Input and while Loops]]