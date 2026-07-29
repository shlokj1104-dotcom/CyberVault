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
> [!summary] `pip` installs packages into a Python interpreter. `venv` gives each project its own isolated interpreter+packages so installs don't collide across projects. `requirements.txt` records what to reinstall. `.env` stores secrets — unrelated mechanism to venv, easy to conflate because the names look similar.

## 1. pip

Installs third-party packages from PyPI that aren't in the standard library (unlike `math`, `os`, etc., which ship with Python).

```
py -m pip install <package>                  # download and install a package
py -m pip install <package>==<version>       # install one specific version, not just the latest
py -m pip install --upgrade <package>        # replace an installed package with its newest version (or -U)
py -m pip uninstall <package>                # remove an installed package
py -m pip list                               # show every package installed in this interpreter
py -m pip show <package>                     # show details for one package: version, location, dependencies
py -m pip install --upgrade pip              # update pip itself, not a project package
```

**Why `py -m pip` instead of bare `pip`:** on a machine with multiple Python installs, a bare `pip` on PATH can silently belong to the wrong interpreter, so the package installs somewhere your current `python`/`py` won't see it. `py -m pip` forces pip to run _as a module of the interpreter you just invoked_ — install and interpreter always match.

By default, everything installs into the _global_ site-packages of that interpreter. That's the problem venv solves.

## 2. Virtual environments (venv)

```
py -m venv .venv       # create a new isolated environment in a folder named .venv
```

Creates a `.venv/` folder containing its own isolated copy of the interpreter + a private `site-packages`. Packages installed while it's active don't touch the global install and don't leak into other projects.

**Activation (Windows PowerShell — your setup):**

```
.venv\Scripts\Activate.ps1     # switch this terminal session into the .venv environment
```

**Windows cmd:**

```
.venv\Scripts\activate.bat     # same switch, cmd.exe syntax
```

**Git Bash / WSL / Linux / macOS:**

```
source .venv/Scripts/activate     # same switch, Git Bash on Windows
source .venv/bin/activate         # same switch, Linux/macOS (note: bin, not Scripts)
```

**Deactivate (same on all):**

```
deactivate      # switch back to the global interpreter, leave .venv
```

> [!note] PowerShell gotcha If `Activate.ps1` gets blocked with an execution-policy error, run once per session: `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` This only affects the current shell process, not the whole system.

When active, your prompt shows `(.venv)` — that's your only visual confirmation you're not installing into the global interpreter.

> [!example] Analogy Global install = one shared garage toolbox for every project in the house — swap a wrench (package version) for one project and every other project feels it. venv = each project gets its own toolbox. Nothing leaks between them.

## 3. requirements.txt

```
py -m pip freeze > requirements.txt      # snapshot exact installed versions
py -m pip install -r requirements.txt    # reinstall that exact snapshot elsewhere
```

You never push `.venv/` to GitHub — it's large and platform-specific (a Windows `.venv` won't run on Linux). Instead you commit `requirements.txt`; anyone cloning the repo runs `venv` + `install -r` to rebuild an equivalent environment locally.

## 4. .env

Stores secrets/config: API keys, DB credentials, tokens. Unrelated to venv — venv isolates _code execution_, `.env` isolates _configuration_. Typically read via a library:

```
py -m pip install python-dotenv      # library that reads a .env file into your program
```

```python
from dotenv import load_dotenv
import os

load_dotenv()                        # reads .env in the project root and loads it into the environment
api_key = os.environ["API_KEY"]      # fetch one value by the name defined in .env
```

## 5. .gitignore — the piece that ties it together

```
.venv/
.env
__pycache__/
*.pyc
```

`.venv/` is excluded because it's rebuildable junk (that's what `requirements.txt` is for). `.env` is excluded because it's a secrets leak waiting to happen. A common pattern: commit a `.env.example` with dummy keys so teammates know what variables are needed, without exposing real ones.

## 6. pipx — for CLI tools, not project dependencies

```
py -m pip install pipx
pipx install black
```

`pip` inside a venv is for _project dependencies_. `pipx` is for _global command-line tools_ (formatters, linters, `httpie`, etc.) — each tool gets its own isolated environment automatically, but the command is available globally, without polluting your system Python's site-packages.

## 7. Managing Python versions (Windows py launcher)

```
py --list                 # show installed Python versions
py -3.11 -m venv .venv    # create a venv with a specific version
```

You're already using `py` as your launcher, so this is the same tool — just pass a version flag when you need a non-default interpreter for a project.

## Comparison table

|Tool|Scope|Purpose|
|---|---|---|
|`pip`|interpreter-wide|install/remove packages|
|`venv`|per-project folder|isolate an interpreter + its packages|
|`requirements.txt`|file, committed to git|reproduce an environment elsewhere|
|`.env`|file, gitignored|store secrets/config, unrelated to venv|
|`pipx`|system-wide, per-tool isolated|install CLI tools without touching project envs|

## Security implications

|Risk|Why it matters|Mitigation|
|---|---|---|
|Committing `.env` to git|Leaks API keys/credentials, often permanently in history|`.gitignore` it; commit `.env.example` instead|
|Installing from PyPI without checking|Typosquatting/supply-chain attacks (malicious package with a similar name)|Double-check spelling, maintainer, download counts before installing|
|Global (non-venv) installs|Version conflicts can silently break unrelated projects or system tools|Default to venv per project|
|Stale dependencies|Known CVEs sit unpatched in installed packages|`py -m pip list --outdated`; `pip-audit` (below)|

```
py -m pip install pip-audit
py -m pip-audit          # scans installed packages against known vulnerability DBs
```

## Workflow

```
py -m venv .venv
        |
        v
   activate .venv               (prompt shows "(.venv)")
        |
        v
py -m pip install <packages>
        |
        v
py -m pip freeze > requirements.txt
        |
        v
        ... work, commit requirements.txt + .env.example, NOT .venv/ or .env ...
        |
        v
   deactivate
```

## 8. Project structure convention

```
project/
├── .venv/                    [gitignored]
├── .env                      [gitignored]
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

**src-layout vs flat-layout:** putting your package under `src/` (instead of loose files in the project root) stops Python from silently importing your uncompiled local folder instead of the actually-installed package — matters once you `pip install -e .` (below). For a single script or small tool, flat layout (files straight in the root) is fine — don't over-engineer a 50-line script.

**README.md** — at minimum: what the project does, how to set it up (`venv` + `pip install -r requirements.txt`), how to run it. This is the first thing anyone (including future you) reads.

## 9. pyproject.toml

The modern standard config file — replacing the older `setup.py`/`setup.cfg`. One file holds build metadata, dependencies, and tool config (black, ruff, pytest settings, etc.) instead of scattering them across many dotfiles. You don't need this for standalone scripts, but expect to see it in most tutorials/repos going forward, and you'll want it once a project grows into something installable.

## 10. Separating dev deps from runtime deps

```
# requirements.txt        <- what the program needs to run
requests==2.30.0

# requirements-dev.txt    <- what you need to work on it
-r requirements.txt
pytest
black
mypy
```

```
py -m pip install -r requirements-dev.txt
```

Keeps testing/linting tools out of what you'd actually ship or hand to someone running the program.

## 11. VS Code + venv

Activating `.venv` in the terminal and having VS Code use it for linting/imports are two separate settings. If imports show red squiggles despite an active venv: `Ctrl+Shift+P` → **Python: Select Interpreter** → pick the one under `.venv`.

## 12. Editable installs

```
py -m pip install -e .
```

Installs your own local package in "link" mode — edits to source are picked up immediately, no reinstall needed. Requires a `pyproject.toml` (or `setup.py`) defining the package. Useful once you're building something with more than one file that imports from itself.

## 13. Pre-commit hooks (secret-scanning tie-in)

```
py -m pip install pre-commit
```

`.gitignore` only stops files you haven't `git add`ed yet — if a `.env` gets committed once by mistake, it's in history permanently even after you delete it later. A pre-commit hook (e.g. `detect-secrets` or `gitleaks`) scans staged changes _before_ the commit completes and blocks it if it sees something that looks like a key or token. Set up once per repo with a `.pre-commit-config.yaml`, then `pre-commit install`.

## 14. Alternatives you'll see referenced

|Tool|What it does differently|
|---|---|
|**Poetry**|Replaces pip+venv as one tool: manages dependencies, venv, and packaging natively through `pyproject.toml`|
|**Conda**|Manages non-Python dependencies too (C libraries, etc.); separate package index from PyPI, common in data-science tooling|

pip + venv is the portable baseline and what most documentation assumes — fine to stay on it; just recognize these names when tutorials use them instead.

## Try this

- [ ] Create a venv, activate it, confirm `(.venv)` shows in the PowerShell prompt
- [ ] Install one package, run `pip freeze`, then `deactivate` and confirm `pip list` outside the venv doesn't show it
- [ ] Write a `.gitignore` with `.venv/`, `.env`, `__pycache__/` before your next project's first commit
- [ ] Run `pip-audit` on an existing project and see what it flags

## Key terms

- **PyPI** — Python Package Index, the default package source for pip
- **site-packages** — the folder where installed packages actually live for a given interpreter
- **venv** — a self-contained interpreter + site-packages folder, isolated per project
- **requirements.txt** — a pinned list of package==version, used to reproduce an environment
- **.env** — a file storing secrets/config, read at runtime, not related to venv isolation
- **pipx** — installs CLI tools each in their own isolated environment, exposed globally
- **execution policy** — PowerShell setting that can block running `.ps1` scripts like `Activate.ps1`
- **pyproject.toml** — modern single config file for build metadata, dependencies, and tool settings
- **editable install** (`pip install -e .`) — links your local source into site-packages so edits apply without reinstalling
- **pre-commit hook** — a check that runs automatically before a commit completes; catches issues (e.g. leaked secrets) before they enter git history
- **src-layout** — placing your package under `src/` to avoid accidentally importing an uninstalled local copy