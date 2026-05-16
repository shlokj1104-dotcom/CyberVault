---
title: FUNCTIONS
date: 2026-05-15
phase: phase-1
tags:
  - concept
  - python
links: []
status: learning
---
# FUNCTIONS

## One-Line Summary

A function is a named, reusable block of code that does one specific job — you `def`ine it once with parameters, and `call` it as many times as you want, optionally getting a `return` value back; store related functions in a _module_ to keep your programs clean.

---

## Core Concept

Think of a function like a vending machine. You press a button (call the function), drop in coins if needed (pass arguments), and it dispenses the exact same snack every time (predictable output). You don't care how the machine works inside — you just know what goes in and what comes out. Without functions, you'd have to rebuild the vending machine from scratch every time you wanted a snack.

Unlike Java, Python functions don't declare a return type upfront — they just return whatever you `return`, or `None` if you return nothing. And instead of `void`, you simply omit the `return` statement.

---

## How It Works

### Defining and Calling a Function

```python
# Python — simplest possible function
def greet_user():
    """Display a simple greeting."""   # ← docstring
    print("Hello!")

greet_user()   # calling the function
# Output: Hello!
```

**Anatomy of a function:**

- `def` — keyword that starts a definition
- `greet_user` — function name (lowercase + underscores by convention)
- `()` — parentheses required, even when empty
- `:` — ends the header
- indented body — everything that runs when called
- `"""..."""` — **docstring**: describes what the function does; Python uses it to generate docs

---

### Parameters and Arguments

**Parameter** = the variable in the function definition (placeholder) **Argument** = the actual value passed in the function call

```python
def greet_user(username):          # Python — 'username' is the parameter
    print(f"Hello, {username.title()}!")

greet_user('jesse')                # 'jesse' is the argument
# Output: Hello, Jesse!
```

> 💡 People use the words _parameter_ and _argument_ interchangeably in conversation — don't be confused if you see that in documentation.

---

### Passing Arguments

Python gives you three ways to pass arguments:

#### 1. Positional Arguments

Matched by **order** — must be in the same order as parameters.

```python
def describe_pet(animal_type, pet_name):    # Python
    """Display information about a pet."""
    print(f"\nI have a {animal_type}.")
    print(f"My {animal_type}'s name is {pet_name.title()}.")

describe_pet('hamster', 'harry')
# Output:
# I have a hamster.
# My hamster's name is Harry.
```

> ⚠️ Order matters! `describe_pet('harry', 'hamster')` gives nonsense output — "I have a harry."

#### 2. Keyword Arguments

Matched by **name** — order doesn't matter.

```python
describe_pet(animal_type='hamster', pet_name='harry')    # Python
describe_pet(pet_name='harry', animal_type='hamster')    # same result
```

#### 3. Default Values

Set a default in the definition — if the caller doesn't pass that argument, Python uses the default.

```python
def describe_pet(pet_name, animal_type='dog'):    # Python
    """Display information about a pet."""
    print(f"\nI have a {animal_type}.")
    print(f"My {animal_type}'s name is {pet_name.title()}.")

describe_pet('willie')                            # uses default 'dog'
describe_pet(pet_name='harry', animal_type='hamster')   # overrides default
```

> ⚠️ Parameters **with** default values must come **after** parameters **without** them.

---

### Avoiding Argument Errors

If you call a function with too few or too many arguments, Python raises a `TypeError` and tells you exactly which arguments are missing:

```python
describe_pet()    # Python — no arguments given
# TypeError: describe_pet() missing 2 required positional arguments:
#            'animal_type' and 'pet_name'
```

Use these error messages — they name the missing parameters, which often lets you fix the call without even opening the function file.

---

### Return Values

A function can process data and send a result back with `return`.

#### Returning a Simple Value

```python
def get_formatted_name(first_name, last_name):    # Python
    """Return a full name, neatly formatted."""
    full_name = first_name + ' ' + last_name
    return full_name.title()

musician = get_formatted_name('jimi', 'hendrix')
print(musician)
# Output: Jimi Hendrix
```

#### Making an Argument Optional

Use an empty string `''` as the default to make a parameter optional, then check with `if`:

```python
def get_formatted_name(first_name, last_name, middle_name=''):    # Python
    """Return a full name, neatly formatted."""
    if middle_name:
        full_name = first_name + ' ' + middle_name + ' ' + last_name
    else:
        full_name = first_name + ' ' + last_name
    return full_name.title()

print(get_formatted_name('jimi', 'hendrix'))           # Jimi Hendrix
print(get_formatted_name('john', 'hooker', 'lee'))     # John Lee Hooker
```

#### Returning a Dictionary

A function can return any data structure — not just strings:

```python
def build_person(first_name, last_name, age=''):    # Python
    """Return a dictionary of information about a person."""
    person = {'first': first_name, 'last': last_name}
    if age:
        person['age'] = age
    return person

musician = build_person('jimi', 'hendrix', age=27)
print(musician)
# Output: {'first': 'jimi', 'last': 'hendrix', 'age': 27}
```

#### Using a Function with a while Loop

```python
while True:                                          # Python
    print("\nPlease tell me your name:")
    print("(enter 'q' at any time to quit)")
    f_name = input("First name: ")
    if f_name == 'q': break
    l_name = input("Last name: ")
    if l_name == 'q': break
    formatted_name = get_formatted_name(f_name, l_name)
    print(f"\nHello, {formatted_name}!")
```

---

### Passing a List

When you pass a list to a function, the function gets direct access to its contents — and any changes are **permanent** (the function works on the original list).

```python
def greet_users(names):          # Python
    """Print a simple greeting to each user in the list."""
    for name in names:
        print(f"Hello, {name.title()}!")

usernames = ['hannah', 'ty', 'margot']
greet_users(usernames)
# Output: Hello, Hannah! / Hello, Ty! / Hello, Margot!
```

#### Modifying a List in a Function

```python
def print_models(unprinted_designs, completed_models):    # Python
    """Simulate printing each design until none are left."""
    while unprinted_designs:
        current_design = unprinted_designs.pop()
        print(f"Printing model: {current_design}")
        completed_models.append(current_design)

unprinted = ['iphone case', 'robot pendant', 'dodecahedron']
completed = []
print_models(unprinted, completed)
# unprinted is now [] — the original list is modified!
```

#### Preventing Modification — Pass a Copy

Use slice notation `[:]` to pass a copy instead of the original:

```python
print_models(unprinted_designs[:], completed_models)    # Python
# unprinted_designs list is untouched; function works on a copy
```

> 💡 Prefer passing the original list (more efficient) unless you specifically need to preserve it.

---

### Passing an Arbitrary Number of Arguments

#### `*args` — Arbitrary Positional Arguments

The `*` prefix tells Python to pack all extra positional arguments into a **tuple**:

```python
def make_pizza(*toppings):          # Python — *toppings catches everything
    """Summarize the pizza we are about to make."""
    print("\nMaking a pizza with the following toppings:")
    for topping in toppings:
        print(f"- {topping}")

make_pizza('pepperoni')
make_pizza('mushrooms', 'green peppers', 'extra cheese')
```

#### Mixing Positional and `*args`

Put `*args` **last** in the parameter list — Python fills positional parameters first, then packs the rest into the tuple:

```python
def make_pizza(size, *toppings):    # Python
    print(f"\nMaking a {size}-inch pizza with the following toppings:")
    for topping in toppings:
        print(f"- {topping}")

make_pizza(16, 'pepperoni')
make_pizza(12, 'mushrooms', 'green peppers', 'extra cheese')
```

#### `**kwargs` — Arbitrary Keyword Arguments

The `**` prefix packs extra keyword arguments into a **dictionary**:

```python
def build_profile(first, last, **user_info):    # Python
    """Build a dictionary containing everything we know about a user."""
    profile = {'first_name': first, 'last_name': last}
    for key, value in user_info.items():
        profile[key] = value
    return profile

user = build_profile('albert', 'einstein',
                     location='princeton',
                     field='physics')
print(user)
# Output: {'first_name': 'albert', 'last_name': 'einstein',
#           'location': 'princeton', 'field': 'physics'}
```

---

### Built-in Functions (from PDF 2 supplement)

Python ships with ready-to-use built-in functions — no import needed:

```python
max('Hello world')    # Python → 'w'   (largest character)
min('Hello world')    # → ' '          (smallest character)
len('Hello world')    # → 11           (number of characters)
```

#### Type Conversion Functions

```python
int('32')       # Python → 32     (string → integer)
int(3.99999)    # → 3             (truncates, does NOT round)
int(-2.3)       # → -2            (truncates toward zero)
float(32)       # → 32.0
float('3.14')   # → 3.14
str(32)         # → '32'
```

> ⚠️ `int('Hello')` raises `ValueError` — `int()` only works if the string actually represents an integer.

#### Math Module

```python
import math                     # Python — must import first

math.log10(ratio)               # log base 10
math.sin(radians)               # trig functions take radians
math.sqrt(2)                    # → 1.4142...
math.pi                         # → 3.14159... (constant)

# Convert degrees → radians manually:
radians = degrees / 360.0 * 2 * math.pi
```

---

### Storing Functions in Modules

A **module** is a `.py` file that contains functions. Splitting functions into modules keeps your main program clean and lets you reuse code across projects.

```python
# pizza.py — the module file
def make_pizza(size, *toppings):
    """Summarize the pizza we are about to make."""
    print(f"\nMaking a {size}-inch pizza with:")
    for topping in toppings:
        print(f"- {topping}")
```

**Five ways to import:**

```python
# 1. Import entire module (use dot notation to call)
import pizza
pizza.make_pizza(16, 'pepperoni')

# 2. Import specific function(s)
from pizza import make_pizza
make_pizza(16, 'pepperoni')

# 3. Import multiple specific functions
from pizza import make_pizza, other_func

# 4. Alias a function
from pizza import make_pizza as mp
mp(16, 'pepperoni')

# 5. Alias a module
import pizza as p
p.make_pizza(16, 'pepperoni')

# 6. Import everything (use sparingly — can cause name collisions)
from pizza import *
make_pizza(16, 'pepperoni')
```

> ⚠️ Prefer options 1 or 2. `from module import *` can silently overwrite names in your program if the module has functions with the same name.

---

### Styling Functions

|Rule|Example|
|---|---|
|Lowercase name with underscores|`get_formatted_name` not `GetFormattedName`|
|Docstring immediately after `def`|`"""Return a full name."""`|
|No spaces around `=` for defaults|`def f(x, animal_type='dog'):`|
|No spaces around `=` for keyword args|`describe_pet(pet_name='harry')`|
|Max 79 chars per line (PEP 8)|Break after `(`, indent body twice|
|Two blank lines between functions|Makes it easy to spot where one ends|
|All `import` statements at top of file|Unless a comment at top describes the program|

---

## Java vs Python

|Java|Python|
|---|---|
|Return type declared: `public String getName()`|No return type declared: `def get_name():`|
|`void` for no return|Omit `return` (implicitly returns `None`)|
|`null`|`None`|
|No default parameter values|`def f(x, y='dog'):` — built in|
|No `*args` / `**kwargs` — use overloads|`*args` (tuple) / `**kwargs` (dict)|
|`import java.util.Math` then `Math.sqrt(2)`|`import math` then `math.sqrt(2)`|
|Checked exceptions from type mismatch|`TypeError` / `ValueError` at runtime|
|No docstrings (JavaDoc uses `/** */` outside)|`"""..."""` docstrings inside the function body|

---

## Quick Reference

```python
# Define
def function_name(param1, param2='default'):    # Python
    """Docstring."""
    body
    return value          # optional

# Call — positional
function_name(val1, val2)

# Call — keyword (order-independent)
function_name(param2='b', param1='a')

# Arbitrary positional → tuple
def f(*args): ...

# Arbitrary keyword → dict
def f(**kwargs): ...

# Both
def f(required, *args, **kwargs): ...

# Pass list copy
function_name(my_list[:])

# Import styles
import module                         # module.func()
from module import func               # func()
from module import func as f          # f()
import module as m                    # m.func()
from module import *                  # func() — avoid

# Built-ins
max(iterable)   min(iterable)   len(iterable)
int(x)   float(x)   str(x)

# Math module
import math
math.sqrt(x)   math.sin(x)   math.log10(x)   math.pi
```

---

## Practice Exercises

- [ ] **Display Message** — Write `display_message()` that prints one sentence about what you're learning. Call it.
- [ ] **Favorite Book** — Write `favorite_book(title)` that prints `One of my favorite books is <title>`. Call it.
- [ ] **T-Shirt** — Write `make_shirt(size, message)`. Default: large + "I love Python". Call it positionally and with keyword args.
- [ ] **Cities** — Write `describe_city(city, country='India')`. Call for 3 cities, at least one outside India.
- [ ] **City Names** — Write `city_country(city, country)` returning `"Santiago, Chile"` format. Call 3 times.
- [ ] **Album** — Write `make_album(artist, title, tracks='')` returning a dictionary. Add optional `tracks` key only if provided.
- [ ] **User Albums** — Add a `while` loop to the above to collect input until the user types `'quit'`.
- [ ] **Magicians** — Pass a list to `show_magicians()`. Then write `make_great()` to prepend "the Great" to each name.
- [ ] **Unchanged Magicians** — Pass a copy so the original list stays intact. Print both lists.
- [ ] **Sandwiches** — Write `make_sandwich(*items)` that prints all items for any number of toppings.
- [ ] **User Profile** — Use `build_profile(first, last, **user_info)` to build your own profile with 3 extra key-value pairs.
- [ ] **Cars** — `make_car(manufacturer, model, **options)` — call with `color='blue', tow_package=True`.
- [ ] **Module Practice** — Move one of your functions to a separate `.py` file and import it 5 different ways.

---

## Questions I Still Have

_Write your open questions here. Return later and answer them._

---

## Related Notes

- [[ITERATIVE STATEMENTS]]
- [[CONDITIONAL STATEMENT]]