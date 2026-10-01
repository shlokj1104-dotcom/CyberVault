---
title: DATA TYPES, VARIABLES AND ARRAYS
date: 2026-09-30
language: Java
phase: phase-1
tags:
links: []
status: learning
---
> **One-line summary:** Java is strongly typed: every variable and expression has a fixed type, and the compiler refuses mismatches. It has 8 primitive types with the same size on every machine, strict rules for when types convert automatically, and arrays that are real objects with bounds checking.

## What it is

This chapter covers the three things every program is built on: **data types** (what kind of value), **variables** (named storage for a value), and **arrays** (a group of same-type values under one name).

### Java is strongly typed

**In plain words:** Java always knows, and enforces, what type of data each thing holds.

- Every variable has a type, every expression has a type, and every type is strictly defined.
- Every assignment, including passing arguments to methods, is checked for compatibility. Java does **not** silently coerce conflicting types the way some languages do.
- A type mismatch is a **compile-time error**. It must be fixed before the class will compile.

**Why it matters:** a whole class of bugs (putting the wrong kind of data in the wrong place) is caught before the program ever runs. This is a big part of Java's safety and robustness.

### The 8 primitive types

Primitive (or "simple") types hold **one plain value**, not an object. There are exactly eight, in four groups:

|Group|Types|Holds|
|---|---|---|
|Integers|`byte`, `short`, `int`, `long`|Whole numbers (signed)|
|Floating-point|`float`, `double`|Numbers with a fractional part|
|Character|`char`|A single Unicode character|
|Boolean|`boolean`|`true` or `false`|

**Why aren't primitives objects, if Java is object-oriented?** Efficiency. Making every `int` a full object would slow programs down too much. So primitives are the one non-object part of the language.

**Why do the sizes never change?** In C/C++, the size of an `int` depends on the machine. In Java, an `int` is **always 32 bits**, on every platform. This is required for portability: a program compiled once must behave identically everywhere ("write once, run anywhere"). The cost is a tiny performance loss on some hardware, which the designers accepted.

## Structure

### Integer types

|Type|Width|Range|
|---|---|---|
|`byte`|8 bits|-128 to 127|
|`short`|16 bits|-32,768 to 32,767|
|`int`|32 bits|about -2.1 billion to 2.1 billion (-2,147,483,648 to 2,147,483,647)|
|`long`|64 bits|about -9.2 quintillion to 9.2 quintillion (-9,223,372,036,854,775,808 to 9,223,372,036,854,775,807)|

- **All are signed.** Java has **no unsigned integer types** (unlike C). The designers felt unsigned was mostly about how the top bit (the sign bit) behaves, and Java handles that with a separate operator (unsigned right shift `>>>`, covered in the operators chapter).
- **"Width" describes behavior, not storage.** It tells you the range and behavior your variable is guaranteed to have. The JVM may store it however it likes.
- **`int` is the default choice.** It is used for loop counters, array indexes, and most whole-number math.
    - You might think `byte`/`short` save time or space. Usually they don't help, because they are **promoted to `int`** whenever they're used in an expression (see type promotion below).
- **`byte`** is useful for raw binary data: network streams, files.
- **`short`** is rarely used.
- **`long`** is for values too big for `int`, like large counts, timestamps in milliseconds, or big products. Example: the distance light travels in 1000 days in miles is about 16 trillion, which cannot fit in an `int`.

```java
int a = 2_000_000_000;
int b = a + a;          // overflow! wraps around to a negative number, no error
long c = 2_000_000_000L + 2_000_000_000L;   // 4,000,000,000: fine
```

_(Overflow wrapping silently is beyond the chapter, but it is a classic interview point.)_

### Floating-point types

|Type|Width|Precision|Notes|
|---|---|---|---|
|`float`|32 bits|single (about 7 decimal digits)|Uses half the memory. Loses precision for very large or very small values|
|`double`|64 bits|double (about 15 decimal digits)|The default for decimal numbers. All math functions (`sin()`, `cos()`, `sqrt()`) return `double`|

- Both follow the IEEE-754 standard.
- **Use `double` by default.** Use `float` only when memory matters and precision doesn't. On many modern processors `double` is just as fast.
- **Never use `float` or `double` for money** _(the book suggests `float` for dollars and cents, but that is bad practice in real code)_. Binary floating-point can't represent many decimals exactly: `0.1 + 0.2` gives `0.30000000000000004`. For money, use `BigDecimal` or store whole cents in a `long`.

### `char` (Java is not C here)

- `char` is **16 bits** and holds a **Unicode** character. In C/C++, `char` is 8 bits.
- Range: **0 to 65,535**. There are no negative `char`s. It is unsigned, the only unsigned type in Java.
- Why Unicode? It covers characters from all human languages (Latin, Greek, Arabic, Hindi, Chinese, and so on), which Java needs for worldwide portability. The cost: it's less compact for English text.
- ASCII is the first 128 values of Unicode (0-127), so all the "old tricks" from C still work: `'X'` is 88.
- A `char` behaves like an integer, so you can do arithmetic on it:

```java
char ch = 'X';
ch++;                   // now 'Y': the next character in the sequence
char c2 = 88;           // 88 is 'X'; an int literal that fits is allowed
System.out.println(ch + 1);   // prints 90 (an int!), not a char
```

### `boolean`

- Only two values: `true` and `false`.
- It is the type returned by all relational operators (`<`, `>`, `==`, ...) and the type **required** by conditions in `if`, `for`, and loops.
- **`true`/`false` are NOT 1/0.** They don't convert to numbers, and numbers don't convert to boolean. In C, `if (1)` works. In Java, it's a compile error.
- No need to write `if (b == true)`. Just write `if (b)`.
- Because the condition must be `boolean`, the C bug `if (x = 5)` (assignment instead of comparison) is a **compile error** in Java for `int` variables. This is a safety win.
- Watch operator precedence when printing: `"10 > 9 is " + (10 > 9)` needs the parentheses because `+` binds tighter than `>`.

### Literals in detail

A **literal** is a constant value written directly in code. The type of the literal matters.

**Integer literals**

|Form|Example|Notes|
|---|---|---|
|Decimal|`42`|Type is `int` by default|
|Octal|`017`|Leading zero. So `09` is a **compile error**, since 9 isn't a valid octal digit|
|Hexadecimal|`0xFF`|Prefix `0x` or `0X`|
|Binary|`0b1010`|Prefix `0b` (Java 7+). Great for bit masks|
|Long|`100L`|Suffix `L` (use uppercase, `l` looks like `1`)|
|With underscores|`1_000_000`|Java 7+, purely for readability. Ignored by the compiler. Only between digits|

- An integer literal is an `int` by default. It may be assigned directly to `byte`, `short`, or `char` **if the value fits in range**. Anything bigger than `int` needs the `L` suffix: `long big = 3000000000L;` (without `L` it's an error, because the literal is too big for an `int`).

**Floating-point literals**

- `3.14` and `2.0` are **`double` by default**. To get a `float`, add `f` or `F`: `float f = 3.14f;`. Without the `f`, it's a compile error, because narrowing `double` to `float` isn't automatic.
- Scientific notation works: `6.022E23`, `2e-5`.

**Boolean literals:** `true`, `false` only.

**Character literals** use single quotes: `'a'`, `'@'`. For special characters, use escape sequences:

|Escape|Meaning|
|---|---|
|`\n`|New line|
|`\t`|Tab|
|`\\`|Backslash|
|`\'`|Single quote|
|`\"`|Double quote|
|`\uXXXX`|Unicode character by hex code (e.g. `'\u0061'` is `'a'`)|

**String literals** use double quotes: `"Hello"`, `"two\nlines"`, `"\"quoted\""`.

- A string literal must start and end **on the same line**. Java has no line-continuation escape.
- **A `String` is an object, not a primitive and not a char array** (unlike C/C++). It has many built-in methods, covered in a later chapter. For now: you can declare `String` variables, assign string literals to them, and print them. `String` is not in the 8 primitives.

### Variables

A **variable** is the basic unit of storage. It's defined by an **identifier** (name), a **type**, and an optional **initializer**, and it has a **scope** and a **lifetime**.

```java
int a, b, c;               // three ints, uninitialized
int d = 3, e, f = 5;       // d and f are initialized
double pi = 3.14159;
char x = 'x';
```

- Declare before use. The name says nothing about the type. Any valid identifier can have any type.
- **Dynamic initialization:** the initial value can be any expression valid at that moment, not just a constant.

```java
double a = 3.0, b = 4.0;
double c = Math.sqrt(a * a + b * b);   // computed at run time: 5.0
```

### Scope and lifetime

**Scope** = where a variable can be seen. **Lifetime** = how long it exists. In Java, both are controlled by **blocks** (`{ }`).

Rules to remember:

1. **A variable is visible only inside the block where it is declared** (and blocks nested inside it), and only **after** the line that declares it.
2. **Outer variables are visible in inner blocks. Inner variables are not visible outside.**
3. **A variable is created when its block is entered and destroyed when the block ends.** It doesn't keep its value between method calls or between visits to the block.
4. **If it has an initializer, that runs every time the block is entered.**
5. **You cannot declare an inner variable with the same name as one in an enclosing scope** (compile error). In C/C++ shadowing is allowed. In Java local variables can't shadow other local variables.

```java
int x = 10;
if (x == 10) {
    int y = 20;        // y exists only inside this block
    x = y * 2;         // fine: x is from the outer scope
}
// y = 100;            // ERROR: y is not visible here

for (int i = 0; i < 3; i++) {
    int y = -1;        // re-created and re-set to -1 on EVERY pass
    y = 100;           // this value is lost when the block ends
}

int bar = 1;
{
    // int bar = 2;    // ERROR: bar already defined in the enclosing scope
}
```

**Why scope matters:** it is the foundation of encapsulation. Limiting where a variable can be seen protects it from unintended changes. Java's two big scope categories are **class scope** and **method scope** (there is no true "global" scope). Class scope comes in a later chapter.

_(Beyond the chapter, and asked constantly: **local variables get no default value.** Using one before assigning it is a compile error. Only array elements and class fields get default values.)_

## Mechanics / reference

### Type conversion and casting

You often assign one type to another. Java has two mechanisms.

**1. Automatic (widening) conversion**

Happens when **both** are true:

- the types are compatible, and
- the **destination is larger** than the source.

```
byte → short → int → long → float → double
              char → int
```

Nothing can be lost, so Java does it for you. Examples: `int` to `long`, `int` to `double`.

- Numeric types (integer and floating-point) are compatible with each other.
- There is **no** automatic conversion from numeric types to `char` or `boolean`, and `char` and `boolean` aren't compatible with each other.
- Special case: an integer **literal** can be assigned to `byte`/`short`/`char` if it fits. (`byte b = 22;` is fine, even though `22` is an `int`.)

**2. Explicit casting (narrowing)**

When the destination is smaller, or the types aren't automatically compatible, you must write a **cast** to say "I know what I'm doing": `(target-type) value`.

```java
int i = 257;
byte b = (byte) i;       // b = 1
double d = 323.142;
int n = (int) d;         // n = 323   (fraction is cut off)
byte bb = (byte) d;      // bb = 67
```

What actually happens (know this for interviews):

- **Integer → smaller integer:** the value is reduced **modulo the target's range**. `257 mod 256 = 1`. Data can be lost or change sign.
- **Floating-point → integer:** **truncation**. The fractional part is dropped (not rounded!): `1.99` becomes `1`, `-1.99` becomes `-1`.
- **Double → byte:** both happen. First truncate to 323, then reduce mod 256 to get 67.

### Automatic type promotion in expressions

Even without assignment, Java converts types **inside expressions**. Why: an intermediate result can exceed the range of the operands. For example, `byte a = 40, b = 50, c = 100; int d = a * b / c;` computes `a * b = 2000`, which doesn't fit in a `byte`.

**The promotion rules** (applied in this order):

1. All `byte`, `short`, and `char` values are promoted to **`int`**.
2. Then, if any operand is `long`, the whole expression becomes **`long`**.
3. Else if any operand is `float`, the whole expression becomes **`float`**.
4. Else if any operand is `double`, the whole expression becomes **`double`**.

**The classic gotcha:**

```java
byte b = 50;
b = b * 2;          // COMPILE ERROR: b * 2 is an int, can't assign to a byte
b = (byte)(b * 2);  // OK: explicit cast, b = 100
```

The value 100 would fit in a `byte`, but the compiler only looks at the **type** (`int`), not the value. Same for `short`, and for `char`: `char c = 'a'; c = c + 1;` is an error.

_(Interview follow-up: `b += 2;` and `b++` **do** compile. Compound assignment operators include an implicit cast back to the variable's type.)_

**Worked example** (why the type of each part matters):

```java
byte b = 42; char c = 'a'; short s = 1024; int i = 50000; float f = 5.67f; double d = .1234;
double result = (f * b) + (i / c) - (d * s);
```

- `f * b` → `b` promoted to float → **float**
- `i / c` → `c` promoted to int (97) → **int division**: 50000 / 97 = 515 (fraction lost!)
- `d * s` → `s` promoted to double → **double**
- float + int → float; float - double → **double**. The final result is a `double`.

### Arrays

An **array** is a group of same-type values referred to by one name, accessed by a numeric **index**. Java arrays differ from C/C++ arrays, so don't assume the C behavior.

**Creating an array is two steps:**

```java
int[] month_days;              // 1. declare: just a variable, NO array exists yet
month_days = new int[12];      // 2. allocate with new: creates 12 ints, links them to the variable

int[] days = new int[12];      // both at once (the normal way)
```

- `new` allocates memory at run time. **All Java arrays are dynamically allocated** (they're objects living on the heap; the variable holds a reference to them).
- **Default values:** elements start as `0` (numeric), `false` (boolean), or `null` (object references).
- **Indexes start at 0.** A 12-element array has indexes 0 to 11.
- **Bounds are checked at run time.** Going outside the range (negative or ≥ length) causes a **run-time error** (`ArrayIndexOutOfBoundsException`), not silent memory corruption as in C. This is a major safety feature.
- The array's size is fixed once created. Use `array.length` to get it instead of hard-coding the number. _(`.length` is a field, not a method; a later chapter covers it.)_

**Array initializer:** create and fill in one step, with no `new` and no size needed:

```java
int[] month_days = { 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31 };   // length 12
double[] nums = { 10.1, 11.2, 12.3 };
```

**Alternative declaration syntax:** both of these are identical:

```java
int a1[] = new int[3];
int[] a2 = new int[3];    // brackets with the type: cleaner, preferred style
```

The bracket-after-type form is handy for declaring several arrays at once: `int[] nums, nums2, nums3;` makes **three arrays**.

**Trap:** `int a[], b;` makes `a` an array but `b` a plain `int`. Compare: `int[] a, b;` makes both arrays.

### Multidimensional arrays

In Java, a multidimensional array is really an **array of arrays**. A 2D array is an array whose elements are themselves 1D arrays.

```java
int[][] twoD = new int[4][5];    // 4 rows, 5 columns; left index = row, right index = column
twoD[2][3] = 7;
```

- With `new`, only the **first (leftmost) dimension is mandatory**. You can allocate the rest separately.
- Because each row is its own array, rows can have **different lengths**. This is called a **jagged (irregular) array**:

```java
int[][] tri = new int[4][];    // 4 rows, each row not yet created
tri[0] = new int[1];
tri[1] = new int[2];
tri[2] = new int[3];
tri[3] = new int[4];           // a triangle shape
```

Useful for saving memory in large, sparsely filled tables. Otherwise, uneven arrays surprise readers, so use regular ones by default.

- **Initializer** for multidimensional arrays: nested braces, one set per dimension.

```java
double[][] m = {
    { 0*0, 1*0, 2*0, 3*0 },
    { 0*1, 1*1, 2*1, 3*1 }
};   // expressions are allowed inside initializers, not just literals
```

- Three or more dimensions work the same way: `new int[3][4][5]`.

### Strings, briefly

`String` is a **class** (an object type), not a primitive and not a `char[]` as in C. A quoted string literal can be assigned to a `String` variable, and `String` variables can be printed and joined with `+`.

```java
String str = "this is a test";
System.out.println(str);
```

_(Beyond the chapter: `String` objects are **immutable**, so operations create new strings. This comes up a lot in interviews.)_

### No pointers in Java

Java does **not** let the programmer use pointers (no `*`, no `&`, no pointer arithmetic). Reason: pointers can hold any memory address, including addresses **outside the Java runtime**, which would let a program break through the safety barrier between the Java environment and the host machine. This is central to Java's security model (see the applet/security discussion in Chapter 1).

**You don't lose anything.** Objects and arrays are accessed through **references**, which behave like safe pointers: you can't do arithmetic on them or point them at arbitrary memory. Inside the safe environment you never need a pointer.

## Pitfalls

- **`byte b = b * 2;` doesn't compile.** `byte`, `short`, `char` are promoted to `int` in expressions, so the result is an `int`. Fix: cast, `b = (byte)(b * 2)`.
- **`float f = 3.14;` doesn't compile.** Decimal literals are `double`. Write `3.14f`.
- **`long big = 3000000000;` doesn't compile.** The literal is an `int`, and it's too big. Write `3000000000L`.
- **Integer overflow is silent.** `int` wraps around with no error. Use `long` (or `Math.addExact`) when values can get large.
- **Integer division truncates.** `7 / 2` is `3`, not `3.5`. Make at least one operand a `double`: `7 / 2.0`.
- **Casting a `double` to `int` truncates, it doesn't round.** `(int) 2.9` is `2`.
- **A narrowing cast can change the value or sign.** `(byte) 200` is `-56`.
- **Using floating-point for money.** Use `BigDecimal` or whole cents in a `long`.
- **A leading zero means octal.** `int x = 010;` is 8, and `09` is an error.
- **`if (x = 5)` in Java** is a compile error for ints (it isn't a boolean). It compiles only if `x` is a `boolean`, so the bug can still exist for `if (flag = true)`.
- **Using a variable outside its block.** A variable declared in a loop or `if` block is gone after it.
- **Re-declaring a local with the same name** in an inner block is an error (unlike C/C++).
- **Expecting array elements to grow.** Array size is fixed. Going out of bounds throws a run-time error.
- **Thinking `String` is a primitive or a `char` array.** It's an object.
- **`char + int` gives an `int`.** `'a' + 1` is `98`, not `'b'`. Cast back with `(char)`.
- **Assuming C-sized types.** In Java, `int` is always 32-bit and `char` is 16-bit Unicode.
- **`int a[], b;`** makes `b` an `int`, not an array.

## Flashcards

- What does "strongly typed" mean in Java? :: Every variable and expression has a defined type, and all assignments and method arguments are checked for compatibility. Mismatches are compile-time errors #card
- Name the 8 primitive types and their four groups. :: Integers: byte, short, int, long. Floating-point: float, double. Character: char. Boolean: boolean #card
- Why are primitives not objects? :: Efficiency. Making them objects would slow programs too much #card
- Why does Java fix the size of every primitive (e.g. int = 32 bits always)? :: Portability. Programs must behave the same on every platform, unlike C/C++ where int size varies by machine #card
- Sizes of byte, short, int, long? :: 8, 16, 32, 64 bits #card
- Range of byte? :: -128 to 127 #card
- Does Java have unsigned integers? :: No. All integer types are signed (only `char` is unsigned) #card
- Why is `int` usually preferred over `byte`/`short`? :: byte and short are promoted to int in expressions anyway, so there's usually no gain #card
- float vs double? :: float is 32-bit single precision, double is 64-bit double precision. double is the default for decimal literals and for math library results #card
- Why shouldn't you use float/double for money? :: Binary floating point can't represent many decimal values exactly (0.1 + 0.2 != 0.3). Use BigDecimal or integer cents #card
- How is `char` in Java different from `char` in C? :: Java `char` is 16-bit Unicode (0 to 65,535, unsigned); C's is 8-bit #card
- Can you do arithmetic on a `char`? :: Yes. It behaves like an integer, so `ch++` moves to the next character. But `ch + 1` yields an int #card
- Is `true` equal to 1 in Java? :: No. Booleans don't convert to or from numbers, and conditions must be boolean #card
- What type is the literal `100`? What about `3.14`? :: `100` is an `int`; `3.14` is a `double` #card
- How do you write a long literal and a float literal? :: Suffix `L` (`100L`) and `f` (`3.14f`) #card
- What are binary and hex literal prefixes, and what's special about a leading 0? :: `0b` binary, `0x` hex; a leading `0` means octal (so `09` is an error) #card
- What are underscores in numeric literals for? :: Readability only (`1_000_000`); the compiler ignores them; only allowed between digits #card
- What are the rules for variable scope? :: A variable is visible only in its block (and nested blocks) after its declaration; it's created on block entry and destroyed on exit #card
- Can an inner block declare a variable with the same name as an outer local? :: No. That's a compile-time error in Java (unlike C/C++) #card
- If a variable has an initializer inside a loop body, when does it run? :: Every time the block is entered #card
- When does Java convert types automatically (widening)? :: When the types are compatible and the destination is larger than the source (e.g. int to long) #card
- Which conversions never happen automatically? :: Numeric to char or boolean, and char to boolean. Narrowing conversions need an explicit cast #card
- What does casting `(byte) 257` give, and why? :: 1. The value is reduced modulo 256 (the byte range) #card
- What happens when a double is cast to an int? :: The fractional part is truncated (not rounded) #card
- What are the type promotion rules in expressions? :: byte/short/char become int; then if any operand is long the result is long, else float, else double #card
- Why does `byte b = 50; b = b * 2;` fail? :: `b * 2` is an int (promotion), and assigning an int to a byte needs an explicit cast #card
- How do you create an array in Java? :: Declare the variable, then allocate with `new` (`int[] a = new int[12];`) #card
- What are the default values of array elements? :: 0 for numeric types, false for boolean, null for references #card
- What happens on an out-of-bounds array index? :: A run-time error (exception). Java checks every index, unlike C #card
- What is a multidimensional array in Java? :: An array of arrays. Rows can have different lengths (jagged arrays) #card
- For `new int[4][]`, what exists? :: An array of 4 row references (all null); each row must be allocated separately #card
- What does `int a[], b;` declare? :: `a` is an int array, `b` is a plain int #card
- Is `String` a primitive or a char array? :: Neither. It's a class, so strings are objects #card
- Why doesn't Java have pointers? :: Pointers could reach memory outside the Java runtime and break its security; references are the safe alternative #card

## Open questions

- [ ] How do wrapper classes (`Integer`, `Double`, ...) and autoboxing connect the primitives to the object world?
- [ ] Why are Strings immutable, and what is the string pool?
- [ ] What exactly happens on integer overflow, and how does two's complement explain `(byte) 200 == -56`?
- [ ] How does `BigDecimal` avoid floating-point errors, and when is it worth the cost?
- [ ] What is the difference between a reference variable and a C pointer, and what does "pass by value" mean for references?

## Key terms

|Term|Definition|
|---|---|
|Strongly typed|Every variable and expression has a fixed type and the compiler enforces type compatibility|
|Primitive (simple) type|One of Java's 8 built-in non-object types: byte, short, int, long, float, double, char, boolean|
|Signed|Can hold both negative and positive values|
|IEEE-754|The standard that defines Java's floating-point types and operations|
|Unicode|A universal character set covering all human languages; Java `char` is 16-bit Unicode|
|Literal|A constant value written directly in code (`42`, `3.14f`, `'a'`, `"text"`)|
|Escape sequence|A backslash code for a special character (`\n`, `\t`, `\\`, `\uXXXX`)|
|Variable|A named storage location with a type, and optionally an initial value|
|Dynamic initialization|Initializing a variable with an expression computed at run time|
|Scope|The region of code where a variable is visible (defined by blocks)|
|Lifetime|How long a variable exists: from entering its scope to leaving it|
|Widening conversion|Automatic conversion to a larger, compatible type (int to long)|
|Narrowing conversion|Conversion to a smaller type; requires an explicit cast and may lose data|
|Cast|An explicit type conversion: `(type) value`|
|Truncation|Dropping the fractional part when converting a floating-point number to an integer|
|Type promotion|Automatic widening of operands inside an expression (byte/short/char to int, then long, float, double)|
|Array|A fixed-size group of same-typed elements accessed by index|
|Array initializer|A comma-separated list in braces that creates and fills an array: `{1, 2, 3}`|
|`new`|Operator that allocates memory (here, for arrays) at run time|
|Jagged array|A multidimensional array whose rows have different lengths|
|Reference|A safe handle to an object or array, Java's replacement for pointers|

## Related


→ Next: [[OPERATORS]]