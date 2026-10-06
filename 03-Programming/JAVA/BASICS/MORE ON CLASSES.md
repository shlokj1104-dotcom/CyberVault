---
title: MORE ON CLASSES
date: 2026-10-02
language: Java
phase: phase-1
tags:
links: []
status: learning
---
lp> **One-line summary:** This chapter makes methods and classes practical. **Overloading** lets one name serve several related methods; Java passes **everything by value** (for objects, the value is a _reference_, so the method can change the object but cannot re-point your variable); **recursion** is a method calling itself and needs a base case; **access control** (`private`/`public`) hides data; **`static`** members belong to the class, not to objects; **`final`** makes constants; arrays carry **`length`**; **inner classes**, **`String`** (immutable), **command-line args** and **varargs** round it off.

## What it is

This chapter covers the tools you use every day once you start writing real classes:

|Topic|The question it answers|
|---|---|
|Method overloading|Can one name do the same job for different inputs?|
|Constructor overloading|Can I create objects in several convenient ways?|
|Objects as parameters / return values|How do objects travel into and out of methods?|
|Argument passing|When a method changes its parameter, does my variable change?|
|Recursion|Can a method call itself, and how does that stay safe?|
|Access control|How do I stop other code from corrupting my data?|
|`static`|How do I share one value or one method across all objects?|
|`final`|How do I make a constant?|
|Arrays revisited|How do I know an array's size?|
|Nested / inner classes|Can a class live inside another class?|
|`String`|How do strings behave, and why are they special?|
|Command-line args, varargs|How do I take input at launch, or accept any number of arguments?|

## Structure

### 1. Overloading methods

**Method overloading** means two or more methods in the **same class** share a **name** but have **different parameter lists** (different number and/or types of parameters).

```java
class OverloadDemo {
    void test()                    { System.out.println("No parameters"); }
    void test(int a)               { System.out.println("a: " + a); }
    void test(int a, int b)        { System.out.println("a and b: " + a + " " + b); }
    double test(double a)          { System.out.println("double a: " + a); return a * a; }
}
```

**How Java picks the version:** at the call, the compiler compares the **number and types of the arguments** with each method's parameters.

**The rule that gets asked in interviews:** the **return type is not part of the distinction**. Two methods that differ only in return type are _not_ valid overloads (compile error), because the compiler can't tell which one you mean from the call.

**Automatic type conversion applies, but only as a fallback.** If there is no exact match, Java widens the argument (e.g. `int` to `double`) and tries again.

```java
// class has test(), test(int,int), test(double), but NO test(int)
int i = 88;
ob.test(i);        // no test(int) → i is widened to double → calls test(double)
```

If `test(int)` existed, it would be chosen instead. **Exact match first, conversion only if no exact match.**

**Why overloading matters (polymorphism):** it is "one interface, multiple methods". In C you need `abs()`, `labs()`, `fabs()` for int, long and float. In Java, `Math.abs()` is overloaded, so you remember one name and the compiler chooses the version. Only overload methods that do **closely related** things. Using `sqr` for both "square an int" and "square root of a double" would defeat the purpose.

### 2. Overloading constructors

Constructors can be overloaded too, and most real classes do this. With only `Box(double w, double h, double d)`, `new Box()` is an error, and you can't make a cube with one number. So provide several:

```java
class Box {
    double width, height, depth;

    Box(double w, double h, double d) { width = w; height = h; depth = d; }  // all three
    Box()                              { width = height = depth = -1; }        // "uninitialized" box
    Box(double len)                    { width = height = depth = len; }       // cube

    double volume() { return width * height * depth; }
}

Box b1 = new Box(10, 20, 15);   // 3000.0
Box b2 = new Box();             // -1.0 (volume of -1 × -1 × -1)
Box b3 = new Box(7);            // 343.0
```

`new` picks the constructor by the arguments you give, exactly like method overloading.

### 3. Using objects as parameters

A parameter's type can be a class, just like `int` or `double`.

```java
class Test {
    int a, b;
    Test(int i, int j) { a = i; b = j; }

    boolean equalTo(Test o) {          // compare the calling object with another Test
        return o.a == a && o.b == b;
    }
}

Test ob1 = new Test(100, 22), ob2 = new Test(100, 22), ob3 = new Test(-1, -1);
ob1.equalTo(ob2);   // true  (same field values)
ob1.equalTo(ob3);   // false
```

Note: `ob1` and `ob2` are two different objects. `equalTo` compares **contents**, which is different from `ob1 == ob2` (that compares references and would be `false`).

**A very common use: the copy constructor.** A constructor that takes an object of its own class, so you can build a new object that starts as a copy of an existing one:

```java
Box(Box ob) {                // copy constructor
    width  = ob.width;
    height = ob.height;
    depth  = ob.depth;
}
Box myclone = new Box(mybox1);   // a NEW, separate Box with the same dimensions
```

This is a true copy, unlike `Box b2 = b1;`, which only copies the reference.

### 4. A closer look at argument passing

Two classic styles in programming languages:

|Style|What the method receives|Effect on the caller's variable|
|---|---|---|
|**Call-by-value**|A **copy** of the argument's value|Changes to the parameter do **not** affect the argument|
|**Call-by-reference**|A reference to the **original variable**|Changes to the parameter **do** change the argument|

**Java is always call-by-value.** What differs is _what the value is_.

#### Primitives: the value is the data itself

```java
void meth(int i, int j) { i *= 2; j /= 2; }

int a = 15, b = 20;
ob.meth(a, b);
// a is still 15, b is still 20
```

`i` and `j` are copies. Changing them cannot touch `a` and `b`.

#### Objects: the value is the reference

When you pass an object, Java copies the **reference** (the address), not the object. So the parameter and the argument are two references to the **same object**:

```java
void meth(Test o) { o.a *= 2; o.b /= 2; }

Test ob = new Test(15, 20);
ob.meth(ob);
// ob.a is now 30, ob.b is now 10
```

```
caller's ob ──┐
              ├──►  [ Test object: a=30, b=10 ]
param o     ───┘      (same object, so changes made through o are visible)
```

So an object parameter _behaves like_ call-by-reference **for changing the object's contents**.

> **Remember:** the reference itself is passed by value. The copy of the reference still points to the same object.

_(Beyond the chapter, and a classic interview trap: because the reference is a **copy**, if the method **re-assigns** it (`o = new Test(1, 1);`), only the method's local copy re-points. The caller's variable still refers to the original object. A method can change **what the object contains**, but cannot change **which object your variable refers to**. That is exactly why Java is called pass-by-value, not pass-by-reference. Java cannot write a method that swaps two variables.)_

### 5. Returning objects

A method can return any type, including a class type:

```java
class Test {
    int a;
    Test(int i) { a = i; }

    Test incrByTen() {
        Test temp = new Test(a + 10);   // build a NEW object
        return temp;                    // return its reference
    }
}

Test ob1 = new Test(2);
Test ob2 = ob1.incrByTen();     // ob1.a = 2, ob2.a = 12
ob2 = ob2.incrByTen();          // ob2.a = 22
```

Each call creates a **new** object. `ob1` is unchanged.

**Key point:** an object created inside a method does **not** disappear when the method ends. It lives on the heap as long as **some reference** to it exists (here the returned reference). Once no references remain, the garbage collector may reclaim it. (Compare with local _primitive_ variables, which do vanish when the method returns.)

### 6. Recursion

**Recursion** means a method that **calls itself**. It solves a problem by reducing it to a smaller version of the same problem.

Factorial: `n! = n × (n-1)!`, and `1! = 1`.

```java
int fact(int n) {
    if (n == 1) return 1;          // BASE CASE: stops the recursion
    return fact(n - 1) * n;        // RECURSIVE CASE: smaller problem
}
```

**Tracing `fact(3)`:** the calls "telescope" out, then the answers return back:

```
fact(3) → needs fact(2) × 3
            fact(2) → needs fact(1) × 2
                        fact(1) → returns 1            (base case reached)
            fact(2) returns 1 × 2 = 2
fact(3) returns 2 × 3 = 6
```

**What happens in memory:** each call gets its **own copy of the parameters and local variables** on the **call stack**. As calls return, those frames are removed. This is why `n` in each call is independent.

**Two things every recursive method needs:**

1. A **base case** (an `if` that returns _without_ calling itself). Without it the method never stops.
2. A **recursive step that moves toward the base case** (here `n - 1`).

**Trade-offs:**

- Often **slower** than a loop because of call overhead.
- Too-deep recursion exhausts the stack and throws an exception (`StackOverflowError`).
- But it can make some algorithms much **clearer** (tree traversal, divide-and-conquer, quicksort).

**Second example, printing an array recursively:**

```java
void printArray(int i) {
    if (i == 0) return;                 // base case
    else printArray(i - 1);             // go deeper FIRST
    System.out.println("[" + (i-1) + "] " + values[i-1]);   // print on the way BACK
}
```

Because the print comes **after** the recursive call, elements print in order `[0], [1], ... [9]`. If you put the print **before** the recursive call, the order would be reversed. This "before vs. after the call" idea is a favorite exam question.

### 7. Introducing access control

**Encapsulation** also means controlling **who may use** each member. In Chapter 6, anyone could write `mystack.tos = 50;` and corrupt the stack. **Access modifiers** fix this. An access modifier goes at the **start** of a member's declaration.

|Modifier|Who can access the member|
|---|---|
|`public`|Any code, anywhere|
|`private`|Only code **inside the same class**|
|_(none) default_|Code in the **same package** (the book says "public within its package")|
|`protected`|Same package, plus subclasses (covered with inheritance)|

```java
class Test {
    int a;            // default access
    public int b;     // public
    private int c;    // private

    void setc(int i) { c = i; }       // public-style access to c through methods
    int  getc()      { return c; }
}

Test ob = new Test();
ob.a = 10;           // OK
ob.b = 20;           // OK
// ob.c = 100;       // COMPILE ERROR: c is private
ob.setc(100);        // OK
```

**The standard pattern:** make data `private`, expose behavior through `public` methods (setters and getters). Now the class controls what is allowed.

**Why `main()` is `public`:** it is called by the Java run-time system, which is code _outside_ your class.

**The fixed `Stack`:**

```java
class Stack {
    private int stck[] = new int[10];
    private int tos;
    // push() and pop() unchanged
}
// mystack.tos = -2;        // now a compile error
// mystack.stck[3] = 100;   // now a compile error
```

Now the only way to change the stack is `push()` and `pop()`, so it can't be put in an invalid state.

_(Beyond the chapter: it is fine to make a field `public` when there's a good reason. The book does it in simple examples. In real code, fields are usually `private`. Exposing them locks you into that internal design forever.)_

### 8. Understanding `static`

Normally you need an object to use a class's members. A `static` member belongs to the **class itself**, so it can be used **without creating any object**, even before any object exists.

```java
class UseStatic {
    static int a = 3;
    static int b;

    static void meth(int x) {
        System.out.println("x = " + x);
        System.out.println("a = " + a);
        System.out.println("b = " + b);
    }

    static {                              // static initialization block
        System.out.println("Static block initialized.");
        b = a * 4;
    }

    public static void main(String args[]) { meth(42); }
}
```

**Output:** `Static block initialized.` then `x = 42`, `a = 3`, `b = 12`.

**Order of events:** when the class is **loaded**, the static variable initializers and the `static { }` block run **once**, top to bottom. Only after that does `main()` run.

**Static variables:** **one shared copy for the whole class**, not one per object. They behave like controlled "global" variables. (Instance variables: one copy per object.)

**Why `main()` is `static`:** the JVM must call it **before any object exists**.

**Restrictions on `static` methods** (very common exam question):

1. They can directly call **only other `static` methods**.
2. They can directly access **only `static` data**.
3. They **cannot use `this` or `super`**, because there is no current object.

```java
class X {
    int count;                       // instance variable
    static void f() { count++; }     // ERROR: static method can't use instance variable directly
}
```

**Static block:** use it when initializing static variables needs actual computation. It runs **once**, at class load.

**Calling static members from outside the class:** use the **class name**, not an object:

```java
StaticDemo.callme();                          // static method
System.out.println(StaticDemo.b);             // static variable
```

You've been doing this already: `Math.abs(...)`, `System.out`.

### 9. Introducing `final`

A `final` variable **cannot be changed after it is given a value**: it is a constant.

```java
final int FILE_NEW  = 1;
final int FILE_OPEN = 2;
```

- Must be initialized **at declaration**, or once in a **constructor**.
- **Convention:** `ALL_UPPERCASE` names.
- `final` can also apply to **method parameters** (can't be reassigned inside the method) and **local variables** (can be assigned only once).
- `final` on **methods** means something different (preventing overriding). That comes with inheritance.

_(Beyond the chapter: `final` on a **reference** means the reference can't be re-pointed, but the **object it points to can still change**. `final Box b = new Box(...); b.width = 5;` is legal. Constants are usually written `static final` so there is one shared copy: `static final double PI = 3.14159;`.)_

### 10. Arrays revisited: `length`

Arrays are **objects**, and every array has a `length` **instance variable** that holds the number of elements it was **created to hold**.

```java
int a1[] = new int[10];
int a2[] = {3, 5, 7, 1, 8, 99, 44, -10};
a1.length   // 10
a2.length   // 8
```

- `length` is the _capacity_, not how many slots you've actually filled.
- It's a **field** (`arr.length`, no parentheses). Contrast with `String`'s **method** `str.length()`. Mixing these up is a classic mistake.

**Improved `Stack`:** the size is now a constructor parameter, and `push` uses `stck.length - 1` instead of a hard-coded `9`, so it works for any size:

```java
Stack(int size) { stck = new int[size]; tos = -1; }

void push(int item) {
    if (tos == stck.length - 1) System.out.println("Stack is full.");
    else stck[++tos] = item;
}
```

### 11. Nested and inner classes

A class can be defined **inside another class**. This is a **nested class**. Its scope is limited to the enclosing class.

|Kind|Description|
|---|---|
|**Static nested class**|Declared `static`. Cannot refer directly to the outer class's non-static members (needs an object). Rarely used.|
|**Inner class**|Non-static nested class. Can use **all** of the outer class's members directly, even `private` ones. This is the important kind.|

```java
class Outer {
    int outer_x = 100;

    void test() {
        Inner inner = new Inner();     // create inner object from inside Outer
        inner.display();
    }

    class Inner {                      // inner class
        void display() { System.out.println("display: outer_x = " + outer_x); }
    }
}
```

**Access goes one way:**

- Inner **can** use outer's members (`outer_x`).
- Outer **cannot** use inner's members directly. If `Inner` has `int y`, then code in `Outer` writing `y` is a compile error. It would have to create an `Inner` object and use `inner.y`.

**More facts:**

- An inner class **instance** normally exists only in the context of an outer instance (it is usually created by the outer class's own code).
- Inner classes can be declared inside **any block** (a method body, even a `for` loop). Then they exist only within that block.
- Their big use is **event handling** (and anonymous inner classes), covered later.

### 12. Exploring the `String` class

Strings are everywhere, so know these essentials:

1. **Every string is an object** of type `String`, even a literal like `"hello"`.
2. **Strings are immutable.** Once created, a `String`'s contents **cannot be changed**. Operations like concatenation produce a **new** `String`. (`StringBuilder` / `StringBuffer` are the mutable versions.)
3. `+` **concatenates** strings. It's the only operator defined for `String`.

```java
String s1 = "First String";
String s2 = "Second String";
String s3 = s1 + " and " + s2;      // "First String and Second String"
```

**Three methods to know now:**

|Method|Returns|Example|
|---|---|---|
|`boolean equals(String other)`|whether contents are equal|`s1.equals(s2)`|
|`int length()`|number of characters|`"First String".length()` → 12|
|`char charAt(int index)`|char at that position (0-based)|`"First String".charAt(3)` → `'s'`|

**`equals()` vs. `==` (top interview question):** `==` compares **references** (are they the same object?). `equals()` compares **contents**. Always use `.equals()` to compare strings.

```java
String strOb1 = "First String";
String strOb3 = strOb1;              // second reference to the SAME string
strOb1.equals(strOb3);               // true
```

**Arrays of strings** work like any object array:

```java
String str[] = { "one", "two", "three" };
for (int i = 0; i < str.length; i++) System.out.println(str[i]);
```

### 13. Using command-line arguments

Information typed **after the program name** when you launch it is passed to `main()` as the `String[] args` parameter.

```java
class CommandLine {
    public static void main(String args[]) {
        for (int i = 0; i < args.length; i++)
            System.out.println("args[" + i + "]: " + args[i]);
    }
}
```

```
java CommandLine this is a test 100 -1
args[0]: this    args[1]: is    args[2]: a    args[3]: test    args[4]: 100    args[5]: -1
```

- `args[0]` is the **first argument after** the class name (unlike C, where `argv[0]` is the program name).
- **Every argument is a `String`**, even `100` and `-1`. To use them as numbers you must convert them (e.g. `Integer.parseInt(args[4])`).
- With no arguments, `args.length` is `0`.

### 14. Varargs: variable-length arguments (Java 5+)

A **varargs** method takes **zero or more** arguments of one type. Mark the parameter with `...`:

```java
static void vaTest(int ... v) {
    System.out.print("Number of args: " + v.length + " Contents: ");
    for (int x : v) System.out.print(x + " ");
    System.out.println();
}

vaTest(10);          // 1 arg
vaTest(1, 2, 3);     // 3 args
vaTest();            // no args
```

**Key idea:** inside the method, `v` **is an array** (`int[]`). The compiler automatically packs the arguments into an array. With no arguments, the array has length `0`.

**Before Java 5** you had to build the array yourself (`int n2[] = {1, 2, 3}; vaTest(n2);`) or write many overloads. Varargs is simply cleaner.

**Rules:**

1. The varargs parameter must be the **last** parameter.
2. There can be only **one** varargs parameter per method.

```java
int doIt(int a, int b, double c, int ... vals) { }                   // OK
int doIt(int a, int ... vals, boolean stopFlag) { }                   // ERROR: must be last
int doIt(int ... vals, double ... morevals) { }                       // ERROR: only one
```

#### Overloading varargs methods

You can overload by changing the varargs **type** (`int...` vs `boolean...`), or by adding normal parameters (`String msg, int...`). A normal non-varargs overload also works: `vaTest(int x)` is called when exactly one `int` is passed, while `vaTest(int...)` handles two or more.

#### Varargs and ambiguity

```java
static void vaTest(int ... v)     { }
static void vaTest(boolean ... v) { }

vaTest(1, 2, 3);                 // OK
vaTest(true, false, false);      // OK
vaTest();                        // ERROR: ambiguous
```

`vaTest()` with no arguments could mean either version, and the compiler can't choose. Another ambiguous pair:

```java
static void vaTest(int ... v)          { }
static void vaTest(int n, int ... v)   { }

vaTest(1);    // ERROR: one varargs element? or n=1 with zero varargs?
```

The overloads are legal, but **calls** can be ambiguous. The fix is usually to use different method names.

## Mechanics / reference

### Overloading at a glance

|Can differ|Can't be the only difference|
|---|---|
|Number of parameters|Return type|
|Types of parameters|Parameter **names**|
|Order of parameter types (`(int, double)` vs `(double, int)`)||

**Resolution order:** exact match first → then automatic widening → otherwise compile error.

### Parameter passing summary

|You pass...|The method receives...|Can the method change the caller's data?|
|---|---|---|
|A primitive (`int`, `double`, ...)|A copy of the value|No|
|An object (reference)|A copy of the reference (same object)|Yes, it can change the **object's contents**|
|An object, then method does `o = new ...`|Copy re-pointed|No effect on the caller's variable|

### `static` vs. instance

| |Instance member|`static` member|
|---|---|---|
|Belongs to|Each object|The class|
|Copies|One per object|One total|
|Access from outside|`object.member`|`ClassName.member`|
|Can use `this`?|Yes|No|
|Can use instance members directly?|Yes|No (needs an object)|
|Exists before any object?|No|Yes|

### Access levels at a glance

|Modifier|Same class|Same package|Subclass (other package)|Anywhere|
|---|---|---|---|---|
|`private`|yes|no|no|no|
|(default)|yes|yes|no|no|
|`protected`|yes|yes|yes|no|
|`public`|yes|yes|yes|yes|

_(The subclass and package columns are covered with inheritance and packages.)_

### `length` vs. `length()`

|Thing|How to get size|
|---|---|
|Array|`arr.length` (field, no parentheses)|
|`String`|`str.length()` (method, with parentheses)|

### Common patterns

```java
// 1. Private field with getter/setter
private int count;
int  getCount()      { return count; }
void setCount(int c) { count = c; }

// 2. Constants
static final int MAX = 100;

// 3. Safe recursion skeleton
int solve(int n) {
    if (n == BASE) return BASE_VALUE;      // base case first
    return combine(n, solve(n - 1));       // move toward the base case
}

// 4. Sum with varargs
static int sum(int ... nums) { int t = 0; for (int x : nums) t += x; return t; }

// 5. Parse a command-line number
int n = Integer.parseInt(args[0]);
```

## Pitfalls

- **Overloading by return type only.** Not allowed. The parameter lists must differ.
- **Surprise conversion in overloads.** `test(88)` with only `test(double)` defined silently widens to `double`. If `test(int)` is added later, the call switches versions.
- **Thinking Java passes objects by reference.** It passes a **copy of the reference**. Changing the object's fields is visible to the caller. Re-assigning the parameter (`o = new ...`) is not.
- **Expecting a method to swap two variables.** Impossible with primitives or object references passed in. The caller's variables never change.
- **Confusing `==` and `.equals()`.** `==` on objects (including `String`) compares references. `equals()` compares contents.
- **Recursion with no base case**, or a recursive step that doesn't approach it. You get infinite recursion and a `StackOverflowError`.
- **`fact(0)` with the book's code.** It only stops at `n == 1`, so `fact(0)` or a negative number recurses forever. A safer base case is `n <= 1`. Also `int` overflows quickly (13! no longer fits).
- **Print before vs. after the recursive call** changes the output order (forward vs. reverse).
- **Using a non-`static` member from a `static` method** (including `main`). `main` is static, so it can't touch instance variables without an object.
- **Using `this` in a `static` method.** There is no current object.
- **Expecting `static` variables to be per-object.** There is one shared copy. Changing it through any object (or the class) changes it for all.
- **Assigning a `final` variable twice**, or never initializing it. Compile error.
- **`final` object ≠ immutable object.** `final` fixes the reference, not the object's contents.
- **`length` vs. `length()`.** Arrays use `.length` (no parentheses). Strings use `.length()`.
- **`length` is capacity, not "elements used".** A `new int[10]` has `length` 10 even if empty.
- **Making fields `private` and then accessing them directly from another class.** Compile error. Add getters/setters.
- **Outer class using an inner class's members directly.** Not allowed. Access goes inner → outer, not the reverse.
- **Creating an inner class instance outside its outer-class context.** Needs an outer object.
- **Trying to modify a `String` in place.** Strings are immutable. Methods return **new** strings, so assign the result.
- **Forgetting that command-line args are all strings.** `args[0] + args[1]` with `"2"` and `"3"` gives `"23"`, not 5. Convert first. Also `args[0]` crashes with `ArrayIndexOutOfBoundsException` if no arguments were given.
- **Varargs parameter not last, or more than one.** Compile error.
- **Ambiguous varargs calls.** `vaTest()` with `int...` and `boolean...` overloads. Rename a method instead.
- **Passing an array vs. varargs.** Inside the method both look like arrays. If a method takes `int...`, you may pass either individual `int`s or an `int[]`.

## Flashcards

- What is method overloading? :: Defining several methods in the same class with the same name but different parameter lists (number and/or types) #card
- Can two overloaded methods differ only by return type? :: No. The return type isn't enough to tell them apart; the parameters must differ #card
- How does Java decide which overloaded method to call? :: It matches the number and types of the arguments: exact match first, then automatic type conversion if needed #card
- If only `test(double)` exists and you call `test(88)`, what happens? :: The `int` is widened to `double` and `test(double)` is called #card
- How does overloading relate to polymorphism? :: It implements "one interface, multiple methods": one general name, with the compiler choosing the specific version (e.g. `Math.abs`) #card
- Why overload constructors? :: To let objects be created in several ways (all values, no values, one value for a cube) #card
- What is a copy constructor? :: A constructor taking an object of its own class and copying its fields, e.g. `Box(Box ob)` #card
- Is `Box b2 = b1;` the same as `new Box(b1)` with a copy constructor? :: No. The first copies the reference (one object); the second creates a new, separate object #card
- What's the difference between call-by-value and call-by-reference? :: By value passes a copy of the value; by reference passes access to the original variable #card
- Is Java call-by-value or call-by-reference? :: Always call-by-value. For objects, the value copied is the reference #card
- What happens to a caller's `int` when the method changes its parameter? :: Nothing, because the method has only a copy #card
- If you pass an object and the method changes its fields, does the caller see it? :: Yes. The parameter and argument are references to the same object #card
- If a method does `o = new Test(1,1);` on its object parameter, does the caller's variable change? :: No. Only the method's local copy of the reference is re-pointed #card
- Does an object created inside a method disappear when the method ends? :: No. It lives as long as a reference to it exists (e.g. a returned reference); then it becomes eligible for garbage collection #card
- What is recursion? :: A method calling itself to solve a smaller version of the same problem #card
- What two parts must every recursive method have? :: A base case that stops the recursion, and a recursive step that moves toward the base case #card
- Where are a recursive method's local variables stored for each call? :: On the call stack; each call gets its own copy, removed when that call returns #card
- What happens with infinite recursion? :: The stack is exhausted and Java throws a `StackOverflowError` #card
- Name a pro and a con of recursion. :: Pro: clearer code for some algorithms (e.g. quicksort, trees). Con: slower and uses stack space #card
- In `printArray`, why does the print come after the recursive call? :: So elements print in forward order; printing before the call would reverse the order #card
- What is access control? :: Using modifiers (`public`, `private`, etc.) to decide which code may use a class member #card
- What does `private` do? :: The member can be used only by code inside the same class #card
- What does `public` do? :: The member can be used by any code #card
- What access does a member have if no modifier is given? :: Default (package) access: usable within the same package, not outside it #card
- Why is `main()` declared `public`? :: It is called by the Java run-time system, which is outside the class #card
- What is the standard pattern for protecting data? :: Make fields `private` and expose `public` getter/setter methods #card
- What does a `static` member belong to? :: The class itself, not any object; it can be used without creating an object #card
- Why is `main()` static? :: The JVM must call it before any object exists #card
- How many copies of a `static` variable exist? :: One, shared by all objects of the class #card
- What are the three restrictions on `static` methods? :: They can directly call only static methods, directly access only static data, and cannot use `this` or `super` #card
- When does a `static { }` block run? :: Exactly once, when the class is first loaded #card
- How do you call a static method from outside its class? :: With the class name: `ClassName.method()` #card
- What does `final` do on a variable? :: Makes it a constant: it can be assigned only once (at declaration or in a constructor) #card
- Does `final` on a reference make the object unchangeable? :: No. It only stops the reference from being re-pointed; the object's fields can still change #card
- What is the naming convention for constants? :: ALL_UPPERCASE, like `FILE_OPEN` #card
- How do you get an array's size? :: The `length` field: `arr.length` (no parentheses) #card
- What does `array.length` represent? :: The number of elements the array was created to hold, not how many are in use #card
- What is a nested class? :: A class defined inside another class, with scope limited to the enclosing class #card
- What is the difference between a static nested class and an inner class? :: A static nested class is declared `static` and can't directly use the outer's non-static members; an inner class is non-static and can use all of the outer's members #card
- Can an outer class directly use an inner class's variables? :: No. Access goes inner → outer only; the outer needs an inner object #card
- What does it mean that `String` is immutable? :: Once created, its contents can't change; operations produce a new `String` #card
- What is the difference between `==` and `equals()` for strings? :: `==` compares references; `equals()` compares contents #card
- Which mutable classes exist for changing text? :: `StringBuilder` and `StringBuffer` #card
- What do `length()` and `charAt(i)` return for a string? :: The number of characters, and the char at index `i` (0-based) #card
- Where do command-line arguments arrive? :: As the `String[] args` parameter of `main()`, starting at `args[0]` #card
- What type are command-line arguments? :: Always `String`; numbers must be converted manually #card
- What is a varargs parameter and how is it written? :: A parameter accepting zero or more arguments of one type, written `type ... name` #card
- What is a varargs parameter inside the method? :: An array of that type (`int ... v` makes `v` an `int[]`) #card
- What are the two rules for varargs parameters? :: It must be the last parameter, and there can be only one per method #card
- Why is `vaTest()` ambiguous with `vaTest(int...)` and `vaTest(boolean...)`? :: An empty argument list matches both, so the compiler can't choose #card
- Why is `vaTest(1)` ambiguous with `vaTest(int...)` and `vaTest(int, int...)`? :: It could be one varargs element or the normal `int` with zero varargs #card

## Open questions

- [ ] Why can't Java swap two variables in a method, and how do you swap array elements or object fields instead?
- [ ] What is the difference between `this(...)` and overloading, and how do I make one constructor call another to avoid repeated code?
- [ ] What are packages, and how do default vs. `protected` access really differ? (Next chapters.)
- [ ] How does the JVM store the call stack vs. the heap, and what exactly triggers a `StackOverflowError`?
- [ ] When should I use recursion vs. a loop, and what is tail recursion (does Java optimize it)?
- [ ] Why is `String` immutable, how does the String pool work, and why does `new String("a") == "a"` give `false`?
- [ ] What are anonymous inner classes and lambdas, and how do they relate to the inner classes here?
- [ ] How does overload resolution rank widening, boxing, and varargs when several methods could match?

## Key terms

|Term|Definition|
|---|---|
|Method overloading|Several methods in one class with the same name but different parameter lists|
|Overload resolution|The compiler's process of picking which overloaded method a call refers to|
|Polymorphism ("one interface, multiple methods")|One general name used for related operations on different data|
|Constructor overloading|Multiple constructors with different parameter lists|
|Copy constructor|Constructor that initializes an object from another object of the same class|
|Call-by-value|Passing a copy of the argument's value to the method|
|Call-by-reference|Passing access to the original variable (Java does not do this)|
|Recursion|A method calling itself|
|Base case|The condition in a recursive method that ends the recursion|
|Call stack|Memory holding each active method call's parameters and local variables|
|Access modifier|Keyword (`public`, `private`, `protected`) controlling who can access a member|
|Default (package) access|Access when no modifier is given: visible within the same package|
|Getter / setter|Public methods that read / write a private field|
|`static`|Marks a member as belonging to the class rather than to objects|
|Static initialization block|`static { }` block that runs once when the class loads|
|`final`|Prevents a variable from being reassigned (a constant)|
|`length` (array)|Field holding the number of elements an array can hold|
|Nested class|A class defined inside another class|
|Inner class|A non-static nested class that can access its outer class's members|
|Static nested class|A nested class declared `static`; needs an object to reach outer instance members|
|Immutable|Cannot be changed after creation (like `String`)|
|Command-line argument|Text typed after the program name, passed to `main()` as `String[] args`|
|Varargs|Parameter (`type ...`) accepting a variable number of arguments, treated as an array|
|Ambiguous call|A call that matches several overloads equally well, causing a compile error|

## Related

→ Next: [[INHERITANCE]]