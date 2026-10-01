---
title: OPERATORS
date: 2026-10-01
language: Java
phase: phase-1
tags:
links: []
status: learning
---
> **One-line summary:** Java's operators fall into four main groups (arithmetic, bitwise, relational, logical), plus assignment and the ternary `? :`. The things that matter most are integer division and `%`, prefix vs. postfix `++`, how bits and shifts work (two's complement, `>>` vs `>>>`), short-circuit `&&`/`||`, and the precedence rules that decide how an expression is evaluated.

## What it is

An **operator** is a symbol that performs an operation on one or more values called **operands.** An expression like `a + b * 2` combines operands and operators to produce a value.

- **Unary** operators take one operand (`-x`, `!flag`, `i++`).
- **Binary** operators take two (`a + b`, `x < y`).
- **Ternary** operator takes three (`cond ? a : b`). Java has exactly one.

The groups, at a glance:

|Group|Operators|Works on|Result|
|---|---|---|---|
|Arithmetic|`+ - * / % ++ --`|numeric types (and `char`)|a number|
|Bitwise|`~ & \| ^ << >> >>>`|integer types (`byte short int long char`)|an integer|
|Relational|`== != > < >= <=`|numbers (all six); any type (`==`, `!=` only)|`boolean`|
|Boolean logical|`& \| ^ ! && \|`|`boolean` only|`boolean`|
|Assignment|`=` and `op=` (e.g. `+=`, `<<=`)|any compatible types|the assigned value|
|Ternary|`? :`|`boolean` condition, two compatible results|one of the two results|

_(The book defers `instanceof` to a later chapter and the lambda arrow `->` to another.)_

## Structure

### Arithmetic operators

|Operator|Meaning|
|---|---|
|`+` `-`|Add, subtract (also unary plus/minus)|
|`*` `/` `%`|Multiply, divide, remainder (modulus)|
|`++` `--`|Increment, decrement|

- Operands must be **numeric**. You can't use them on `boolean`. You _can_ use them on `char`, because `char` is essentially an integer type.
- **Integer division throws away the fraction.** `7 / 2` is `3`, not `3.5`. If either operand is floating-point, you get floating-point division.

```java
int a = 6 / 4;           // 1       (integer division)
double d = 6 / 4.0;      // 1.5     (one operand is double)
double e = 6 / 4;        // 1.0     (!) the division happens in ints first, THEN converts
```

That last line is a classic trap: the type of the _expression_ is decided before the assignment, so the fraction is already gone.

### The modulus operator `%`

Returns the **remainder** of a division. Unlike in some languages, it works on **floating-point** numbers too.

```java
42 % 10        // 2
42.25 % 10     // 2.25
```

_(Beyond the chapter, but asked a lot: the result takes the **sign of the left operand**. `-7 % 3` is `-1`, not `2`. For a "wrap-around" index, use `Math.floorMod(-7, 3)` which gives `2`.)_

Common uses: even/odd check (`n % 2 == 0`), wrapping an index around an array, extracting digits (`n % 10`).

### Compound assignment operators

`a = a + 4;` can be written `a += 4;`. This works for every binary arithmetic and bitwise operator: `+= -= *= /= %= &= |= ^= <<= >>= >>>=`.

General form: `var = var op expression;` becomes `var op= expression;`

```java
int a = 1, b = 2, c = 3;
a += 5;       // a = 6
b *= 4;       // b = 8
c += a * b;   // c = 3 + 48 = 51
c %= 6;       // c = 51 % 6 = 3
```

Two benefits: shorter code, and the variable is only evaluated once.

**Interview point (beyond the chapter):** a compound assignment includes an **implicit cast** back to the variable's type. That's why this compiles and the longer form doesn't:

```java
byte b = 50;
b += 2;          // OK: acts like b = (byte)(b + 2)
b = b + 2;       // ERROR: b + 2 is an int
```

### Increment and decrement: prefix vs. postfix

`++` adds 1, `--` subtracts 1. They can go **before** (prefix) or **after** (postfix) the variable. On their own, there's no difference. Inside a larger expression, there is:

|Form|What happens|Value used in the expression|
|---|---|---|
|Prefix `++x`|increment first|the **new** value|
|Postfix `x++`|use the value, then increment|the **old** value|

```java
int x = 42;
int y = ++x;    // x becomes 43, then y = 43
// vs
x = 42;
y = x++;        // y = 42 (old value), then x becomes 43
```

Equivalent code: `y = ++x;` means `x = x + 1; y = x;`. And `y = x++;` means `y = x; x = x + 1;`.

Worked example: `a = 1, b = 2; c = ++b; d = a++; c++;` gives `a = 2, b = 3, c = 4, d = 1`.

_(Beyond the chapter: `x = x++;` leaves `x` unchanged. The old value is saved, `x` is incremented, then the old value is assigned back over it. A classic interview trick question.)_

### Bitwise operators

These work on the **individual bits** of integer values. To use them well you need two ideas.

**1. Integers are stored as binary.** Each bit position is a power of 2, starting from bit 0 on the right. `42` as a byte is `00101010`, which is 32 + 8 + 2.

**2. Negative numbers use two's complement.** All Java integer types (except `char`) are signed. To make a negative number:

1. **Invert** every bit,
2. then **add 1**.

```
 42  =  00101010
invert  11010101
add 1   11010110  =  -42
```

To decode a negative number, do the same thing again (invert, add 1) and put a minus sign in front.

**Why two's complement?** It avoids "negative zero." With simple inversion, `00000000` (zero) inverts to `11111111`, which would be a second zero. With two's complement, adding 1 to `11111111` overflows out of the byte and gives `00000000` again, so there is exactly one zero, and `11111111` becomes `-1`. It also lets the same hardware add positive and negative numbers.

**The high-order (leftmost) bit is the sign bit.** If you set it, the value becomes negative, whether you meant that or not. Remember this when you do bit manipulation.

**Bitwise logical operators** work on each bit position independently:

| A   | B   | `A \| B` (OR) | `A & B` (AND) | `A ^ B` (XOR) | `~A` (NOT) |
| --- | --- | ------------- | ------------- | ------------- | ---------- |
| 0   | 0   | 0             | 0             | 0             | 1          |
| 1   | 0   | 1             | 0             | 1             | 0          |
| 0   | 1   | 1             | 0             | 1             | 1          |
| 1   | 1   | 1             | 1             | 0             | 0          |
In plain words:

- **AND** `&`: 1 only if **both** bits are 1. Use it to **clear** bits or **test/extract** bits (a mask).
- **OR** `|`: 1 if **either** bit is 1. Use it to **set** bits.
- **XOR** `^`: 1 if the bits **differ**. Use it to **flip** bits (wherever the mask has a 1, the bit inverts; where it has a 0, the bit is unchanged).
- **NOT** `~`: flips **every** bit.

```
  00101010  (42)           00101010  (42)           00101010  (42)
& 00001111  (15)         | 00001111  (15)         ^ 00001111  (15)
  --------                 --------                 --------
  00001010  (10)           00101111  (47)           00100101  (37)
```

`42 & 15` keeps only the low 4 bits (a **mask**). That is exactly how the book converts a byte to hex digits.

### Shift operators

All three shift the bits of a value by a number of positions: `value << num`, `value >> num`, `value >>> num`.

**Left shift `<<`:** moves bits left, fills the right with **zeros**. Bits shifted off the left end are **lost**.

- Each left shift **doubles** the value (as long as nothing overflows): `5 << 1` is 10, `5 << 3` is 40.
- If a 1 reaches the top bit (bit 31 for `int`, bit 63 for `long`), the number turns **negative**.

**Right shift `>>` (signed):** moves bits right and fills the left with a copy of the **sign bit** (this is called **sign extension**). This preserves the sign.

- Each right shift **divides by 2** and discards the remainder: `35 >> 2` is `8`.
- Negative numbers stay negative: `-8 >> 1` is `-4`. Shifting `-1` right always gives `-1`.

**Unsigned right shift `>>>`:** moves bits right and always fills the left with **zeros**, so the result is non-negative (for shifts of 1 or more).

- `-1 >>> 24` is `255`: all 32 bits of `-1` are 1s, and shifting right 24 zero-fills the top 24 bits.
- It exists because Java has no unsigned types. Use it when the bits are raw data (pixels, hashes, flags) and not a signed number.

```
int a = -1;           // 11111111 11111111 11111111 11111111
a >> 24   -->  -1     // sign bit copied in from the left: still all 1s
a >>> 24  -->  255    // zeros fill in from the left:       00000000 00000000 00000000 11111111
```

**The byte/short trap:** `byte` and `short` are **promoted to `int`** (with sign extension!) before the shift, and the result is an `int`.

- Left shift: `byte a = 64; a << 2` is **256** (an `int`). Cast back with `(byte)` and the 1 bit is gone: you get **0**.
- Unsigned right shift on a negative `byte` seems to do nothing: `byte b = (byte)0xf1;` then `b >>> 4` gives `0xff`, not the `0x0f` you'd expect, because `b` was sign-extended to `0xFFFFFFF1` first, shifted, and then cut back to a byte. **Fix:** mask first, `(b & 0xff) >> 4` gives `0x0f`.
- Masking with `0xff` (or `0x0f`) removes the sign-extended bits. You'll see this pattern whenever bytes are handled.

**Using shifts as arithmetic:** `x << 1` doubles and `x >> 1` halves (rounding toward negative infinity for negatives: `-7 >> 1` is `-4`, but `-7 / 2` is `-3`). Compilers do this optimization themselves, so write `x * 2` for clarity and keep shifts for real bit work.

_(Beyond the chapter: the shift distance is taken modulo the type's size. For `int`, `x << 32` is the same as `x << 0`, i.e. nothing happens, not zero.)_

**Bitwise compound assignment:** every binary bitwise operator has one: `&= |= ^= <<= >>= >>>=`. Example: `a = 1, b = 2, c = 3; a |= 4; b >>= 1; c <<= 1; a ^= c;` gives `a = 3, b = 1, c = 6`.

### Relational operators

`==`, `!=`, `>`, `<`, `>=`, `<=`. Every one returns a `boolean`.

- **`==` and `!=` work on any type** (numbers, characters, booleans).
- **The four ordering operators (`<`, `>`, `<=`, `>=`) work only on numeric types** (integers, floating-point, `char`). You can't order two booleans.
- Result is a `boolean`, so `boolean c = a < b;` is valid.
- Remember: equality is `==` (two signs). A single `=` is assignment.

**C/C++ difference (important for you):** in C, any non-zero value is true and zero is false, so `if (done)` and `if (!done)` work on an `int`. **In Java they don't compile.** `true`/`false` are not numbers. You must write the comparison out:

```java
int done = 0;
if (done == 0) { ... }     // Java style
if (done != 0) { ... }
// if (done) { ... }       // ERROR in Java: int cannot be used as a condition
```

_(Beyond the chapter: for **objects**, `==` compares **references** (are they the same object?), not contents. Compare strings with `s1.equals(s2)`. Also, never compare floating-point numbers with `==` after calculations: `0.1 + 0.2 == 0.3` is `false`.)_

### Boolean logical operators

These work **only on `boolean`** operands and give a `boolean`.

|Operator|Meaning|Operator|Meaning|
|---|---|---|---|
|`&`|Logical AND|`!`|NOT|
|`\|`|Logical OR|`&&`|**Short-circuit** AND|
|`^`|Logical XOR|`\|`|**Short-circuit** OR|

Truth table (same logic as bitwise, with true/false):

|A|B|`A \| B`|`A & B`|`A ^ B`|`!A`|
|---|---|---|---|---|---|
|false|false|false|false|false|true|
|true|false|true|false|true|false|
|false|true|true|false|true|true|
|true|true|true|true|false|false|

### Short-circuit operators: `&&` and `||`

**The idea:** `A && B` is false whenever `A` is false, no matter what `B` is. And `A || B` is true whenever `A` is true. So why bother evaluating `B`? The short-circuit versions **skip the right-hand side** when the left side already decides the result.

|Expression|Right side evaluated when...|
|---|---|
|`A && B`|`A` is **true** (if `A` is false, result is false, `B` skipped)|
|`A \| B`|`A` is **false** (if `A` is true, result is true, `B` skipped)|

The plain `&` and `|` **always evaluate both sides**.

**Why it matters (safety guard):**

```java
if (denom != 0 && num / denom > 10) { ... }   // safe: if denom is 0, the division never runs
if (denom != 0 &  num / denom > 10) { ... }   // CRASH when denom is 0: both sides always run
```

Same idea for null checks: `if (s != null && s.length() > 0)`.

**Standard practice:** use `&&` and `||` for boolean logic, and keep single `&` / `|` for bitwise work. The only time to use `&` on booleans is when you **need** the right side to run for its side effect:

```java
if (c == 1 & e++ < 100) d = 100;   // e++ happens whether or not c == 1
```

_(The language spec calls `&&` / `||` the "conditional-and" and "conditional-or.")_

### The assignment operator `=`

`var = expression;` The type of the expression must be compatible with the variable.

`=` is an **operator that produces a value** (the value assigned), and it associates **right to left**. This allows chained assignment:

```java
int x, y, z;
x = y = z = 100;    // z = 100 first, then y = (that value), then x
```

### The ternary operator `? :`

A compact if-else that **produces a value**: `condition ? valueIfTrue : valueIfFalse`

```java
int k = (i < 0) ? -i : i;                      // absolute value
int ratio = (denom == 0) ? 0 : num / denom;    // avoid divide by zero
```

- The condition must be a `boolean`. Only the chosen branch is evaluated.
- Both result expressions must have the **same or compatible type** and can't be `void`.
- Use it for simple choices. For anything longer, a normal `if` is easier to read.

## Mechanics / reference

### Operator precedence

**Precedence** decides which operator is applied first when an expression has several. Operators on the same row have equal precedence; higher rows bind tighter.

|Level (highest at top)|Operators|
|---|---|
|1|`[ ]` `( )` `.` (act like operators)|
|2|`++` `--` (postfix)|
|3|`++` `--` (prefix), `~`, `!`, unary `+`, unary `-`, type-cast|
|4|`*` `/` `%`|
|5|`+` `-`|
|6|`>>` `>>>` `<<`|
|7|`>` `>=` `<` `<=` `instanceof`|
|8|`==` `!=`|
|9|`&`|
|10|`^`|
|11|`\|`|
|12|`&&`|
|13|`\|`|
|14|`? :`|
|15|`->` (lambda)|
|16 (lowest)|`=` `op=`|

**Associativity (order among equal precedence):**

- Binary operators evaluate **left to right**: `a - b - c` is `(a - b) - c`.
- **Assignment evaluates right to left**: `x = y = 5` is `x = (y = 5)`. _(Unary operators and the ternary also group right to left.)_

**Memory aids:**

- Rough order: **unary → arithmetic (`* /` before `+ -`) → shifts → relational → equality → `&` → `^` → `|` → `&&` → `||` → `? :` → assignment.**
- Bitwise operators rank **below** comparison, so `a & b == c` is parsed as `a & (b == c)`, which is usually a bug. Write `(a & b) == c`.
- `+` ranks above shifts and relational operators: `a >> b + 3` means `a >> (b + 3)`, and `"x is " + a > b` is a compile error.

### Using parentheses

Parentheses **override precedence** and make intent clear. They cost nothing at run time.

```java
a >> b + 3        // same as a >> (b + 3)
(a >> b) + 3      // shift first, then add

a | 4 + c >> b & 7           // hard to read
(a | (((4 + c) >> b) & 7))   // same meaning, clear
```

When an expression mixes bitwise, shift, and comparison operators, add parentheses even if they're technically redundant.

### Which operators care about types

- **Arithmetic** on `byte`/`short`/`char` promotes to `int` (Chapter 3 rules).
- **Bitwise and shift** follow the same promotion. The result of `byte << n` is an `int`.
- **Compound assignment** casts back to the variable's type for you.
- **Relational** results are always `boolean`.
- **Logical** operators accept only `boolean`.

## Pitfalls

- **Integer division loses the fraction.** `1 / 2` is `0`. `double x = 1 / 2;` is `0.0`. Make one operand a `double` (`1 / 2.0`) before dividing.
- **Dividing by zero:** integers throw `ArithmeticException`; floating-point gives `Infinity` or `NaN` with no error. _(Beyond the chapter.)_
- **`%` with negatives:** `-7 % 3` is `-1`. The sign follows the left operand.
- **`i++` vs `++i` inside an expression.** Postfix gives the old value, prefix the new. And `x = x++` does nothing.
- **`b = b + 1` on a `byte` fails; `b += 1` works.** Compound assignment casts implicitly.
- **`=` vs. `==`** in conditions. In Java this is a compile error for numbers, but `if (flag = true)` compiles for `boolean` variables.
- **C-style truthiness doesn't exist.** `if (n)` or `while (1)` is an error. Write `n != 0`.
- **`==` on objects compares references.** Use `.equals()` for contents (especially strings).
- **`&` vs `&&` (and `|` vs `||`).** The single versions don't short-circuit, which can cause null-pointer or divide-by-zero crashes. They also have much lower precedence than comparisons, so `a & b == c` is a surprise.
- **Right shift of a negative number keeps it negative** (sign extension). Use `>>>` for a zero-fill shift.
- **`>>>` on `byte`/`short` appears to do nothing.** Values are promoted (sign-extended) to `int` first. Mask with `& 0xff` before shifting.
- **Left shift can flip the sign.** A 1 reaching bit 31 makes an `int` negative.
- **Left-shifting a `byte` then casting back loses bits.** `(byte)(64 << 2)` is `0`.
- **Operator precedence surprises.** `+` binds tighter than shifts and comparisons; bitwise ops bind looser than `==`. When unsure, add parentheses.
- **Chained `+` with strings goes left to right.** `1 + 2 + "a"` is `"3a"`, but `"a" + 1 + 2` is `"a12"`. _(Beyond the chapter.)_
- **Overflow is silent.** `int` arithmetic wraps around (`Integer.MAX_VALUE + 1` is negative).

## Flashcards

- What is the result of `7 / 2` in Java, and why? :: `3`. Both operands are integers, so integer division discards the fraction #card
- Why is `double d = 7 / 2;` equal to `3.0`? :: The division is performed on ints first (giving 3), then the result is converted to double #card
- What does `%` do, and does it work on doubles? :: It returns the remainder of a division. Yes, it works on floating-point too (`42.25 % 10` is `2.25`) #card
- What is the sign of `-7 % 3`? :: Negative (`-1`). The result takes the sign of the left operand #card
- What do compound assignment operators do? :: Combine an operation with assignment: `a += 4` is `a = a + 4`. They exist for all binary arithmetic and bitwise operators #card
- Why does `byte b = 50; b += 2;` compile but `b = b + 2;` doesn't? :: `b + 2` is an int. Compound assignment includes an implicit cast back to the variable's type #card
- Difference between `y = ++x` and `y = x++`? :: Prefix increments first, so y gets the new value. Postfix gives y the old value, then increments x #card
- What does `int x = 5; x = x++;` leave `x` as? :: `5`. The old value is used for the assignment after the increment, overwriting it #card
- How does Java store negative integers? :: Two's complement: invert all bits, then add 1 #card
- Why two's complement instead of simple bit inversion? :: It avoids a "negative zero" and lets the same hardware add positive and negative numbers #card
- Which bit determines the sign of an integer? :: The high-order (leftmost) bit #card
- What do `&`, `\|`, `^`, `~` do bitwise? :: AND (1 if both bits 1), OR (1 if either is 1), XOR (1 if bits differ), NOT (flip every bit) #card
- What is each bitwise operator typically used for? :: `&` to clear or extract bits (mask), `\|` to set bits, `^` to flip bits #card
- What does `<<` do, and what's the effect of one left shift? :: Shifts bits left, filling with zeros; each shift doubles the value (until a 1 reaches the sign bit) #card
- What does `>>` do to the new left bits? :: Fills them with the sign bit (sign extension), so negative numbers stay negative; each shift halves the value #card
- What's the difference between `>>` and `>>>`? :: `>>` copies the sign bit into the new high bits; `>>>` always fills with zeros #card
- What is `-1 >>> 24`? :: `255` #card
- What is `-8 >> 1`? :: `-4` #card
- Why does `>>>` seem to do nothing on a negative `byte`? :: The byte is promoted to int with sign extension before the shift, and the result is cast back to a byte #card
- How do you correctly zero-fill shift a byte? :: Mask first: `(b & 0xff) >> 4` #card
- Which type do `byte << 2` and `short >> 1` produce? :: `int`, because of type promotion #card
- Which relational operators work on any type, and which only on numbers? :: `==` and `!=` work on any type; `<`, `>`, `<=`, `>=` only on numeric types (including char) #card
- Why can't you write `if (x)` for an int `x` in Java? :: Conditions must be boolean; `true`/`false` aren't related to zero/nonzero. Write `x != 0` #card
- What does `==` compare for objects? :: References (identity), not contents. Use `.equals()` for contents #card
- What is short-circuit evaluation? :: With `&&` and `\|\|`, the right operand is skipped if the left already determines the result #card
- Difference between `&` and `&&` on booleans? :: `&` always evaluates both sides; `&&` skips the right side when the left is false #card
- Give a use of `&&` as a guard. :: `denom != 0 && num / denom > 10`, or `s != null && s.length() > 0`: it prevents a crash #card
- When might you deliberately use `&` instead of `&&`? :: When the right side has a side effect that must always run, e.g. `c == 1 & e++ < 100` #card
- Is assignment an operator? What does it return? :: Yes. It returns the assigned value, which allows chaining: `x = y = z = 100` #card
- What is the ternary operator and what are its rules? :: `cond ? a : b`; the condition must be boolean, only the chosen branch is evaluated, and both results must be compatible types (not void) #card
- In what order are operators evaluated at equal precedence? :: Left to right for binary operators; right to left for assignment #card
- Rank these from highest to lowest precedence: `&&`, `==`, `+`, `<<`, `=`. :: `+`, `<<`, `==`, `&&`, `=` #card
- How does `a & b == c` parse? :: As `a & (b == c)`, because `==` binds tighter than `&` #card
- Do redundant parentheses slow a program down? :: No. They only affect parsing, and they improve readability #card

## Open questions

- [ ] How exactly do `Integer.MAX_VALUE + 1` and other overflows behave, and when should I use `Math.addExact`?
- [ ] What is the difference between `==` and `.equals()` for `String` and for boxed types like `Integer`?
- [ ] How is `instanceof` used, and how does it relate to casting between object types? (Later chapter.)
- [ ] What are some common bit-manipulation tricks (check power of two with `n & (n - 1)`, swap with XOR, set/clear/toggle the k-th bit), and how do they appear in interviews?
- [ ] Why does `i++ + ++i` give a defined result in Java but undefined behavior in C?

## Key terms

|Term|Definition|
|---|---|
|Operator|A symbol that performs an operation on one or more operands|
|Operand|A value an operator acts on|
|Unary / binary / ternary|Operator taking one / two / three operands|
|Integer division|Division of two integers; the fractional part is discarded|
|Modulus (`%`)|The remainder of a division|
|Compound assignment|An operator combining an operation with assignment (`+=`, `<<=`, ...)|
|Prefix / postfix|`++x` increments before the value is used; `x++` after|
|Bitwise operator|An operator acting on the individual bits of an integer|
|Two's complement|Encoding of negative integers: invert all bits and add 1|
|Sign bit / high-order bit|Leftmost bit of an integer; 1 means negative|
|Mask|A value used with `&` (or `\|`, `^`) to isolate, set, or flip specific bits|
|Sign extension|Copying the sign bit into the new high bits during `>>` (or when widening)|
|Unsigned right shift (`>>>`)|Right shift that always fills the high bits with zeros|
|Relational operator|Compares two values and returns a boolean (`==`, `<`, ...)|
|Boolean logical operator|Operator on boolean values (`&`, `\|`, `^`, `!`, `&&`, `\|`)|
|Short-circuit evaluation|Skipping the right operand when the left already determines the result|
|Ternary operator (`? :`)|Compact conditional that yields one of two values|
|Precedence|The rule for which operator applies first in an expression|
|Associativity|The direction (left-to-right or right-to-left) operators of equal precedence group|

## Related


→ Next: [[5 - Control Statements]]