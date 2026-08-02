---
title: PYTHON ENVIORMENT SETUP
date: 2026-07-29
language: Python
phase: Phase 0 · Setup
tags:
  - python
links: []
status: learning
---
> [!summary] `pip` = an app store for Python (downloads and installs packages). `venv` = a separate toybox per project so packages don't mix between projects. `requirements.txt` = a shopping list of what to reinstall. `.env` = a locked diary for secrets — totally separate from venv, easy to mix up because the names sound alike.

## 1. pip — installing packages

Think of pip as an app store built into Python. Some tools (`math`, `os`) already come with Python — no install needed. Anything else (like a package to talk to a website, or read Excel files) you get through pip.

```
py -m pip install <package>                  # download this package and add it to Python, like installing an app
py -m pip install <package>==<version>       # install one exact version, not whatever is newest right now
py -m pip install --upgrade <package>        # update it to the newest version (or -U for short)
py -m pip uninstall <package>                # remove it, like uninstalling an app
py -m pip list                               # show every package you currently have installed
py -m pip show <package>                     # show one package's info card: version, where it lives, what it needs
py -m pip install --upgrade pip              # this one updates the app-store app itself, not a package
```

**Why type `py -m pip` instead of just `pip`?** A computer can sometimes have more than one copy of Python on it. Typing `pip` alone can accidentally talk to the wrong copy — so the package gets installed somewhere your code can't find it. `py -m pip` says "use the pip that belongs to _this exact_ Python I'm about to run" — no mix-up.

By default, everything you install goes onto one shared shelf that every Python project on your computer can see. That's the problem `venv` solves, below.

## 2. Virtual environments (venv) — a separate toybox per project

> [!example] Analogy Imagine every project gets its own toybox instead of sharing one big shelf. Project A can keep an old toy (package version) in its box while Project B keeps a brand-new one in its box — they never bump into each other. Without separate boxes, updating a toy for one project could break a different project that needed the old one.

```
py -m venv .venv       # create a new empty toybox for this project, in a folder called .venv
```

This makes a `.venv/` folder with its own private copy of Python's package shelf inside it. Anything you install _while inside this box_ only affects this project.

**Step into the box (activate) — Windows PowerShell, your setup:**

```
.venv\Scripts\Activate.ps1     # step into the .venv toybox — installs now go here, not the shared shelf
```

**Windows cmd (if you're not using PowerShell):**

```
.venv\Scripts\activate.bat     # same thing, cmd.exe version
```

**Git Bash / WSL / Linux / macOS:**

```
source .venv/Scripts/activate     # same thing, Git Bash on Windows
source .venv/bin/activate         # same thing, Linux/macOS (folder is bin, not Scripts, here)
```

**Step back out (deactivate) — same everywhere:**

```
deactivate      # leave the toybox, go back to the shared shelf
```

> [!note] PowerShell gotcha Sometimes PowerShell refuses to run `Activate.ps1` and shows a red "execution policy" error. This just means PowerShell is being extra cautious about running scripts. Fix it for the current window only by running: `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` This doesn't change any permanent setting — just allows scripts for this one terminal session.

When you're "inside the box," your terminal prompt shows `(.venv)` at the start of the line. That's the only visible sign — always check for it before installing something.

## 3. requirements.txt — a shopping list for your toybox

> [!example] Analogy `pip freeze` writes down everything currently sitting in your toybox onto a shopping list. `pip install -r` reads that list somewhere else and buys everything on it, rebuilding the same toybox from scratch.

```
py -m pip freeze > requirements.txt      # write down every package + version currently installed, into this file
py -m pip install -r requirements.txt    # read that file and install everything on it
```

Why this matters: you never upload the `.venv/` folder itself to GitHub — it's huge, and a toybox built on Windows won't even work on Linux. Instead, you upload just the _list_ (`requirements.txt`). Anyone else (or future you, on a new laptop) makes their own empty toybox and runs the install command to rebuild an identical one.

## 4. .env — a locked diary for secrets

Stores things you don't want anyone else to see: API keys, passwords, database logins. This has nothing to do with `venv` — venv keeps _packages_ separate, `.env` keeps _secrets_ separate. Easy to confuse because of the similar name.

```
py -m pip install python-dotenv      # install a small helper package that knows how to read .env files
```

```python
from dotenv import load_dotenv
import os

load_dotenv()                        # open the .env file and load everything in it into memory
api_key = os.environ["API_KEY"]      # grab one specific secret by the name you gave it in .env
```

## 5. .gitignore — "don't pack these when mailing the box"

> [!example] Analogy When you upload a project to GitHub, `.gitignore` is your packing list of things to leave behind — junk you can regenerate, or secrets you don't want mailed out with the package.

```
.venv/           # leave the whole toybox behind — it's rebuildable from requirements.txt
.env              # leave your secrets diary behind — never mail this out
__pycache__/      # leave Python's auto-generated scratch files behind
*.pyc              # leave any leftover compiled files behind
```

Common habit: also make a `.env.example` file with fake placeholder values, and _do_ upload that one — it tells others "you'll need an API_KEY and a DB_PASSWORD here" without giving away your real ones.

## 6. pipx — tools you want everywhere, not just in one project

```
py -m pip install pipx      # install pipx itself, once
pipx install black          # install "black" (a code formatter) as a tool you can run from anywhere
```

Regular `pip` inside a venv installs things _for one project_. `pipx` is for things you want available _everywhere_, like a screwdriver you keep in your main toolbox instead of packing a spare into every single toybox. Each tool still gets kept separate behind the scenes, but the command works globally.

## 7. Managing Python versions — the py launcher

```
py --list                 # show every Python version installed on this computer
py -3.11 -m venv .venv    # make a toybox using Python 3.11 specifically, not whatever the default is
```

You're already using `py` to run your code — this is the same tool, just telling it "use this specific version" when a project needs one.

## Quick comparison

|Tool|What it's for, in plain terms|
|---|---|
|`pip`|the app store — install/remove packages|
|`venv`|a separate toybox per project|
|`requirements.txt`|the shopping list to rebuild a toybox|
|`.env`|the locked diary for secrets|
|`pipx`|install a tool once, use it from any project|

## Security notes (why this matters for you specifically)

|Risk|In plain terms|What to do|
|---|---|---|
|Uploading `.env` by accident|Your passwords/keys become visible to anyone who can see the GitHub repo, forever in its history|Put `.env` in `.gitignore` _before_ your first commit; share an `.env.example` instead|
|Installing a package without checking|Someone can name a fake package almost identically to a real one, hoping you mistype and install theirs instead|Double check the exact spelling and that it has real downloads/maintainers before installing|
|Skipping venv, installing everything globally|One project's update can quietly break a totally different project|Always make a venv per project|
|Never updating packages|Old versions can have known security holes|Check with the commands below occasionally|

```
py -m pip install pip-audit    # install a tool that checks your packages for known security holes
py -m pip-audit                # run the check — it lists anything risky it finds
```

## The full workflow, start to finish

```
py -m venv .venv
        |
        v
   step into .venv               (prompt shows "(.venv)")
        |
        v
py -m pip install <packages>
        |
        v
py -m pip freeze > requirements.txt
        |
        v
        ... do your work, upload requirements.txt + .env.example, NOT .venv/ or .env ...
        |
        v
   deactivate
```

## 8. How a typical project folder is laid out

```
project/
├── .venv/                    [never uploaded]
├── .env                      [never uploaded]
├── .gitignore
├── README.md
├── LICENSE
├── requirements.txt
├── requirements-dev.txt
├── pyproject.toml
├── src/
│   └── package_name/
│       ├── __init__.py
│       └── module.py
└── tests/
    └── test_module.py
```

**Do you need the `src/` folder?** Only once a project grows into something with several files that import each other. For a single script, just keep it as a plain file in the folder — don't build a whole house for one toy.

**README.md** is basically a note taped to the front of the box explaining what's inside: what the project does, how to set it up, how to run it. First thing anyone (including future you, six months from now) reads.

## 9. pyproject.toml — the project's info card

A single file that describes the project: its name, what it needs to run, and settings for other tools. It's slowly replacing older, messier setup files. You won't need one for a quick script, but you'll start seeing it in most repos and tutorials, so it's worth recognizing.

## 10. Two shopping lists: what to run it vs. what to work on it

> [!example] Analogy `requirements.txt` is the list of ingredients the recipe actually needs. `requirements-dev.txt` is your list of kitchen tools (thermometer, extra bowls) — helpful while cooking, but not something you'd hand to someone just eating the meal.

```
# requirements.txt        <- what the program needs to actually run
requests==2.30.0

# requirements-dev.txt    <- extra tools just for you, while building it
-r requirements.txt
pytest
black
mypy
```

```
py -m pip install -r requirements-dev.txt    # installs your dev tools AND everything requirements.txt needs
```

## 11. VS Code needs to be told which toybox to use, separately

Activating `.venv` in the terminal doesn't automatically tell VS Code's editor (the part that gives you autocomplete and underlines errors) to look in that same box. If you see red squiggly lines under imports that should work: `Ctrl+Shift+P` → **Python: Select Interpreter** → pick the one inside `.venv`.

## 12. Editable installs — "watch this folder live"

```
py -m pip install -e .    # install your own project as a package, but linked to your live source files
```

Normally, installing a package copies a snapshot of it. `-e` (editable) instead links directly to your working folder — so when you edit your own code, the change is instantly "installed," no reinstalling needed. Useful once your project has multiple files that import from each other.

## 13. Pre-commit hooks — a bag check before you leave the house

`.gitignore` only stops files from being uploaded _if you remembered to list them first_. If a `.env` accidentally gets uploaded once, it's stuck in the project's history forever, even if you delete it in a later update.

```
py -m pip install pre-commit    # install a tool that runs automatic checks before every commit
```

A pre-commit hook is like a bag check at the door before you leave the house — it looks at what you're about to upload and blocks it if it spots something that looks like a password or key, catching the mistake before it ever leaves your computer. Set up once per project with a `.pre-commit-config.yaml` file, then run `pre-commit install`.

## 14. Other names you'll run into

|Tool|What it does differently|
|---|---|
|**Poetry**|Does the job of pip + venv together, as one tool|
|**Conda**|Similar idea, but can also install non-Python software (not just PyPI packages) — common in data science|

pip + venv is the default everyone assumes you're using, and what most tutorials teach first — fine to stick with it. Just recognize these other names when you see them.

## Try this

- [ ] Make a venv, step into it, confirm `(.venv)` shows up in your PowerShell prompt
- [ ] Install one package, run `pip freeze`, then step out (`deactivate`) and confirm `pip list` outside the box doesn't show it
- [ ] Write a `.gitignore` with `.venv/`, `.env`, `__pycache__/` before your very first commit on a new project
- [ ] Run `pip-audit` on an existing project and see what it flags

## Key terms, in plain words

- **PyPI** — the actual app store pip downloads from
- **site-packages** — the "shelf" where installed packages physically sit
- **venv** — your project's own private toybox
- **requirements.txt** — the shopping list to rebuild a toybox elsewhere
- **.env** — the locked diary for secrets, unrelated to venv
- **pipx** — installs a tool once, usable from any project
- **execution policy** — PowerShell's caution setting that can block running `.ps1` scripts
- **pyproject.toml** — the project's info card / settings file
- **editable install** — links your live code instead of copying a snapshot of it
- **pre-commit hook** — an automatic bag check before every commit
- **src-layout** — keeping your package's code inside a `src/` folder, for bigger projects