---
title: JAVA OVERVIEW
date: 2026-09-29
language: Java
phase: phase-1
tags:
links: []
status: learning
---
> **One-line summary:** Java is built around objects, and every object-oriented design rests on three ideas: encapsulation (protect the data), inheritance (reuse and specialize), and polymorphism (one name, many behaviors). Every Java program lives inside a class and starts at `main()`.

## What it is

Every program is made of two things: **code** (the steps) and **data** (the information the steps work on). There are two ways to organize a program around them.

**1. Process-oriented model ("what is happening?")**

- The program is a series of steps, and the data is just passed around to those steps. Think of it as _code acting on data_.
- C works this way. It's fine for small programs, but as a program grows, any function can touch any data, so one change can break things in many places.

**2. Object-oriented model ("who is being affected?")**

- The program is organized around _objects_. Each object bundles its own data together with the code allowed to work on that data. Think of it as _data controlling access to code_.
- The rest of the program can only use the object through a small set of approved methods (its interface).
- Result: changes stay local, so big programs stay manageable. This is why OOP was invented.

**Example:** a bank program.

- _Process-oriented:_ one global `balance` variable and separate functions `deposit()`, `withdraw()`, and so on. Any function can change `balance` in any way.
- _Object-oriented:_ a `BankAccount` object owns its `balance`, and the only way to change it is through its `deposit()` and `withdraw()` methods. Those methods can check the rules (no negative deposits, no overdraft).

### Abstraction: the idea underneath all of OOP

**In plain words:** abstraction means _using something without needing to know how it works inside_. You show the important "what it does" and hide the "how it does it".

**Why it exists:** the human brain can't hold thousands of details at once, so we group them into one simple concept. You think "car" and drive to the store. You don't think about the engine, transmission, and braking system. The car is one object with its own behavior.

**Hierarchical abstraction** stacks these layers:

- Level 1: the car is one object.
- Level 2: the car is made of subsystems (steering, brakes, sound system).
- Level 3: each subsystem is made of smaller parts (the sound system has a radio, a CD player, and so on).
- You work at whichever level you need and ignore the levels below.

**How it looks in a program:**

- A process-oriented program is a list of steps. An object-oriented program is a set of objects that send **messages** to each other, like "account, deposit 500". Each object decides for itself how to respond.
- Because other code only relies on the interface (the message), you can **rewrite or replace the inside of an object** without touching the code that uses it. You do the same with a car: swap the engine, and the driver still uses the pedals in the same way.
- That is what "replace or retire parts of an old system without fear" means. Clean interfaces let you change one part without breaking the rest.

**Abstraction vs. encapsulation** (a common interview question):

- _Abstraction_ is the **design idea**: show only what's necessary and hide the complexity.
- _Encapsulation_ is the **mechanism** Java gives you to do it: bundle data and methods in a class, and use `private`/`public` to control access.
- Short version: abstraction decides _what_ to hide, and encapsulation is _how_ you hide it.

## Structure

The three principles that every OOP language provides:

```
              ┌── Encapsulation  (bundle code + data, hide the insides)
OOP  ─────────┼── Inheritance    (reuse and specialize: superclass → subclass)
              └── Polymorphism   (one interface, many implementations)
```

|Principle|One-line meaning|Car analogy|
|---|---|---|
|Encapsulation|Keep data and its code together, and control access to them|The gear lever is the only way to affect the transmission. The turn signal can't|
|Inheritance|A new class gets everything from an existing class and adds its own extras|You can drive any car because they all share steering, brakes, and pedals|
|Polymorphism|The same call behaves differently depending on the object|Same brake pedal, whether ABS or ordinary brakes|

### Encapsulation

**In plain words:** wrap data and the methods that use it into one unit (a class), and don't let outside code reach in. Outsiders go through the public methods only.

**Why it matters:**

- _Safety:_ nobody can put the object into an invalid state, such as a negative balance.
- _Freedom to change:_ you can rewrite the inside later, and outside code doesn't break as long as the public methods stay the same.

**Key terms:**

- A **class** is the blueprint, a logical description of data and behavior. An **object** is a real instance created from that blueprint. Think of a cookie cutter (class) and the cookies (objects).
- The data in a class is called **instance variables**. The code is called **methods**. Together they are the **members** of the class.
- `private` means only code inside the same class can use it. `public` means anyone can use it. The public part is the class's **interface**.

```java
class BankAccount {
    private double balance;              // hidden: outsiders can't touch it directly

    public void deposit(double amount) { // controlled way in
        if (amount > 0) balance += amount;
    }
    public double getBalance() {         // controlled way out
        return balance;
    }
}
// account.balance = -999;   // compile error, because balance is private
```

**Design tip:** keep the public interface small. Expose only what users need, and keep the rest private. Public instance variables are legal but bad practice, because they break encapsulation.

### Inheritance

**In plain words:** a class can be built on top of another class. It automatically gets everything the parent has, and only needs to define what is _new or different_. It models "is-a" relationships: a Dog _is an_ Animal.

**Why it matters:**

- _Code reuse:_ write common behavior once in the parent instead of copying it into every child.
- _Growth stays manageable:_ each new subclass adds only its own bits, so complexity grows **linearly**, not explosively.

**Terms:** the general class is the **superclass** (parent), and the specialized class is the **subclass** (child). A subclass inherits from _all_ its ancestors up the **class hierarchy**.

**The book's example:** Animal → Mammal → Canine → Domesticus → Retriever → Labrador.

- Animal defines Age, Sex, Weight.
- Mammal adds Litter Size and Gestation Period.
- Canine adds Hunting Skills and Tail Length, and so on down the chain.
- A Labrador ends up with all of these without redefining them, and only adds its own field (AKC Certified?).

```java
class Animal {
    void eat() { System.out.println("eating"); }
}
class Dog extends Animal {              // Dog inherits eat()
    void bark() { System.out.println("woof"); }
}
// new Dog().eat();   // works: inherited from Animal
// new Dog().bark();  // works: Dog's own method
```

### Polymorphism

**In plain words:** "many forms". One name or interface can trigger different behavior depending on the situation. You call the same method, and the correct version runs for you.

**Book's example:** you need stacks for `int`, `float`, and `char`. The logic is identical, only the data type differs. Without polymorphism you'd write three sets of functions with three different names. With it, you write one set of stack methods that all share the same names, and Java picks the right one.

**Everyday analogy:** a dog's sense of smell is the same "interface", but the reaction differs. It smells a cat and chases it. It smells food and runs to the bowl.

**Two flavors** (this goes beyond the chapter, but interviewers ask it constantly):

- **Compile-time polymorphism (method overloading):** several methods with the same name but different parameters. The compiler picks the right one from the arguments.
- **Runtime polymorphism (method overriding):** a subclass replaces a parent's method. Which version runs is decided by the _actual object_ when the program is running.

```java
class Shape {
    void draw() { System.out.println("some shape"); }
}
class Circle extends Shape {
    @Override
    void draw() { System.out.println("circle"); }
}
Shape s = new Circle();   // a Shape variable holding a Circle object
s.draw();                 // prints "circle": the object's real type decides
```

### How the three work together

- **Inheritance** builds a reusable hierarchy of classes.
- **Encapsulation** lets you change a class's insides later without breaking the code that depends on it.
- **Polymorphism** lets code stay simple and general, because you work with the general type and the right specific behavior happens automatically.
- Together they give programs that are easier to scale, reuse, and maintain than the process-oriented style. The book says the car is the better analogy for programs, since a car is built from parts that use all three ideas.
- **Every Java program uses all three**, even a tiny one. The built-in class libraries are built on them.

## Mechanics / reference

### Your first program

```java
class Example {
    public static void main(String args[]) {
        System.out.println("This is a simple Java program.");
    }
}
```

**Two-step process: compile, then run**

```
Example.java --(javac)--> Example.class (bytecode) --(java Example)--> JVM runs it
```

- `javac` (the compiler) turns source code into **bytecode** in a `.class` file. Bytecode isn't machine code, so you can't run it directly.
- `java` (the launcher) starts the JVM, which loads the class and runs it. You give it the **class name** (`java Example`), not the file name. Writing `java Example.class` is wrong.
- One `.class` file is produced per class.

**File naming:**

- A Java source file is officially called a _compilation unit_. It holds one or more classes.
- The convention is that the file name matches the main class name exactly, including capitalization, because Java is **case-sensitive**.
- (Extra: this becomes a hard rule when the class is `public`. A public class must live in a file with the same name.)

**Why every word in `public static void main(String args[])` is there** (a classic interview question):

|Word|Why it's needed|
|---|---|
|`public`|The JVM is outside your class and needs permission to call `main()`. If it were `private`, the JVM couldn't reach it|
|`static`|The JVM calls `main()` **before any object exists**. `static` means "belongs to the class, no object needed"|
|`void`|`main()` doesn't return a value to the JVM|
|`main`|The fixed name the JVM looks for. It must be lowercase, because `Main` is a different name|
|`String args[]`|An array that receives command-line arguments. You don't need it in this example, but it must be declared|

**Compiler vs. JVM:** if you misspell `main` as `Main`, the code **still compiles**, because it is legal Java. The error appears only when you run it, when `java` can't find `main()`. A class without `main()` is fine as a library class. It just can't be the starting point.

**Other facts about `main()`:**

- It is just the _starting point_. A big program has many classes, and only one needs `main()`.
- Applets don't use `main()`, because the browser launches them.

**`System.out.println(...)`**

- `System` is a built-in class, `out` is the output stream connected to the console, and `println()` is a method that prints the text and then moves to a new line.
- `print()` does the same but **stays on the same line**.
- Every Java **statement** ends with `;`. Lines like `class Example {` and the `main` header are not statements, so they have no semicolon.

### Variables

```java
int num;                                  // declare: reserve a named memory location
num = 100;                                // assign
System.out.println("This is num: " + num); // + joins a String with the value
num = num * 2;                            // now 200
```

- A **variable** is a named memory location whose value can change. It must be **declared before use**.
- Declaration form is `type name;`. You can declare several of the same type at once: `int x, y;`.
- `=` **assigns**. `==` **compares**. Mixing them up is a very common bug.
- `+` with a String converts the other value to text and joins them. This is called concatenation.

### Control statements

**`if`**: runs code only when a condition is true.

```java
if (x < y) System.out.println("x is less than y");
```

The condition must be a boolean expression. Relational operators: `<`, `>`, `==`.

**`for`**: repeats code. It has three parts, `for (initialization; condition; iteration)`.

```java
for (x = 0; x < 10; x++)
    System.out.println("This is x: " + x);
```

1. _Initialization_ runs once at the start (`x = 0`).
2. _Condition_ is checked before **every** pass, including the first (`x < 10`). If false, the loop ends.
3. _Iteration_ runs after every pass (`x++`).

- `x++` increments by 1, and `x--` decrements by 1. Professional code uses `x++` instead of `x = x + 1`.

**Code blocks**

- `{ ... }` groups several statements into **one unit** that can be used wherever a single statement is allowed.
- Without braces, `if` and `for` control only **one** statement. Anything you want controlled together must go in a block.

```java
if (x < y) {   // both lines run, or neither does
    x = y;
    y = 0;
}
```

- Bug to watch for: without braces, only the first line belongs to the `if`. The second line always runs.

### Lexical issues: the building blocks of Java code

A Java program is made of whitespace, identifiers, literals, comments, operators, separators, and keywords.

- **Whitespace:** Java is _free-form_, so layout and indentation don't affect meaning. You just need at least one space, tab, or newline between tokens that aren't already separated by an operator or separator.
- **Identifiers** (names for classes, variables, methods):
    - Can contain letters, digits, `_`, and `$`.
    - **Cannot start with a digit** (it would look like a number).
    - **Case-sensitive**: `VALUE` and `Value` are different names.
    - Valid: `AvgTemp`, `count`, `a4`, `$test`, `this_is_ok`. Invalid: `2count`, `high-temp` (has `-`), `Not/ok` (has `/`).
- **Literals:** constant values written directly in code: `100` (integer), `98.6` (floating-point), `'X'` (character), `"This is a test"` (string).
- **Comments:** ignored by the compiler. Java has three kinds: `// single-line`, `/* multi-line */`, and `/** documentation */` (used to auto-generate HTML documentation).
- **Keywords:** Java has **50** reserved words (`class`, `public`, `static`, `if`, `for`, and so on). You **cannot use them as identifiers**.
    - `const` and `goto` are reserved but **not used** (no functionality).
    - `true`, `false`, and `null` are also reserved, but they are **literal values**, not keywords. You still can't use them as names.

### The Java class libraries

- Java is really **two things together**: the language itself, plus a huge set of built-in classes (the standard library) for I/O, strings, networking, graphics, and GUIs.
- `System` is one of these built-in classes, and it's included in every program automatically.
- A lot of what makes Java powerful comes from the library, so learning Java means learning its standard classes too.

## Pitfalls

- **Using `=` when you mean `==`.** `=` assigns a value and `==` compares. They look almost the same, but behave very differently.
- **Missing braces on `if`/`for`.** Without `{ }`, only the next single statement is controlled. The line after it always runs, no matter what.
- **`Main` vs. `main`.** The compiler accepts both. Only `java` fails at run time, because it can't find the entry point.
- **Running with `java Example.class`.** Wrong. Use `java Example`.
- **File name doesn't match the class name.** This causes errors, especially for public classes. Also remember that Java is case-sensitive.
- **Putting `;` on the wrong lines.** Statements need it. Class and method headers don't.
- **Thinking `main()` is required everywhere.** Only the class that starts the program needs it. Applets don't use it.
- **Making instance variables `public`.** It's legal but defeats encapsulation. Make data `private` and expose methods.
- **Mixing up abstraction and encapsulation.** Abstraction is the idea of hiding complexity. Encapsulation is the mechanism (class + access control) that implements it.

## Flashcards

- What are the two ways to organize a program, and how do they differ? :: Process-oriented (code acting on data, like C) and object-oriented (data controlling access to code). OOP scales better because each object protects its own data #card
- Why was OOP created? :: To manage the growing complexity of large programs, where any function could change any data in a process-oriented design #card
- What is abstraction? :: Hiding complex details and exposing only what's needed to use something, like driving a car without knowing how the engine works #card
- How is abstraction different from encapsulation? :: Abstraction is the design idea (show what, hide how). Encapsulation is the mechanism (bundle data and methods in a class, control access with `private`/`public`) #card
- Why does abstraction let you replace parts of a system safely? :: Other code depends only on the interface, so you can change the inside of an object without breaking the code that uses it #card
- What are the three OOP principles? :: Encapsulation, inheritance, polymorphism #card
- What is encapsulation, and what is its benefit? :: Bundling data with the methods that use it and controlling access. It prevents invalid changes and lets you change the internals without breaking users of the class #card
- Class vs. object? :: A class is a blueprint (logical). An object is a real instance created from it, like a cookie cutter and the cookies #card
- `public` vs. `private`? :: `public` members can be used from any code. `private` members can be used only inside their own class #card
- What is inheritance, and what relationship does it model? :: A subclass acquires the members of its superclass and adds its own. It models an "is-a" relationship (Dog is an Animal) #card
- Why does inheritance keep complexity growing linearly? :: Each subclass only adds what's new, and it doesn't interact unpredictably with most of the rest of the system #card
- What is polymorphism? :: "Many forms": one interface or method name works for a general class of actions, and the specific behavior is chosen for the situation #card
- Overloading vs. overriding? :: Overloading is the same method name with different parameters (chosen at compile time). Overriding is a subclass replacing a parent's method (chosen at run time by the object's actual type) #card
- What does `javac` produce, and what runs it? :: `javac` produces a `.class` file of bytecode. The `java` launcher starts the JVM, which runs it #card
- Why is `main()` `public`, `static`, and `void`? :: `public` so the JVM outside the class can call it. `static` because it runs before any object exists. `void` because it returns nothing to the JVM #card
- What happens if you write `Main` instead of `main`? :: It compiles fine, but `java` can't find the entry point and reports an error at run time #card
- What does the class name have to do with the file name? :: By convention they match exactly (Java is case-sensitive), and a `public` class must be in a file with its own name #card
- What does an `if` or `for` without braces control? :: Only the single next statement. Use `{ }` to group several statements into one block #card
- What are the three parts of a `for` loop, and when does each run? :: Initialization runs once. The condition is checked before every pass. The iteration expression runs after every pass #card
- What is `=` vs. `==`? :: `=` assigns a value. `==` compares two values for equality #card
- Which are valid identifiers: `2count`, `high-temp`, `$test`, `this_is_ok`? :: `$test` and `this_is_ok`. Identifiers can't start with a digit or contain `-`, and they are case-sensitive #card
- What is special about `const`, `goto`, `true`, `false`, and `null`? :: `const` and `goto` are reserved but unused. `true`, `false`, and `null` are reserved literal values. None of them can be used as identifiers #card
- What are the three kinds of comments in Java? :: `//` single-line, `/* */` multi-line, `/** */` documentation comment (generates HTML docs) #card

## Open questions

- [ ] When exactly does a public class's file name have to match, and what happens with several classes in one file?
- [ ] How does method overriding work under the hood: how does the JVM decide at run time which version to call?
- [ ] What is the difference between an abstract class and an interface, and when do you use each? (This extends the abstraction idea in Java code.)
- [ ] What does `static` mean beyond `main()`: static variables and static methods in general?

## Key terms

|Term|Definition|
|---|---|
|Process-oriented model|Program organized around code: a series of steps acting on data|
|Object-oriented programming|Program organized around objects that bundle data with the code allowed to use it|
|Abstraction|Hiding complexity and exposing only what's needed, layered hierarchically|
|Encapsulation|Bundling data and methods in a class and controlling access to protect them|
|Inheritance|A class acquiring the members of another class and adding its own|
|Polymorphism|One interface or method name with many behaviors, chosen by context|
|Class|A blueprint that defines the data and behavior shared by its objects|
|Object|A real instance of a class|
|Instance variable|Data stored inside each object of a class|
|Method|Code inside a class that works on its data (a "function" in C/C++)|
|Superclass / subclass|The parent class being inherited from / the child class that inherits|
|Access modifier|`public` or `private`: controls which code can use a member|
|Bytecode|Machine-independent instructions produced by `javac`, run by the JVM|
|`main()`|The entry point where a Java application starts running|
|Parameter|A variable in a method's parentheses that receives values passed in|
|Identifier|A name for a class, variable, or method|
|Literal|A constant value written directly in code, like `100` or `"text"`|
|Code block|Statements grouped in `{ }` and treated as a single unit|
|Class libraries|Built-in Java classes for I/O, strings, networking, graphics, and GUIs|

## Related

[[1 - The History and Evolution of Java]] · [[3 - Data Types, Variables, and Arrays]]

→ Next: [[3 - Data Types, Variables, and Arrays]]