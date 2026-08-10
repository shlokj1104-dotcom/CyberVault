---
title: EXPLORING THE SYSTEM
date: 2026-08-10
phase: phase-1
tags:
  - linux
links: []
status: learning
---
# 1.3 Exploring the System

> **One-Line Summary:** `ls` reveals far more than filenames once given the right **options and arguments** — especially `-l`, whose long-format output packs permissions, ownership, size, and modification time into a single line — while `file` identifies what a file actually contains and `less` lets you safely read plain-text files like configs and scripts; together these three commands turn a guided walk through the Linux **Filesystem Hierarchy Standard** (`/bin`, `/etc`, `/var`, and friends) into something genuinely informative, with **symbolic links** acting as the mechanism that lets one file quietly answer to several different names.

---

## Core Idea: From Moving Around to Actually Looking Around

The previous note covered how to get anywhere in the filesystem. This one covers what to do once you've arrived: `ls`, `file`, and `less` are the three commands that turn navigation into genuine exploration, and the **options/arguments** pattern they introduce — a short or long flag followed by the thing being acted on — is the same pattern nearly every other Linux command will use from here on. The chapter closes with a real tour of the standard directories every Linux system shares, plus the concept of symbolic links, which quietly solve a versioning problem that shows up constantly once real software is involved.

This note builds up in four stages:

1. **More with `ls`, and the options/arguments pattern (1.3.1)** — multiple directories at once, long format, and how command syntax works in general.
2. **Identifying and reading files with `file` and `less` (1.3.2)** — figuring out what a file actually is, and safely viewing plain text.
3. **A guided tour of the standard directories (1.3.3)** — what `/etc`, `/var`, `/usr`, and the rest are actually for.
4. **Symbolic links (1.3.4)** — how one file can answer to more than one name, and why that's useful.

---

## 1.3.1 More with `ls`, and the Options/Arguments Pattern

Beyond listing the current working directory, `ls` accepts a pathname as an argument to list a _different_ directory instead, and can even be given several pathnames at once to list each of them in turn — including the shorthand `~` for a user's home directory.

Adding `-l` switches `ls` into **long format**, which reveals considerably more about each file: permissions, ownership, size, and modification time, not just a bare name. This single change illustrates a pattern used throughout Linux commands in general:

|Part|Role|
|---|---|
|`command`|The program being run|
|`-options`|Modify how the command behaves — a short option is a single dash plus one character (`-l`), a long option is a double dash plus a word (`--reverse`)|
|`arguments`|The items the command actually acts on, such as a filename or directory|

Short options can be strung together (`ls -lt` runs long format _and_ sorts by modification time in one go), and command options — like Linux filenames themselves — are case-sensitive.

|Option|Long option|Description|
|---|---|---|
|`-a`|`--all`|List every file, including hidden ones (names starting with a period)|
|`-A`|`--almost-all`|Like `-a`, but skips the `.` and `..` entries|
|`-d`|`--directory`|List a directory itself rather than its contents — useful together with `-l`|
|`-F`|`--classify`|Append a marker to each name (e.g. a trailing `/` for directories)|
|`-h`|`--human-readable`|Show file sizes in a human-friendly unit rather than raw bytes|
|`-l`|—|Long format output|
|`-r`|`--reverse`|Reverse the sort order (`ls` normally sorts alphabetically)|
|`-S`|—|Sort by file size|
|`-t`|—|Sort by modification time|

Reading a long-format line is its own small skill — each column means something specific:

```
Fig 1.5 -- Reading a Long-Format ls Entry
──────────────────────────────────────────
-rw-r--r--  1  root  root  32059  2017-04-03 11:05  oo-cd-cover.odf
   [1]      [2][3]   [4]    [5]         [6]               [7]
```

|#|Field|Meaning|
|---|---|---|
|1|`-rw-r--r--`|File type + permissions. The first character names the type (`-` for a regular file, `d` for a directory); the next nine are read/write/execute rights for the owner, the group, and everyone else in three-character blocks|
|2|`1`|Number of hard links pointing to this file|
|3|`root`|Username of the file's owner|
|4|`root`|Name of the group that owns the file|
|5|`32059`|Size, in bytes|
|6|`2017-04-03 11:05`|Date and time of the last modification|
|7|`oo-cd-cover.odf`|The filename itself|

> **Analogy — A Library's Catalogue Card:** A long-format `ls` line is a lot like an old library catalogue card: it doesn't just say a book exists, it says who can check it out, who's currently responsible for it, how thick it is, when it was last touched, and finally, its title. Skimming past all of that and reading only the filename is like reading only the title on the card and ignoring everything the librarian actually needed to know.

---

## 1.3.2 Identifying and Reading Files with `file` and `less`

Because Linux filenames are never _required_ to reflect what a file actually contains — a file called `picture.jpg` isn't guaranteed to be a JPEG — the **`file`** command inspects a file's actual content and reports what kind of data it holds, regardless of what its name claims. This connects to a broader idea common to Unix-like systems: in a real sense, "everything is a file," from ordinary documents down to devices and running processes.

**`less`** is a program for viewing the contents of text files — and a considerable number of important Linux files genuinely are plain, human-readable text: configuration files that control system settings, and scripts that are themselves just text instructions the shell can run. Being able to simply read these files, without needing any special editor, is one of the more transparent aspects of how Linux works.

Under the hood, "text" here means something specific: a simple, compact one-to-one mapping between characters and numbers (classically ASCII), containing only the characters themselves plus a few basic control codes like tabs and line breaks. This is a fundamentally different, far simpler format than something like a word-processor document, which bundles in extensive formatting and structural information alongside the visible text.

|Command|Action|
|---|---|
|`Page Up` / `b`|Scroll back one page|
|`Page Down` / `space`|Scroll forward one page|
|`Up arrow`|Scroll up one line|
|`Down arrow`|Scroll down one line|
|`G`|Jump to the end of the file|
|`1G` / `g`|Jump to the beginning of the file|
|`/characters`|Search forward for the next occurrence|
|`n`|Repeat the previous search|
|`h`|Show the help screen|
|`q`|Quit `less`|

`less` takes its name as a pun on an older, more limited pager program called `more` — "less is more" — and belongs to a general class of programs called **pagers**, which display long text documents a page at a time. Unlike its predecessor, `less` can page both forward _and_ backward, among other improvements.

---

## 1.3.3 A Guided Tour of the Standard Directories

The overall layout of a Linux filesystem isn't arbitrary — it's described by a published standard called the **Filesystem Hierarchy Standard (FHS)**, which most distributions follow closely even if not perfectly. Wandering through it directly, applying the navigation skills from the previous note, is a genuinely effective way to build intuition: `cd` into a directory, `ls -l` its contents, `file` anything unfamiliar, and `less` anything that looks like text. Regular users are largely prevented from breaking anything by exploring — that protection is precisely what file permissions are for — and if a non-text file ever scrambles the terminal display, the `reset` command restores it.

|Directory|What it's for|
|---|---|
|`/`|The root directory — where the entire tree begins|
|`/bin`|Essential program binaries needed for the system to boot and run|
|`/boot`|The Linux kernel, initial RAM disk image, and boot loader files|
|`/dev`|Device nodes — under "everything is a file," this is where the kernel exposes the devices it knows about|
|`/etc`|System-wide configuration files, plus startup scripts for system services|
|`/home`|Each user's own directory; ordinary users can generally only write files here|
|`/lib`|Shared libraries used by core system programs (comparable to DLLs on Windows)|
|`/lost+found`|Used to recover files after a filesystem-corruption event; normally empty|
|`/media`|Mount points for removable media such as USB drives, added automatically|
|`/mnt`|Mount points for devices that have been mounted manually (older convention)|
|`/opt`|Optional, typically commercial, software packages|
|`/proc`|A virtual filesystem — not real files on disk, but a live window into the kernel itself|
|`/root`|The home directory belonging specifically to the root (superuser) account|
|`/sbin`|System binaries reserved for administrative tasks|
|`/tmp`|Temporary files created by running programs; sometimes cleared on reboot|
|`/usr`|Programs and support files for regular users — typically the largest tree on the system|
|`/usr/bin`|The bulk of a distribution's installed executable programs|
|`/usr/lib`|Shared libraries supporting the programs in `/usr/bin`|
|`/usr/local`|Software installed outside the distribution's own packages, e.g. compiled from source|
|`/usr/sbin`|Additional system-administration programs|
|`/usr/share`|Shared, architecture-independent data: default configs, icons, backgrounds, sounds|
|`/usr/share/doc`|Documentation bundled with installed packages|
|`/var`|Data that changes routinely — logs, spool files, mail, and similar|
|`/var/log`|System log files; important to monitor, and often restricted to the superuser to read|

---

## 1.3.4 Symbolic Links

While exploring, a directory listing will occasionally show an entry whose type character is `l` and whose name is followed by an arrow pointing to a second filename — this is a **symbolic link** (also called a _soft link_ or _symlink_): a special file that simply refers to another file by name, letting one underlying file effectively answer to more than one identity.

```
Fig 1.6 -- A Symbolic Link in Action
────────────────────────────────────────
lrwxrwxrwx 1 root root 11 ... libc.so.6 -> libc-2.6.so

  "foo"  ──(symlink)──▶  "foo-2.6"   (the real file)

Any program that opens "foo" transparently
ends up reading whichever version foo
currently points to.
```

The practical payoff shows up with version-numbered software: a library shipped as `libc-2.6.so` can have a plain symlink named `libc.so.6` pointing at it. Every program that asks for `libc.so.6` transparently gets whichever real file the link currently points to. When a newer version arrives, only the symlink needs repointing — nothing that depends on the old name has to be tracked down and edited, and if the new version turns out to be broken, reverting is just a matter of pointing the link back.

> **Analogy — A Building's Directory Sign, Not the Office Itself:** A symbolic link is like the nameplate on a building's lobby directory rather than the office door itself — "Accounting, 4th floor." When Accounting relocates to the 7th floor, only the lobby sign needs updating; nobody visiting has to learn a new department name, they just get redirected once the sign changes.

A related mechanism, **hard links**, achieves multiple names for a file in a structurally different way — worth knowing exists, though its details are more naturally covered alongside file creation and linking commands in the following note.

---

## Why It Matters for Security

|Concept|Attacker's Perspective|Defender's Perspective|
|---|---|---|
|**Long-format permissions are the first thing worth checking**|Overly permissive files (world-writable scripts, readable secrets) are exactly what an attacker who's gained any foothold will scan for using `ls -l`|Regularly audit permissions on sensitive files and directories, especially anything under `/etc`, `/var/log`, or a home directory that shouldn't be group- or world-writable|
|**`/etc/passwd` and `/etc/shadow` are prime reconnaissance targets**|`/etc/passwd` lists every user account on the system and is world-readable by design, making it a routine first stop for enumerating valid usernames|`/etc/shadow` (which holds the actual password hashes) is deliberately restricted to root — verify that restriction hasn't been loosened, and treat `/etc/passwd` readability as expected, not a finding on its own|
|**`file` reveals content that a filename can lie about**|Malware is routinely disguised with an innocuous-looking extension or name to blend in during casual review|Never trust a filename's apparent type at face value during an investigation — `file` (or deeper analysis) confirms what a file actually is|
|**Symbolic links can be abused to redirect trusted paths**|A symlink swapped in to point somewhere unexpected (a classic TOCTOU / symlink-attack pattern) can trick a privileged process into reading or writing the wrong file|Be cautious with symlinks in world-writable directories, and prefer safe file-handling APIs that resist symlink redirection over naive path-based access|

---

## Questions I Still Have

- [ ] The book's version-upgrade example swaps a symlink from `foo-2.6` to `foo-2.7` — in practice, which actual Linux command performs that repointing, and does it need any special privilege to do so?
- [ ] `/proc` is described as a virtual filesystem giving a live view into the kernel — what does actually reading one of its files look like in practice, and what kind of information tends to live there?
- [ ] Given that `/var/log` is often restricted to the superuser, what's the normal, non-root way an ordinary sysadmin task or monitoring tool is supposed to read logs day to day?

---

## Key Terms — Quick Reference

| Term                                    | Definition                                                                                    |
| --------------------------------------- | --------------------------------------------------------------------------------------------- |
| **Option**                              | A flag that modifies a command's behavior — short (`-l`) or long (`--reverse`)                |
| **Argument**                            | The item(s) a command actually acts on, such as a filename                                    |
| **Long format**                         | `ls -l` output showing permissions, ownership, size, and modification time                    |
| **`file`**                              | Command that reports what a file actually contains, independent of its name                   |
| **`less`**                              | A pager program for viewing text files a page at a time                                       |
| **Pager**                               | A class of program that displays long text a page at a time                                   |
| **Filesystem Hierarchy Standard (FHS)** | The published convention most Linux distributions follow for organizing top-level directories |
| **Symbolic link (symlink)**             | A special file that refers to another file by name, letting one file answer to multiple names |
| **Hard link**                           | A second mechanism for giving a file multiple names, structurally different from a symlink    |

---

## Related Concepts

---

→ Next: [[1.4 - Manipulating Files and Directories]]