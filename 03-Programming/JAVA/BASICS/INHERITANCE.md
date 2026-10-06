---
title: INHERITANCE
date: 2026-10-03
language: Java
phase: phase-1
tags:
links: []
status: learning
---
> **One-line summary:** Inheritance lets a **subclass** reuse and extend a **superclass** (`extends`). The things that matter most are what a subclass can and cannot access (`private` stays hidden), the two uses of `super` (call the parent's constructor; reach a hidden member), constructors running **parent first**, **overriding** vs. overloading, **dynamic method dispatch** (the _object's_ type picks the method at run time, which is how Java does run-time polymorphism), **abstract** classes, **`final`** to stop overriding/inheriting, and the **`Object`** class at the top of everything.

## What it is

**Inheritance** lets you build a new class from an existing one instead of starting from scratch.

- The class being inherited from is the **superclass** (parent, base class).
- The class doing the inheriting is the **subclass** (child, derived class).
- The subclass gets **all the members** of the superclass and can **add its own** (new fields, new methods) or **change** inherited behavior (overriding).

Analogy: a general class `Vehicle` holds what every vehicle has (speed, `start()`). `Car` and `Truck` extend it, reuse all of that, and add only what is specific to them. This is an **"is-a"** relationship: a `Car` _is a_ `Vehicle`.

Why it matters:

1. **Code reuse.** Write the common part once.
2. **Hierarchical design.** General → specific.
3. **Run-time polymorphism.** One piece of code can work with many related types (the big idea of this chapter).

## Structure

### 1. Inheritance basics: `extends`

```java
class A {                      // superclass
    int i, j;
    void showij() { System.out.println("i and j: " + i + " " + j); }
}

class B extends A {            // subclass: inherits i, j, showij()
    int k;
    void showk() { System.out.println("k: " + k); }
    void sum()   { System.out.println("i+j+k: " + (i + j + k)); }   // i and j used directly
}
```

```java
B subOb = new B();
subOb.i = 7; subOb.j = 8; subOb.k = 9;   // i, j inherited; k is B's own
subOb.showij();                           // inherited method
subOb.sum();                              // 24
```

**Rules:**

- General form: `class SubClass extends SuperClass { ... }`.
- **Only one superclass** per class. Java has **no multiple inheritance of classes**. (Interfaces, a later chapter, cover that need.)
- Hierarchies can be **multiple levels** deep: `A → B → C`. `C` gets everything from `B` and `A`.
- **A class can't be its own superclass** (no cycles).
- The superclass remains a normal, usable class by itself. Having children doesn't change it.

### 2. Member access and inheritance

A subclass **includes** all superclass members, but it can **use** only the ones it is _allowed_ to see. **`private` members are inherited but not accessible** from the subclass.

```java
class A {
    int i;             // default access
    private int j;     // private to A
    void setij(int x, int y) { i = x; j = y; }
}

class B extends A {
    int total;
    void sum() { total = i + j; }   // ERROR: j is private in A
}
```

> **Remember:** a `private` member stays private to its own class, even from subclasses.

**The way around it:** the subclass uses `public`/`protected` methods of the parent (like `setij()`), or the parent provides getters. This is how encapsulation survives inheritance.

_(Beyond the chapter: `protected` is the middle ground: visible to the subclass (and the same package) but not to unrelated code. Note the book's example comment "public by default" for `int i;` really means **default (package) access**.)_

### 3. A practical example: `Box` and `BoxWeight`

```java
class BoxWeight extends Box {
    double weight;
    BoxWeight(double w, double h, double d, double m) {
        width = w; height = h; depth = d;   // set Box's fields
        weight = m;
    }
}
```

`BoxWeight` gets `width`, `height`, `depth`, and `volume()` from `Box` for free, and adds `weight`. You can create many specialized subclasses (e.g. `ColorBox` adds `color`) from the same `Box`. That is the point of inheritance.

**Problem with this version:** it duplicates `Box`'s initialization, and it only works because `Box`'s fields are accessible. If `Box` makes them `private` (good encapsulation), `BoxWeight` can't set them. That is why `super` exists.

### 4. A superclass variable can reference a subclass object

```java
BoxWeight weightbox = new BoxWeight(3, 5, 7, 8.37);
Box plainbox = new Box();

plainbox = weightbox;           // OK: a BoxWeight IS A Box
plainbox.volume();              // OK: volume() is defined in Box
// plainbox.weight;             // ERROR: Box has no member called weight
```

**Key rule:** **the type of the reference variable decides which members you can access**, not the type of the object it points to. `plainbox` is a `Box` reference, so you only see what `Box` declares, even though the real object is a `BoxWeight`. (The superclass knows nothing about what subclasses add.)

```
plainbox (type Box) ──►  [ BoxWeight object: width, height, depth, weight ]
                          you can only "see" the Box part through plainbox
```

The reverse is not allowed without a cast: you can't assign a `Box` reference to a `BoxWeight` variable directly. This idea powers dynamic dispatch below.

### 5. Using `super`

`super` refers to the **immediate superclass**. It has **two uses**.

#### Use 1: `super(args)` calls the superclass constructor

```java
class BoxWeight extends Box {
    double weight;
    BoxWeight(double w, double h, double d, double m) {
        super(w, h, d);      // runs Box(double, double, double)
        weight = m;
    }
}
```

- `super(...)` **must be the first statement** in the subclass constructor.
- It lets the subclass initialize inherited data **without needing access** to it. `Box` can now make `width`, `height`, `depth` `private`.
- Which parent constructor runs is chosen by **the arguments you pass** (overload resolution again).
- A complete version has `BoxWeight` constructors that each call a matching `super(...)`:

```java
BoxWeight(BoxWeight ob)            { super(ob);          weight = ob.weight; }  // clone
BoxWeight(double w,double h,double d,double m) { super(w,h,d); weight = m; }
BoxWeight()                        { super();            weight = -1; }
BoxWeight(double len, double m)    { super(len);         weight = m; }          // cube
```

- Look at `super(ob)` where `ob` is a **`BoxWeight`**: it matches `Box(Box ob)` because **a subclass object can be passed where the superclass type is expected** (section 4). `Box` only copies its own fields.

#### Use 2: `super.member` reaches a hidden superclass member

When a subclass declares a field or method with the **same name** as the parent's, the subclass's version **hides** the parent's. `super.name` reaches the parent's.

```java
class A { int i; }
class B extends A {
    int i;                          // hides A's i
    B(int a, int b) {
        super.i = a;                // A's i
        i = b;                      // B's i
    }
    void show() {
        System.out.println("i in superclass: " + super.i);   // 1
        System.out.println("i in subclass: " + i);           // 2
    }
}
```

`super` here works like `this`, except it refers to the superclass part of the object.

### 6. Multilevel hierarchy

`Box → BoxWeight → Shipment`. `Shipment` adds `cost` and inherits everything above.

```java
class Shipment extends BoxWeight {
    double cost;
    Shipment(double w, double h, double d, double m, double c) {
        super(w, h, d, m);    // calls BoxWeight's constructor, which calls Box's
        cost = c;
    }
}
```

- `super(...)` always calls the constructor of the **closest (immediate)** superclass. `Shipment`'s `super` → `BoxWeight`; `BoxWeight`'s `super` → `Box`.
- If a superclass constructor **requires parameters**, **every subclass must pass them up the chain**, even if the subclass itself needs no extra data.
- In real projects each class normally goes in its own file. The book puts them together only for convenience.

### 7. When constructors run

Constructors execute in **order of derivation: superclass first, then subclass**.

```java
class A { A() { System.out.println("Inside A's constructor."); } }
class B extends A { B() { System.out.println("Inside B's constructor."); } }
class C extends B { C() { System.out.println("Inside C's constructor."); } }

new C();
// Inside A's constructor.
// Inside B's constructor.
// Inside C's constructor.
```

**Why parent first:** the superclass knows nothing about its subclasses, and its setup may be a prerequisite for the subclass's setup. So it must finish first.

**Important detail:** this order holds **even if you don't write `super()`**. If you leave it out, the compiler inserts an implicit `super();` (a call to the parent's **no-argument** constructor) as the first line.

_(Beyond the chapter: that implicit call causes a very common compile error. If the parent defines only a constructor **with parameters** (so there is no no-arg constructor), then a subclass constructor that doesn't call `super(args)` explicitly will not compile: "constructor ... in class ... cannot be applied". Also, you can't use `this(...)` and `super(...)` together, since each must be the first statement.)_

### 8. Method overriding

A subclass method **overrides** a superclass method when it has the **same name and same signature** (same parameter types and order). When called on a subclass object, the **subclass version runs**; the superclass version is hidden.

```java
class A {
    int i, j;
    A(int a, int b) { i = a; j = b; }
    void show() { System.out.println("i and j: " + i + " " + j); }
}

class B extends A {
    int k;
    B(int a, int b, int c) { super(a, b); k = c; }
    void show() { System.out.println("k: " + k); }     // OVERRIDES A.show()
}

new B(1, 2, 3).show();      // k: 3
```

**Calling the parent's version from the override** with `super.method()`:

```java
void show() {
    super.show();                                  // A's show(): "i and j: 1 2"
    System.out.println("k: " + k);                 // then k: 3
}
```

#### Overriding vs. overloading (classic interview question)

||Overloading|Overriding|
|---|---|---|
|Where|Same class (or sub/super)|Subclass vs. superclass|
|Signature|**Different** parameters|**Identical** name and parameters|
|Resolved|At compile time (by argument types)|At run time (by actual object type)|
|Purpose|Several ways to call "the same action"|Replace inherited behavior|

If the signatures differ, it is just overloading, not overriding:

```java
class B extends A {
    void show(String msg) { System.out.println(msg + k); }   // different signature → OVERLOAD
}
subOb.show("This is k: ");   // B's show(String)
subOb.show();                // A's show() is still there
```

_(Beyond the chapter: put `@Override` above an overriding method. The compiler then errors if you didn't really override anything, which catches typos and signature mismatches. Rules: the overriding method can't have **weaker access** than the original, and its return type must be the same (or a subtype). `static` methods are **hidden**, not overridden.)_

### 9. Dynamic method dispatch (run-time polymorphism)

This is the most important idea in the chapter. **Dynamic method dispatch** means a call to an overridden method is resolved **at run time**, not compile time.

**The rule:** when you call an overridden method through a **superclass reference**, Java picks the version based on the **type of the object actually being referred to**, not the type of the reference variable.

```java
class A { void callme() { System.out.println("Inside A's callme method"); } }
class B extends A { void callme() { System.out.println("Inside B's callme method"); } }
class C extends A { void callme() { System.out.println("Inside C's callme method"); } }

A a = new A();  B b = new B();  C c = new C();
A r;                    // one superclass reference

r = a;  r.callme();     // Inside A's callme method
r = b;  r.callme();     // Inside B's callme method
r = c;  r.callme();     // Inside C's callme method
```

The same line `r.callme()` runs three different methods, depending on what `r` currently points to. If the **reference type** decided it, you'd get `A`'s version three times.

**Two rules, side by side (this trips people up):**

|Question|Decided by|When|
|---|---|---|
|Which **members may I use** through this variable?|The **reference type**|Compile time|
|Which **version of an overridden method runs**?|The **object's actual type**|Run time|

_(C++/C# readers: overridden methods in Java behave like **virtual functions**, and in Java every instance method is "virtual" by default.)_

#### Why overriding matters

It lets a general class define **what methods all its subclasses have**, while each subclass supplies **how**. This is Java's "one interface, multiple methods" form of polymorphism. Code written against the superclass keeps working when new subclasses are added later, with no changes.

#### Applying it: `Figure`, `Rectangle`, `Triangle`

```java
class Figure {
    double dim1, dim2;
    Figure(double a, double b) { dim1 = a; dim2 = b; }
    double area() { System.out.println("Area for Figure is undefined."); return 0; }
}
class Rectangle extends Figure {
    Rectangle(double a, double b) { super(a, b); }
    double area() { return dim1 * dim2; }          // override
}
class Triangle extends Figure {
    Triangle(double a, double b) { super(a, b); }
    double area() { return dim1 * dim2 / 2; }      // override
}

Figure figref;
figref = new Rectangle(9, 5);  figref.area();   // 45
figref = new Triangle(10, 8);  figref.area();   // 40
figref = new Figure(10, 10);   figref.area();   // "undefined", 0
```

One reference type (`Figure`), one call (`area()`), different behavior per object. The caller doesn't need to know which kind of figure it has.

### 10. Abstract classes and methods

Sometimes a superclass can't give a **meaningful** implementation. `Figure.area()` above prints "undefined", which is only a placeholder. An **abstract method** forces subclasses to supply the real one.

```java
abstract class Figure {                    // abstract class
    double dim1, dim2;
    Figure(double a, double b) { dim1 = a; dim2 = b; }
    abstract double area();                // abstract method: NO body, ends with ;
}
```

**Rules:**

1. An **abstract method** has **no body**: `abstract type name(params);`.
2. **Any class with an abstract method must itself be declared `abstract`.**
3. You **cannot create objects** of an abstract class with `new`. (`new Figure(10, 10)` is a compile error.)
4. A subclass must **override all** inherited abstract methods, **or** be declared `abstract` itself. Otherwise compile error.
5. You **cannot** declare **abstract constructors** or **abstract `static` methods**.
6. An abstract class **can contain normal (concrete) methods and fields**, as much implementation as it wants.
7. You **can** declare **reference variables** of an abstract type. That is exactly what makes run-time polymorphism work:

```java
Figure figref;            // OK: just a reference, no object created
figref = new Rectangle(9, 5);
```

```java
abstract class A {
    abstract void callme();                          // subclasses must implement
    void callmetoo() { System.out.println("This is a concrete method."); }
}
class B extends A {
    void callme() { System.out.println("B's implementation of callme."); }
}
```

**When to use abstract:** when a class is a general idea (shape, animal, vehicle) that makes no sense as a standalone object but defines a contract for its children.

### 11. Using `final` with inheritance

`final` has three uses: (1) constants (Chapter 7); (2) prevent **overriding**; (3) prevent **inheritance**.

#### `final` method: cannot be overridden

```java
class A { final void meth() { System.out.println("This is a final method."); } }
class B extends A { void meth() { } }     // COMPILE ERROR: can't override
```

Use it when the method's behavior must never change in subclasses (e.g. something security- or correctness-critical).

**Early vs. late binding:** normally Java resolves calls **dynamically at run time** (**late binding**). A `final` method can't be overridden, so the call can be resolved at **compile time** (**early binding**), and the compiler may **inline** small final methods for speed. _(Beyond the chapter: modern JVMs optimize this well on their own, so don't use `final` just for speed.)_

#### `final` class: cannot be extended

```java
final class A { }
class B extends A { }     // COMPILE ERROR: can't subclass A
```

- All methods of a `final` class are implicitly `final`.
- A class **can't be both `abstract` and `final`**. Abstract means "must be extended"; final means "can't be extended". They contradict each other.
- _(Beyond the chapter: `String` is a `final` class, which is one reason strings are safe to rely on.)_

### 12. The `Object` class

**Every class in Java is a subclass of `Object`**, directly or indirectly. If you write `class A { }` with no `extends`, Java treats it as `class A extends Object`. So:

- An `Object` reference can refer to **any object** (and any array).
- Every object has `Object`'s methods:

|Method|Purpose|
|---|---|
|`boolean equals(Object obj)`|Is this object "equal" to another?|
|`String toString()`|A string describing the object|
|`int hashCode()`|Hash code of the object|
|`Object clone()`|Creates a copy of the object|
|`Class<?> getClass()`|The object's class at run time|
|`void finalize()`|Called before garbage collection|
|`wait()`, `notify()`, `notifyAll()`|Thread coordination (later chapters)|

- `getClass()`, `notify()`, `notifyAll()`, `wait()` are `final` (can't override). The rest can be overridden.

**Two to know now:**

- **`equals()`** returns `true` if the objects are "equal". What _equal_ means depends on the class, so you override it to define it for your own type.
- **`toString()`** returns a text description. It is **called automatically by `println()`** (and string concatenation) when you print an object. Many classes override it to print something meaningful.

_(Beyond the chapter, very commonly asked:_

- _The default `equals()` is just `==` (same reference), and the default `toString()` prints something like `ClassName@1b6d3586`. Override both to get useful behavior._
- _**If you override `equals()`, also override `hashCode()`** so equal objects have equal hash codes. Otherwise hash-based collections (`HashMap`, `HashSet`) misbehave._
- _For `String`, `equals()` is already overridden to compare contents, which is why `.equals()` works for strings.)_

## Mechanics / reference

### `super` at a glance

|Form|What it does|Rules|
|---|---|---|
|`super(args);`|Calls a superclass constructor|Must be the **first** statement in a constructor; refers to the **immediate** parent|
|`super.field`|Accesses a hidden superclass field|Same-named field in the subclass hides the parent's|
|`super.method()`|Calls the superclass version of an overridden method|Use inside the overriding method|

### Who can see what

|Member in superclass|Accessible in subclass?|
|---|---|
|`public`|Yes|
|`protected`|Yes|
|default (package)|Yes if same package|
|`private`|**No** (inherited, but hidden). Use getters/setters.|

### Constructor execution order

```
new C()   →   A() body   →   B() body   →   C() body
              (top of chain first, then down to the actual class)
```

### `final` summary

|Applied to|Meaning|
|---|---|
|Variable|Value can't change (constant)|
|Method|Can't be **overridden**|
|Class|Can't be **inherited** (subclassed)|

### `abstract` summary

|Item|Rule|
|---|---|
|Abstract method|No body; must be overridden|
|Abstract class|Can't be instantiated; may have both abstract and concrete members|
|Subclass of abstract class|Implements all abstract methods, or is itself abstract|
|Can't combine with|`final` (and `private`/`static` for methods)|

### Common patterns

```java
// 1. Subclass constructor passes data up
class Dog extends Animal {
    Dog(String name) { super(name); }
}

// 2. Extend, don't replace, parent behavior
@Override
void show() { super.show(); System.out.println("extra"); }

// 3. Polymorphic collection
Figure[] shapes = { new Rectangle(9, 5), new Triangle(10, 8) };
for (Figure f : shapes) System.out.println(f.area());   // each calls its own area()

// 4. Meaningful printing
@Override
public String toString() { return "Box[" + width + "x" + height + "x" + depth + "]"; }
```

## Pitfalls

- **Using a parent's `private` field in a subclass.** Compile error. Use a getter or a `protected` field.
- **Multiple inheritance.** `class C extends A, B` is illegal. A class has one superclass.
- **`super(...)` not first.** It must be the first statement in the constructor.
- **Parent has no no-arg constructor, and the subclass constructor doesn't call `super(args)`.** Compile error (the implicit `super()` has nothing to call).
- **Thinking `private` members aren't inherited.** They exist in the subclass object. They just can't be accessed directly by name.
- **Accessing a subclass member through a superclass reference.** `plainbox.weight` is an error even though the object is a `BoxWeight`. The reference type limits what you can see.
- **Confusing overriding and overloading.** Same name + **same** parameters = override. Same name + **different** parameters = overload. A typo in a parameter type silently creates an overload instead of overriding. Use `@Override`.
- **Expecting field access to be polymorphic.** Only **methods** are dispatched by object type. A field accessed through a superclass reference gives the **superclass's** field. _(Beyond the chapter.)_
- **Weakening access when overriding.** A `public` method can't be overridden as package-private or `private`. _(Beyond the chapter.)_
- **Forgetting `super.method()` when you meant to extend behavior.** Without it, the parent's logic is replaced, not added to.
- **Trying to instantiate an abstract class.** `new Figure(...)` fails. Instantiate a concrete subclass.
- **Subclass of an abstract class missing an override.** Compile error unless the subclass is also `abstract`.
- **`abstract final` together.** Illegal.
- **Overriding a `final` method, or extending a `final` class.** Compile error.
- **Overriding `equals()` without `hashCode()`.** Breaks `HashMap`/`HashSet`.
- **Comparing objects with `==` and expecting content equality.** Use `equals()` (and override it for your classes).
- **Calling an overridable method from a constructor.** The subclass override may run before the subclass's fields are initialized (because the parent constructor runs first). _(Beyond the chapter, a classic bug.)_
- **Using inheritance when the relationship isn't "is-a".** A `Stack` is not an `ArrayList`. Prefer composition ("has-a") in that case. _(Beyond the chapter.)_

## Flashcards

- What is inheritance? :: A mechanism where a subclass acquires the members of a superclass and can add or change behavior #card
- Which keyword creates a subclass? :: `extends`: `class B extends A` #card
- Can a Java class extend more than one class? :: No. Java allows only one superclass per class (no multiple inheritance of classes) #card
- Can a class be its own superclass? :: No; cyclic inheritance is not allowed #card
- Can a subclass access the superclass's private members? :: No. They are inherited but remain private to the superclass; use accessor methods instead #card
- A subclass includes all superclass members. Why can it still fail to use some? :: Access control: `private` members aren't accessible to the subclass #card
- Can a superclass reference variable refer to a subclass object? :: Yes (a subclass object "is a" superclass object), e.g. `Box b = new BoxWeight(...)` #card
- Given `Box plainbox = new BoxWeight(...)`, why can't you use `plainbox.weight`? :: The reference type (`Box`) determines accessible members, and `Box` has no `weight` #card
- What are the two uses of `super`? :: `super(args)` to call a superclass constructor, and `super.member` to access a hidden superclass member or overridden method #card
- Where must `super(...)` appear in a constructor? :: As the first statement #card
- Why use `super(...)` instead of setting the parent's fields directly? :: The subclass may not have access to them (private) and it avoids duplicating the parent's initialization #card
- In a multilevel hierarchy, which constructor does `super()` call? :: The constructor of the immediate (closest) superclass #card
- If a superclass constructor needs parameters, what must subclasses do? :: Pass those arguments up the chain via `super(...)` #card
- In what order are constructors executed in `A → B → C` when creating `C`? :: `A`'s, then `B`'s, then `C`'s (superclass first) #card
- Why do superclass constructors run first? :: The superclass doesn't know about subclasses, and its initialization may be a prerequisite for theirs #card
- What happens if you don't write `super()` in a subclass constructor? :: The compiler inserts an implicit call to the superclass's no-argument constructor #card
- What is method overriding? :: A subclass defines a method with the same name and signature as a superclass method, replacing it for subclass objects #card
- How do you call the superclass version of an overridden method? :: `super.methodName()` #card
- What is the difference between overriding and overloading? :: Overriding: same name and identical parameters in a subclass (resolved at run time). Overloading: same name, different parameters (resolved at compile time) #card
- If the subclass method has the same name but different parameters, is it overriding? :: No, it is overloading; both methods exist #card
- What does `@Override` do? :: Tells the compiler to verify you are actually overriding a method, catching signature mistakes #card
- What is dynamic method dispatch? :: Resolving a call to an overridden method at run time, based on the actual object's type #card
- Which decides which overridden method runs: the reference type or the object type? :: The object type, at run time #card
- Which decides which members you can access: the reference type or the object type? :: The reference type, at compile time #card
- How does Java achieve run-time polymorphism? :: Through overridden methods called via superclass reference variables #card
- Why is overriding useful? :: A superclass defines a common interface and subclasses supply specifics, so code written for the superclass works with any subclass #card
- What is an abstract method? :: A method declared with `abstract` and no body; subclasses must implement it #card
- What must be true of a class containing an abstract method? :: The class itself must be declared `abstract` #card
- Can you instantiate an abstract class? :: No, but you can declare reference variables of its type #card
- What must a subclass of an abstract class do? :: Implement all abstract methods, or be declared abstract itself #card
- Can an abstract class have concrete methods? :: Yes, it can have as much implementation as needed #card
- What can't be abstract? :: Constructors and static methods #card
- What does a `final` method prevent? :: Overriding in subclasses #card
- What does a `final` class prevent? :: Being extended (subclassed) #card
- Can a class be both `abstract` and `final`? :: No; abstract requires extension while final forbids it #card
- What are early binding and late binding? :: Early: resolved at compile time (e.g. final methods). Late: resolved at run time (overridable methods) #card
- What is the `Object` class? :: The root superclass of all Java classes; every class inherits from it #card
- Which methods does every object inherit from `Object`? :: `equals`, `toString`, `hashCode`, `clone`, `getClass`, `finalize`, `wait`, `notify`, `notifyAll` #card
- Which `Object` methods are final? :: `getClass()`, `notify()`, `notifyAll()`, and `wait()` #card
- When is `toString()` called automatically? :: When you print an object with `println()` (or concatenate it into a string) #card
- Why override `equals()` in your own classes? :: The default compares references; overriding lets you define equality by content #card
- If you override `equals()`, what else should you override? :: `hashCode()`, so equal objects have equal hash codes #card

## Open questions

- [ ] What are interfaces, and how do they give Java a form of multiple inheritance? (Next chapter.)
- [ ] What are packages, and how do `protected` and default access differ across packages? (Next chapter.)
- [ ] How do `instanceof` and casting (downcasting `Box` to `BoxWeight`) work, and when does a cast throw `ClassCastException`?
- [ ] Why aren't fields polymorphic, and what exactly is "field hiding"?
- [ ] How should I write a correct `equals()` and `hashCode()` pair for my own class?
- [ ] When should I choose composition ("has-a") over inheritance ("is-a")?
- [ ] How does the JVM implement dynamic dispatch internally (virtual method table)?
- [ ] What does `Object.clone()` do, and why is it considered tricky?

## Key terms

|Term|Definition|
|---|---|
|Inheritance|A subclass acquiring the members of a superclass|
|Superclass|The class being inherited from (parent / base class)|
|Subclass|The class that inherits (child / derived class)|
|`extends`|Keyword declaring that a class inherits from another|
|Multilevel hierarchy|A chain of inheritance (`A → B → C`)|
|`super`|Refers to the immediate superclass: its constructor or its members|
|Name hiding|A subclass member with the same name masking the superclass's member|
|Method overriding|Subclass method with the same name and signature as a superclass method|
|Dynamic method dispatch|Run-time resolution of an overridden method based on the object's actual type|
|Run-time polymorphism|Different behavior from the same call depending on the object's real type|
|Abstract method|A declaration with no body that subclasses must implement|
|Abstract class|A class that cannot be instantiated and may contain abstract methods|
|`final` method|A method that cannot be overridden|
|`final` class|A class that cannot be subclassed|
|Early binding|Method call resolved at compile time|
|Late binding|Method call resolved at run time|
|`Object`|The root class of the Java class hierarchy|
|`equals()`|Method testing whether two objects are equal|
|`toString()`|Method returning a text description of an object|
|"is-a" relationship|The relationship inheritance models (a `Car` is a `Vehicle`)|

## Related

→ Next: [[9 - Packages and Interfaces]]