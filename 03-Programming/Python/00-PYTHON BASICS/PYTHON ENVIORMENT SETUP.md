---
title: Python Environment Setup
date: 2026-07-28
language: Python
phase: Phase 0 · Setup
tags:
  - python
status: learning
---
> **One-line summary:** Getting Python running on your own machine comes down to the same five steps everywhere — check what's installed, install what's missing, wire up a text editor to the right interpreter, save and run a one-line hello_world.py, and troubleshoot if it doesn't fire — only the exact commands change between Linux, macOS, and Windows.

## Core Idea

Python is cross-platform, but "cross-platform" doesn't mean "identical setup." Three things vary chapter to chapter as you move between operating systems: whether Python ships pre-installed, which command actually invokes it (`python` vs `python3`), and how your editor is told to use that command. Two extra wrinkles sit underneath all of this:

- **Python 2 vs Python 3.** Both still exist in the wild. Code written for one doesn't always run cleanly on the other, but for a beginner the differences are minor. Default to Python 3 if you're installing fresh; if a system only has Python 2, it's fine to start there and upgrade once you're comfortable.
- **Snippets vs saved programs.** Typing code directly at a `>>>` prompt in a terminal session is for trying out small isolated ideas — nothing is saved. Writing a `.py` file in an editor and running it is how you'll build anything real. Both use the same interpreter underneath.

## Structure

_(the setup sequence is identical across every OS — only the commands inside each step change)_

```
+------------------------+
|1. Check Python version |
+------------------------+
            |
            v
+------------------------+
|2. Install if missing   |
+------------------------+
            |
            v
+------------------------+
|3. Install text editor  |
+------------------------+
            |
            v
+------------------------+
|4. Point editor at      |
|   python3 interpreter  |
+------------------------+
            |
            v
+------------------------+
|5. Save & run           |
|   hello_world.py       |
+------------------------+
            |
            v
+------------------------+
|6. Troubleshoot if it   |
|   doesn't run          |
+------------------------+
```

## Analogy

Setting up a Python environment is like prepping a car before a road trip. Checking your Python version is checking the fuel gauge — you need to know what you've already got before doing anything else. Installing Python is filling the tank. The text editor is your dashboard — Geany or Sublime Text just gives you dials and buttons instead of a bare engine bay. Pointing the editor's Build/Execute commands at the right interpreter path is making sure the dashboard is actually wired to _this_ engine and not a different one sitting in the garage — get that wrong (wrong path, mismatched capitalization) and every gauge reads nothing, even though the engine itself is fine.

## Mechanics / reference

**Checking and installing Python, by OS**

| |Linux|macOS|Windows|
|---|---|---|---|
|Usually pre-installed?|Yes|Yes|No|
|Check version|`python` then `python3` in a terminal|`python` then `python3` in Terminal|`python` in a Command Prompt|
|If missing / need 3|`sudo apt-get install` route, or system package manager|Rarely needed — check `python3` first|Download installer from python.org, check **Add Python to PATH**|
|Recommended editor|Geany (`sudo apt-get install geany`)|Sublime Text|Geany|

**`python` vs `python3` command**

|Situation|What it means|What to do|
|---|---|---|
|`python` opens a `>>>` prompt showing 2.x|Python 2 is the system default|Try `python3` next — most systems now have both|
|`python3` also works|Both versions are installed|Use `python3` everywhere in this book from now on|
|`python` returns "not recognized" (Windows)|Python isn't on PATH|Find the actual install folder (e.g. `C:\Python35\`) and use the full path, or reinstall with **Add Python to PATH** checked|

## Worked example

**Linux terminal check (Ubuntu):**

```
$ python
Python 2.7.6 (default, Mar 22 2014, 22:59:38)
>>>
$ python3
Python 3.5.0 (default, Sep 17 2015, 13:05:18)
>>>
```

Both exist here, so `python3` becomes the standing command for the rest of the book.

**macOS — confirming the real interpreter path before touching editor settings:**

```
$ type -a python3
python3 is /usr/local/bin/python3
```

This is the path that goes straight into Sublime Text's custom build system (`"cmd": ["/usr/local/bin/python3", "-u", "$file"]`) — skipping this step is the single most common reason a "Build" command silently fails.

**Windows — the PATH failure mode:**

```
C:\> python
'python' is not recognized as an internal or external command,
operable program or batch file.
```

This means Windows has no idea where the interpreter lives. Locating the install folder (e.g. `C:\Python35\python`) and testing that full path first, before configuring the editor around it, is the fix.

![[Figure 1-1.png]] _Geany's Set Build Commands dialog — Compile and Execute both explicitly call `python3`._

![[Figure 1-2.png]] _The Windows installer's Add Python to PATH checkbox — easy to miss, and the direct cause of the "not recognized" error above if left unchecked._

![[Figure 1-3.png]] _The same Geany dialog on Windows, with the full install-folder path (`C:/Python35/python`) baked into every command since PATH wasn't configured._

## When to use it

Any time you're starting Python work on a machine (or VM, or container) you haven't set up before — including switching between a personal Linux box, a work Windows laptop, and a Mac, where muscle memory for one OS's commands will actively mislead you on another.

## Why it matters for security

|Concept|Attacker's perspective|Defender's perspective|
|---|---|---|
|PATH resolution when running `python`/`python3`|If a malicious script named `python3` sits in a directory earlier in PATH than the real interpreter, it runs instead — silently|Confirm the real path with `type -a python3` (or `where python` on Windows) before trusting the command, especially on shared or unfamiliar machines|
|Installer trust chain|A tampered or fake "Python installer" downloaded from a spoofed site can run arbitrary code with the privileges you grant it|Download only from python.org / official OS package managers, and avoid running installers with elevated privileges you don't need|

## Pitfalls

- Assuming `python` means Python 3 — on many systems it's still 2.x, and only `python3` is guaranteed to be the newer version
- Mismatched spacing or capitalization in a Build/Execute command — the book calls this out explicitly, since editors won't warn you, they'll just fail to run
- Forgetting to check **Add Python to PATH** during the Windows installer, which produces the "not recognized" error afterward
- Configuring the editor's build command before confirming the real interpreter path — easy to do the steps out of order and debug the wrong thing

## Flashcards

- #card What's the safe first command to check for Python 3 on a system where `python` shows 2.x? >> `python3`
- #card On macOS/Linux, what command reveals the full path to the real `python3` binary? >> `type -a python3`
- #card What Windows installer checkbox avoids the "'python' is not recognized" error? >> Add Python to PATH
- #card What's the practical difference between typing code at a `>>>` prompt and saving a `.py` file? >> The prompt is for quick, unsaved experiments; a saved file is a real, rerunnable program — both run on the same interpreter

## Try It Yourself

- Spend a few minutes browsing python.org itself so the site becomes familiar territory to come back to later.
- Deliberately introduce a typo into `hello_world.py` and rerun it — try to produce both a hard error and a typo that runs without complaint, and think through why some mistakes surface as errors and others don't.
- Jot down three programs worth building eventually — starting an "ideas notebook" now gives the rest of this learning path a concrete target.

## Questions I still have

- [ ] Once virtual environments (venv) enter the picture, do these same Build/Execute path configurations need to change, or does venv activation handle that transparently?
- [ ] Is there a more robust fix for the PATH-hijacking risk than manually checking `type -a python3` every time — e.g. shell config that pins a known-good path?
- [ ] For Windows specifically, is there a reason the installer doesn't default "Add Python to PATH" to checked, given how common that failure mode is?

## Key terms

|Term|Definition|
|---|---|
|REPL|The interactive `>>>` prompt session where Python code runs and prints results immediately, without saving a file|
|PATH|The ordered list of directories a shell searches through to find the program behind a typed command like `python3`|
|Build system|An editor's saved configuration for which command compiles/runs the current file — what Geany's "Set Build Commands" and Sublime's custom build files both are|
|`python_work`|The book's naming convention for the folder holding all your practice programs — lowercase, underscored, per Python naming conventions|

## Related

[[]]

---

→ Next: [[Variables and Simple Data Types]]