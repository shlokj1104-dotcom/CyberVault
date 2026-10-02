---
title: INTRODUCING CLASSES
date: 2026-10-01
language: Java
phase: phase-1
tags:
links: []
status: learning
---
> **One-line summary:** A **class** is a template that defines a new data type (instance variables for data, methods for behavior); an **object** is a real instance of it, created with `new`, which calls a **constructor**. The things that matter most are that object variables are **references** (assigning one copies the reference, not the object), that each object has its own copy of the instance variables, how constructors and `this` work, and that Java cleans up memory through **garbage collection**, not manual `delete`.

## What it is

A **class** is the core of Java. Any code you write must live inside a class. A class does two jobs:

1. It **defines a new data type** (like `Box`), so you can declare variables of that type.
2. It acts as a **template** (a blueprint). The class itself uses no memory for data. It only describes the shape of objects.

An **object** is one real **instance** of that class. It occupies memory and holds actual values. "Object" and "instance" mean the same thing in practice.

Analogy: a class is the architectural plan for a house; an object is one house built from that plan. You can build many houses from one plan, and painting one house does not repaint the others.

A class bundles two things together. This bundling is called **encapsulation**:

- **Data** (variables), called **instance variables**
- **Code that works on that data** (methods)

Together, the instance variables and methods are called the **members** of the class. In a well-designed class, the methods define how the data may be used.

## Structure

### The general form of a class

```java
class ClassName {
    type instanceVariable1;
    type instanceVariable2;
    // ...

    type methodName1(parameter-list) {
        // body
    }
    type methodName2(parameter-list) {
        // body
    }
}
```

- A class does **not** need a `main()`. You write `main()` only in the class where the program **starts**.
- Most methods are **not** `static` or `public` in this chapter's examples. Those modifiers come in Chapter 7.

### A simple class and creating objects

```java
class Box {
    double width;
    double height;
    double depth;
}
```

This only creates the template. **No `Box` exists yet.** To create one:

```java
Box mybox = new Box();    // now a real Box exists in memory
```

Access members with the **dot operator** (`object.member`):

```java
mybox.width = 100;                                  // set
double vol = mybox.width * mybox.height * mybox.depth;   // read
```

**Each object has its own copy of every instance variable.** This is why they are called _instance_ variables. Changing one object never affects another:

```java
Box mybox1 = new Box();
Box mybox2 = new Box();
mybox1.width = 10;      // mybox2.width is still 0.0
mybox2.width = 3;
```

**File and compile facts:**

- If the file has several classes, name it after the class that contains `main()` (e.g. `BoxDemo.java`).
- The compiler produces **one `.class` file per class**. You run the class that has `main()`: `java BoxDemo`.
- Each class can also sit in its own `.java` file. It is not required to put them together.

### Declaring objects: a two-step process

`Box mybox = new Box();` is really **two steps** written together:

```java
Box mybox;              // Step 1: declare a REFERENCE variable (no object yet)
mybox = new Box();      // Step 2: create the object, store its reference in mybox
```

- **Step 1** creates only a variable that _can refer to_ a `Box`. Until step 2, it refers to nothing (`null`).
- **Step 2:** `new` **allocates memory at run time** for the object and **returns a reference** (essentially its address). That reference is stored in `mybox`.
- So `mybox` does not _contain_ the box. It _points to_ it.

```
mybox  ──────►  [ width | height | depth ]   ← the actual Box object
(reference)            (on the heap)
```

**Coming from C/C++:** a reference is like a pointer, but **safe**. You cannot do pointer arithmetic on it, cast it to an integer, or point it at an arbitrary memory address. That restriction is much of what makes Java safe.

**Why don't `int`, `char`, etc. need `new`?** Primitive types are plain variables, not objects, for efficiency. They hold their value directly. Only class types (objects) need `new`.

**What if memory runs out?** `new` throws a run-time exception (covered in the Exceptions chapter).

### Assigning reference variables (very important)

```java
Box b1 = new Box();
Box b2 = b1;
```

This does **not** copy the box. It copies the **reference**. Now `b1` and `b2` both point to the **same single object**:

```
b1 ──┐
     ├──►  [ Box object ]
b2 ──┘
```

- A change made through `b2` is visible through `b1`, because it is the same object.
- Setting `b1 = null;` only detaches `b1`. The object still exists, and `b2` still points to it.

> **Remember:** assigning one reference variable to another copies the _reference_, not the _object_.

Compare with primitives, where `int a = 5; int b = a;` makes two independent copies.

### Methods

General form:

```java
type name(parameter-list) {
    // body
}
```

- **`type`** is the return type: any valid type (including your own classes), or `void` if nothing is returned.
- **`name`** is any legal identifier.
- **parameter list** is a comma-separated list of `type name` pairs. It can be empty.
- A non-`void` method hands back a value with `return value;`.

#### A method inside the class (no parameters, no return)

```java
class Box {
    double width, height, depth;

    void volume() {
        System.out.print("Volume is ");
        System.out.println(width * height * depth);
    }
}
```

Calling it: `mybox1.volume();` runs `volume()` **on the object** `mybox1`.

**Key idea:** inside a method, instance variables are used **directly** (`width`), with no object name and no dot. A method is always called _on_ some object, so `width` automatically means "the `width` of the object that called me". So `mybox1.volume()` uses `mybox1`'s dimensions and `mybox2.volume()` uses `mybox2`'s.

**Access rule:**

- Code **outside** the class must use an object and the dot operator: `mybox.width`.
- Code **inside** the class can use the member name directly. This holds for methods too.

#### Returning a value

Printing inside the method is inflexible, since other code may want the number without printing it. Better:

```java
double volume() {
    return width * height * depth;
}
```

```java
double vol = mybox1.volume();                       // store it
System.out.println("Volume is " + mybox1.volume()); // or use it directly, no temp variable
```

Two rules:

1. The returned value's type must be **compatible with the declared return type** (a `boolean` method cannot `return 5;`).
2. The variable receiving the value must also be compatible with that return type.

#### Methods with parameters

Parameters make a method general-purpose.

```java
int square(int i) { return i * i; }

x = square(5);    // 25
x = square(9);    // 81
y = 2;
x = square(y);    // 4
```

**Parameter vs. argument** (a common interview question):

|Term|Meaning|Example|
|---|---|---|
|**Parameter**|The variable in the method's **definition** that receives a value|`i` in `int square(int i)`|
|**Argument**|The actual value you **pass in** when calling|`100` in `square(100)`|

A parameterized method is also better design for `Box`. Setting fields one by one (`mybox1.width = 10; ...`) is clumsy, easy to forget, and exposes the variables directly. In good Java design, **instance variables should be changed through methods**, because you can change a method's behavior later but you cannot change what a directly exposed variable does.

```java
void setDim(double w, double h, double d) {
    width = w;
    height = h;
    depth = d;
}
// mybox1.setDim(10, 20, 15);   → w=10, h=20, d=15, then copied into the fields
```

### Constructors

Calling `setDim()` after every `new` is still tedious. A **constructor** initializes an object **at the moment it is created**.

**Rules for a constructor:**

- **Same name as the class.**
- **No return type**, not even `void`. (Its implicit "return" is the new object itself.)
- It is **called automatically** by `new`, before `new` hands back the reference.
- Its job is to put the object into a valid, ready-to-use state.

```java
class Box {
    double width, height, depth;

    Box() {                       // no-argument constructor
        width = 10;
        height = 10;
        depth = 10;
    }
    double volume() { return width * height * depth; }
}
```

**Now you can see why `new Box()` has parentheses.** It is not decoration. `Box()` is a **call to the constructor**.

#### The default constructor

If you write **no** constructor, Java supplies a **default constructor** (no arguments, empty body). It sets instance variables to their default values:

|Variable type|Default|
|---|---|
|numeric (`int`, `double`, ...)|`0` (or `0.0`)|
|`boolean`|`false`|
|reference (objects, arrays, `String`)|`null`|

This is why `new Box()` worked in the early examples that had no constructor.

> **Once you write any constructor of your own, Java stops providing the default one.** If you only write `Box(double w, double h, double d)`, then `new Box()` no longer compiles unless you also write a no-argument constructor.

#### Parameterized constructors

A constructor that takes arguments lets each object start with different values:

```java
Box(double w, double h, double d) {
    width = w;
    height = h;
    depth = d;
}
```

```java
Box mybox1 = new Box(10, 20, 15);    // volume 3000.0
Box mybox2 = new Box(3, 6, 9);       // volume 162.0
```

The arguments in `new Box(10, 20, 15)` go straight to the constructor's parameters.

### The `this` keyword

Inside any method or constructor, **`this` is a reference to the current object**, meaning the object the method was called on.

```java
Box(double w, double h, double d) {
    this.width  = w;     // same as width = w
    this.height = h;
    this.depth  = d;
}
```

Here `this` is redundant but legal. It becomes necessary in the next case.

#### Instance variable hiding

You cannot declare two local variables with the same name in the same scope. But a **local variable or parameter may have the same name as an instance variable**. When it does, the local one **hides** the instance variable. A plain name now refers to the parameter, not the field.

```java
Box(double width, double height, double depth) {
    width = width;          // BUG: assigns the parameter to itself; the field is never set
}

Box(double width, double height, double depth) {
    this.width  = width;    // correct: this.width is the field, width is the parameter
    this.height = height;
    this.depth  = depth;
}
```

`this.width` means "the `width` field of this object". This is **the** standard use of `this`, and you will see this pattern in almost every Java constructor and setter. (Some programmers prefer different parameter names instead. It is a matter of taste.)

### Garbage collection

In C/C++ you must free heap memory yourself (`free`, `delete`). In Java you **never** free objects. The **garbage collector** does it:

- When **no references to an object remain**, the object is unreachable and the JVM may reclaim its memory.
- It runs **occasionally and unpredictably** (if at all). It does not run the instant an object becomes unused.
- Different JVMs use different strategies. You normally don't think about it.

```java
Box b = new Box();
b = null;        // the Box is now unreachable → eligible for garbage collection
```

"Eligible" does not mean "collected right now". It only means the JVM is free to do it.

### The `finalize()` method

Before the collector reclaims an object, the JVM may call that object's `finalize()` method. The idea was to release non-Java resources (file handles, etc.).

```java
protected void finalize() {
    // cleanup code
}
```

Why it should **not** be relied on:

- It is called only **just before garbage collection**, not when a variable goes out of scope.
- You **cannot know when, or even if,** it will run.
- So your program must release resources by other means and must not depend on `finalize()`.
- Java has **no destructors** like C++. `finalize()` only roughly resembles one.

_(Beyond the chapter: `finalize()` is **deprecated** in modern Java (deprecated in Java 9, deprecated for removal from Java 18). The modern way to release resources is `try-with-resources` with `AutoCloseable`. Interviewers like to ask about this.)_

### Worked example: a `Stack` class

A **stack** is a **LIFO** (last-in, first-out) structure, like a pile of plates: the last plate placed on top is the first one taken off. Two operations:

- **push** puts an item on top.
- **pop** removes and returns the top item.

```java
class Stack {
    int stck[] = new int[10];   // storage
    int tos;                    // "top of stack": index of the top item

    Stack() { tos = -1; }       // -1 means empty

    void push(int item) {
        if (tos == 9)
            System.out.println("Stack is full.");
        else
            stck[++tos] = item;     // increment first, then store
    }

    int pop() {
        if (tos < 0) {
            System.out.println("Stack underflow.");
            return 0;
        } else
            return stck[tos--];     // return, then decrement
    }
}
```

How it works:

- `tos` always holds the **index of the top element**. Empty stack means `tos == -1`.
- `push` uses **prefix** `++tos` (move up, then store). `pop` uses **postfix** `tos--` (read the top, then move down). This is the prefix/postfix difference from the Operators notes in action.
- Full means `tos == 9` (indices 0 to 9 = 10 slots). Empty means `tos < 0`.
- Each `Stack` object has its **own** `stck` and `tos`, so two stacks are completely independent.

```java
Stack mystack1 = new Stack();
Stack mystack2 = new Stack();
for (int i = 0; i < 10; i++)  mystack1.push(i);
for (int i = 10; i < 20; i++) mystack2.push(i);
// popping mystack1 prints 9 8 7 ... 0, and mystack2 prints 19 18 ... 10
```

**Why this example matters (encapsulation):** users only touch `push()` and `pop()`. They don't need to know the data is in an array. You could swap the array for a linked list and the code that uses the stack would not change.

**Weakness of this version:** `stck` and `tos` are exposed, so outside code can write `mystack1.tos = 50;` and corrupt the stack. The next chapter fixes this with access control (`private`).

## Mechanics / reference

### Class vs. object

|Class|Object|
|---|---|
|A template / new data type|An instance of that template|
|Logical construct|Has physical reality (uses memory)|
|Declared once|Many can be created|
|Defines the instance variables|Holds its own copy of them|

### What `new ClassName(args)` does, in order

1. Allocates memory for the object (on the heap) at run time.
2. Sets all instance variables to default values (`0`, `false`, `null`).
3. Runs the constructor body with the given arguments.
4. Returns a reference to the new object, which is then stored in your variable.

### Declaring vs. creating

|Code|What it creates|
|---|---|
|`Box b;`|Just a reference variable (holds `null`/nothing). **No object.**|
|`b = new Box();`|The actual object. `b` now points to it.|
|`Box c = b;`|A second reference to the **same** object. No new object.|

### Constructor vs. method

|Constructor|Method|
|---|---|
|Same name as class|Any legal name|
|No return type (not even `void`)|Has a return type (or `void`)|
|Called automatically by `new`|Called explicitly on an object|
|Initializes a new object|Performs an operation|
|Java gives a default one only if you write none|Never supplied automatically|

### Where a name is looked up

|Situation|How to refer to the instance variable|
|---|---|
|Code in another class|`obj.width` (needs an object and the dot)|
|Method of the same class|`width` (direct)|
|Parameter has the same name as the field|`this.width` for the field, `width` for the parameter|

### Common patterns

```java
// 1. Standard constructor: parameter names match fields, resolved with this
class Point {
    int x, y;
    Point(int x, int y) { this.x = x; this.y = y; }
}

// 2. Getter-style method: compute and return, don't print
double volume() { return width * height * depth; }

// 3. Reference aliasing
Box a = new Box(1, 2, 3);
Box b = a;          // same object
b.width = 99;       // a.width is now 99 too
```

## Pitfalls

- **Declaring a reference but never creating the object.** `Box b; b.width = 5;` fails to compile (uninitialized local). If `b = null` was assigned, it compiles but throws `NullPointerException` at run time.
- **Thinking `b2 = b1` copies the object.** It copies only the reference. Both names point to one object, so a change through either is seen by both.
- **Forgetting `new`.** A class-type variable is just a reference, not an object.
- **Parameter name hides the field.** `width = width;` inside a constructor assigns the parameter to itself and leaves the field at its default. Use `this.width = width;`.
- **Writing a constructor and losing the default one.** After you define `Box(double, double, double)`, `new Box()` is a compile error unless you add a no-arg constructor too.
- **Giving a constructor a return type.** `void Box() { ... }` is a normal _method_ named `Box`, **not** a constructor, and it will never run on `new Box()`. The compiler won't warn you.
- **Mismatched return type.** `double volume()` that has no `return`, or a `boolean` method returning an `int`, is a compile error.
- **Confusing parameter and argument.** The parameter is in the definition; the argument is the value you pass.
- **Relying on `finalize()` to release resources.** It may never run. Close files and resources explicitly.
- **Expecting garbage collection to be immediate.** Setting a reference to `null` makes the object _eligible_, not instantly destroyed.
- **Exposing instance variables directly.** Any code can set them to bad values (as in `Stack`, where `tos` can be overwritten). Control access through methods.
- **Stack edge cases (from the example):** `pop()` returns `0` on underflow, which looks like a legitimate value. Real code should throw an exception instead. The hard-coded `9` in `push()` breaks if the array size changes. Use `stck.length - 1`.
- **Public class and filename.** If a class is `public`, the file must have exactly that class name.

## Flashcards

- What is a class in Java? :: A template that defines a new data type, made of instance variables (data) and methods (code). It does not itself hold data #card
- What is the difference between a class and an object? :: A class is a logical template; an object is a real instance of it that occupies memory and has its own data #card
- Does writing `class Box { ... }` create a Box? :: No. It only defines the template. An object exists only after `new Box()` runs #card
- What are the "members" of a class? :: Its instance variables and its methods #card
- Why are they called instance variables? :: Each object (instance) has its own separate copy of them #card
- How do you access an object's instance variable or method from outside the class? :: With the dot operator: `object.member` #card
- If you change `mybox1.width`, does `mybox2.width` change? :: No. Each object has its own copy of the instance variables #card
- Does a Java class need a `main()` method? :: No. Only the class that is the program's starting point needs one #card
- How many `.class` files are created for a source file with two classes? :: Two, one per class #card
- What are the two steps hidden inside `Box b = new Box();`? :: Declare a reference variable (`Box b;`), then allocate an object and store its reference in it (`b = new Box();`) #card
- What does `new` do? :: Dynamically allocates memory for an object at run time, calls the constructor, and returns a reference to the object #card
- What does an object variable actually hold? :: A reference (essentially the address) to the object, not the object itself #card
- How do Java references differ from C/C++ pointers? :: They can't be manipulated: no pointer arithmetic, no casting to an integer, no pointing at arbitrary memory. This is key to Java's safety #card
- Why don't primitive types like `int` need `new`? :: They are not objects; they are plain variables holding their value directly, which is more efficient #card
- What happens after `Box b2 = b1;`? :: `b1` and `b2` refer to the same single object. Only the reference was copied #card
- After `Box b2 = b1; b1 = null;`, what is `b2`? :: Still a valid reference to the original object. Setting `b1` to `null` only detaches `b1` #card
- Inside a method, how do you refer to the object's own instance variables? :: Directly by name (e.g. `width`), because the method is always called on a specific object #card
- Why is returning a value from `volume()` better than printing inside it? :: Callers can use the value however they like (store it, print it, compute with it), which makes the method reusable #card
- What are the two rules for a method's return value? :: Its type must be compatible with the declared return type, and the receiving variable must be compatible too #card
- What is the difference between a parameter and an argument? :: A parameter is the variable in the method definition; an argument is the actual value passed in the call #card
- Why is setting fields through a method (like `setDim`) better than direct access? :: Less error-prone, and you can change a method's behavior later, but you can't change the behavior of an exposed variable #card
- What is a constructor? :: A special member that initializes an object at creation; it has the class's name and no return type and is called automatically by `new` #card
- Why does a constructor have no return type, not even `void`? :: Its implicit return type is the class itself, since it produces the new object #card
- Why does `new Box()` have parentheses? :: Because it calls the `Box()` constructor #card
- What is the default constructor and when does Java supply it? :: A no-argument constructor that sets fields to defaults (0, false, null). Java supplies it only if you define no constructor #card
- What are the default values of instance variables? :: `0` for numeric types, `false` for boolean, `null` for references #card
- After you define a parameterized constructor, can you still call `new Box()`? :: Not unless you also write a no-argument constructor; the default one is no longer provided #card
- What does `this` refer to? :: The current object, the one on which the method or constructor is being executed #card
- What is instance variable hiding and how do you fix it? :: A local variable or parameter with the same name as a field hides the field. Use `this.fieldName` to reach the field #card
- What does `width = width;` do in a constructor whose parameter is also `width`? :: Assigns the parameter to itself and leaves the field unchanged (a bug). Use `this.width = width;` #card
- How does Java free object memory? :: Automatically via garbage collection: when no references to an object remain, its memory can be reclaimed #card
- Does garbage collection happen immediately when an object becomes unreferenced? :: No. It runs sporadically (if at all) and the timing is up to the JVM #card
- What is `finalize()` and why shouldn't you depend on it? :: A method the JVM may call just before collecting an object. You can't know when or whether it runs, so don't use it for essential cleanup (and it is deprecated in modern Java) #card
- Does Java have destructors? :: No. `finalize()` only loosely approximates one #card
- What does LIFO mean, and which two operations does a stack have? :: Last-in, first-out. `push` adds on top; `pop` removes and returns the top #card
- In the `Stack` class, what does `tos == -1` mean? :: The stack is empty. `tos` holds the index of the top element #card
- Why is the `Stack` class a good example of encapsulation? :: Users only call `push()` and `pop()`; the internal storage (array, or later a linked list) can change without affecting them #card
- What is the weakness of the chapter's `Stack` class? :: Its array and `tos` are accessible from outside, so other code can corrupt them. Access control (next chapter) fixes this #card

## Open questions

- [ ] What is constructor overloading, and how does one constructor call another with `this(...)`? (Chapter 7.)
- [ ] What do `public`, `private`, and `protected` do, and how do they fix the exposed-fields problem in `Stack`? (Chapter 7.)
- [ ] Is Java pass-by-value or pass-by-reference when you pass an object to a method? (Passing a reference by value; Chapter 7.)
- [ ] Where do objects, references, and local variables live in memory (heap vs. stack), and how does garbage collection decide what is unreachable?
- [ ] What is the difference between `==` and `.equals()` on two references that point to different objects with the same contents?
- [ ] What replaces `finalize()` today (`try-with-resources`, `AutoCloseable`, `Cleaner`)?

## Key terms

|Term|Definition|
|---|---|
|Class|A template that defines a new data type: its instance variables and methods|
|Object / instance|A real, memory-occupying realization of a class|
|Instance variable|A variable declared in a class; every object gets its own copy|
|Method|Code inside a class that operates on its data|
|Member|An instance variable or method of a class|
|Dot operator (`.`)|Accesses a member of an object (`obj.member`)|
|Reference variable|A variable that refers to an object, holding its address rather than its data|
|`new`|Operator that allocates an object at run time and returns a reference to it|
|`null`|The value of a reference that refers to no object|
|Constructor|Special member, named like the class with no return type, that initializes a new object|
|Default constructor|No-argument constructor Java supplies when a class declares none|
|Parameterized constructor|A constructor that takes arguments to initialize an object with chosen values|
|Parameter|Variable in a method's definition that receives a value|
|Argument|Value passed to a method when it is called|
|Return type|The type of value a method gives back (`void` if none)|
|`this`|Reference to the current object inside a method or constructor|
|Instance variable hiding|A local variable or parameter with the same name shadowing an instance variable|
|Garbage collection|Automatic reclaiming of memory from objects with no remaining references|
|`finalize()`|Method the JVM may call just before collecting an object (deprecated)|
|Encapsulation|Bundling data with the code that operates on it and hiding the internals behind methods|
|Stack (LIFO)|Structure where the last item added is the first removed (`push` / `pop`)|
|`tos`|"Top of stack": index of the top element in the `Stack` example|

## Related

→ Next: [[7 - A Closer Look at Methods and Classes]]