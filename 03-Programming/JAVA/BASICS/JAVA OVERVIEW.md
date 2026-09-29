---
title: JAVA OVERVIEW
date: 2026-09-29
language: Java
phase: phase-1
tags:
links: []
status: learning
---
> **One-line summary:** Java is object-oriented at its core (encapsulation + inheritance + polymorphism), every program lives inside a class that starts at `main()`, and the language itself is built from a small set of atoms — whitespace, identifiers, literals, comments, operators, separators, and 50 keywords — plus a big standard class library.

## What it is

**Two paradigms for organizing a program** (every program = code + data):

- **Process-oriented model** — organized around _code_: "what is happening." A series of linear steps, i.e. _code acting on data_. This is how C works, and it starts to break down as programs grow.
- **Object-oriented model** — organized around _data_: "who is being affected." Data plus a well-defined interface to that data, i.e. _data controlling access to code_. Created to manage growing complexity.

**Abstraction** is how humans manage complexity — you think "car," not tens of thousands of parts. Hierarchical abstraction layers a system into manageable pieces (car → subsystems → specialized units). In programs, process steps become _messages between objects_, each object describing its own behavior. Payoff: with clean interfaces, you can replace or retire parts of an old system without fear.

## Structure

```
              ┌── Encapsulation  (bind code + data, hide internals)
OOP  ─────────┼── Inheritance    (reuse + hierarchy: superclass → subclass)
              └── Polymorphism   (one interface, many implementations)
```

**The three OOP principles**

|Principle|Core idea|Car analogy|
|---|---|---|
|Encapsulation|Binds code and the data it manipulates; keeps both safe from outside misuse; access only via a well-defined interface|Gear-shift lever is the only way to affect the transmission — turn signal can't|
|Inheritance|One object acquires the properties of another; subclass = superclass attributes **plus** its own|All drivers can drive any vehicle because steering, brakes, accelerator are shared|
|Polymorphism|"One interface, multiple methods" — one general interface for a class of actions; compiler picks the specific action|Same brake pedal whether ABS or traditional brakes|

**Encapsulation in Java** — the basis is the **class**:

- A class = logical construct defining structure + behavior (data + code) shared by a set of objects. An **object** = an instance of a class, with physical reality (the "mold" vs. the thing stamped out).
- Members: **member/instance variables** (data) and **member methods** (code). Java "method" = C/C++ "function".
- `public` = the interface external users may use. `private` = accessible only by other members of the same class. Keep the public interface small so it doesn't leak inner workings. _(Book's Fig 2-1: public instance variables are "not recommended.")_

**Inheritance in Java**

- Book's example: Animal → Mammal → Canine → Domesticus → Retriever → Labrador / Golden.
- Each level only defines what makes it unique; the rest is inherited from ancestors. Labrador ends up with all attributes of every ancestor (Age, Sex, Weight, Litter Size, … AKC Certified?).
- Why it matters: a new subclass inherits everything from its ancestors and doesn't interact unpredictably with most of the rest of the system, so programs grow in complexity **linearly rather than geometrically**.

**Polymorphism in Java**

- Example: three stacks (int, float, char) use the same algorithm. Non-OO language → three sets of routines with different names. Java → one general set of stack routines sharing the same names.
- Dog analogy: the same "sense of smell" produces different reactions to different data (cat vs. food).

**How the three work together:** a good class hierarchy = reusable, tested code; encapsulation lets you change implementations without breaking dependents; polymorphism keeps code clean and readable. Every Java program involves all three, even if tiny examples don't visibly show it — and the built-in class libraries use them heavily.

## Mechanics / reference

### Your first program

```java
class Example {
  public static void main(String args[]) {
    System.out.println("This is a simple Java program.");
  }
}
```

**Entering it:** save as `Example.java`. A source file is officially a **compilation unit** — a text file containing one or more class definitions, with the `.java` extension. By convention the main class name = file name, and capitalization must match because Java is case-sensitive.

**Compile & run (JDK 8 command line):**

```
javac Example.java   --> produces Example.class (bytecode)
java Example         --> JVM runs the class; prints the string
```

- `javac` output is **not** directly executable — it's bytecode for the JVM.
- Each class in the source gets its own `.class` file named after the class. `java` takes the _class name_ (not the filename) and searches for `ClassName.class`.
- Using an IDE? The compile/run procedure differs — check the IDE's docs.

**Line-by-line breakdown**

|Piece|Meaning|
|---|---|
|`/* ... */`|Multiline comment — ignored by compiler|
|`// ...`|Single-line comment — to end of line|
|`class Example {`|`class` declares a new class; `Example` is an identifier (its name). Everything between `{ }` is the class body. All program activity happens inside a class|
|`public`|Access modifier — `main()` must be public because it's called from outside the class|
|`static`|`main()` can be called without creating an object — needed because the JVM calls it before any objects exist|
|`void`|`main()` returns no value|
|`main`|Program entry point. Case-sensitive: `Main` ≠ `main`|
|`String args[]`|Parameter: array of `String` holding command-line arguments|
|`System.out.println(...)`|`System` = predefined class giving system access; `out` = output stream connected to the console; `println()` prints the argument + newline|

- Only **one** class in a program needs `main()`. Applets don't use `main()` at all — the browser starts them differently.
- All _statements_ end with `;`. Lines like `class Example {` and `public static void main(...) {` aren't statements, so no semicolon.
- Console I/O is rare in real-world apps (they're GUI/windowed) — used mostly for utilities, demos, and server-side code.

### Variables and output (Example2)

```java
int num;              // declare
num = 100;            // assign (single = is assignment)
System.out.println("This is num: " + num);   // + concatenates
num = num * 2;
System.out.print("The value of num * 2 is ");
System.out.println(num);
```

Output: `This is num: 100` then `The value of num * 2 is 200`.

- Variables must be declared before use. General form: `type var-name;` — multiple of the same type via comma list: `int x, y;`
- `+` in `println` converts `num` to its string form and joins it. You can chain as many items as you like.
- `print()` = like `println()` but **no newline** afterward. Both work with any built-in type.

### Two control statements

**`if`** — same syntax as C/C++/C#: `if (condition) statement;` — runs the statement only if the Boolean condition is true.

|Operator|Meaning|
|---|---|
|`<`|Less than|
|`>`|Greater than|
|`==`|Equal to (double equals!)|

**`for`** — `for (initialization; condition; iteration) statement;`

- Initialization sets the loop variable; condition is tested at the start of each iteration (including the first); iteration updates the variable after each pass.
- Idiomatic Java uses the **increment operator**: `x++` instead of `x = x + 1`. Decrement is `--`.
- `for (x = 0; x < 10; x++)` prints 0 through 9.

**Blocks of code** — `{ ... }` groups statements into a logical unit usable anywhere a single statement is allowed (target of `if` / `for`). Main purpose: **logically inseparable units** — if the condition holds, _all_ statements in the block run together.

```java
if (x < y) {   // begin block
  x = y;
  y = 0;
}              // end block
```

### Lexical issues (the atoms of Java)

Java programs = whitespace + identifiers + literals + comments + operators + separators + keywords.

- **Whitespace:** Java is free-form — no indentation rules. Need at least one whitespace char (space, tab, newline) between tokens not already separated by an operator/separator.
- **Identifiers:** names for classes, variables, methods. Letters, digits, `_`, `$`; **cannot start with a digit**; case-sensitive. `$` isn't intended for general use.
    - Valid: `AvgTemp`, `count`, `a4`, `$test`, `this_is_ok`
    - Invalid: `2count`, `high-temp`, `Not/ok`
    - Since JDK 8, a lone `_` as an identifier is discouraged.
- **Literals:** constant values written directly — `100` (integer), `98.6` (floating-point), `'X'` (char), `"This is a test"` (string). Usable anywhere a value of that type is allowed.
- **Comments (3 kinds):** multiline `/* */`, single-line `//`, and **documentation comment** `/** */` — used to generate an HTML file documenting your program.
- **Separators:**

|Symbol|Name|Purpose|
|---|---|---|
|`( )`|Parentheses|Parameter lists, precedence in expressions, control-statement conditions, cast types|
|`{ }`|Braces|Array initializer values; blocks for classes, methods, local scopes|
|`[ ]`|Brackets|Declare array types; dereference array values|
|`;`|Semicolon|Terminates statements|
|`,`|Comma|Separates identifiers in declarations; chains statements inside `for`|
|`.`|Period|Separates package/subpackage/class names; separates a variable or method from a reference variable|
|`::`|Colons|Method or constructor reference (added in JDK 8)|

- **Keywords:** 50 defined. Can't be used as identifiers (variable, class, or method names). `const` and `goto` are **reserved but unused**. `true`, `false`, `null` are also reserved — they're _values_, not keywords.

### The Java class libraries

`println()` / `print()` come from `System.out`; `System` is a predefined class automatically included in every program. Java as a whole = **the language + its standard class libraries** (I/O, string handling, networking, graphics, GUI). Much of Java's functionality lives in the libraries, so learning to use them is part of learning Java. Part II of the book covers several in detail.

## Pitfalls

- **"`main` vs. `Main` — the compiler will catch it."** No. `javac` happily compiles classes without a `main()`; it's `java` that fails at runtime because it can't find `main`.
- **"The file name doesn't matter."** Unlike most languages, it does in Java. Match the file name to the main class (including capitalization), so the `.java` name lines up with the `.class` name.
- **"Run it with `java Example.class`."** No — pass the class name only: `java Example`. It finds `Example.class` itself.
- **"Every line needs a semicolon."** Only statements do. Class and method headers aren't statements.
- **`=` vs. `==`.** Single `=` assigns; `==` tests equality.
- **"Every program starts at `main()`."** Applets don't — the browser starts them.
- **"`goto` and `const` are real keywords I can use."** They're reserved but do nothing. And `true`/`false`/`null` can't be identifiers even though they aren't technically keywords.
- **"Public instance variables are fine."** Legal, but discouraged — expose behavior via public methods and keep data private.

## Flashcards

- What are the two paradigms for organizing a program? :: Process-oriented (code acting on data) and object-oriented (data controlling access to code) #card
- Why was OOP conceived? :: To manage the growing complexity that breaks down process-oriented programs as they get larger #card
- What are the three OOP principles? :: Encapsulation, inheritance, polymorphism #card
- What does encapsulation do? :: Binds code with the data it manipulates and keeps both safe from outside interference/misuse via a well-defined interface #card
- Class vs. object? :: A class is a logical construct (the mold); an object is a physical instance of it #card
- What's the difference between `public` and `private` members? :: Public = accessible from outside the class (the interface); private = accessible only by other members of the same class #card
- Why does inheritance keep complexity growth linear? :: A subclass inherits everything from its ancestors and adds only what's unique, without unpredictable interactions with most of the rest of the system #card
- What does "one interface, multiple methods" describe? :: Polymorphism — a general interface for a class of actions; the compiler picks the specific method #card
- What must the source file be named for `class Example`, and why? :: `Example.java` — Java is case-sensitive and the `.class` output is named after the class, so matching keeps things organized #card
- What does `javac` produce, and what runs it? :: `Example.class` (bytecode); the `java` launcher/JVM runs it #card
- Why is `main()` declared `static`? :: The JVM calls it before any objects exist, so it can't require an instance #card
- What does `String args[]` in `main()` receive? :: Any command-line arguments passed when the program runs #card
- Difference between `print()` and `println()`? :: `println()` appends a newline after output; `print()` doesn't #card
- What does `x++` do? :: Increments `x` by one — the idiomatic replacement for `x = x + 1` #card
- What is a block of code, and why use one? :: Statements grouped in `{ }` that act as one unit; makes multiple statements logically inseparable (e.g. as an `if`/`for` target) #card
- What are Java's three comment styles? :: Multiline `/* */`, single-line `//`, documentation `/** */` #card
- Which of `2count`, `high-temp`, `$test`, `this_is_ok` are valid identifiers? :: `$test` and `this_is_ok` (identifiers can't start with a digit or contain `-`) #card
- How many keywords does Java define, and which two are reserved but unused? :: 50; `const` and `goto` #card
- Name Java's reserved literal values. :: `true`, `false`, `null` #card
- What is the `::` separator for? :: Method or constructor references (added in JDK 8) #card

## Open questions

- [ ] What exactly are the "full complement" of relational operators beyond `<`, `>`, `==`? (Chapter 4)
- [ ] What else besides `main()` can the JVM use as an entry point — e.g. how does an applet get started by the browser?
- [ ] How does `System.out` actually work internally — what is `out`, exactly, as a field of `System`?
- [ ] How does the compiler choose which method to call for a polymorphic call — compile time or runtime?
- [ ] What happens when a source file contains multiple classes — which name must the file match?

## Key terms

|Term|Definition|
|---|---|
|Process-oriented model|Program organized around code — a series of linear steps acting on data|
|Object-oriented programming|Program organized around data (objects) and well-defined interfaces to it|
|Abstraction|Managing complexity by treating a complex system as a single well-defined object, layered hierarchically|
|Encapsulation|Mechanism binding code and data together and protecting both from outside misuse|
|Inheritance|Process by which one object acquires the properties of another|
|Polymorphism|"Many forms" — one interface used for a general class of actions|
|Class|Logical construct defining structure and behavior shared by a set of objects|
|Object|A physical instance of a class|
|Member variable / instance variable|Data defined by a class|
|Method|Code that operates on a class's data (a C/C++ "function")|
|Superclass / subclass|The more general class inherited from / the more specific class that inherits|
|Compilation unit|A Java source file — text file with one or more class definitions, `.java` extension|
|`javac`|Java compiler; produces `.class` bytecode files|
|`java`|Java application launcher; runs a compiled class on the JVM|
|`main()`|Entry point where all Java applications begin executing|
|Access modifier|Keyword (`public`/`private`) controlling visibility of class members|
|Parameter|Variable in a method's parentheses that receives information passed to the method|
|Identifier|Name for a class, variable, or method|
|Literal|Constant value written directly in code (`100`, `98.6`, `'X'`, `"text"`)|
|Separator|Symbol like `; , . ( ) { } [ ] ::` that structures code|
|Code block|Statements grouped between `{ }` and treated as a single unit|
|Class libraries|Built-in classes supplying I/O, strings, networking, graphics, GUI support|

## Related

[[1 - The History and Evolution of Java]] · [[3 - Data Types, Variables, and Arrays]]

→ Next: [[3 - Data Types, Variables, and Arrays]]