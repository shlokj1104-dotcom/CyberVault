---
title: VARIABLES AND SIMPLE DATA TYPES
date: 2026-11-05
language: Python
phase: phase-1
tags:
links: []
status: learning
---
> **One-line summary:** A **variable** is a name attached to a value (`message = "Hello"`), and the value can change at any time. The simple data types are **strings** (text in quotes, with methods like `.title()` and `.strip()`) and **numbers** (`int` and `float`). The classic early errors are **`NameError`** (misspelled or undefined name), **`SyntaxError`** (mismatched quotes) and **`TypeError`** (mixing `int` with `str`, fixed with `str()`). **Comments** start with `#` and are ignored by Python.

## Structure

### 1. What happens when you run a `.py` file

```python
print("Hello Python world!")
```

- The `.py` ending tells your editor the file is a Python program.
- The editor passes it to the **Python interpreter**, which reads the program and works out what each word means. When it sees `print`, it prints whatever is inside the parentheses.
- **Syntax highlighting:** the editor colours code (function names, strings, etc.) differently. It is a free hint for spotting mistakes, e.g. if code looks like plain text or text looks like code, you probably have a mismatched quote.

### 2. Variables

```python
message = "Hello Python world!"
print(message)
```

- A **variable** is a name; its **value** is the information attached to it. Here the value is the text `"Hello Python world!"`.
- Line 1: the interpreter associates the text with the name `message`. Line 2: it prints the value associated with `message`.
- You can **change the value at any time**; Python always tracks the current one.

```python
message = "Hello Python world!"
print(message)

message = "Hello Python Crash Course world!"   # reassigned
print(message)
```

Output: two lines, one per `print`.

### 3. Naming and using variables

**Rules (break these and you get errors):**

- Only **letters, numbers, underscores**.
- May start with a letter or underscore, **never a number**: `message_1` OK, `1_message` not.
- **No spaces**. Use underscores: `greeting_message` OK, `greeting message` error.
- Don't use **Python keywords or built-in function names** (like `print`) as variable names.

**Guidelines (readability only):**

- Short but descriptive: `name` > `n`, `student_name` > `s_n`, `name_length` > `length_of_persons_name`.
- Be careful with lowercase `l` and uppercase `O`: they look like `1` and `0`.
- Stick to **lowercase** variable names for now. Uppercase won't error, but it is a convention to avoid.

### 4. Name errors and tracebacks

```python
message = "Hello Python Crash Course reader!"
print(mesage)          # typo: missing the 's'
```

```
Traceback (most recent call last):
  File "hello_world.py", line 2, in <module>
    print(mesage)
NameError: name 'mesage' is not defined
```

**How to read a traceback** (a record of where the interpreter ran into trouble):

1. **File and line number** where the error happened (`line 2`).
2. **The offending line** of code.
3. **The type of error and message** (`NameError: name 'mesage' is not defined`).

A `NameError` usually means you **forgot to assign** the variable before using it, or **misspelled** its name.

**Python doesn't spell-check, it checks consistency.** If you misspell the name _everywhere_ the same way, the program runs fine:

```python
mesage = "Hello Python Crash Course reader!"
print(mesage)          # works: spelled consistently
```

Tiny one-character typos are among the most common bugs, even for experienced programmers.

### 5. Strings

A **string** is a series of characters. Anything inside quotes is a string, and you can use **single or double** quotes.

```python
"This is a string."
'This is also a string.'
```

The flexibility lets you put quotes and apostrophes _inside_ strings:

```python
'I told my friend, "Python is my favorite language!"'
"The language 'Python' is named after Monty Python, not the snake."
"One of Python's strengths is its diverse and supportive community."
```

### 6. Changing case with methods

A **method** is an action Python can perform on a piece of data. It is written as `variable.method()`: the **dot** tells Python to apply the method to that variable, and the **parentheses** are for any extra information the method needs (often empty).

```python
name = "ada lovelace"
print(name.title())     # Ada Lovelace
```

```python
name = "Ada Lovelace"
print(name.upper())     # ADA LOVELACE
print(name.lower())     # ada lovelace
```

- `title()` → each word starts with a capital letter. Useful because `Ada`, `ADA` and `ada` can all be treated as the same name and displayed as `Ada`.
- `lower()` is especially useful for **storing** data: don't trust the capitalisation users type, convert to lowercase before storing, then display in whichever case makes sense.

### 7. Concatenation (combining strings)

Use **`+`** to join strings. This is called **concatenation**.

```python
first_name = "ada"
last_name = "lovelace"
full_name = first_name + " " + last_name     # note the " " for the space
print(full_name)                              # ada lovelace
```

Use it to build complete messages, and combine it with methods:

```python
print("Hello, " + full_name.title() + "!")   # Hello, Ada Lovelace!
```

You can also store the whole message in a variable first, which keeps the final `print` simple:

```python
message = "Hello, " + full_name.title() + "!"
print(message)
```

### 8. Whitespace: tabs and newlines

**Whitespace** = any non-printing character: spaces, tabs, end-of-line symbols. It helps organise output.

|Sequence|Meaning|
|---|---|
|`\t`|Tab|
|`\n`|Newline (move to next line)|
|`\n\t`|New line, then start it with a tab|

```python
print("\tPython")                          # indented Python
print("Languages:\nPython\nC\nJavaScript") # four lines
print("Languages:\n\tPython\n\tC\n\tJavaScript")  # four lines, items indented
```

### 9. Stripping whitespace

To a program, `'python'` and `'python '` (trailing space) are **two different strings**. This matters when comparing strings, e.g. checking a username at login.

|Method|Removes whitespace from|
|---|---|
|`rstrip()`|Right end|
|`lstrip()`|Left end|
|`strip()`|Both ends|

```python
favorite_language = 'python '
favorite_language.rstrip()        # 'python'   (temporary!)
favorite_language                 # 'python '  (unchanged)
```

The strip is **temporary**. To make it permanent, store the result back in the variable:

```python
favorite_language = favorite_language.rstrip()
```

In real programs, stripping is used most often to **clean user input** before storing it.

### 10. Syntax errors with strings

A **syntax error** means Python doesn't recognise part of your code as valid Python. The classic string case is an **apostrophe inside single quotes**:

```python
message = "One of Python's strengths is its diverse community."   # OK
message = 'One of Python's strengths is its diverse community.'   # SyntaxError: invalid syntax
```

Why: Python reads from the first `'` to the apostrophe as the string, then tries to read the rest as Python code. The error points just after the second single quote. Syntax errors are the **least specific** kind of error, so they can be frustrating to track down.

**Spotting it:** if your editor highlights Python code as if it's English (or English as if it's code), you probably have a mismatched quotation mark.

### 11. Numbers: integers

Operators: `+` `-` `*` `/` and **`**` for exponents**.

```python
>>> 2 + 3
5
>>> 3 / 2
1.5          # division always gives a float in Python 3
>>> 3 ** 2
9
>>> 10 ** 6
1000000
```

Python follows the **order of operations**; use parentheses to override it:

```python
>>> 2 + 3*4
14
>>> (2 + 3) * 4
20
```

Spaces around operators don't affect evaluation; they just help you read priority.

### 12. Numbers: floats

Any number with a **decimal point** is a **float** ("the decimal point can appear at any position"). Mostly they behave as you expect:

```python
>>> 0.1 + 0.1
0.2
>>> 2 * 0.1
0.2
```

But you can sometimes get **extra decimal places**:

```python
>>> 0.2 + 0.1
0.30000000000000004
>>> 3 * 0.1
0.30000000000000004
```

This happens in **all** languages (computers can't always represent decimals exactly). Ignore it for now; there are ways to handle it later.

### 13. Avoiding type errors with `str()`

```python
age = 23
message = "Happy " + age + "rd Birthday!"   # TypeError
```

```
TypeError: Can't convert 'int' object to str implicitly
```

Python can't tell whether `age` means the **number 23** or the **characters "2" and "3"**. Inside a string, you must say explicitly that you want the characters. Wrap it in **`str()`**:

```python
age = 23
message = "Happy " + str(age) + "rd Birthday!"
print(message)       # Happy 23rd Birthday!
```

If numbers behave unexpectedly, check whether Python is treating them as a **number or a string**.

### 14. Comments

A **comment** is a note in English inside your program. In Python it starts with **`#`**; everything after the `#` on that line is ignored.

```python
# Say hello to everyone.
print("Hello Python people!")
```

**Why write them:** explain **what the code is supposed to do and how**. When you come back to a project after time away you'll have forgotten details, and comments summarise your approach faster than re-reading the code.

**Which comments to write:** if you had to consider several approaches before finding one that works, write a comment about your solution. It is easier to delete extra comments later than to go back and add them to a sparsely commented program. Clear, concise comments are one of the best habits for a new programmer, especially for collaboration.

### 15. The Zen of Python

Run `import this` in the interpreter to see Tim Peters' principles for good Python code. The ones the chapter highlights:

|Principle|Meaning|
|---|---|
|Beautiful is better than ugly|Code can be elegant; people notice|
|Simple is better than complex|If both work, pick the simple one|
|Complex is better than complicated|When real life is messy, use the simplest thing that works|
|Readability counts|Even complex code should be readable; comment it|
|One obvious way to do it|Two programmers solving the same problem should reach compatible solutions|
|Now is better than never|Write code that works, then decide whether to improve it. Don't chase perfection|

(Contrast: Perl's motto was "there's more than one way to do it", which made large projects hard to maintain.)

## Mechanics / reference

### String methods in this chapter

|Method|Example|Result|
|---|---|---|
|`title()`|`"ada lovelace".title()`|`Ada Lovelace`|
|`upper()`|`"Ada".upper()`|`ADA`|
|`lower()`|`"Ada".lower()`|`ada`|
|`rstrip()`|`"python ".rstrip()`|`python`|
|`lstrip()`|`" python".lstrip()`|`python`|
|`strip()`|`" python ".strip()`|`python`|

All of these **return a new string**; they don't change the variable unless you reassign.

### Arithmetic operators

|Operator|Meaning|Example|Result|
|---|---|---|---|
|`+`|Add|`2 + 3`|`5`|
|`-`|Subtract|`3 - 2`|`1`|
|`*`|Multiply|`2 * 3`|`6`|
|`/`|Divide|`3 / 2`|`1.5`|
|`**`|Exponent|`3 ** 3`|`27`|

### The three errors you meet in this chapter

|Error|Cause|Fix|
|---|---|---|
|`NameError`|Variable not defined, or misspelled|Assign it first; fix the spelling|
|`SyntaxError`|Python can't parse the code (e.g. apostrophe in single-quoted string)|Fix quoting; use double quotes around text containing `'`|
|`TypeError`|Mixing incompatible types (`str + int`)|Convert with `str()`|

### Python 2 vs Python 3 (only if you read older code)

|Thing|Python 3|Python 2|
|---|---|---|
|Print|`print("Hi")` (it's a function)|`print "Hi"` (no parentheses needed)|
|`3 / 2`|`1.5`|`1` (integer division, remainder simply dropped, not rounded)|
|Get a float in Py2|n/a|Make at least one operand a float: `3.0 / 2` → `1.5`|

### Common patterns

```python
# 1. Clean and normalise user input
name = input_name.strip().title()

# 2. Build a message from variables
message = "Hello, " + name + "!"

# 3. Number inside a string
message = "Happy " + str(age) + "rd Birthday!"

# 4. Make a strip permanent
text = text.rstrip()
```

## Pitfalls

- **Starting a variable name with a number** (`1_message`). Error.
- **Spaces in a variable name** (`greeting message`). Use underscores.
- **Using a built-in name like `print` as a variable.** Avoid reserved words.
- **Misspelling a variable name once** → `NameError`. (Misspelling it _everywhere_ is silently fine.)
- **Apostrophe inside single quotes** → `SyntaxError`. Wrap the string in double quotes.
- **Forgetting that methods don't change the variable.** `name.title()` alone changes nothing; use `name = name.title()` or print it directly.
- **Expecting `rstrip()` to be permanent.** It's temporary until you store the result back.
- **Treating `'python'` and `'python '` as equal.** Extra whitespace makes them different strings.
- **Concatenating a number with a string** (`"Happy " + age`). `TypeError`; use `str(age)`.
- **Forgetting the space in concatenation.** `first_name + last_name` gives `adalovelace`; add `" "`.
- **Panicking about `0.30000000000000004`.** It's normal floating-point behaviour in every language.
- **Assuming `/` gives an integer.** In Python 3 it always gives a float (`3 / 2` → `1.5`). Python 2 differs.
- **Confusing `\t` / `\n` with `/`.** Escape sequences use a backslash.
- **Skipping comments** on tricky code, then forgetting how it works.

## Flashcards

- What is a variable? :: A name associated with a value; the value can be changed at any time #card
- What does the Python interpreter do? :: Reads your program and determines what each word means, then runs it #card
- What is syntax highlighting? :: The editor colouring different parts of code to help you read it and spot errors #card
- What characters may a variable name contain? :: Letters, numbers and underscores only #card
- Can a variable name start with a number? :: No; it may start with a letter or underscore #card
- How do you separate words in a variable name? :: Underscores (`greeting_message`), never spaces #card
- Why avoid naming a variable `print`? :: It's a Python built-in function name, reserved for a purpose #card
- Why are `l` and `O` risky variable names? :: They can be confused with `1` and `0` #card
- Which case should variable names use for now? :: Lowercase #card
- What is a traceback? :: A record of where the interpreter ran into trouble while running your code #card
- What does a `NameError` usually mean? :: The variable was never assigned, or its name is misspelled #card
- Does Python spell-check your variable names? :: No, it only checks that names are used consistently #card
- What is a string? :: A series of characters inside single or double quotes #card
- How do you put an apostrophe inside a string? :: Wrap the string in double quotes: `"Python's"` #card
- What is a method? :: An action Python can perform on a piece of data, called with `variable.method()` #card
- What does `title()` do? :: Makes each word start with a capital letter #card
- What do `upper()` and `lower()` do? :: Convert a string to all uppercase / all lowercase #card
- Why is `lower()` useful for storing data? :: You can't trust users' capitalisation, so normalise before storing #card
- What is concatenation? :: Combining strings with the `+` operator #card
- What does `"Hello, " + name.title() + "!"` do? :: Builds a greeting with the name correctly capitalised #card
- What is whitespace? :: Any non-printing character: spaces, tabs, end-of-line symbols #card
- How do you add a tab to a string? :: `\t` #card
- How do you add a newline to a string? :: `\n` #card
- What does `"\n\t"` do? :: Moves to a new line and starts it with a tab #card
- Why do `'python'` and `'python '` differ? :: Python treats the extra space as significant, so they are different strings #card
- Which methods strip whitespace? :: `rstrip()` (right), `lstrip()` (left), `strip()` (both) #card
- Is `rstrip()` permanent? :: No; store the result back (`x = x.rstrip()`) to make it permanent #card
- What is a syntax error? :: Python doesn't recognise part of your code as valid Python #card
- What's the common string syntax error? :: An apostrophe inside a single-quoted string #card
- What are the arithmetic operators in Python? :: `+`, `-`, `*`, `/`, and `**` for exponents #card
- What does `3 ** 3` give? :: `27` #card
- What does `3 / 2` give in Python 3? :: `1.5` #card
- What is a float? :: Any number with a decimal point #card
- Why does `0.2 + 0.1` show `0.30000000000000004`? :: Computers can't represent some decimals exactly; it happens in all languages #card
- What does `2 + 3*4` give vs `(2 + 3) * 4`? :: `14` vs `20` #card
- Why does `"Happy " + age + "rd"` fail (age = 23)? :: TypeError: Python can't tell if `age` is the number 23 or the characters "23" #card
- How do you fix that error? :: Use `str(age)` #card
- What does `str()` do? :: Represents a non-string value as a string #card
- How do you write a comment in Python? :: Start the line with `#`; everything after it is ignored #card
- Why write comments? :: To explain what code should do and how, saving time when you return to it later #card
- How do you see the Zen of Python? :: Enter `import this` in the interpreter #card
- Who wrote the Zen of Python? :: Tim Peters #card
- What does `3 / 2` give in Python 2? :: `1` (integer division; the remainder is dropped, not rounded) #card
- How do you get a float result from division in Python 2? :: Make at least one number a float, e.g. `3.0 / 2` #card
- How is `print` different in Python 2? :: It's a statement: `print "Hello"` without parentheses #card

## Open questions

- [ ] What is an f-string, and how does it compare to `+` concatenation and `str()`?
- [ ] Why exactly can't computers store `0.1` exactly (floating-point representation)? What is `decimal.Decimal` for?
- [ ] Are strings mutable? What does it mean that methods return a _new_ string?
- [ ] What are Python's keywords, and how do I list them (`keyword.kwlist`)?
- [ ] How do `//` (floor division) and `%` (modulo) work?
- [ ] Triple-quoted strings and multi-line comments/docstrings: how do they differ from `#` comments?

## Key terms

|Term|Definition|
|---|---|
|Interpreter|Program that reads and runs Python code|
|Syntax highlighting|Editor colouring that distinguishes parts of code|
|Variable|Name associated with a value|
|Value|Information attached to a variable|
|Traceback|Record of where the interpreter hit an error|
|`NameError`|Using a name that hasn't been defined|
|`SyntaxError`|Code that isn't valid Python (e.g. mismatched quotes)|
|`TypeError`|Using a value of the wrong type for an operation|
|String|Series of characters inside quotes|
|Method|Action performed on data, called with a dot (`name.title()`)|
|Concatenation|Joining strings with `+`|
|Whitespace|Non-printing characters: spaces, tabs, newlines|
|`\t` / `\n`|Tab / newline escape sequences|
|`strip()` family|Methods that remove leading/trailing whitespace|
|Integer (`int`)|Whole number|
|Float|Number with a decimal point|
|`**`|Exponent operator|
|`str()`|Function that converts a value to a string|
|Comment|`#` note ignored by Python, written for humans|
|Zen of Python|Tim Peters' guiding principles (`import this`)|

## Try it yourself (chapter exercises)

- [ ] 2-1 Simple Message: store a message in a variable and print it
- [ ] 2-2 Simple Messages: print a message, change the variable, print the new one
- [ ] 2-3 Personal Message: store a name, print a greeting to that person
- [ ] 2-4 Name Cases: print a name in lowercase, uppercase and titlecase
- [ ] 2-5 Famous Quote: print a quote and its author, including quotation marks
- [ ] 2-6 Famous Quote 2: same, using `famous_person` and `message` variables
- [ ] 2-7 Stripping Names: name with `\t`/`\n` whitespace; print it raw, then with `lstrip()`, `rstrip()`, `strip()`
- [ ] 2-8 Number Eight: four operations (add, subtract, multiply, divide) that each print `8`
- [ ] 2-9 Favorite Number: store it in a variable, build a message with `str()`, print it
- [ ] 2-10 Adding Comments: add at least one comment to two programs
- [ ] 2-11 Zen of Python: run `import this` and skim the rest

## Related

→ Next: [[INTRODUCTION TO LIST]]