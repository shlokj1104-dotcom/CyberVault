---
title: INTRODUCTION
date: 2026-08-19
language: Java
phase: phase-1
tags:
links: []
status: learning
---
> **One-line summary:** Java code gets typed as text, turned into bytecode by a compiler, then run by a Java Virtual Machine (JVM) — and that one design choice is why "write once, run anywhere" actually works.

## What it is

Java has been around since 1996 and is still one of the most-used languages out there. Two things make it stand out:

- **Portability.** You compile your code once, and the result (bytecode) can run on any device that has a JVM — laptop, phone, tablet, whatever.
- **A safety net built into the language itself.** Java checks your code twice — once when it compiles, once while it runs — before letting sketchy stuff happen.

Old Java (early versions) was genuinely slow. Modern Java uses a JVM feature called HotSpot that optimizes your code _while it's running_, so it's now close to C/Rust in speed — though it does use noticeably more memory than those languages. Java is also famous for backward compatibility: code written for old JVMs still runs fine on new ones, so you'll bump into both old-style and new-style code out in the wild.

## Structure

**The path your code takes to actually run:**

```
[Source .java] -> [Compiler javac] -> [Bytecode .class]
                                            |
                                            v
                                   [JVM — any device]
```

1. You write a **source file** (`.java`), using the Java language.
2. The **compiler** (`javac`) reads it, checks it for errors, and — if everything's fine — produces a new file made of **bytecode** (`.class`). The compiler won't let broken code through.
3. Bytecode is _platform-independent_. Any device with a JVM can read and run it.
4. The **JVM** — Java's virtual machine, sitting inside the device — reads the bytecode and actually executes it. This is the step that makes the program run.

**How code is organized inside the source file itself:**

```
source file (.java)
  └── class
        └── method
              └── statement;
```

- A source file holds a **class**.
- A class holds one or more **methods**.
- A method holds **statements** — the actual instructions.

## Mechanics / reference

|Term|What goes there|Example|
|---|---|---|
|Source file|One class, `.java` extension|`Robot.java`|
|Class|One or more methods, wrapped in `{ }`|`public class Robot { ... }`|
|Method|A block of statements — think function/procedure|`void beep() { ... }`|
|Statement|One instruction, always ends in `;`|`x = x + 1;`|

**The required entry point** — every Java program needs exactly one method that looks like this, and the JVM starts here:

```java
public class Robot {
    public static void main(String[] args) {
        // your code goes here
    }
}
```

- A program can use many classes, but only **one** of them needs a `main` method — the one that kicks the whole thing off.
- Compile it: `javac Robot.java` → produces `Robot.class`
- Run it: `java Robot` → JVM loads the class and runs everything inside `main`'s curly braces, top to bottom.

**Basic syntax rules:**

|Rule|Example|
|---|---|
|Every statement ends in a semicolon|`int x = 5;`|
|Code blocks (classes, methods, loops, if-blocks) live inside `{ }`|`void go() { ... }`|
|Declare a variable with a type + a name|`int weight;`|
|One `=` assigns a value|`x = 3;`|
|Two `==` checks for equality|`if (x == 3) { }`|
|Two slashes start a single-line comment|`// this is a note to yourself`|
|Most extra whitespace doesn't matter|`x = 3 ;` still works|

**Loops and branching** (this is where a program actually _does_ something repeatedly or conditionally):

```java
// while loop — repeats as long as the condition is true
int count = 5;
while (count > 0) {
    System.out.println("count is " + count);
    count = count - 1;
}

// for loop — same idea, more compact for counting
for (int i = 0; i < 5; i = i + 1) {
    System.out.println("i is " + i);
}

// if / else — branching
if (count == 0) {
    System.out.println("done counting");
} else {
    System.out.println("still going");
}
```

A key rule: the condition inside `while (...)` or `if (...)` **must evaluate to a boolean** (`true` or `false`). Unlike some other languages, Java won't let you test a plain `int` directly — you need a comparison like `x > 0` or `x == 3`, or an actual `boolean` variable.

**`print` vs `println`:** `System.out.print(...)` keeps writing on the same line. `System.out.println(...)` adds a line break after — think of the `ln` as "line."

## When to use it

This is foundational syntax, so "when to use it" really means: every single Java program you ever write starts from these pieces. A few situations where the specific ideas in this chapter matter most:

- **Choosing while vs for:** use `for` when you know how many times you're looping (a counter); use `while` when you're looping until some condition changes (waiting for user input, reading a file until it ends, etc.).
- **Comparing with `==` vs assigning with `=`:** this trips up almost everyone at some point — a classic bug source, covered below in Pitfalls.
- **Multiple classes, one `main`:** useful to remember once you start writing multi-class programs — you don't need a `main` in every class, just the one that starts the app.

## Pitfalls

- Using `=` (assignment) when you meant `==` (equality check) inside an `if` or `while` — very easy typo, and Java's strict typing actually protects you here more than in some languages, since it won't let you assign a non-boolean where a boolean condition is expected.
- Writing a `while` loop and forgetting to change the variable being tested — this causes an infinite loop, since the condition never becomes false.
- Trying to test a plain `int` (or any non-boolean) directly in a condition, e.g. `while (x) { }` — Java requires an actual boolean result.
- Forgetting the class wrapper entirely (statements can't just float outside a class), or forgetting `main` isn't inside a method.
- Mismatched or missing curly braces — every class, method, loop, and if-block needs its own matching pair.
- Confusing `print` and `println` and ending up with output all mashed onto one line by accident.

## Flashcards

- What are the four things bytecode goes through before a program runs? :: source (.java) → compiler → bytecode (.class) → JVM #card
- What's the one method every runnable Java program needs? :: `public static void main(String[] args)` #card
- What's the difference between `=` and `==`? :: `=` assigns a value, `==` checks equality #card
- Why can bytecode run on any device? :: it's platform-independent — any device with a JVM can read and run it #card
- What must the condition inside a `while` or `if` evaluate to? :: a boolean (true or false), never a plain int #card
- What's the difference between `print` and `println`? :: `println` adds a line break after printing, `print` doesn't #card
- Name one thing the compiler catches vs one thing only the JVM can catch. :: compiler catches most type errors at compile time; JVM catches runtime-only issues like invalid object casts #card

## Open questions

- [ ] How exactly does HotSpot decide which parts of the code to optimize while running?
- [ ] What does Java's bytecode verification process actually check for, step by step?
- [ ] Why does Java use more memory than C/Rust — is that mostly the JVM's own overhead, or something about how Java objects are stored?

## Key terms

|Term|Definition|
|---|---|
|Bytecode|The compiled, platform-independent form of your code, stored in a `.class` file|
|JVM|Java Virtual Machine — the program that actually reads and runs bytecode on a given device|
|Compiler (javac)|Converts your `.java` source file into bytecode, checking for errors along the way|
|Class|A blueprint / container that holds one or more methods|
|Method|A named block of statements — Java's version of a function|
|Statement|One instruction inside a method, always ending in a semicolon|
|Argument|A value passed into a method — e.g. the `String[] args` in `main`|
|Boolean|A value that's either `true` or `false` — required for loop/if conditions|

## Related

→ Next: [[2 - A Trip to Objectville]]