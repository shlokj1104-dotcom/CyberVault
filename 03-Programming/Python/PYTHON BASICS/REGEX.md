---
title: REGEX
date: 2026-05-20
phase: phase-1
tags:
  - concept
  - python
links: []
status: learning
---
# REGULAR EXPRESSIONS (REGEX)

## One-Line Summary

**Regular expressions** are a _mini programming language for pattern matching_ — you describe the **shape** of the text you're looking for using special characters, and Python's **`re` module** finds, extracts, or replaces it in any string.

---

## PART 1 — WHY REGEX EXISTS

### The Problem Without Regex

Before regex, to find something like a phone number in a string you'd write a clunky function like `isPhoneNumber()` — checking each character individually, checking length, checking hyphens. That function is 17 lines and can only match **one specific format** (e.g. `415-555-4242`). What about `(415) 555-4242` or `415.555.4242`? You'd need to rewrite the whole thing.

> Think of regex as **a search bar that understands patterns** — instead of typing an exact word, you describe the rule: "a number that looks like three digits, a dash, three digits, a dash, four digits."

### The Power of Regex

- **Pattern-first thinking:** describe structure, not exact content.
- **10× shorter code:** `\d{3}-\d{3}-\d{4}` replaces the 17-line phone checker.
- **Extract, match, AND replace** — all with the same tool.
- **Built into Python's standard library:** no install needed (`import re`).

> **Java comparison:** Java has `java.util.regex.Pattern` and `.Matcher`. Python's `re` module works similarly but is less ceremonial — no type declarations, no `Pattern.compile()` chaining.

---

## PART 2 — THE `re` MODULE BASICS

### The 3-Step Workflow

Every regex operation in Python follows the same three steps:

```python
import re

# Step 1: Compile the pattern into a Regex object
phoneNumRegex = re.compile(r'\d\d\d-\d\d\d-\d\d\d\d')

# Step 2: Call .search() on the Regex with the string you want to scan
mo = phoneNumRegex.search('My number is 415-555-4242.')

# Step 3: Call .group() on the Match object to get the matched text
print(mo.group())   # 415-555-4242
```

**Why the `r` before the string?** The `r` marks a **raw string** — backslashes are treated literally, not as escape characters. Without it, `\d` would be interpreted as an invalid escape sequence. Always use raw strings with regex.

```python
# Normal string:  '\n' = newline character
# Raw string:     r'\n' = backslash + n (two characters)

# Regex needs literal backslashes, so always write:
re.compile(r'\d{3}')    # ✅ correct
re.compile('\\d{3}')    # ✅ also works but ugly — avoid
```

### `search()` vs `findall()` — Critical Difference

|Method|Returns|Use when|
|---|---|---|
|`search()`|A **Match object** for the **first** match (or `None`)|You just want to know if the pattern exists|
|`findall()`|A **list** of all matching strings|You want every match|

```python
import re

phoneNumRegex = re.compile(r'\d\d\d-\d\d\d-\d\d\d\d')

# search() — returns Match object for FIRST match
mo = phoneNumRegex.search('Cell: 415-555-9999 Work: 212-555-0000')
print(mo.group())   # 415-555-9999   ← only the first one!

# findall() — returns LIST of ALL matches
lst = phoneNumRegex.findall('Cell: 415-555-9999 Work: 212-555-0000')
print(lst)   # ['415-555-9999', '212-555-0000']   ← both!
```

> ⚠️ **`search()` returns `None` if nothing found.** Always check before calling `.group()`, or you'll get an `AttributeError`:

```python
mo = phoneNumRegex.search('No phone here')
if mo:
    print(mo.group())
else:
    print('Not found')
```

---

## PART 3 — CHARACTER MATCHING

### The Most Important Special Characters

|Character|Meaning|Example Match|
|---|---|---|
|`.`|Any single character (except newline)|`r'F..m:'` matches `From:`, `Fxxm:`, `F12m:`|
|`^`|Start of line / start of string|`r'^From'` matches only if line starts with `From`|
|`$`|End of line / end of string|`r'java$'` matches only if line ends with `java`|
|`*`|Zero or more of preceding character|`r'Bat(wo)*man'` matches `Batman`, `Batwoman`, `Batwowowowoman`|
|`+`|One or more of preceding character|`r'Bat(wo)+man'` matches `Batwoman` but NOT `Batman`|
|`?`|Zero or one of preceding group (optional)|`r'Bat(wo)?man'` matches both `Batman` and `Batwoman`|
|`\`|Escape special characters|`r'\.'` matches a literal period|
|`|`|OR — match one of several alternatives|

> **Mnemonic for `^` and `$`:** "Carrots cost dollars" — caret comes first (beginning), dollar comes last (end).

### Quantifiers — Controlling Repetition

```python
# ? = optional (0 or 1)
batRegex = re.compile(r'Bat(wo)?man')
# Matches: 'Batman', 'Batwoman'
# No match: 'Batwowoman'

# * = zero or more
batRegex = re.compile(r'Bat(wo)*man')
# Matches: 'Batman', 'Batwoman', 'Batwowowowoman'

# + = one or more (MUST appear at least once)
batRegex = re.compile(r'Bat(wo)+man')
# Matches: 'Batwoman', 'Batwowoman'
# No match: 'Batman' ← fails because wo must appear at least once

# Curly brackets = exact or range
haRegex = re.compile(r'(Ha){3}')       # exactly 3 Ha's → 'HaHaHa'
haRegex = re.compile(r'(Ha){3,5}')     # 3 to 5 Ha's
haRegex = re.compile(r'(Ha){3,}')      # 3 or more
haRegex = re.compile(r'(Ha){,5}')      # 0 to 5
```

### Greedy vs Non-Greedy

Python's regex is **greedy by default** — it matches the **longest possible** string. Add `?` after a quantifier to make it **non-greedy** (shortest match):

```python
greedyHaRegex    = re.compile(r'(Ha){3,5}')    # greedy
nongreedyHaRegex = re.compile(r'(Ha){3,5}?')   # non-greedy

mo1 = greedyHaRegex.search('HaHaHaHaHa')
print(mo1.group())   # 'HaHaHaHaHa'   ← grabbed all 5

mo2 = nongreedyHaRegex.search('HaHaHaHaHa')
print(mo2.group())   # 'HaHaHa'       ← grabbed the minimum 3
```

> ⚠️ **`?` has TWO different meanings in regex:**
> 
> 1. After a group `(wo)?` → optional match (0 or 1 times)
> 2. After a quantifier `{3,5}?` or `*?` or `+?` → non-greedy mode These meanings are completely unrelated.

---

## PART 4 — GROUPS AND EXTRACTION

### Grouping with Parentheses

Parentheses in a regex serve two purposes at once:

1. **Group** part of the pattern for quantifiers to apply to
2. **Extract** just that part when using `.group()` or `findall()`

```python
phoneNumRegex = re.compile(r'(\d\d\d)-(\d\d\d-\d\d\d\d)')
mo = phoneNumRegex.search('My number is 415-555-4242.')

mo.group(1)   # '415'         ← first set of parentheses
mo.group(2)   # '555-4242'    ← second set of parentheses
mo.group(0)   # '415-555-4242'  ← the whole match
mo.group()    # '415-555-4242'  ← same as group(0)

# Get all groups at once as a tuple:
areaCode, mainNumber = mo.groups()   # multiple assignment trick!
print(areaCode)    # 415
print(mainNumber)  # 555-4242
```

**To match a literal parenthesis** (not a group), escape it:

```python
re.compile(r'\(\d\d\d\)')   # matches (415)
```

### `findall()` with Groups — Returns List of Tuples

When the regex **has groups**, `findall()` returns a list of **tuples** (one per match, one string per group):

```python
# No groups → list of strings
phoneNumRegex = re.compile(r'\d\d\d-\d\d\d-\d\d\d\d')
phoneNumRegex.findall('Cell: 415-555-9999 Work: 212-555-0000')
# ['415-555-9999', '212-555-0000']

# With groups → list of TUPLES
phoneNumRegex = re.compile(r'(\d\d\d)-(\d\d\d)-(\d\d\d\d)')
phoneNumRegex.findall('Cell: 415-555-9999 Work: 212-555-0000')
# [('415', '555', '9999'), ('212', '555', '0000')]
```

### The Pipe `|` — OR Matching

```python
heroRegex = re.compile(r'Batman|Tina Fey')

mo1 = heroRegex.search('Batman and Tina Fey')
print(mo1.group())   # 'Batman'   ← first match wins

mo2 = heroRegex.search('Tina Fey and Batman')
print(mo2.group())   # 'Tina Fey'

# Combine pipe with grouping — share a common prefix
batRegex = re.compile(r'Bat(man|mobile|copter|bat)')
mo = batRegex.search('Batmobile lost a wheel')
print(mo.group())    # 'Batmobile'
print(mo.group(1))   # 'mobile'   ← just the group part
```

---

## PART 5 — CHARACTER CLASSES

### Shorthand Character Classes

These are the most important ones — memorise them:

|Shorthand|Represents|Opposite|
|---|---|---|
|`\d`|Any digit `[0-9]`|`\D` = any non-digit|
|`\w`|Any letter, digit, or underscore `[a-zA-Z0-9_]`|`\W` = opposite|
|`\s`|Any space, tab, or newline|`\S` = any non-whitespace|
|`.`|Any character except newline (wildcard)|`\.` = literal period|
|`\b`|Word boundary (empty string at start/end of word)|`\B` = non-boundary|

```python
# \d+ = one or more digits, \s = whitespace, \w+ = one or more word chars
xmasRegex = re.compile(r'\d+\s\w+')
xmasRegex.findall('12 drummers, 11 pipers, 10 lords, 9 ladies')
# ['12 drummers', '11 pipers', '10 lords', '9 ladies']
```

### Custom Character Classes with `[ ]`

Square brackets let you define your own set of characters to match:

```python
# Match any vowel (upper or lower)
vowelRegex = re.compile(r'[aeiouAEIOU]')
vowelRegex.findall('RoboCop eats baby food.')
# ['o', 'o', 'e', 'a', 'a', 'o', 'o']

# Range shorthand inside brackets
re.compile(r'[a-zA-Z0-9]')   # all letters and digits

# Negative character class — ^ INSIDE brackets inverts it
consonantRegex = re.compile(r'[^aeiouAEIOU]')   # anything NOT a vowel
# matches 'R', 'b', 'c', 'p', ' ', 't', 's', ...
```

> 💡 **Key insight:** Inside `[ ]`, special characters lose their special meaning. You don't need to escape `.`, `*`, `?`, or `()` inside brackets. `[0-5.]` matches digits 0–5 **and** a literal period — no backslash needed.

> ⚠️ **`^` has TWO different meanings:**
> 
> 1. At the start of a regex `^From` → anchors to beginning of string
> 2. At the start inside brackets `[^aeiou]` → negates the class

---

## PART 6 — ANCHORS AND THE WILDCARD

### `^` and `$` — Start and End Anchors

```python
# ^ anchors to beginning of string
beginsWithHello = re.compile(r'^Hello')
beginsWithHello.search('Hello world!')      # Match ✅
beginsWithHello.search('He said hello.')    # None ❌

# $ anchors to end of string
endsWithNumber = re.compile(r'\d$')
endsWithNumber.search('Your number is 42') # Match ✅
endsWithNumber.search('forty two')         # None ❌

# Both together → entire string must match
wholeStringIsNum = re.compile(r'^\d+$')
wholeStringIsNum.search('1234567890')   # Match ✅
wholeStringIsNum.search('12345xyz890')  # None ❌
```

### The Wildcard `.` and Dot-Star `.*`

```python
# . matches ANY single character (except newline)
atRegex = re.compile(r'.at')
atRegex.findall('The cat in the hat sat on the flat mat.')
# ['cat', 'hat', 'sat', 'lat', 'mat']
# Note: 'flat' only matched 'lat' because . is ONE character

# .* = match EVERYTHING (dot-star) — greedy
nameRegex = re.compile(r'First Name: (.*) Last Name: (.*)')
mo = nameRegex.search('First Name: Al Last Name: Sweigart')
mo.group(1)   # 'Al'
mo.group(2)   # 'Sweigart'

# .*? = match everything NON-GREEDY
nongreedy = re.compile(r'<.*?>')
greedy    = re.compile(r'<.*>')
text = '<To serve man> for dinner.>'

nongreedy.search(text).group()   # '<To serve man>'   ← stops at first >
greedy.search(text).group()      # '<To serve man> for dinner.>'   ← grabs all
```

### Matching Newlines with `re.DOTALL`

By default, `.` does NOT match newlines. To make it match everything including newlines, pass `re.DOTALL` as a flag:

```python
noNewlineRegex = re.compile('.*')
newlineRegex   = re.compile('.*', re.DOTALL)

text = 'Serve the public trust.\nProtect the innocent.\nUphold the law.'

noNewlineRegex.search(text).group()
# 'Serve the public trust.'   ← stopped at newline

newlineRegex.search(text).group()
# 'Serve the public trust.\nProtect the innocent.\nUphold the law.'
```

---

## PART 7 — EXTRACTING DATA WITH `findall()`

### Basic `findall()` Usage

```python
import re

s = 'A message from csev@umich.edu to cwen@iupui.edu about meeting @2PM'
lst = re.findall(r'\S+@\S+', s)
print(lst)   # ['csev@umich.edu', 'cwen@iupui.edu']
```

Breaking down `\S+@\S+`:

- `\S+` = one or more non-whitespace characters (before `@`)
- `@` = literal at-sign
- `\S+` = one or more non-whitespace characters (after `@`)

### Cleaning Up Dirty Matches

Sometimes results contain junk characters. Use a more precise pattern with `[ ]`:

```python
# Messy version (may grab <source@email.org>;)
re.findall(r'\S+@\S+', line)

# Cleaner version — must start AND end with letter or digit
re.findall(r'[a-zA-Z0-9]\S*@\S*[a-zA-Z]', line)
```

### Combining Search and Extract

Use groups `( )` inside `findall()` to extract just the part you care about:

```python
import re
hand = open('mbox-short.txt')

# Extract only the hour from "From: stephen@uct.ac.za Sat Jan 5 09:14:16 2008"
for line in hand:
    line = line.rstrip()
    x = re.findall(r'^From .* ([0-9][0-9]):', line)
    if len(x) > 0:
        print(x)
# ['09']
# ['18']
# ['16']
# ...
```

Breaking down `^From .* ([0-9][0-9]):`:

- `^From` = line must start with "From "
- `.*` = any characters (the email + date parts)
- `([0-9][0-9])` = capture two digits (the hour) — the parentheses extract this
- `:` = followed by a colon

Without the parentheses, `findall()` returns the whole match. With them, it returns only what's inside the group.

---

## PART 8 — SUBSTITUTION WITH `sub()`

### Basic `sub()` Usage

`sub()` finds all matches and replaces them with a new string:

```python
namesRegex = re.compile(r'Agent \w+')
result = namesRegex.sub('CENSORED', 'Agent Alice gave documents to Agent Bob.')
print(result)   # 'CENSORED gave documents to CENSORED.'
```

### Using Matched Groups in the Replacement

Use `\1`, `\2`, etc. in the replacement string to insert the text from capture groups:

```python
# Censor agent names but keep their first letter
agentNamesRegex = re.compile(r'Agent (\w)\w*')
result = agentNamesRegex.sub(r'\1****', 'Agent Alice told Agent Carol that Agent Eve knew Agent Bob.')
print(result)   # 'A**** told C**** that E**** knew B****.'
```

`\1` in the replacement refers to whatever was captured by group 1 `(\w)` — i.e., the first letter.

---

## PART 9 — REGEX FLAGS

### Using Flags as Second Argument to `re.compile()`

|Flag|Short Form|Effect|
|---|---|---|
|`re.IGNORECASE`|`re.I`|Case-insensitive matching|
|`re.DOTALL`|—|`.` matches newlines too|
|`re.VERBOSE`|`re.X`|Allow whitespace and comments in regex|

```python
# Case-insensitive
robocop = re.compile(r'robocop', re.I)
robocop.search('RoboCop is part man, part machine.').group()   # 'RoboCop'
robocop.search('ROBOCOP protects the innocent.').group()       # 'ROBOCOP'

# Combining flags with | (bitwise OR)
someRegex = re.compile('foo', re.IGNORECASE | re.DOTALL | re.VERBOSE)
```

### `re.VERBOSE` — Readable Complex Regexes

For long patterns, `re.VERBOSE` lets you add whitespace and `#` comments inside the regex string (spaces are ignored, `#` starts a comment):

```python
# Without VERBOSE — hard to read:
phoneRegex = re.compile(r'((\d{3}|\(\d{3}\))?(\s|-|\.)?\d{3}(\s|-|\.)\d{4}(\s*(ext|x|ext.)\s*\d{2,5})?)')

# With VERBOSE — same pattern, readable:
phoneRegex = re.compile(r'''(
    (\d{3}|\(\d{3}\))?        # area code
    (\s|-|\.)?                # separator
    \d{3}                     # first 3 digits
    (\s|-|\.)                 # separator
    \d{4}                     # last 4 digits
    (\s*(ext|x|ext.)\s*\d{2,5})?  # extension
    )''', re.VERBOSE)
```

---

## PART 10 — ESCAPE CHARACTER

Regular expressions have special characters like `^`, `$`, `.`, `*` etc. To match them **literally**, prefix with a backslash:

```python
import re

x = 'We just received $10.00 for cookies.'
y = re.findall(r'\$[0-9.]+', x)
print(y)   # ['$10.00']

# Without the backslash: \$ matches a literal dollar sign
# [0-9.] matches digits OR a period (inside brackets, . loses special meaning)
```

**Characters that need escaping outside brackets:** `. ^ $ * + ? { } [ ] \ | ( )`

---

## PART 11 — SUMMARY TABLE OF ALL SYMBOLS

|Symbol|Meaning|
|---|---|
|`^`|Start of string / line|
|`$`|End of string / line|
|`.`|Any character except newline|
|`\s`|Whitespace character (space, tab, newline)|
|`\S`|Non-whitespace character|
|`\d`|Digit (0–9)|
|`\D`|Non-digit|
|`\w`|Word character (letter, digit, underscore)|
|`\W`|Non-word character|
|`\b`|Word boundary|
|`*`|Zero or more of preceding group|
|`*?`|Zero or more (non-greedy)|
|`+`|One or more of preceding group|
|`+?`|One or more (non-greedy)|
|`?`|Zero or one of preceding group (optional)|
|`??`|Zero or one (non-greedy)|
|`{n}`|Exactly n repetitions|
|`{n,}`|n or more repetitions|
|`{,m}`|0 to m repetitions|
|`{n,m}`|n to m repetitions (greedy)|
|`{n,m}?`|n to m repetitions (non-greedy)|
|`[abc]`|Any character in the set (a, b, or c)|
|`[^abc]`|Any character NOT in the set|
|`[a-z0-9]`|Range of characters|
|`(abc)`|Group — captures matched text|
|`\1`, `\2`|Backreference to group 1, 2 in sub()|
|`a|b`|
|`\`|Escape the next special character|

---

## PART 12 — REAL PROJECT: PHONE + EMAIL EXTRACTOR

This is the capstone project from _Automate the Boring Stuff_ — a program that scans clipboard text and extracts all phone numbers and email addresses.

```python
#! python3
# phoneAndEmail.py - Finds phone numbers and email addresses on the clipboard.

import pyperclip, re

# PHONE NUMBER REGEX (verbose for readability)
phoneRegex = re.compile(r'''(
    (\d{3}|\(\d{3}\))?          # area code (optional) — plain or in parens
    (\s|-|\.)?                  # separator (optional)
    (\d{3})                     # first 3 digits
    (\s|-|\.)                   # separator
    (\d{4})                     # last 4 digits
    (\s*(ext|x|ext.)\s*\d{2,5})? # extension (optional)
    )''', re.VERBOSE)

# EMAIL ADDRESS REGEX
emailRegex = re.compile(r'''(
    [a-zA-Z0-9._%+-]+           # username
    @                           # @ symbol
    [a-zA-Z0-9.-]+              # domain name
    (\.[a-zA-Z]{2,4})           # dot-something (.com, .org, .co.uk)
    )''', re.VERBOSE)

# GET TEXT FROM CLIPBOARD
text = str(pyperclip.paste())

# FIND ALL MATCHES
matches = []

for groups in phoneRegex.findall(text):
    phoneNum = '-'.join([groups[1], groups[3], groups[5]])
    if groups[8] != '':
        phoneNum += ' x' + groups[8]
    matches.append(phoneNum)

for groups in emailRegex.findall(text):
    matches.append(groups[0])

# COPY RESULTS TO CLIPBOARD
if len(matches) > 0:
    pyperclip.copy('\n'.join(matches))
    print('Copied to clipboard:')
    print('\n'.join(matches))
else:
    print('No phone numbers or email addresses found.')
```

---

## PART 13 — DEBUGGING REGEX

### Use Python's `help()` and `dir()`

```python
>>> import re
>>> dir(re)
[..., 'compile', 'findall', 'match', 'purge', 'search', 'split', 'sub', 'subn', ...]

>>> help(re.search)
# search(pattern, string, flags=0)
#     Scan through string looking for a match to the pattern...
```

### Use regex101.com

Paste your pattern and test string at **https://regex101.com** — it shows exactly which part of the string each group matches, with colour coding.

### Common Debugging Mistakes

**1. Forgetting the `r` prefix:**

```python
re.compile('\d{3}')    # ❌ \d is not a valid escape sequence in normal strings
re.compile(r'\d{3}')   # ✅ always use raw strings
```

**2. `search()` returns `None` — calling `.group()` crashes:**

```python
mo = re.compile(r'\d+').search('no digits here')
print(mo.group())   # ❌ AttributeError: 'NoneType' has no attribute 'group'

# ✅ Always check first:
if mo:
    print(mo.group())
```

**3. Confusing `search()` and `findall()`:**

```python
# search() returns a Match object (use .group())
# findall() returns a LIST of strings (no .group() needed)

mo = re.compile(r'\d+').search('12 and 34')
print(mo.group())              # '12'   — only first match

lst = re.compile(r'\d+').findall('12 and 34')
print(lst)                     # ['12', '34']   — all matches
```

**4. Forgetting that `.` doesn't match newlines by default:**

```python
re.compile(r'.*').search('line1\nline2').group()                    # 'line1'
re.compile(r'.*', re.DOTALL).search('line1\nline2').group()         # 'line1\nline2'
```

**5. Greedy matching eating more than intended:**

```python
re.compile(r'<.*>').search('<tag1> text <tag2>').group()    # '<tag1> text <tag2>'
re.compile(r'<.*?>').search('<tag1> text <tag2>').group()   # '<tag1>'
# When in doubt, use .*? (non-greedy)
```

**6. `^` and `[^...]` are completely different:**

```python
re.compile(r'^hello')     # line must START with 'hello'
re.compile(r'[^hello]')   # match any char that is NOT h, e, l, o
```

---

## Glossary

|Term|Definition|
|---|---|
|**regular expression (regex)**|A pattern string using special characters to describe what text to match|
|**`re` module**|Python's built-in regex library|
|**`re.compile()`**|Takes a regex string and returns a Regex object|
|**Regex object**|Compiled pattern — has `.search()`, `.findall()`, `.sub()` methods|
|**Match object**|Returned by `.search()` — has `.group()`, `.groups()` methods|
|**`search()`**|Find first match → returns Match object or `None`|
|**`findall()`**|Find all matches → returns list of strings or tuples|
|**`sub()`**|Find and replace → returns modified string|
|**raw string**|`r'...'` — backslashes are literal, not escape chars|
|**group**|A portion of a pattern inside `( )` — captured separately|
|**greedy matching**|Default — matches the longest possible string|
|**non-greedy**|`?` after quantifier — matches the shortest possible string|
|**wildcard**|`.` — matches any single character except newline|
|**character class**|`[abc]` or `\d`, `\w`, `\s` — a set of acceptable characters|
|**negative class**|`[^abc]` — matches any character NOT in the set|
|**anchor**|`^` (start) or `$` (end) — locks match to position in string|
|**pipe**|`|
|**`re.IGNORECASE`**|Flag to make matching case-insensitive|
|**`re.DOTALL`**|Flag to make `.` match newlines too|
|**`re.VERBOSE`**|Flag to allow whitespace and comments in regex string|
|**brittle code**|Code that works only for one exact format and breaks on variations|
|**`grep`**|Unix command-line tool that uses regex — Generalized Regular Expression Parser|

---

## Java vs Python — Regex

|Concept|Java|Python|
|---|---|---|
|Import|`import java.util.regex.*;`|`import re`|
|Compile|`Pattern p = Pattern.compile(r"\\d+");`|`p = re.compile(r'\d+')`|
|Match object|`Matcher m = p.matcher(str);`|`mo = p.search(str)`|
|Get matched text|`m.group()`|`mo.group()`|
|Find all|Loop with `m.find()`|`p.findall(str)`|
|Replace|`m.replaceAll("new")`|`p.sub('new', str)`|
|Raw string|`"\\d+"` (double backslash)|`r'\d+'` (r-prefix)|
|Ignore case|`Pattern.CASE_INSENSITIVE`|`re.IGNORECASE`|

---

## Quick Reference

```python
# ══ BASIC WORKFLOW ═══════════════════════════════════════════════  # Python

import re

regex = re.compile(r'your_pattern_here')  # Step 1: compile
mo    = regex.search('string to scan')   # Step 2: search (first match)
lst   = regex.findall('string to scan')  # Step 2 alt: all matches
if mo:
    print(mo.group())    # Step 3: get matched text


# ══ COMMON PATTERNS ══════════════════════════════════════════════

re.compile(r'\d+')                  # one or more digits
re.compile(r'\d{3}-\d{4}')         # phone number format
re.compile(r'[a-zA-Z]+')            # one or more letters
re.compile(r'\S+@\S+')             # email-like pattern
re.compile(r'^From:')              # lines starting with 'From:'
re.compile(r'[0-9]+\.[0-9]+')      # decimal numbers

# ══ GROUPS ═══════════════════════════════════════════════════════

regex = re.compile(r'(\d{3})-(\d{4})')
mo = regex.search('555-1234')
mo.group(1)      # '555'
mo.group(2)      # '1234'
mo.groups()      # ('555', '1234')

# ══ FINDALL WITH GROUPS → TUPLES ════════════════════════════════

regex = re.compile(r'(\d{3})-(\d{4})')
regex.findall('555-1234 and 777-9999')
# [('555', '1234'), ('777', '9999')]

# ══ SUBSTITUTION ════════════════════════════════════════════════

regex = re.compile(r'Agent (\w)\w*')
regex.sub(r'\1****', 'Agent Alice and Agent Bob')
# 'A**** and B****'

# ══ FLAGS ════════════════════════════════════════════════════════

re.compile(r'hello', re.I)                          # case-insensitive
re.compile(r'.*', re.DOTALL)                        # dot matches newline
re.compile(r'pattern # comment', re.VERBOSE)        # allow comments
re.compile(r'x', re.I | re.DOTALL | re.VERBOSE)     # combine flags

# ══ USEFUL ONE-LINERS ════════════════════════════════════════════

# Extract all email addresses from text:
re.findall(r'[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,4}', text)

# Extract all URLs:
re.findall(r'https?://\S+', text)

# Check if entire string is a number:
bool(re.compile(r'^\d+$').search(s))

# Replace multiple spaces with one:
re.sub(r' +', ' ', text)
```

---

## Practice Exercises

**Basic:**

- [ ] Use `re.search()` to find lines containing `'From:'` in a text file. Print matching lines.
- [ ] Modify the above to only match `'From:'` at the **start** of a line using `^`.
- [ ] Write a regex to match lines that contain an `@` sign somewhere after `From:`.

**Extraction:**

- [ ] Extract all email addresses from a string using `re.findall()`.
- [ ] Extract only the **domain** part (after `@`) of each email.
- [ ] Extract the **hour** from lines of the form `From stephen@uct.ac.za Sat Jan 5 09:14:16 2008`.

**Combining Search + Extract:**

- [ ] Write a regex that finds lines starting with `X-` and extracts the floating-point number at the end.
- [ ] Write a program that reads a file and counts lines matching `^Author`.
- [ ] Write a program that finds lines matching `New Revision:` followed by numbers, extracts all revision numbers, and prints their average as an integer.

**Advanced:**

- [ ] Build a regex that matches US phone numbers in ALL of these formats: `415-555-4242`, `(415) 555-4242`, `415.555.4242`, `415 555 4242`, `555-4242` (no area code).
- [ ] Use `re.sub()` to censor all occurrences of agent names (words following "Agent") in a string, keeping only the first letter.
- [ ] Build the Phone + Email extractor clipboard program from Part 12.

---

## Questions I Still Have

_Write your open questions here. Return later and answer them._

---

## Related Notes

- [[OBJECT-ORIENTED PROGRAMMING (OOP)]]
- [[FUNCTIONS]]
- [[FILE HANDLING]]
- [[STRINGS]]
- [[ITERATIVE STATEMENTS]]