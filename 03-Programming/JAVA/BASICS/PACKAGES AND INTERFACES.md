---
title: PACKAGES AND INTERFACES
date: 2026-10-08
language: Java
phase: phase-1
tags:
links: []
status: learning
---
> **One-line summary:** **Packages** are folders-for-classes that prevent name clashes and add a visibility level (default/package access, plus `protected`); the package name must match the directory layout, and `import` is only a typing shortcut. **Interfaces** declare _what_ a class must do without saying _how_; a class can `implements` many of them (Java's answer to multiple inheritance), methods implementing them must be `public`, and a variable of interface type calls the right version at run time (polymorphism again). Interfaces can hold constants, can `extend` other interfaces, and since JDK 8 can contain **`default`** methods (a body that classes may override, so an interface can grow without breaking old code) and **`static`** methods. When two interfaces supply the same default, the class must resolve the clash itself.

## What it is

Two features for organizing and abstracting larger programs:

|Topic|The question it answers|
|---|---|
|Package|How do I group classes and avoid two classes with the same name colliding?|
|`CLASSPATH`|How does the JVM find my packages on disk?|
|Access protection|Who may see a member now that packages exist?|
|`import`|Do I really have to type `java.util.Date` every time?|
|Interface|How do I specify _what_ a class must do, independent of _how_?|
|`implements`|How does a class promise to follow an interface?|
|Interface reference|How do I write code that works with any class that follows the interface?|
|Interface constants|How do I share constants across classes?|
|Extending interfaces|Can one interface build on another?|
|`default` methods|How do I add a method to a widely used interface without breaking every implementer?|
|Default-method conflicts|What if two interfaces give the same default method?|
|`static` interface methods|Can an interface carry a utility method?|

**Why packages exist:** until now every class lived in one shared name space, so each class needed a globally unique name. A package partitions that name space. You can have your own `List` class in your package without clashing with someone else's `List`. A package is both a **naming** mechanism and a **visibility-control** mechanism.

**Why interfaces exist:** an abstract class can only be extended once (single inheritance), and it pushes shared behavior up the class hierarchy. An interface lives in a **separate hierarchy**, so completely unrelated classes can implement the same interface, and one class can implement many interfaces.

## Structure

### 1. Defining a package

Put a `package` statement as the **first statement** in the source file. Every class in that file belongs to that package.

```java
package MyPack;     // general form: package pkg;
```

- No `package` statement → the class goes in the **default package** (unnamed). Fine for small demos, not for real applications. This is why you never needed packages so far.
- **Packages are mirrored by directories.** The `.class` files of `MyPack` must be in a directory named exactly `MyPack` (case-sensitive).
- Many files can declare the same package. The statement only says "this file's classes belong here", it doesn't exclude other files. Real packages are spread across many files.
- **Hierarchy:** separate levels with dots: `package pkg1.pkg2.pkg3;`. It must match the folder tree (`java.awt.image` → `java/awt/image/`).
- You can't rename a package without renaming the directory.

### 2. Finding packages: `CLASSPATH`

The JVM needs to know where the package directories are. It looks in three ways:

1. **Current working directory** is the default starting point. If the package folder is a subdirectory of where you run `java`, it is found.
2. The **`CLASSPATH` environment variable** lists one or more directories to search.
3. The **`-classpath` option** of `java` / `javac` gives the path for that run.

**The point that gets tested:** the class path must name the directory that **contains** the package folder, not the package folder itself.

```
Package folder:   C:\MyPrograms\Java\MyPack
Class path to use: C:\MyPrograms\Java        (NOT ...\MyPack)
```

### 3. A short package example

```java
// file: MyPack/AccountBalance.java
package MyPack;

class Balance {
    String name;  double bal;
    Balance(String n, double b) { name = n; bal = b; }
    void show() {
        if (bal < 0) System.out.print("--> ");
        System.out.println(name + ": $" + bal);
    }
}

class AccountBalance {
    public static void main(String args[]) {
        Balance current[] = new Balance[3];
        current[0] = new Balance("K. J. Fielding", 123.23);
        current[1] = new Balance("Will Tell", 157.02);
        current[2] = new Balance("Tom Jackson", -12.33);
        for (int i = 0; i < 3; i++) current[i].show();
    }
}
```

Run it from the directory **above** `MyPack`, using the **fully qualified** class name:

```
java MyPack.AccountBalance      // correct
java AccountBalance             // WRONG: the class now lives in a package
```

Because `AccountBalance` is in package `MyPack`, its real name is `MyPack.AccountBalance`.

### 4. Access protection (the table to memorize)

Packages add a new dimension to access control. Four situations now matter for a class member: same class, same package, subclass in a different package, anything else.

|Where the code is|`private`|(no modifier)|`protected`|`public`|
|---|---|---|---|---|
|Same class|Yes|Yes|Yes|Yes|
|Same package, subclass|No|Yes|Yes|Yes|
|Same package, non-subclass|No|Yes|Yes|Yes|
|Different package, subclass|No|**No**|Yes|Yes|
|Different package, non-subclass|No|No|No|Yes|

**How to remember it:**

- `public`: everywhere.
- `private`: only inside its own class.
- **default (no modifier)**: the **package** only. Subclasses in other packages do **not** get it.
- `protected`: the package **plus subclasses anywhere** (including other packages).

_(The book's loose sentence says default access is "visible to subclasses as well as to other classes in the same package". Trust the table: a subclass in a **different** package cannot see default members.)_

**Class-level access (not members):** a top-level class has only two levels: **default** (package-only) or **`public`** (anywhere). A `public` class must be the **only public class in its file**, and the file name must match the class name.

### 5. The access example (`p1` and `p2`)

`p1.Protection` has four fields, one per level: `n` (default), `n_pri` (private), `n_pro` (protected), `n_pub` (public).

|Class|Relationship to `Protection`|Can use|
|---|---|---|
|`p1.Derived`|Subclass, **same** package|`n`, `n_pro`, `n_pub` (not `n_pri`)|
|`p1.SamePackage`|Non-subclass, **same** package|`n`, `n_pro`, `n_pub` (not `n_pri`)|
|`p2.Protection2`|Subclass, **different** package|`n_pro`, `n_pub` (not `n`, not `n_pri`)|
|`p2.OtherPackage`|Non-subclass, **different** package|`n_pub` only|

Notice the one that surprises people: `Protection2` is a subclass and **still cannot** use `n`, because default access stops at the package boundary.

_(Beyond the chapter: a subtle `protected` rule. In a different package, a subclass may use the inherited protected member **through itself (`this`/its own type)**, not through an arbitrary reference to the superclass type. Writing `Protection p = new Protection(); p.n_pro` inside `Protection2` fails, while plain `n_pro` works.)_

### 6. Importing packages

All built-in Java classes live in named packages (the root is `java`). Writing the full name every time is tedious, so `import` brings classes into direct visibility.

```java
import java.util.Date;     // one class
import java.io.*;          // every class in the package (the * form)
```

**Where it goes:** right after the `package` statement (if any), before any class definition.

**Key facts:**

- `import` is **just a convenience**. You can always use the **fully qualified name** instead: `class MyDate extends java.util.Date { }`. It changes nothing at run time.
- `java.lang` (`String`, `System`, `Math`, ...) is **implicitly imported** into every program.
- `*` imports classes only from that package, not from its sub-packages (`import java.*;` does not give you `java.util`). _(Beyond the chapter.)_
- **Name clash:** if two star-imported packages both contain a class with the same name, the compiler stays quiet **until you use that name**. Then you get a compile error and must write the fully qualified name.
- Only `public` items of an imported package are usable by non-subclass code outside it.

**Making `Balance` usable outside `MyPack`:** the class, its constructor, and `show()` must all be `public`, and the class goes in its own file.

```java
package MyPack;
public class Balance {
    String name;  double bal;
    public Balance(String n, double b) { name = n; bal = b; }
    public void show() { /* ... */ }
}
```

```java
import MyPack.*;
class TestBalance {
    public static void main(String args[]) {
        Balance test = new Balance("J. J. Jaspers", 99.88);
        test.show();
    }
}
```

Remove `public` from `Balance` and `TestBalance` stops compiling. Making the class public is not enough: a non-public constructor or method would still be hidden.

### 7. Interfaces: the idea

An **interface** specifies **what** a class must do, not **how**. Syntactically similar to a class, but (traditionally) **no instance variables** and **no method bodies**.

- Any number of classes can implement one interface.
- One class can implement any number of interfaces.
- It is Java's fullest form of **"one interface, multiple methods"** polymorphism.
- Interface methods are resolved **dynamically at run time**, so a class written _later_ can plug into code written _earlier_, as long as it follows the interface.
- Because interfaces sit in a **different hierarchy** from classes, **unrelated classes** can share one. That is their real power.

### 8. Defining an interface

```java
access interface Name {
    return-type method1(parameter-list);     // no body, ends with ;
    type VAR = value;                        // constant
}
```

```java
interface Callback {
    void callback(int param);
}
```

**Rules:**

- **Access:** no modifier → **default** (package-only). `public` → anywhere; then it must be the only public interface in its file, with a matching file name.
- Methods have **no body** (they are implicitly **abstract**) and every implementing class must provide all of them.
- **Variables** in an interface are implicitly `public static final` (constants, must be initialized).
- **All members are implicitly `public`.**
- **JDK 8 change:** an interface may now give a method a **default implementation**, so it can specify some behavior too. The book treats it as a special-use feature and covers it at the end of the chapter. The traditional "pure contract" form is still the normal mental model.

### 9. Implementing interfaces

Use the `implements` clause, then write every required method.

```java
class classname [extends superclass] [implements interface1, interface2, ...] {
    // body
}
```

```java
class Client implements Callback {
    public void callback(int p) {                 // MUST be public
        System.out.println("callback called with " + p);
    }
    void nonIfaceMeth() { System.out.println("Extra members are allowed."); }
}
```

**Rules that get asked:**

1. The implementing method must be declared **`public`**. The interface method is implicitly public, and you cannot **weaken** access when implementing. Forgetting `public` is the most common compile error here.
2. The signature must **match exactly** (name, parameters, return type).
3. A class may add **extra members** of its own (like `nonIfaceMeth()`).
4. `extends` (one superclass) comes **before** `implements` (many interfaces).
5. If two interfaces declare the same method, one implementation serves both.

### 10. Accessing implementations through interface references

You can declare a variable whose **type is an interface**. It can refer to **any object whose class implements** that interface.

```java
Callback c = new Client();
c.callback(42);          // callback called with 42
// c.nonIfaceMeth();     // ERROR: Callback doesn't declare it
```

This is the same two-rule idea as in inheritance:

|Question|Decided by|When|
|---|---|---|
|Which **methods may I call** through `c`?|The **interface type** (`Callback`)|Compile time|
|Which **version runs**?|The **actual object** (`Client`, `AnotherClient`, ...)|Run time|

Polymorphism in action:

```java
class AnotherClient implements Callback {
    public void callback(int p) {
        System.out.println("Another version of callback");
        System.out.println("p squared is " + (p * p));
    }
}

Callback c = new Client();
c.callback(42);                  // callback called with 42
c = new AnotherClient();         // same variable, different object
c.callback(42);                  // Another version of callback / p squared is 1764
```

The calling code knows nothing about `Client` or `AnotherClient`. That is what lets you add new implementations later without touching it.

**Caution from the book:** dynamic lookup has overhead compared with a normal call, so don't use interfaces casually in performance-critical code. _(Beyond the chapter: on modern JVMs this cost is usually small. Treat it as a historical caveat, not a reason to avoid interfaces.)_

### 11. Partial implementations

If a class says `implements` but does not implement **all** the interface's methods, it **must be declared `abstract`**.

```java
abstract class Incomplete implements Callback {
    int a, b;
    void show() { System.out.println(a + " " + b); }
    // callback() not implemented
}
```

Any concrete subclass of `Incomplete` must implement `callback()` (or be abstract too). Same rule as abstract classes in the last chapter.

### 12. Nested interfaces

An interface can be a **member of a class or another interface**.

- Unlike a top-level interface (default or public only), a nested interface can be `public`, `private`, or `protected`.
- Outside its enclosing scope it must be referred to by its **qualified name** (`A.NestedIF`).

```java
class A {
    public interface NestedIF { boolean isNotNegative(int x); }
}

class B implements A.NestedIF {
    public boolean isNotNegative(int x) { return x < 0 ? false : true; }
}

A.NestedIF nif = new B();                 // interface reference to a B object
nif.isNotNegative(10);                    // true
nif.isNotNegative(-12);                   // false
```

### 13. Applying interfaces: the `IntStack` example

A stack's **interface** (`push`, `pop`) stays the same however it is stored (array, linked list, tree, fixed or growable). So define the interface once and let each implementation decide the details.

```java
interface IntStack {
    void push(int item);     // store an item
    int pop();               // retrieve an item
}
```

**Implementation 1, `FixedStack`:** fixed-size array. `push` prints "Stack is full." when `tos == stck.length - 1`; `pop` prints "Stack underflow." and returns 0 when `tos < 0`. Fields are `private`.

**Implementation 2, `DynStack`:** a growable stack. When full, `push` allocates a new array **twice as large**, copies the old elements across, and continues.

```java
public void push(int item) {
    if (tos == stck.length - 1) {                 // full → grow
        int temp[] = new int[stck.length * 2];    // double the size
        for (int i = 0; i < stck.length; i++) temp[i] = stck[i];
        stck = temp;
        stck[++tos] = item;
    } else
        stck[++tos] = item;
}
```

**Using both through one interface variable (`IFTest3`):**

```java
IntStack mystack;                     // interface reference
DynStack ds = new DynStack(5);
FixedStack fs = new FixedStack(8);

mystack = ds;                         // load the dynamic stack
for (int i = 0; i < 12; i++) mystack.push(i);   // grows as needed
// then mystack = fs; ... the same push/pop calls now hit FixedStack
```

The `push()` / `pop()` calls are resolved **at run time** from whichever object `mystack` currently refers to. Pushing 12 items works on `DynStack(5)`, while the same pushes on a `FixedStack(8)` would print "Stack is full." for the extra items.

### 13b. Finishing `IFTest3`: one variable, two implementations

```java
mystack = ds;                       // dynamic stack
for (int i = 0; i < 12; i++) mystack.push(i);

mystack = fs;                       // fixed stack
for (int i = 0; i < 8; i++) mystack.push(i);

mystack = ds;                       // pop from the dynamic one
for (int i = 0; i < 12; i++) System.out.println(mystack.pop());

mystack = fs;                       // pop from the fixed one
for (int i = 0; i < 8; i++) System.out.println(mystack.pop());
```

`mystack` is only an `IntStack` reference. Each time you re-point it, the same `push()`/`pop()` text runs a different implementation. The book calls this **the most powerful way Java achieves run-time polymorphism.** Both stacks keep their own contents, since they are separate objects.

### 14. Variables in interfaces (shared constants)

An interface can carry constants. A class that **implements** it gets those names in scope directly, as if it had defined them. (Like a C/C++ header full of `#define`/`const`.)

```java
interface SharedConstants {
    int NO = 0;  int YES = 1;  int MAYBE = 2;
    int LATER = 3;  int SOON = 4;  int NEVER = 5;
}

class Question implements SharedConstants {       // uses the constants
    int ask() { /* returns NO, YES, LATER, SOON or NEVER by probability */ }
}

class AskMe implements SharedConstants {          // uses the same constants
    static void answer(int result) {
        switch (result) {
            case NO:    System.out.println("No");    break;
            case YES:   System.out.println("Yes");   break;
            case MAYBE: System.out.println("Maybe"); break;
            /* ... */
        }
    }
}
```

- Fields are implicitly **`public static final`**, so the implementing class **cannot change** them, and they must be **initialized**.
- If the interface has **no methods**, implementing it forces the class to implement **nothing**. It just imports the constants.
- The example also uses `java.util.Random`; `nextDouble()` returns a pseudorandom value from 0.0 up to 1.0, so `(int)(100 * rand.nextDouble())` gives 0-99, which the code maps to percentage ranges.
- The book itself notes this technique is **controversial** and includes it "for completeness".

_(Beyond the chapter: it is considered poor style because it makes a class's constants part of its public API just by `implements`. Modern code puts constants in a class (`static final`) or uses an **`enum`**. You may still see this pattern in older code and exam questions.)_

### 15. Interfaces can be extended

One interface can inherit another with **`extends`** (same syntax as classes). A class implementing the sub-interface must implement **every method in the whole chain**.

```java
interface A { void meth1(); void meth2(); }
interface B extends A { void meth3(); }          // B = meth1 + meth2 + meth3

class MyClass implements B {                     // must implement all three
    public void meth1() { System.out.println("Implement meth1()."); }
    public void meth2() { System.out.println("Implement meth2()."); }
    public void meth3() { System.out.println("Implement meth3()."); }
}
```

Remove `meth1()` from `MyClass` and it will not compile, because `meth1()` is inherited through `B`.

_(Beyond the chapter: unlike classes, an interface may `extend` **several** interfaces: `interface C extends A, B`. This is legal and is another way Java gets "multiple inheritance" of types.)_

### 16. Default interface methods (JDK 8)

Before JDK 8, every interface method was abstract. A **default method** has a body and is marked with the keyword `default`. (Also called an **extension method** during development.)

```java
public interface MyIF {
    int getNumber();                         // normal abstract method
    default String getString() {             // default method, has a body
        return "Default String";
    }
}

class MyIFImp implements MyIF {              // only getNumber() is required
    public int getNumber() { return 100; }
}

class MyIFImp2 implements MyIF {             // may also override the default
    public int getNumber()  { return 100; }
    public String getString() { return "This is a different string."; }
}

MyIFImp obj = new MyIFImp();
obj.getNumber();     // 100
obj.getString();     // "Default String"   (default was used)
```

**Why they were added (two motivations):**

1. **Evolve an interface without breaking existing code.** If you add a new abstract method to a popular interface, every class that implements it stops compiling. A default method gives old classes a working version automatically.
2. **Optional methods.** A method like `remove()` may make no sense for a read-only sequence. A default that does nothing (or throws an exception) means implementers don't have to write placeholder code.

**Practical example:** adding `clear()` to `IntStack` without touching `FixedStack` or `DynStack`:

```java
interface IntStack {
    void push(int item);
    int pop();
    default void clear() {                                   // new, optional
        System.out.println("clear() not implemented.");
    }
}
```

Old implementations still compile and simply get this default. A new implementation can override `clear()` to really empty the stack. _(The book suggests that real code should throw `UnsupportedOperationException` here, which you can do after the exceptions chapter.)_

**What defaults do NOT change:**

- An interface **still cannot have instance variables**, so it **cannot hold state**. That is the defining difference between an interface and a class.
- You still **cannot instantiate** an interface. A class must implement it.
- Interfaces are still mainly for **what**, not **how**. Defaults are a special-purpose feature.

### 17. Multiple inheritance issues with default methods

Java still has **no multiple inheritance of classes**, and default methods don't change that (no state). But a class implementing two interfaces can now inherit **behavior** from both, so **name conflicts** become possible. Suppose `Alpha` and `Beta` both define `default void reset()`.

**The resolution rules (interview favorite):**

|Situation|Result|
|---|---|
|1. The **class** itself defines/overrides `reset()`|**The class wins.** Class implementation always beats an interface default, even if it implements both `Alpha` and `Beta`|
|2. Class implements `Alpha` and `Beta`, both have a default `reset()`, class does **not** override|**Compile error** (ambiguous)|
|3. `Beta extends Alpha`, both define `reset()`|**The more specific (inheriting) interface wins**: `Beta`'s version is used|
|4. You want a particular interface's version|Call it explicitly with `InterfaceName.super.methodName()`|

```java
interface Alpha { default void reset() { System.out.println("Alpha reset"); } }
interface Beta  { default void reset() { System.out.println("Beta reset"); } }

class MyClass implements Alpha, Beta {
    public void reset() {                    // required, otherwise error (rule 2)
        Alpha.super.reset();                 // explicitly pick Alpha's version (rule 4)
    }
}
```

The book's example of the explicit form: if `Beta` extends `Alpha` and wants Alpha's version, it writes `Alpha.super.reset();`. Note this is **not** the same as the plain `super.method()` from inheritance, which names the superclass. Here you name the **interface** before `.super`.

### 18. `static` methods in an interface (JDK 8)

An interface can define `static` methods, called by the **interface name**. No object and no implementing class is needed.

```java
public interface MyIF {
    int getNumber();
    default String getString() { return "Default String"; }
    static int getDefaultNumber() { return 0; }       // static interface method
}

int defNum = MyIF.getDefaultNumber();                 // call via the interface name
```

**Key rule:** static interface methods are **not inherited** by implementing classes or by sub-interfaces. `MyIFImp.getDefaultNumber()` or `obj.getDefaultNumber()` will not compile. You must always write `MyIF.getDefaultNumber()`.

_(Beyond the chapter: Java 9 added **`private`** interface methods (and private static), so default and static methods can share helper code without exposing it. The book's edition stops at default and static.)_

### 19. Final thoughts

Almost every real Java program lives inside packages, and many implement interfaces. Be comfortable with both: package layout and visibility on one side, "code to the interface" on the other.

## Mechanics / reference

### Package vs. class access

|Level|Applies to|Options|
|---|---|---|
|Top-level class / interface|The type itself|default (package) or `public`|
|Class member|Fields, methods, constructors|`private`, default, `protected`, `public`|
|Nested interface|Interface inside a class/interface|`public`, `private`, `protected`, (default)|

### Running a packaged class

|Situation|Command|
|---|---|
|Class `AccountBalance` in package `MyPack`, run from the folder above `MyPack`|`java MyPack.AccountBalance`|
|Package is elsewhere|Add `-classpath <dir containing MyPack>` or set `CLASSPATH`|

### Interface vs. abstract class (preview)

||Interface|Abstract class|
|---|---|---|
|Keyword to use it|`implements`|`extends`|
|How many can a class use|**Many**|**One**|
|Instance variables|No (only constants)|Yes|
|Method bodies|Traditionally none (default methods since JDK 8)|Any mix of abstract and concrete|
|Constructors|No|Yes|
|Can be instantiated|No|No|
|Can be a reference type|Yes|Yes|

### Implementing rules at a glance

|Rule|Detail|
|---|---|
|Implement **all** methods|Or declare the class `abstract`|
|Methods must be `public`|Implicitly public in the interface; can't narrow|
|Signature must match|Same name, parameters, return type|
|Order in the header|`extends` first, then `implements`|
|Interface reference can call|Only methods declared in the interface|

### What an interface can contain (JDK 8)

|Member|Has body?|Notes|
|---|---|---|
|Abstract method|No|Implicitly `public abstract`; classes must implement it (as `public`)|
|`default` method|Yes|Optional to override; inherited by implementers|
|`static` method|Yes|Called as `Interface.method()`; **not inherited**|
|Constant (variable)|n/a|Implicitly `public static final`, must be initialized|
|Instance variable|n/a|**Not allowed** (no state)|
|Constructor|n/a|**Not allowed**|
|Nested interface/class|n/a|Allowed (nested interface can be public/private/protected)|

### Interface vs. default-method conflict resolution

|Case|Winner|
|---|---|
|Class overrides the method|Class|
|Sub-interface overrides super-interface's default|Sub-interface|
|Two unrelated interfaces with the same default, class silent|**Error**, class must override|
|Need a specific one|`InterfaceName.super.method()`|

### Common patterns

```java
// 1. Package + public class in its own file (MyPack/Balance.java)
package MyPack;
public class Balance { public Balance(String n, double b) { } public void show() { } }

// 2. Using it from another package
import MyPack.*;

// 3. Code against the interface, not the class
IntStack s = new DynStack(5);       // swap in FixedStack later, rest of code unchanged

// 4. Implementing several interfaces
class Report extends Base implements Printable, Savable { }

// 5. Evolve an interface safely
interface IntStack { void push(int i); int pop(); default void clear() { /* optional */ } }

// 6. Resolve a default-method clash
public void reset() { Alpha.super.reset(); }
```

## Pitfalls

- **Package name doesn't match the directory** (or wrong case). The JVM can't find the class.
- **Running a packaged class by its short name.** `java AccountBalance` fails. Use `java MyPack.AccountBalance` from the folder above `MyPack`.
- **Putting the package folder itself on the class path.** The path must be the directory that **contains** the package folder.
- **`package` not the first statement**, or `import` placed before it. Order is: `package` → `import`s → classes.
- **Expecting default access to reach subclasses in other packages.** It doesn't. Use `protected` for that.
- **Assuming `protected` means "subclasses only".** It also includes the **whole package**.
- **Making a class `public` but leaving its constructor/methods package-private.** Code outside the package can see the class but can't use it.
- **Two `public` classes in one file**, or a file name that doesn't match the public class. Compile error.
- **Star-importing two packages with the same class name** (e.g. `java.util.*` and `java.sql.*` both have `Date`). Error appears only when you use `Date`. Fully qualify it.
- **Thinking `import` loads or links anything.** It only saves typing.
- **Forgetting `public` on an implementing method.** Compile error (attempting to assign weaker access).
- **Not implementing every interface method** in a non-abstract class. Compile error. Make the class `abstract` if the partial implementation is intended.
- **Calling a class-only method through an interface reference.** `c.nonIfaceMeth()` doesn't compile because `Callback` doesn't declare it.
- **Trying to put instance variables or constructors in an interface.** Not allowed. Variables are constants.
- **Trying to change an interface variable.** It is implicitly `final`.
- **Confusing "implements" and "extends" order.** `class X extends Y implements Z`, never the reverse.
- **Using a `public` interface in a file with a different name.** File name must match.
- **`DynStack` doubling cost.** Growing copies the whole array. Fine here, but know that it is O(n) for that push. _(Beyond the chapter: amortized O(1) overall, which is why doubling is the standard strategy.)_
- **Treating interface variables as changeable.** They are `public static final`. Assigning to them is a compile error.
- **Forgetting to initialize an interface variable.** Compile error.
- **Using the shared-constants-interface trick as a design habit.** Controversial; prefer a constants class or `enum`.
- **Not implementing inherited interface methods.** If `B extends A`, a class implementing `B` must implement methods of both.
- **Two interfaces with the same default method and no override in the class.** Compile error. Override and choose with `Alpha.super.reset()`.
- **Writing `super.reset()` instead of `Alpha.super.reset()`.** To reach an interface's default you must name the interface.
- **Expecting an interface default to override a class's own method.** It never does: the **class implementation always wins** over an interface default.
- **Calling a static interface method on an object or implementing class.** `obj.getDefaultNumber()` fails. Use `MyIF.getDefaultNumber()`.
- **Believing default methods give interfaces state or make them instantiable.** They don't: no instance variables, no `new`.
- **Thinking default methods remove the need to implement the interface's abstract methods.** Only the `default` ones are optional.
- **Adding an abstract method to a widely used interface.** Breaks every implementer. Add a `default` method instead.

## Flashcards

- What is a package? :: A container for classes that partitions the class name space and controls visibility #card
- Why do packages prevent name collisions? :: Two classes with the same simple name can live in different packages because their fully qualified names differ #card
- How do you put a class in a package? :: Make `package pkg;` the first statement in the source file #card
- What package does a class belong to if there is no `package` statement? :: The default (unnamed) package #card
- How must a package be stored on disk? :: In a directory with the exact same name (case-sensitive); a hierarchy `a.b.c` needs `a/b/c/` #card
- Can several files declare the same package? :: Yes; the package statement only says which package a file's classes belong to #card
- What are the three ways the JVM finds packages? :: Current working directory, the `CLASSPATH` environment variable, and the `-classpath` option #card
- If package `MyPack` is in `C:\MyPrograms\Java\MyPack`, what is the class path? :: `C:\MyPrograms\Java` (the directory containing the package, not the package itself) #card
- How do you run class `AccountBalance` in package `MyPack`? :: `java MyPack.AccountBalance` from the directory above `MyPack` #card
- What does default (no modifier) access allow? :: Access from the same class and the same package only #card
- What does `protected` allow? :: Access from the same class, the same package, and subclasses (even in other packages) #card
- Can a subclass in a different package access a default-access member? :: No #card
- What is the difference between default and protected access? :: Default stops at the package boundary; protected also reaches subclasses in other packages #card
- What access levels can a top-level class have? :: Only default (package-private) or public #card
- What must be true of a public class's file? :: It must be the only public class in the file and the file name must match the class name #card
- What does `import` do? :: Lets you refer to a class by its simple name instead of its fully qualified name; it is purely a convenience #card
- Is `import` required? :: No; you can always use the fully qualified name, e.g. `java.util.Date` #card
- Which package is imported automatically? :: `java.lang` #card
- What happens if two star-imported packages contain the same class name? :: No error until you use the name; then you must fully qualify it #card
- Where do `import` statements go? :: After the `package` statement and before any class definitions #card
- What must be `public` to use a class from another package? :: The class itself, plus any constructors and methods you want to call #card
- What is an interface? :: A type that declares what a class must do (method signatures and constants) without specifying how #card
- How is an interface different from an abstract class? :: A class can implement many interfaces but extend only one class; interfaces have no instance variables or constructors #card
- Why do interfaces allow polymorphism between unrelated classes? :: They live in a separate hierarchy, so any class can implement the same interface regardless of its superclass #card
- What are variables in an interface implicitly? :: `public static final` constants that must be initialized #card
- What access do interface methods have? :: Implicitly `public` #card
- What did JDK 8 add to interfaces? :: Default implementations for methods #card
- Which keyword does a class use to adopt an interface? :: `implements` #card
- What is the correct order of `extends` and `implements`? :: `class X extends Super implements I1, I2` #card
- Why must an implementing method be `public`? :: The interface declares it public, and an implementation cannot have weaker access #card
- Can a class that implements an interface add its own methods? :: Yes, extra members are allowed #card
- Can an interface reference call a class's extra methods? :: No, it only knows the methods declared in the interface #card
- Can an interface reference variable refer to an object? :: Yes, any object whose class implements that interface #card
- When is the interface method version chosen? :: At run time, based on the object the reference actually points to #card
- What must a class do if it implements an interface only partially? :: Declare itself `abstract` #card
- What is a nested (member) interface? :: An interface declared inside a class or another interface; it can be public, private, or protected #card
- How do you name a nested interface outside its enclosing class? :: With the qualified name, e.g. `A.NestedIF` #card
- Why is a stack a good use of an interface? :: `push`/`pop` stay the same no matter how the stack is stored, so fixed and growable versions share one interface #card
- How does `DynStack` grow? :: When full it allocates an array twice as large, copies the elements, and continues #card
- What does `IntStack mystack = ds;` allow? :: Calling `push`/`pop` through one variable that can refer to either `DynStack` or `FixedStack` #card
- What does the book call the most powerful way Java achieves run-time polymorphism? :: Accessing multiple implementations of an interface through an interface reference variable #card
- How can an interface share constants among classes? :: Declare initialized variables in the interface; implementing classes see them as constants #card
- What modifiers do interface variables implicitly have? :: `public static final` #card
- If an interface has only constants and no methods, what must an implementing class do? :: Nothing extra; it simply gets the constants in scope #card
- Why is the shared-constants interface technique considered controversial? :: It leaks constants into the implementing class's API; a constants class or `enum` is preferred #card
- How does one interface inherit another? :: With `extends`, e.g. `interface B extends A` #card
- What must a class implementing `B extends A` do? :: Implement all methods from both `B` and `A` #card
- What is a default method? :: An interface method with a body, declared with the `default` keyword (JDK 8) #card
- What is another name for a default method? :: Extension method #card
- What are the two motivations for default methods? :: Evolving interfaces without breaking existing code, and making some methods optional #card
- Must an implementing class override a default method? :: No; if it doesn't, the default is used #card
- What still can't an interface have even with default methods? :: Instance variables (state), and it still can't be instantiated directly #card
- Why was `clear()` added to `IntStack` as a default method? :: So existing implementations keep compiling without writing `clear()` #card
- Can default methods give Java multiple inheritance of classes? :: No; they give limited inheritance of behavior, but interfaces still hold no state #card
- Class and interface both provide the same method: which is used? :: The class implementation always takes priority #card
- A class implements two interfaces with the same default method and doesn't override it. What happens? :: Compile error #card
- If `Beta extends Alpha` and both define a default `reset()`, which does `Beta` use? :: Beta's own (the inheriting interface's version) #card
- How do you call a specific interface's default method? :: `InterfaceName.super.methodName()` #card
- How do you call a static interface method? :: Through the interface name: `InterfaceName.method()` #card
- Are static interface methods inherited by implementing classes? :: No, nor by sub-interfaces #card
- Does a static interface method need an instance or implementing class? :: No #card
- What is the key difference between an interface and a class even after JDK 8? :: A class can maintain state (instance variables); an interface cannot #card

## Open questions

- [ ] How do functional interfaces (one abstract method) tie into lambdas, and why do default/static methods not break that?
- [ ] What are `private` interface methods (Java 9) and when are they useful?
- [ ] What is the diamond problem, and why do default methods bring it back in a limited form?
- [ ] What are marker interfaces and common standard ones (`Comparable`, `Runnable`, `Serializable`)?
- [ ] How does the `protected` rule work precisely for access through a superclass reference in another package?
- [ ] How do `jar` files, the module system (`module-info.java`), and `import static` fit with packages?
- [ ] When should I choose an abstract class over an interface now that interfaces can have default methods?
- [ ] What is the reverse-domain naming convention (`com.company.project`) and why is it used?
- [ ] How do package-private classes help with encapsulation in real projects?

## Key terms

|Term|Definition|
|---|---|
|Package|Named container for classes (and sub-packages); also controls visibility|
|Default package|The unnamed package used when no `package` statement is given|
|Fully qualified name|Class name with its full package path, e.g. `java.util.Date`|
|`CLASSPATH`|Environment variable listing directories the JVM searches for packages|
|`-classpath`|`java`/`javac` option that sets the search path for one run|
|Default (package) access|No modifier; visible inside the same package only|
|`protected`|Visible in the same package and to subclasses in any package|
|`import`|Statement that brings classes or a whole package into direct visibility|
|`java.lang`|Core package imported automatically into every program|
|Interface|Contract of method signatures and constants with no implementation (traditionally)|
|`implements`|Clause by which a class agrees to provide an interface's methods|
|Interface reference|Variable whose type is an interface; can hold any implementing object|
|Partial implementation|A class that implements only some interface methods and must be `abstract`|
|Nested (member) interface|Interface declared inside a class or another interface|
|`IntStack`|The book's example interface with `push()` and `pop()`|
|`FixedStack` / `DynStack`|Two implementations of `IntStack`: fixed-size and doubling|
|Interface constant|Variable declared in an interface; implicitly `public static final`|
|Shared constants interface|Interface used only to hold constants for implementers (controversial)|
|Extending an interface|`interface B extends A`; B inherits A's methods|
|`default` method|Interface method with a body that implementers may override (JDK 8)|
|Extension method|Alternative name for a default method|
|`InterfaceName.super.method()`|Syntax for calling a specific interface's default implementation|
|`static` interface method|Utility method in an interface, called via the interface name and not inherited|
|Interface evolution|Adding functionality to an interface without breaking existing implementers|
|State|Instance data an object keeps; classes have it, interfaces don't|

## Related

← Previous: [[INHERITANCE]] → Next: [[10 - Exception Handling]]