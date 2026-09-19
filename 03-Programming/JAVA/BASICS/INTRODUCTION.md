---
title: INTRODUCTION
date: 2026-08-19
language: Java
phase: phase-1
tags:
links: []
status: learning
---
> **One-line summary:** Every language exists because the ones before it hit a wall — C replaced assembly, C++ added objects to manage growing complexity, and Java added true portability (via bytecode + JVM) for a world of many different machines, first embedded devices, then the internet.

## What it is

Languages evolve for two reasons: to adapt to a changing environment, and to refine the actual craft of programming. Java is a clean example of both forces acting at once.

**The chain that leads to Java:**

- **C** replaced assembly and older languages like FORTRAN/BASIC/COBOL because those forced an ugly trade-off — you got either power, or safety, or structure, but rarely all three. Older languages also leaned on `GOTO`-driven control flow, producing tangled "spaghetti code" that was nearly impossible to follow. C was structured, efficient, _and_ reasonably easy to learn — and critically, it was built by working programmers refining it through real use, not designed by committee.
- **C++** exists because, past a certain size, even well-structured C programs become too complex to hold in your head. C++ added object-oriented programming (inheritance, encapsulation, polymorphism) on top of C specifically to manage that complexity.
- **Java** inherits its syntax from C and its object-oriented feel from C++, but it isn't source-compatible with either — the designers used the familiarity to make Java approachable, then built a language from a clean slate. Java's real trigger, though, wasn't "let's improve on C++" — it was portability.

## Structure

```
C  --(syntax)-->  C++  --(OOP model)-->  Java  --(cross-pollination)-->  C#
```

Java didn't just borrow from its ancestors — it fed forward too. C# (built by Microsoft for .NET) shares Java's general syntax, object model, and support for distributed programming — the strongest sign of how much Java reshaped language design afterward.

## Mechanics / reference

**Why Java pivoted to the internet:** Java's _original_ goal (1991) had nothing to do with the Web — it was a small, architecture-neutral language for embedded consumer devices like microwaves and remote controls, where many different CPUs are used as controllers and building a full compiler for each one is too expensive. By 1993, the team realized the exact same problem — "this code needs to run correctly on hardware I don't control" — applies just as much to the internet, which is a diverse mix of CPUs, operating systems, and platforms. That realization is what actually launched Java into the mainstream.

**Applets, security, and portability:** An _applet_ is a small Java program that gets automatically downloaded and run by a Java-enabled browser — no install step, no user interaction needed beyond clicking a link. This was a genuinely new category: unlike passive downloaded data (an email, a file), an applet is a _self-executing_ program, initiated by the server but actively running on the client. That combination raised two hard problems:

- **Security** — a program that auto-runs on your machine the moment you visit a page has to be stopped from doing anything harmful.
- **Portability** — the same applet has to run correctly across every CPU, OS, and browser combination on the internet; shipping different versions per platform isn't practical.

**Bytecode is the single mechanism that solves both.** The Java compiler doesn't output native machine code — it outputs _bytecode_, a CPU-independent instruction set designed to be run by the Java Virtual Machine (JVM):

```
source.java --(compiler)--> bytecode --(JVM)--> runs on any platform
```

- **Portability:** only the JVM itself needs to be built per-platform. Once that exists, _any_ Java program — already compiled — runs on it unchanged.
- **Security:** because the JVM is the one actually controlling execution, it can confine an applet to its own execution environment and block it from touching the rest of the system.
- **Performance:** interpreting an intermediate form is normally slower than running native code directly, but Java's bytecode is heavily optimized, and modern JVMs use a **Just-In-Time (JIT) compiler** (Sun's HotSpot) that translates the frequently-used pieces of bytecode into native machine code on the fly, piece by piece, only where it actually pays off. You keep bytecode's portability and safety without eating the full performance cost of interpretation.

**Servlets are the server-side mirror of applets.** A servlet is a small Java program that runs _on the server_ instead of the client, dynamically generating content (e.g. looking up a price in a database and building the resulting page). Since servlets are still just bytecode run by a JVM, the same servlet is portable across any server that supports a JVM and a servlet container.

## The Java buzzwords

|Buzzword|What it actually means|
|---|---|
|Simple|Familiar C/C++ syntax and OOP concepts make it an easy transition _if_ you already know one of those — not necessarily easy for a total beginner|
|Object-Oriented|Not built to be source-compatible with any earlier language — a clean-slate design that balances "everything is an object" purity with pragmatism (primitive types like `int` stay high-performance non-objects)|
|Robust|Strict typing checked at _both_ compile time and runtime, closing off a huge class of bugs before they can even happen|
|Multithreaded|Built-in support for programs that do multiple things at once, with a clean synchronization model, so you can focus on your program's logic instead of building your own multitasking system|
|Architecture-Neutral|Solves "will this even run tomorrow, on this same machine" — OS and CPU upgrades used to silently break programs; the goal was "write once, run anywhere, any time"|
|Interpreted & High-Performance|Bytecode gives cross-platform reach without the usual performance penalty, thanks to JIT compilation|
|Distributed|Built-in TCP/IP support means grabbing a remote resource by URL feels like opening a local file; also supports Remote Method Invocation (RMI) — calling methods on objects across a network|
|Dynamic|Carries rich runtime type information, so new code (even small bytecode fragments) can be safely linked into a program that's already running|

_(Secure and Portable are covered above under bytecode — that's the actual mechanism behind both.)_

**Two real memory/error problems Java's "Robust" design removes:**

- Manual memory allocation mistakes (forgetting to free memory, or freeing memory something else still needs) — Java handles this automatically via garbage collection.
- Ad-hoc, clumsy error handling for things like division-by-zero or a missing file — Java uses structured, object-oriented exception handling instead.

## When to use it

**Where Java's design pays off in practice:**

- Cross-platform client software, where "the same code has to run everywhere" is a hard requirement.
- Server-side applications — servlets (and the frameworks built on top of them) let one codebase serve dynamic content regardless of the underlying server OS.
- Anywhere you need to safely run code you didn't write yourself, since the JVM's sandboxing model was built exactly for that case.

## Pitfalls

- **"Java is just the internet version of C++."** Not true — Java isn't upward or downward compatible with C++. It was designed to solve a different set of problems, and both languages have continued to coexist.
- **"All Java programs run inside a browser."** Only applets do, by definition. Servlets run entirely on the server, and most Java code today has nothing to do with a browser at all.
- **"Bytecode interpretation must be slow."** Not necessarily — the JIT compiler translates the "hot" parts of bytecode into native code on demand, keeping performance close to native speed.
- **"Java was designed for the internet from day one."** Its original goal was actually embedded consumer electronics — the internet only became the primary driver once the Web reached critical mass a couple of years in.

## Flashcards

- Why did older languages like BASIC/COBOL/FORTRAN tend to produce unmaintainable large programs? :: They relied on `GOTO`-driven control flow, producing tangled "spaghetti code" #card
- What problem did C++ solve that plain structured C couldn't? :: Managing complexity beyond a certain size, via object-oriented programming (inheritance, encapsulation, polymorphism) #card
- What was Java's original design goal, before the internet took over? :: A small, architecture-neutral language for embedded consumer devices #card
- What is bytecode, and why does it matter? :: CPU-independent instructions produced by the Java compiler; only the JVM needs porting per platform, which is what makes Java programs portable #card
- How does Java stay fast despite bytecode being interpreted? :: A Just-In-Time (JIT) compiler translates frequently-used bytecode into native machine code while the program runs #card
- What's the difference between an applet and a servlet? :: Same bytecode/JVM model, opposite ends of the client/server connection — an applet runs on the client, a servlet runs on the server #card
- What was the single biggest language feature added in Java SE 8? :: Lambda expressions #card
- Name two features introduced in J2SE 5. :: Any two of: generics, annotations, autoboxing/unboxing, the enhanced for-loop, varargs, static import #card

## Open questions

- [ ] How exactly does the JVM decide which bytecode sequences are worth JIT-compiling versus just interpreting?
- [ ] What does Remote Method Invocation (RMI) actually look like in code, compared to a normal local method call?
- [ ] Are servlets still commonly used directly today, or have frameworks mostly abstracted them away?

## Key terms

|Term|Definition|
|---|---|
|Bytecode|CPU-independent instructions produced by the Java compiler, run by the JVM|
|JVM|Java Virtual Machine — interprets (or JIT-compiles) bytecode|
|JIT compiler|Translates frequently-run bytecode into native machine code while the program runs|
|Applet|A small Java program automatically downloaded and run by a browser|
|Servlet|A small Java program that runs on the server — the server-side counterpart to an applet|
|RMI|Remote Method Invocation — calling methods on objects across a network|
|Structured programming|Programming without `GOTO`-driven control flow, using clear blocks, loops, and conditionals|

## Related

[[]]

→ Next: [[2 - An Overview of Java]]