---
title: NAVIGATING THE FILESYSTEM
date: 2026-08-07
phase: phase-1
tags:
  - linux
links: []
status: learning
---
# 1.2 Navigating the Linux Filesystem

> **One-Line Summary:** Linux organizes everything into one single **hierarchical directory tree** rooted at `/`, a shell session always has a **current working directory** it reports via `pwd`, movement between directories happens with `cd` using either **absolute pathnames** (always starting at root) or **relative pathnames** (starting from wherever you're standing, using `.` and `..`), and directory contents are listed with `ls` — all built on filename conventions and a handful of `cd` shortcuts that reward a small amount of memorization.

---

## Core Idea: The First Skill That Actually Moves You Somewhere

Chapter 1.1 was purely conceptual — what a shell even is. This is the first chapter where the shell actually does something visible: it moves a person around a filesystem and shows them where they've landed. Three commands carry almost the entire chapter — `pwd`, `cd`, `ls` — and everything else here exists to explain how those three commands actually work under the hood. Given how much of scripting, log-hunting, and privilege boundaries later on depend on knowing exactly where a file lives relative to where you're standing, this is worth internalizing properly rather than memorizing by rote.

This note builds up in three stages:

1. **The filesystem tree and the current working directory (1.2.1)** — how Linux organizes files, and what it means for a shell to be "standing" somewhere in that tree.
2. **Absolute and relative pathnames (1.2.2)** — the two ways to tell `cd` where to go, and why they always agree.
3. **Listing contents, filename rules, and quick shortcuts (1.2.3)** — the practical commands and conventions used every day.

---

## 1.2.1 The File System Tree and the Current Working Directory

A Unix-like system such as Linux organizes its files in a **hierarchical directory structure**: directories (also called folders) arranged in a tree-like pattern, where any directory can contain files and further directories. The very first directory in this tree is the **root directory**, written simply as `/`, and every other file or directory in the system descends from it — directly or through some chain of subdirectories.

One detail is worth dwelling on because it trips up anyone coming from Windows: Windows gives each storage device (`C:`, `D:`, a USB drive) its own separate tree. Linux does not. However many drives or storage devices are attached to a Linux machine, there is only ever **one** filesystem tree, and each device is **mounted** — attached — at some specific point within that single tree, at a location a system administrator chooses.

```
Fig 1.3 -- A Partial Linux Filesystem Tree
────────────────────────────────────────────
/
├── bin
├── boot
├── dev
├── etc
├── home
│     ├── shlok
│     │     ├── Documents
│     │     ├── Downloads
│     │     └── Projects
│     └── otheruser
├── usr
│     └── bin
└── var
```

At any given moment, a shell session is standing in exactly one directory — its **current working directory (CWD)** — and can see the files inside it, the pathway up to its parent, and any subdirectories below. The `pwd` (print working directory) command reports exactly where that is:

```
[shlok@linuxbox ~]$ pwd
/home/shlok
```

> **Analogy — A Building With One Elevator Shaft:** Picture a tall building where every single room, on every floor, is reachable only by walking down from one central lobby (the root) through a chain of hallways and doors. There is no second lobby, no separate building next door for the "D: drive" — if a new wing gets built (a new storage device attached), it simply gets a door installed somewhere inside the existing building, and from then on it's just part of the same structure. Wherever a person is currently standing inside that building is their current working directory; `pwd` is simply them reading the room number off the door.

When first logging in — or opening a fresh terminal emulator session — the current working directory is automatically set to the **home directory**, and every user account is given its own. Critically, the home directory is also, by default, the _only_ place a regular (non-administrator) user account is permitted to write files at all — a boundary worth remembering well before it becomes relevant to permissions and privilege escalation later in this roadmap.

---

## 1.2.2 Absolute and Relative Pathnames

Moving the current working directory is done with `cd`, followed by a **pathname** — the route taken through the tree's branches to reach the desired directory. There are exactly two ways to specify that route.

An **absolute pathname** begins at the root directory and follows the tree branch by branch until the destination is fully specified. `/usr/bin`, for example, means: starting from root, find the directory `usr`, and inside it find the directory `bin`.

A **relative pathname**, by contrast, begins from wherever the current working directory already is. Two special notations make this possible: `.` (a single dot) refers to the current working directory itself, and `..` (two dots) refers to that directory's _parent_.

```
Fig 1.4 -- Two Routes to the Same Directory
────────────────────────────────────────────
Absolute pathname (always starts at root):

   /  ──▶  usr  ──▶  bin        => /usr/bin

Relative pathname (starts from wherever
you're standing -- here, cwd = /usr):

   .  ──▶  bin                 => ./bin

Both commands land in the exact same place.
```

|Property|Absolute Pathname|Relative Pathname|
|---|---|---|
|**Starting point**|Root directory (`/`)|Current working directory|
|**Typing effort**|Usually longer|Usually shorter|
|**Result depends on where you're standing?**|No — always the same regardless of cwd|Yes — the same command means something different depending on cwd|
|**Example**|`cd /usr/bin`|`cd ./bin` (only correct while standing in `/usr`)|

A small but genuinely useful shortcut: the leading `./` can almost always be omitted. Typing `cd bin` while standing in `/usr` does exactly the same thing as `cd ./bin` — if no pathname notation is specified at all, the shell assumes it's relative to the working directory by default.

---

## 1.2.3 Listing Contents, Filename Rules, and Quick Shortcuts

### Listing directory contents

The `ls` command lists the files and directories inside the current working directory:

```
[shlok@linuxbox ~]$ ls
Desktop  Documents  Music  Pictures  Public  Templates  Videos
```

`ls` isn't limited to the current directory either — passing it any pathname lists that directory's contents instead. It has a considerable number of further options and behaviors worth a dedicated look later on.

### Filename conventions

A handful of rules govern how Linux treats filenames, several of which differ meaningfully from Windows habits:

- **Hidden files** are simply any filename beginning with a period (`.`) — `ls` on its own will not display them; `ls -a` is needed to see them. Several hidden files are placed in every home directory automatically at account creation to hold configuration and settings.
- **Filenames are case-sensitive.** `File1` and `file1` are two entirely different files, not the same file with inconsistent capitalization.
- **Spaces in filenames are technically allowed but strongly discouraged**, since they complicate typing commands correctly; an underscore or dash is the conventional substitute.
- **Linux has no built-in notion of a "file extension."** A file's actual content and purpose are determined by the operating system through other means, not by whatever text follows a dot in its name — even though plenty of individual applications still choose to use extensions as a convention of their own.

### Quick `cd` shortcuts

|Shortcut|Result|
|---|---|
|`cd`|Changes to your home directory|
|`cd -`|Changes to the previous working directory (wherever you just came from)|
|`cd ~username`|Changes to the home directory of the specified user, e.g. `cd ~bob`|

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**Home directory as a write boundary**|Escalation attempts on a low-privilege account often specifically hunt for the rare paths that _are_ writable outside the account's home directory — misconfigured permissions, a stray world-writable directory|Enforce that regular users can only write within their own home directory by default; periodically audit for accidental world-writable paths elsewhere on the system|
|**`..` and path traversal**|Directory traversal (also called path traversal, catalogued as CWE-22) abuses the `..` notation embedded in a filename or a URL parameter to escape an application's intended directory and reach sensitive files like `/etc/passwd`|Never build a file path by directly concatenating untrusted input; canonicalize the resulting path and verify it still falls within the allowed root directory before using it|
|**Hidden (dotfile) files**|Malware and persistence mechanisms are frequently disguised as dotfiles, since a plain `ls` won't reveal their presence at all|Always include hidden files (`ls -a`) during any forensic or security review of a directory, rather than trusting the default listing|

---

## Questions I Still Have

- [ ] Beyond the home directory, which other paths does a regular user typically get write access to by default (e.g. `/tmp`), and how does that permission differ mechanically from being confined to `$HOME`?
- [ ] What does a real, minimal path-traversal vulnerability actually look like in vulnerable server-side code — what's the smallest realistic snippet an attacker could exploit with a crafted `../` payload?
- [ ] Where does a typical fresh Ubuntu install actually mount its storage devices in the tree, and how would I go about inspecting that on my own system once I have Linux installed?

---

## Key Terms — Quick Reference

|Term|Definition|
|---|---|
|**Hierarchical directory structure**|A tree-like file organization where directories can contain files and other directories|
|**Root directory (`/`)**|The top of the Linux filesystem tree, from which every absolute pathname starts|
|**Mounting**|Attaching a storage device at a specific point within the single, unified filesystem tree|
|**Current working directory (CWD)**|The directory a shell session is currently "standing in"|
|**Absolute pathname**|A path that starts at the root directory and specifies the full route to a destination|
|**Relative pathname**|A path that starts from the current working directory|
|**`.` (dot)**|Shorthand for the current working directory|
|**`..` (dot dot)**|Shorthand for the parent of the current working directory|
|**Home directory**|The directory a user's shell starts in at login; by default, the only place a regular user can write files|

---

## Related Concepts

---

→ Next: [[EXPLORING THE SYSTEM]]