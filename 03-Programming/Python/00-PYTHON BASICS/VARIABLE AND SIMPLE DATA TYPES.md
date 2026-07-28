---
title: VARIABLE AND SIMPLE DATA TYPES
date: 2026-07-28
language: Python
phase: Phase 0 · Setup
tags:
  - python
links: []
status: learning
---
> **One-line summary:** Python variables are just labels bound to a value with no type declaration required, strings are quoted character sequences with a set of built-in methods for case and whitespace handling, and numbers split into ints and floats that Python won't silently mix with strings — you have to convert explicitly with `str()`; on top of that, every name has to dodge 35 reserved keywords, every expression obeys a strict PEMDAS-style precedence, and every value has a type you can check directly with `type()`.

## Core Idea

When you run `hello_world.py`, the editor hands the file to the Python interpreter, which reads it top to bottom and executes each line as it goes — that's true whether the file has one line or a hundred. Adding a variable doesn't change this process, it just gives the interpreter something to remember between lines: `message = "Hello Python world!"` binds the name `message` to that string, and `print(message)` looks the name up and displays whatever it currently points to.

Every line the interpreter runs is a **statement** — a unit of code it can execute. `print(...)` is an _expression statement_: it evaluates something and shows a result. `message = "..."` is an _assignment statement_: it creates or updates a binding and produces no visible output at all. A script is just a sequence of statements executed one at a time.

Four building blocks follow from that:

- **Variables** — no type keyword, no declaration step. A name is just created the first time you assign to it, and it can point to a completely different type of value later without complaint. `type(x)` always tells you what a variable currently refers to.
- **Strings** — any text inside quotes, single or double. Python doesn't care which one you use as long as it's consistent, and gives you methods for common cleanup jobs (case, whitespace, joining).
- **Numbers** — ints and floats behave like you'd expect arithmetically, but Python refuses to auto-convert between numbers and strings, unlike languages that concatenate anything to a string without asking.
- **Keywords** — 35 reserved words the interpreter uses to recognize the structure of a program (`if`, `class`, `for`, `import`, etc.). They can never be used as variable names, no exceptions, no workarounds.

## Structure

_(what actually happens when the interpreter processes a two-line program)_

```
+------------------------------+
|Read hello_world.py           |
+------------------------------+
              |
              v
+------------------------------+
|Line 1: message = "..."       |
|  -> bind name to string      |
+------------------------------+
              |
              v
+------------------------------+
|Line 2: print(message)        |
|  -> look up variable,        |
|     display its value        |
+------------------------------+
```

_(order in which Python evaluates a mixed expression — PEMDAS)_

```
+----------------------------+
|1. Parentheses (innermost) |
+----------------------------+
              |
              v
+----------------------------+
|2. Exponentiation  **      |
+----------------------------+
              |
              v
+----------------------------+
|3. Multiply / Divide       |
+----------------------------+
              |
              v
+----------------------------+
|4. Add / Subtract          |
+----------------------------+
              |
              v
+----------------------------+
|Same precedence: left->right|
+----------------------------+
```

## Analogy

A Python variable is a sticky note stuck on a storage box, not a labeled, fixed-size compartment. The sticky note (`message`) doesn't care what's inside the box — today it might say "Hello Python world!", tomorrow you can peel it off and stick it on a box holding the number `23` instead, and Python won't stop you. Java makes you build a differently-shaped compartment for each type up front (`String message = ...` vs `int age = ...`); Python just moves the sticky note wherever you point it.

Keywords are the sticky-note equivalent of a parking spot that already has the building manager's name painted on it — `class`, `for`, `if`, and 32 others. You can read them, you can park _near_ them, but you cannot ever write your own name over one and call it your variable.

## Mechanics / reference

**Java vs Python — variables and typing**

|Concept|Java|Python|
|---|---|---|
|Declaring a variable|`String message = "Hi";` — type, name, value|`message = "Hi"` — name, value, no type keyword|
|Reassigning to a different type|Not allowed without a cast|Freely allowed — `message = 5` just works|
|Mixing a number into a string|`"Happy " + age + "rd!"` auto-converts `age` to text|Raises `TypeError` — must wrap as `str(age)`|
|Changing string case|`str.toUpperCase()` / `.toLowerCase()`|`.upper()` / `.lower()` / `.title()`|

**String methods quick reference**

|Method|What it does|Example|
|---|---|---|
|`.title()`|Capitalizes each word|`"ada lovelace".title()` → `Ada Lovelace`|
|`.upper()` / `.lower()`|Forces all caps / all lowercase|`"Ada".upper()` → `ADA`|
|`.rstrip()` / `.lstrip()` / `.strip()`|Removes trailing / leading / both-side whitespace|`"python ".rstrip()` → `"python"`|
|`\t` / `\n` inside a string|Inserts a tab / newline where it appears|`"a\tb"` prints `a b`|

**String operations — concatenation and repetition**

|Operator|What it does on strings|Example|
|---|---|---|
|`+`|Concatenation — joins two strings end to end (not addition)|`'100' + '150'` → `'100150'`|
|`*`|Repetition — repeats a string content by an integer count|`'Test ' * 3` → `'Test Test Test '`|

`+` between two strings requires **both** operands to already be strings — mixing an `int` in raises the same `TypeError` covered below. `*` requires one string operand and one `int` operand.

**Operators quick reference (PEMDAS precedence, high → low)**

|Operator|Meaning|Example|
|---|---|---|
|`()`|Parentheses — force evaluation order|`(2 + 3) * 4` → `20`|
|`**`|Exponent|`3 ** 2` → `9`|
|`* / // %`|Multiply, divide, floor-divide, modulus|`7 // 3` → `2`, `7 % 3` → `1`|
|`+ -`|Add, subtract|`3 / 2` → `1.5`|

Operators sharing the same precedence level evaluate strictly **left to right** — `5 - 3 - 1` is `1`, not `3`, because `5 - 3` happens first.

**Modulus operator (`%`)** Works only on integers and returns the _remainder_ of dividing the first operand by the second — `7 % 3` is `1`, because `7 // 3` is `2` with `1` left over. Two practical uses worth remembering:

- Divisibility check: `x % y == 0` means `x` is evenly divisible by `y`.
- Digit extraction: `x % 10` gives the right-most digit of `x`; `x % 100` gives the last two digits.

**Variable naming rules**

- Can be arbitrarily long; can contain letters, digits, and underscores.
- Cannot start with a digit (`76trombones` → `SyntaxError`).
- Cannot contain other punctuation (`more@` → `SyntaxError`).
- Cannot be one of Python's 35 reserved keywords (`class` → `SyntaxError`, since it's a keyword, not just a bad name).
- Are case-sensitive: `LaTeX` and `latex` are two completely different names.
- Convention: start with lowercase; reserve a leading underscore for library code.

**Python's 35 reserved keywords**

```
False   await   else    import  pass
None    break   except  in      raise
True    class   finally is      return
and     continue for     lambda try
as      def     from    nonlocal while
assert  del     global  not     with
async   elif    if      or      yield
```

None of these can ever be a variable name — the interpreter needs them to parse program structure.

**Getting input from the user**

|Call|Behavior|
|---|---|
|`input()`|Pauses execution, waits for Enter, returns what was typed **as a string**, always|
|`input('prompt text\n')`|Displays the prompt first, `\n` puts the typed input on its own line|
|`int(some_input)`|Converts the string to an `int` — raises `ValueError` if it isn't a clean digit string|

`input()` never returns anything but a `str` — even `"17"` from digit-only input has to be explicitly converted with `int()` or `float()` before it can be used in arithmetic.

**Comments** Start with `#` — everything from `#` to the end of the line is ignored by the interpreter and has zero effect on execution. Valid on their own line or trailing after code. Useful comments explain _why_ something is done, not _what_ the code obviously already says; a comment restating the code (`v = 5 # assign 5 to v`) is dead weight, while one adding context the code can't express (`v = 5 # velocity in meters/second`) earns its place.

## Worked example

**The TypeError from mixing an int into a string, and the fix:**

```python
age = 23
message = "Happy " + age + "rd Birthday!"
# TypeError: can only concatenate str (not "int") to str

message = "Happy " + str(age) + "rd Birthday!"
print(message)
# Happy 23rd Birthday!
```

Python knows `age` could mean the number 23 or the two characters "2" and "3" — `str()` tells it explicitly which one you mean.

**The floating-point quirk worth knowing before it surprises you mid-program:**

```python
>>> 0.2 + 0.1
0.30000000000000004
```

This isn't a Python bug — it's how binary floating point represents decimals in every mainstream language. Worth remembering once you start comparing floats for exact equality later on.

**A semantic error that produces no error message at all — commas in big numbers:**

```python
>>> print(1,000,000)
1 0 0
```

Python doesn't read `1,000,000` as one big integer with thousands separators — it reads a comma-separated sequence of three separate integers (`1`, `0`, `0`), and `print()` happily displays all three with spaces between. The code runs cleanly; it just doesn't do the "right" thing.

**A semantic error from misreading order of operations — no crash, just the wrong number:**

```python
>>> 1.0 / 2.0 * pi
```

This is meant to compute `1/(2π)` but Python evaluates division first (same precedence as multiplication, left to right), so it actually computes `(1.0 / 2.0) * pi`, which is `pi / 2` — a different value entirely, with no error to flag the mistake. Parenthesize explicitly whenever intent isn't obvious from precedence alone.

**A debugging session showing three distinct error types:**

```python
>>> 76trombones = 'big parade'
SyntaxError: invalid syntax          # starts with a digit

>>> more@ = 1000000
SyntaxError: invalid syntax          # illegal character @

>>> class = 'Advanced Theoretical Zymurgy'
SyntaxError: invalid syntax          # 'class' is a keyword

>>> bad name = 5
SyntaxError: invalid syntax          # space splits it into two tokens

>>> principal = 327.68
>>> interest = principle * rate
NameError: name 'principle' is not defined   # typo: 'principle' vs 'principal'
```

The `NameError` case is the important one to internalize: the typo itself doesn't fail — only the _later reference_ to the misspelled name does, which is why the traceback points at line 2, not line 1.

**Choosing mnemonic variable names — three functionally identical programs:**

```python
a = 35.0
b = 12.50
c = a * b
print(c)

hours = 35.0
rate = 12.50
pay = hours * rate
print(pay)

x1q3z9ahd = 35.0
x1q3z9afd = 12.50
x1q3p9afd = x1q3z9ahd * x1q3z9afd
print(x1q3p9afd)
```

The interpreter treats all three identically. Humans don't: `hours`/`rate`/`pay` communicates intent instantly, `a`/`b`/`c` communicates nothing, and the third version actively obscures intent. "Mnemonic" just means memory aid — the whole point of the name is to help the _next reader_ (often future-you) remember why the variable exists.

**Asking for input and converting it, including the failure case:**

```python
>>> prompt = 'What...is the airspeed velocity of an unladen swallow?\n'
>>> speed = input(prompt)
What...is the airspeed velocity of an unladen swallow?
17
>>> int(speed)
17
>>> int(speed) + 5
22

>>> speed = input(prompt)
What...is the airspeed velocity of an unladen swallow?
What do you mean, an African or a European swallow?
>>> int(speed)
ValueError: invalid literal for int() with base 10: 'What do you mean, an African or a European swallow?'
```

## When to use it

Any program past `hello_world.py` needs variables the moment you want to reuse a value instead of retyping it, strings the moment you're building any user-facing message, and explicit `str()`/numeric conversion the moment a program takes numeric input that will end up inside printed text. Reach for `input()` whenever a program needs a runtime value from the user, and treat its return value as a plain string until you've explicitly converted it. Add comments only where the _why_ isn't obvious from the code itself — resist commenting what the code already says clearly. Favor mnemonic names for anything beyond a throwaway script, but don't over-engineer them into something a beginner might mistake for a keyword.

## Why it matters for security

|Concept|Attacker's perspective|Defender's perspective|
|---|---|---|
|Whitespace/case normalization|The book's own example is directly relevant: two usernames that differ only by trailing whitespace or case (`"admin "` vs `"admin"`) could be treated as different accounts if a login system compares raw strings — opening the door to lookalike or duplicate accounts|Always normalize identifiers with `.strip()` and `.lower()` (or equivalent) before comparing or storing them, exactly as these string methods are built to do|
|Unchecked type coercion in concatenation|A program that blindly concatenates unvalidated input into a string (instead of validating/converting first) is the same habit that, in more sensitive contexts, leads to command or query injection|Convert and validate explicitly (as `str()` forces you to do here) rather than assuming a value's type, and prefer parameterized approaches over raw concatenation once the output feeds something like a shell command or database query|
|Trusting `input()`'s type|Code that assumes user input is already numeric (skipping `int()`/validation) fails loudly with `ValueError` today — but in less obviously broken code, unvalidated input is exactly the seam attackers probe first|Treat every `input()` result as untrusted text by default; convert and validate explicitly, and plan for the conversion to fail rather than assuming clean data|
|Comments as unverified claims|A comment describing behavior that the code no longer matches — whether from bit rot or a deliberately misleading edit — is invisible to the interpreter and easy for a reviewer to trust over the actual logic|Treat comments as documentation to verify against the code during review, never as a substitute for reading what the code actually does|
|Reserved keyword collisions|Naive input-driven code generation or templating that doesn't account for Python's 35 reserved words can let user-supplied text collide with language syntax|Know the exact keyword list when building anything that turns user input into identifiers or generated code, and validate against it|

## Pitfalls

- A single misspelled variable name (`mesage` vs `message`) produces a `NameError`, not a warning — Python doesn't spellcheck, it just fails on lookup, and only at the point of _use_, not at the point of the typo
- Concatenating a number directly into a string without `str()` raises a `TypeError`
- Mismatched quote types around an apostrophe (`'Python's strengths...'`) causes a `SyntaxError`, since Python reads everything up to the _next_ matching quote as the string
- `.rstrip()` / `.lstrip()` / `.strip()` don't modify the variable in place — the stripped result has to be reassigned back, or the original whitespace is still there next time you check
- Lowercase `l` and uppercase `O` in variable names are easy to misread as `1` and `0`
- Variable names can't start with a digit, can't contain illegal characters like `@`, can't contain spaces, and can't match any of the 35 reserved keywords — all produce the same unhelpful `SyntaxError: invalid syntax`
- Variable names are case-sensitive: `LaTeX` and `latex` are different bindings, a common source of silent bugs
- Order-of-operations mistakes (e.g. `1.0 / 2.0 * pi` instead of `1.0 / (2.0 * pi)`) raise no error at all — they just silently compute the wrong value, which is harder to catch than a crash
- Typing a large number with comma thousands-separators (`1,000,000`) is legal syntax but means something completely different — a 3-tuple of small integers, not one big number
- `input()` always returns a `str`, even for pure-digit input — forgetting the `int()`/`float()` conversion is a common source of a downstream `TypeError`

## Flashcards

- #card What error does Python raise when you reference a variable name that was never assigned (often from a typo)? >> `NameError`
- #card What error does Python raise when you try to concatenate an `int` directly into a `str`? >> `TypeError`
- #card Which string method removes whitespace from both ends without touching the middle? >> `.strip()`
- #card Why does `0.1 + 0.2` print `0.30000000000000004` instead of `0.3`? >> Binary floating-point can't represent most decimals exactly — a universal quirk across languages, not a Python bug
- #card What error results from a variable name that starts with a digit, contains an illegal character, or matches a reserved keyword? >> `SyntaxError: invalid syntax`
- #card How many reserved keywords does Python have, and can any of them be used as a variable name? >> 35, and no — none of them can ever be a variable name
- #card What type does `input()` always return, no matter what the user types? >> `str` (a string)
- #card Which operator returns the remainder of integer division in Python? >> `%` (modulus)
- #card What does PEMDAS represent in Python's order of operations, and what happens when two operators tie in precedence? >> Parentheses, Exponents, Multiply/Divide, Add/Subtract — same-precedence operators evaluate left to right
- #card What character starts a Python comment, and how far does it extend? >> `#`, extending to the end of that line only
- #card Why is `1.0 / 2.0 * pi` (meant to compute 1/(2π)) a semantic error rather than a syntax error? >> It runs with no error message but computes `pi/2` instead, because division and multiplication share precedence and evaluate left to right
- #card What does "mnemonic" mean in the context of variable naming? >> A memory aid — a name chosen to help you (or another reader) remember what the variable is for

## Try It Yourself

- Store a message in a variable and print it (`simple_message.py`).
- Store a message in a variable, print it, then reassign the variable to a new message and print that too (`simple_messages.py`).
- Store a name in a variable and print it in lowercase, uppercase, and titlecase.
- Store a greeting to a specific person using their name inside the message.
- Find a quote from someone you admire and print it attributed to them, quotation marks included.
- Repeat the quote exercise, but store the person's name in one variable and the composed message in a separate variable before printing.
- Store a name with extra whitespace and `\t`/`\n` characters at both ends, print it as-is, then print it again after applying `lstrip()`, `rstrip()`, and `strip()`.
- Deliberately trigger each illegal-name `SyntaxError` (a name starting with a digit, one with an illegal character, one that's a reserved keyword) to see the exact error text for each.
- Write a short script that uses `input()` to collect two numbers, converts both with `int()`, and prints their sum — then break it on purpose by typing non-numeric text and read the resulting `ValueError`.
- Rewrite the same short calculation three ways — single-letter names, mnemonic names, and deliberately obfuscated names — and time how long each takes to read back and explain out loud.

## Questions I still have

- [ ] Is there a more readable alternative to wrapping every number in `str()` for output (f-strings, `.format()`) that a later chapter covers?
- [ ] Beyond `.strip()` and `.lower()`, what else counts as proper normalization for something like a username or email before comparing it — is casefold() different from lower() in edge cases?
- [ ] Does Python have a built-in way to compare floats safely (tolerance-based) instead of running into the `0.1 + 0.2` problem directly?
- [ ] Beyond raising `ValueError`, what's the idiomatic way to validate `input()` before converting it — does a later chapter cover `try`/`except` for this?
- [ ] Is left-to-right evaluation of same-precedence operators ever something I'd actually need to reason about in real code, or is it mostly relevant to floating-point edge cases like the `0.1 + 0.2` quirk?

## Key terms

|Term|Definition|
|---|---|
|Variable|A name bound to a value; in Python the binding can be reassigned to a different type at any time|
|String|An immutable sequence of characters, written inside single or double quotes|
|Method|A function attached to a value, called with dot notation (e.g. `name.title()`)|
|Traceback|The interpreter's report of where and why a program failed to run|
|Concatenation|Joining strings together, in Python done with `+`|
|Float|A number containing a decimal point; subject to binary floating-point precision limits|
|Statement|A unit of code the interpreter can execute — e.g. an assignment or a print expression statement|
|Expression|A combination of values, variables, and operators that evaluates to a single result|
|Operator|A special symbol representing a computation, like `+` or `*`|
|Operand|A value an operator is applied to|
|Keyword|One of Python's 35 reserved words; cannot be used as a variable name|
|Mnemonic|A memory aid — used to describe variable names chosen to be self-explanatory|
|Evaluate|To simplify an expression by performing its operations to produce a single value|
|Modulus operator|`%` — yields the remainder when the first operand is divided by the second|
|Rules of precedence|The ordering (PEMDAS-style) that determines which operator in an expression evaluates first|
|Type|A category of value — the types covered so far are `int`, `float`, and `str`|
|Value|A basic unit of data, like a number or string, that a program works with|

## Related

---

→ Next: [[Lists]]