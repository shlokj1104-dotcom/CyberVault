---
title: PACKAGES AND INTERFACES
date: 2026-10-08
language: Java
phase: phase-1
tags:
links: []
status: learning
---
> **One-line summary:** **Packages** group classes and stop name clashes; they also add the _default (package-private)_ access level, and the exam favorite is how **default vs. `protected`** differ across packages. **Interfaces** declare _what_ a class must do without saying _how_; a class can `implements` many of them, implementing methods must be `public`, and an **interface reference** runs the implementing object's version at run time. Since JDK 8 interfaces can have **`default`** and **`static`** methods; if two interfaces give the same default, the class must resolve the clash itself.



## Structure

### 1. Packages

A **package** is a named container for classes. It does two jobs:

1. **Naming:** two classes both called `List` can coexist if they live in different packages. Their _fully qualified names_ differ (`java.util.List` vs `myapp.List`).
2. **Visibility:** it creates a boundary for the **default access** level (see section 2).

```java
package MyPack;          // must be the FIRST statement in the file

public class Balance { /* ... */ }
```

**Rules to know:**

- No `package` statement → the class goes in the **default (unnamed) package**. Fine for practice programs, not for real projects.
- Package name must match the **folder structure**: `package a.b.c;` → folder `a/b/c/`.
- A class in a package must be run with its full name: `java MyPack.AccountBalance` (from the folder _above_ `MyPack`). Plain `java AccountBalance` fails. This is a very common beginner error.
- File order is always: `package` → `import`s → class.

_(For finding packages, the JVM uses the current directory, the `CLASSPATH` variable, or the `-classpath` option. The path must point to the folder that **contains** the package folder. In real projects your IDE or Maven/Gradle sets this, so just remember the idea.)_

### 2. Access levels with packages (interview favorite)

|Where the code is|`private`|(no modifier = default)|`protected`|`public`|
|---|---|---|---|---|
|Same class|Yes|Yes|Yes|Yes|
|Same package (subclass or not)|No|Yes|Yes|Yes|
|Different package, **subclass**|No|**No**|**Yes**|Yes|
|Different package, non-subclass|No|No|No|Yes|

**How to remember it:**

- `public` → everywhere.
- `private` → only inside the same class.
- **default** → the **same package** only.
- `protected` → the same package **plus subclasses in other packages**. This is the only difference between default and `protected`.

**Class-level access:** a top-level class can only be **default** or **`public`**. A `public` class must be the only public class in its file, and the file name must match the class name.

**Making a class usable from another package:** the class itself must be `public`, _and_ any constructor or method you want to call must be `public` too. Making only the class public is not enough.

### 3. `import`

`import` lets you use a class by its short name. It is **only a typing shortcut**: you can always write the fully qualified name instead, and it changes nothing at run time.

```java
import java.util.ArrayList;     // one class
import java.util.*;             // every class in java.util (not its sub-packages)
```

- `java.lang` (`String`, `System`, `Math`, `Integer`) is **imported automatically**.
- If two star-imported packages both have a class with the same name (e.g. `java.util.Date` and `java.sql.Date`), there is no error until you _use_ the name. Then you must write the full name.

### 4. Interfaces: the idea

An **interface** says **what** a class must do, not **how**. It lists method signatures (no bodies, traditionally) and constants.

```java
interface Callback {
    void callback(int param);       // no body, implicitly public abstract
}
```

**Why interfaces exist:**

- A class can extend only **one** superclass but can implement **many** interfaces. This is how Java gets around having no multiple inheritance of classes.
- Unrelated classes can implement the same interface, so code can be written against the interface and work with any of them.

**Rules:**

- Methods are implicitly **`public`** (and abstract). Variables are implicitly **`public static final`** (constants).
- **No instance variables, no constructors**, and you **can't `new` an interface**.
- A top-level interface is either default access or `public`.

### 5. Implementing an interface

```java
class Client implements Callback {
    public void callback(int p) {                  // MUST be public
        System.out.println("callback called with " + p);
    }
}
```

**Rules that get asked:**

1. The implementing method must be **`public`**. The interface method is public, and you can't reduce access. Forgetting `public` is the most common error.
2. Implement **all** the interface's methods, **or** declare the class `abstract`.
3. Header order: `class X extends Super implements I1, I2`.
4. A class can add extra methods of its own.

### 6. Interface references (run-time polymorphism)

A variable whose type is an interface can hold **any object whose class implements it**.

```java
Callback c = new Client();
c.callback(42);                      // Client's version

c = new AnotherClient();             // same variable, different implementation
c.callback(42);                      // AnotherClient's version
```

|Question|Decided by|When|
|---|---|---|
|Which methods can I call through `c`?|The **interface type** (`Callback`)|Compile time|
|Which version actually runs?|The **object's real class**|Run time|

So `c.nonIfaceMeth()` (a method only `Client` has) does not compile, even though the object has it. This is the same rule as superclass references from the inheritance chapter.

**The stack example (the book's main demo):** one interface, two implementations.

```java
interface IntStack { void push(int item); int pop(); }

class FixedStack implements IntStack { /* fixed-size array */ }
class DynStack  implements IntStack { /* array that doubles when full */ }

IntStack s = new DynStack(5);        // or new FixedStack(8): rest of the code unchanged
s.push(10);
```

The calling code never needs to know which stack it has. **Swapping an implementation without changing the code that uses it** is the whole point.

### 7. Extending interfaces

An interface can inherit another with `extends`. A class implementing the sub-interface must implement **all** methods in the chain.

```java
interface A { void meth1(); void meth2(); }
interface B extends A { void meth3(); }       // B = meth1 + meth2 + meth3
class MyClass implements B { /* must implement all three, all public */ }
```

_(An interface may extend several interfaces: `interface C extends A, B`.)_

### 8. Default methods (JDK 8)

A **default method** has a body and is marked `default`. Classes that implement the interface **may** override it, but don't have to.

```java
interface MyIF {
    int getNumber();                           // abstract: must implement
    default String getString() {              // default: optional
        return "Default String";
    }
}

class MyIFImp implements MyIF {
    public int getNumber() { return 100; }    // getString() inherited from the interface
}
```

**Why they were added (know both reasons):**

1. **Evolve an interface without breaking existing code.** Adding a new abstract method to a widely used interface would break every class that implements it. A default method gives them a working version automatically. (The book's example: adding `clear()` to `IntStack`.)
2. **Optional methods.** Implementers don't need placeholder code for methods they don't care about.

**What defaults do NOT change:** an interface still has **no instance variables** (it can't hold state), and still can't be instantiated. A class can keep state; an interface can't. That is the defining difference.

### 9. Default-method conflicts (interview favorite)

Suppose interfaces `Alpha` and `Beta` both have `default void reset()`.

|Situation|Result|
|---|---|
|The **class** defines its own `reset()`|**Class wins.** A class implementation always beats an interface default|
|Class implements both, **doesn't** override|**Compile error** (ambiguous)|
|`Beta extends Alpha`, both define `reset()`|The **more specific interface** (`Beta`) wins|
|You want one interface's version|`Alpha.super.reset();`|

```java
class MyClass implements Alpha, Beta {
    public void reset() {          // required, otherwise compile error
        Alpha.super.reset();       // explicitly choose Alpha's version
    }
}
```

Note the syntax: `InterfaceName.super.method()` names the **interface**, unlike the plain `super.method()` for a superclass.

### 10. `static` methods in an interface (JDK 8)

An interface can have `static` methods, called through the **interface name**.

```java
interface MyIF {
    static int getDefaultNumber() { return 0; }
}
int n = MyIF.getDefaultNumber();       // correct
```

They are **not inherited** by implementing classes or sub-interfaces, so `obj.getDefaultNumber()` does not compile.

_(Java 9 also added `private` interface methods to share helper code between default methods. Not in this chapter; just know it exists.)_

## Mechanics / reference

### Interface vs. abstract class

| |Interface|Abstract class|
|---|---|---|
|Used with|`implements` (**many** allowed)|`extends` (**one** only)|
|Instance variables / state|No (only constants)|Yes|
|Constructors|No|Yes|
|Method bodies|`default` / `static` only|Any mix|
|Can `new` it|No|No|
|Can be a reference type|Yes|Yes|
|Use when|Defining a **capability** unrelated classes can share (`Comparable`, `Runnable`)|Sharing **code and state** among closely related classes|

### What an interface can contain (JDK 8)

|Member|Body?|Notes|
|---|---|---|
|Abstract method|No|Implicitly `public`; implementers must write it as `public`|
|`default` method|Yes|Optional to override|
|`static` method|Yes|Called as `Interface.method()`; not inherited|
|Constant|n/a|Implicitly `public static final`|
|Instance variable, constructor|n/a|**Not allowed**|

### Common patterns

```java
// 1. Package + import
package myapp;
import java.util.*;

// 2. Code to the interface
List<Integer> list = new ArrayList<>();

// 3. Implementing an interface (note public)
class Dog implements Animal { public void speak() { } }

// 4. Resolve a default-method clash
public void reset() { Alpha.super.reset(); }
```

## Pitfalls

- **Default access assumed to reach subclasses in other packages.** It doesn't. Use `protected`.
- **Thinking `protected` means "subclasses only".** It also includes the whole package.
- **Running a packaged class by its short name.** Use `java MyPack.AccountBalance` from the folder above.
- **`package` not first** (or `import` before it). Order: `package` → `import` → class.
- **Public class but non-public constructor/method.** Other packages can see the class but can't use it.
- **Two public classes in one file**, or a file name that doesn't match the public class. Compile error.
- **Star-importing two packages that share a class name** (`java.util.*` and `java.sql.*` both have `Date`). Fully qualify it.
- **Forgetting `public` on a method that implements an interface.** Compile error (weaker access).
- **Not implementing every abstract method** in a concrete class. Compile error; make the class `abstract` if intended.
- **Calling a class-only method through an interface reference.** The interface type decides what you may call.
- **Trying to put instance variables or a constructor in an interface**, or changing an interface constant (`final`).
- **Two interfaces with the same default method, no override in the class.** Compile error. Override and pick with `Alpha.super.reset()`.
- **Expecting an interface default to override the class's own method.** Never: the class wins.
- **Calling a static interface method on an object.** Use `MyIF.method()`.
- **Believing default methods give interfaces state or allow `new`.** They don't.

## Flashcards

- What is a package? :: A named container for classes that prevents name clashes and controls visibility #card
- How do you put a class in a package? :: `package pkg;` as the first statement in the file #card
- What must a package name match on disk? :: The folder structure (`a.b.c` → `a/b/c/`), case-sensitive #card
- How do you run class `AccountBalance` in package `MyPack`? :: `java MyPack.AccountBalance` from the directory above `MyPack` #card
- What does default (no modifier) access allow? :: Access from the same class and the same package only #card
- What does `protected` allow? :: Same class, same package, and subclasses (even in other packages) #card
- What is the difference between default and `protected` access? :: Default stops at the package boundary; `protected` also reaches subclasses in other packages #card
- What access levels can a top-level class have? :: Only default or `public` #card
- What must be `public` to use a class from another package? :: The class, plus the constructors/methods you call #card
- What does `import` do? :: Lets you use a class's short name; it's only a convenience and the fully qualified name always works #card
- Which package is imported automatically? :: `java.lang` #card
- Does `import java.util.*;` import sub-packages? :: No, only classes directly in `java.util` #card
- What is an interface? :: A type that declares what a class must do (method signatures and constants) without how #card
- How is an interface different from an abstract class? :: A class can implement many interfaces but extend only one class; interfaces have no instance variables or constructors #card
- What are interface methods and variables implicitly? :: Methods `public abstract`; variables `public static final` #card
- Which keyword does a class use to adopt an interface? :: `implements` #card
- Why must an implementing method be `public`? :: The interface method is public and an implementation can't reduce access #card
- What must a class do if it doesn't implement all interface methods? :: Be declared `abstract` #card
- Correct header order for extending and implementing? :: `class X extends Super implements I1, I2` #card
- Can an interface reference variable refer to an object? :: Yes, any object whose class implements the interface #card
- What decides which methods you can call through an interface reference? :: The interface type, at compile time #card
- What decides which version runs? :: The object's actual class, at run time #card
- Why is `List<Integer> l = new ArrayList<>();` good practice? :: It codes to the interface, so the implementation can be swapped by changing one word #card
- How does one interface inherit another? :: With `extends`; implementers must implement the whole chain #card
- What is a default method? :: An interface method with a body, declared with `default` (JDK 8) #card
- What are the two motivations for default methods? :: Evolving interfaces without breaking existing code, and making methods optional #card
- Must an implementing class override a default method? :: No; if it doesn't, the default is used #card
- What still can't an interface have after JDK 8? :: Instance variables (state), constructors, and direct instantiation #card
- Class and interface both provide the same method. Which is used? :: The class implementation always wins #card
- Class implements two interfaces with the same default method and doesn't override it. What happens? :: Compile error #card
- If `Beta extends Alpha` and both define a default `reset()`, which does `Beta` use? :: Beta's own version #card
- How do you call a specific interface's default method? :: `InterfaceName.super.methodName()` #card
- How do you call a static interface method? :: `InterfaceName.method()` #card
- Are static interface methods inherited? :: No, neither by implementing classes nor by sub-interfaces #card

## Open questions

- [ ] When do I choose an abstract class over an interface now that interfaces have default methods?
- [ ] What is a functional interface, and how does it connect to lambdas?
- [ ] What is the diamond problem, and why do default methods bring it back in a limited form?
- [ ] What are the main collection interfaces (`Collection`, `List`, `Set`, `Queue`, `Map`) and how do they relate?

## Key terms

|Term|Definition|
|---|---|
|Package|Named container for classes; prevents name clashes and controls visibility|
|Default package|The unnamed package used when no `package` statement is given|
|Fully qualified name|Class name with its package path, e.g. `java.util.Date`|
|Default (package) access|No modifier; visible inside the same package only|
|`protected`|Visible in the same package and to subclasses in any package|
|`import`|Brings classes into direct visibility; only a typing convenience|
|`java.lang`|Core package imported automatically into every program|
|Interface|A contract of method signatures (and constants) classes can implement|
|`implements`|Clause by which a class agrees to provide an interface's methods|
|Interface reference|Variable whose type is an interface; holds any implementing object|
|`default` method|Interface method with a body that implementers may override (JDK 8)|
|`InterfaceName.super.method()`|Calls a specific interface's default implementation|
|`static` interface method|Utility method called via the interface name; not inherited|
|Interface evolution|Adding functionality to an interface without breaking implementers|
|State|Instance data an object keeps; classes have it, interfaces don't|

## Related

→ Next: [[10 - Exception Handling]]