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

| Buzzword                       | What it actually means (in plain terms)                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Simple                         | Easy to learn if you already know C, C++, or basic OOP — Java reuses familiar syntax and ideas instead of inventing new ones. (Not necessarily easy for someone with zero programming background.)                                                                                                                                                                                                                                                                                     |
| Object-Oriented                | Code is organized around objects — bundles of data plus the actions that work on that data — instead of one long list of separate functions. Java makes one practical exception: simple number types like `int` stay fast, ordinary values rather than being forced into full objects.                                                                                                                                                                                                 |
| Robust                         | Java is built to catch your mistakes as early as possible. Because it's strict about types, most errors get caught when you compile the code, before you ever run it — and it keeps checking a smaller set of things while running too. Two classic trouble spots are handled for you automatically: memory cleanup (garbage collection, so you don't have to manually free memory) and error handling (structured `try`/`catch` instead of manually checking error codes everywhere). |
| Multithreaded                  | A single program can do multiple things at once out of the box — like downloading a file while updating a progress bar and still responding to clicks — without you having to build your own scheduling system from scratch.                                                                                                                                                                                                                                                           |
| Architecture-Neutral           | A Java program should keep working correctly on any machine, and keep working years later even after the OS or hardware changes underneath it. That's the "write once, run anywhere, any time" idea.                                                                                                                                                                                                                                                                                   |
| Interpreted & High-Performance | A Java program compiles down to bytecode, not instructions for one specific machine — which would normally run slower since it has to be interpreted. But Java's JIT compiler quietly turns the bytecode that runs often into fast native instructions while the program is running, so you get close to native speed without losing the "runs everywhere" benefit.                                                                                                                    |
| Distributed                    | Talking to something over a network — like fetching data from a URL — is built in and feels almost as easy as opening a file on your own computer. Java can even let one program call a method on an object that lives on a completely different machine (this is called RMI).                                                                                                                                                                                                         |
| Dynamic                        | A Java program can figure out what type of object it's dealing with while it's actually running, not just when it was written — and can even safely load small new pieces of code into a program that's already running. This flexibility is part of what makes Java robust.                                                                                                                                                                                                           |

**Two real memory/error problems Java's "Robust" design removes:**

- Manual memory allocation mistakes (forgetting to free memory, or freeing memory something else still needs) — Java handles this automatically via garbage collection.
- Ad-hoc, clumsy error handling for things like division-by-zero or a missing file — Java uses structured, object-oriented exception handling instead.

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