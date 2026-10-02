---
title: CONTROL STATEMENT
date: 2026-10-01
language: Java
phase: phase-1
tags:
links: []
status: learning
---
> **One-line summary:** Control statements decide which code runs and how often. Java has selection statements (`if`, `switch`) to choose a path, iteration statements (`while`, `do-while`, `for`, for-each) to repeat code, and jump statements (`break`, `continue`, `return`) to leave or skip code. The details that matter most are `switch` fall-through, loop evaluation order, the for-each loop's read-only variable, and labeled `break`/`continue`.

## What it is

By default a program runs top to bottom, one statement after another. **Control statements** change that flow depending on the program's state. They fall into three groups:

|Group|Purpose|Statements|
|---|---|---|
|Selection|Choose between paths|`if`, `switch`|
|Iteration|Repeat code (loops)|`while`, `do-while`, `for`, for-each|
|Jump|Transfer control somewhere else|`break`, `continue`, `return`|

_(Exception handling with `try`/`catch`/`throw`/`finally` is a fourth way to change flow. It gets its own chapter later.)_

## Structure

### Selection statements

#### `if` and `else`

```java
if (condition)
    statement1;
else
    statement2;       // else is optional
```

- The **condition must be a `boolean` expression**. (No C-style "non-zero means true"; see the Operators note.)
- `statement1` / `statement2` can each be a single statement or a **block** `{ ... }`.
- **Never both branches run.** Exactly one (or neither, if there's no `else` and the condition is false).

```java
if (a < b) a = 0;
else       b = 0;           // a and b are never both set to 0
```

**Braces matter.** Only **one statement** belongs directly to an `if` or `else`. Indentation means nothing to the compiler:

```java
if (bytesAvailable > 0) {
    processData();
    bytesAvailable -= n;
} else
    waitForMoreData();
    bytesAvailable = n;      // looks like part of the else, but ALWAYS runs
```

Fix: always use braces, even for one statement. It's safer when you add lines later, and "forgetting to make a block when one is needed" is a classic bug.

#### Nested `if`s and the dangling `else`

A **nested if** is an `if` that is the target of another `if` or `else`.

**The rule for matching `else`:** an `else` belongs to the **nearest preceding `if` in the same block that doesn't already have an `else`.**

```java
if (i == 10) {
    if (j < 20) a = b;
    if (k > 100) c = d;    // this if ...
    else a = c;            // ... owns this else  (nearest if in the same block)
}
else a = d;                // this else belongs to if (i == 10)
```

The final `else` does **not** go with `if (j < 20)`. Even though that is the nearest unmatched `if`, it's inside a different block. Braces control which `if` an `else` binds to.

#### The `if-else-if` ladder

A chain of tests evaluated **top to bottom**. As soon as one condition is true, its statement runs and **the rest of the ladder is skipped**. The final `else` is the default (runs if nothing above matched). With no final `else` and no match, nothing happens.

```java
if (month == 12 || month == 1 || month == 2)      season = "Winter";
else if (month == 3 || month == 4 || month == 5)  season = "Spring";
else if (month == 6 || month == 7 || month == 8)  season = "Summer";
else if (month == 9 || month == 10 || month == 11) season = "Autumn";
else                                              season = "Bogus Month";
```

Exactly one assignment runs, whatever the value of `month`.

#### `switch`

A **multiway branch**: it compares one value against a list of constants and jumps to the match. It is often a cleaner alternative to a long if-else-if ladder.

```java
switch (expression) {
    case value1:
        // statements
        break;
    case value2:
        // statements
        break;
    default:
        // runs if no case matches
}
```

**How it works:**

1. The expression is evaluated and compared against each `case` constant.
2. Execution **starts at the matching case** and continues until a `break` (or the end of the switch).
3. If nothing matches, `default` runs. `default` is optional, and if it's missing and nothing matches, nothing happens.

**Rules for the expression and the cases:**

- The expression can be `byte`, `short`, `int`, `char`, an **enum**, or (since Java 7) a **`String`**. _(Beyond the chapter: **not** `long`, `float`, `double`, or `boolean`. A `String` switch on `null` throws `NullPointerException`.)_
- Each `case` value must be a **unique constant** (a literal or compile-time constant), and its type must be compatible with the expression. **Duplicate case values are a compile error.**
- `switch` can only test for **equality** against constants. `if` can test any boolean condition (`x > 5`, ranges, `&&`, ...).

```java
switch (i) {
    case 0: System.out.println("zero");  break;
    case 1: System.out.println("one");   break;
    default: System.out.println("more than one");
}
```

**`break` and fall-through** (a favorite interview topic): `break` is **optional**. Without it, execution **falls through** into the next case, running its code too, until a `break` or the end.

```java
switch (i) {
    case 0:
    case 1:
    case 2:
    case 3:
    case 4:
        System.out.println("i is less than 5");   // 0, 1, 2, 3, 4 all land here
        break;
    case 5: case 6: case 7: case 8: case 9:
        System.out.println("i is less than 10");
        break;
    default:
        System.out.println("i is 10 or more");
}
```

- **Intentional fall-through** is used to group cases that share code (as above, or the seasons example: `case 12: case 1: case 2: season = "Winter"; break;`).
- **Accidental fall-through** (forgetting `break`) is a very common bug: it runs the next case's code as well.
- The `default` case also needs care: if it isn't last, it needs its own `break`, or it falls through into the next case.

**String switch** (Java 7+):

```java
switch (str) {
    case "one": System.out.println("one"); break;
    case "two": System.out.println("two"); break;
    default:    System.out.println("no match");
}
```

Convenient, but **more expensive than switching on integers**. Use it when the data is already a string, don't convert to strings just to switch.

**Nested `switch`:** a `switch` can be inside another `switch`'s case. Each `switch` defines its own block, so the inner and outer can reuse the same case values (e.g. both have `case 1:`) without conflict.

**Why `switch` is usually faster than a long if-else-if chain:** the compiler knows all cases are constants of one type compared for equality, so it can build a **jump table** and go straight to the right branch. With a series of `if`s the compiler has no such knowledge, so it checks conditions one by one.

**`if` vs `switch`, summary:**

| |`if`|`switch`|
|---|---|---|
|Tests|any boolean expression|equality with constants only|
|Types|boolean condition|byte, short, int, char, enum, String|
|Many branches|slower as the chain grows|usually faster (jump table)|
|Flow|exactly one branch|falls through unless `break`|

### Iteration statements (loops)

A **loop** repeats statements until a termination condition is met. Java has `while`, `do-while`, `for`, and the for-each form of `for`.

#### `while`

```java
while (condition) {
    // body
}
```

- The condition is checked **before** each pass (**top-tested**). If it's false from the start, the body **never runs**.
- Braces are optional for a single statement, but use them.

```java
int n = 10;
while (n > 0) {
    System.out.println("tick " + n);
    n--;                  // forget this and you get an infinite loop
}
```

**Empty body:** a loop body can be empty (a lone `;` is a valid "null statement"). The whole job happens in the condition:

```java
int i = 100, j = 200;
while (++i < --j);        // find the midpoint: i ends up at 150
```

This is legal and sometimes used on purpose. But **an accidental `;` after `while(...)`, `for(...)` or `if(...)` is a bug**: the loop (or `if`) controls nothing, and the block below always runs once.

#### `do-while`

```java
do {
    // body
} while (condition);      // note the semicolon
```

- The condition is checked **after** each pass (**bottom-tested**), so the body **always runs at least once**.
- Use it when you must do something before you can test: **menu loops, input validation** ("ask, then check if valid; if not, ask again").

```java
do {
    System.out.println("tick " + n);
} while (--n > 0);        // decrement and test in one expression
```

`--n > 0` decrements first (prefix) and then tests the new value.

**`while` vs `do-while`:**

| |`while`|`do-while`|
|---|---|---|
|Condition checked|before the body|after the body|
|Minimum runs|0|1|
|Ends with `;`|no|yes|

#### `for`

```java
for (initialization; condition; iteration) {
    // body
}
```

**Order of execution** (know this precisely):

1. **Initialization** runs **once**, at the start. It usually sets a loop control variable.
2. **Condition** is evaluated. If false, the loop ends. If true, continue.
3. **Body** runs.
4. **Iteration** expression runs (usually `i++`), then go back to step 2.

```java
for (int n = 10; n > 0; n--)
    System.out.println("tick " + n);
```

**Declaring the loop variable inside the `for`:** `for (int i = ...)`. Its **scope ends when the loop ends**. After the loop, `i` doesn't exist. Do this when you only need the variable for the loop (most Java code does). Declare it outside if you need its value afterwards.

**Example (prime test) and why it works:**

```java
boolean isPrime = num >= 2;
for (int i = 2; i <= num / i; i++) {     // same as i * i <= num, without overflow
    if (num % i == 0) {
        isPrime = false;
        break;                           // found a divisor, no need to continue
    }
}
```

A number can only have a divisor larger than its square root if it also has one smaller, so you only test up to the square root.

**Using the comma:** you can run **several statements** in the initialization and iteration parts by separating them with commas:

```java
for (a = 1, b = 4; a < b; a++, b--) {
    System.out.println("a = " + a + ", b = " + b);   // two variables control the loop
}
```

In Java the comma is a **separator**, not an operator as in C/C++. It works only here (and in declarations), not in arbitrary expressions.

**`for` loop variations:** the three parts are flexible.

- The **condition can be any boolean**, it needn't test the counter: `for (int i = 1; !done; i++)`.
- **Any part can be empty.** `for ( ; !done; )` is legal (but usually poor style).
- **All three empty is an infinite loop:** `for ( ; ; ) { ... }`. This is intentional in things like command processors, and is normally exited with `break`.

#### The for-each loop (enhanced `for`, Java 5+)

Walks through every element of an array (or other collection) from first to last, with no counter and no indexing:

```java
for (type itr_var : collection) {
    // use itr_var
}
```

```java
int[] nums = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
int sum = 0;
for (int x : nums) sum += x;      // x takes each value in turn: 1, 2, 3, ...
```

Compare with the old style `for (int i = 0; i < nums.length; i++) sum += nums[i];`. The for-each version:

- needs no counter, start, end, or indexing,
- **prevents boundary errors** (no off-by-one, no `ArrayIndexOutOfBounds`),
- is clearer when you just need every element.

Rules and limits to know:

- **`type` must be compatible with the element type** of the array/collection.
- **Strictly sequential, start to finish, forward only.** You can still exit early with `break`.
- **The iteration variable is read-only with respect to the array.** Assigning to it changes only the loop's local copy, **not** the array:

```java
for (int x : nums) {
    x = x * 10;          // changes the copy only; nums is unchanged
}
```

- **No index available.** If you need the index, or need to modify elements, or iterate backwards or skip elements, use the classic `for`.
- It works with arrays and with collections (collections come in a later part).

**Multidimensional arrays:** a 2D array is an **array of arrays**, so each iteration of the outer loop gives you a whole **row** (a 1D array), not a single element. For an N-dimensional array, each outer iteration gives an (N-1)-dimensional array.

```java
int[][] nums = new int[3][5];
for (int[] row : nums)          // row is a 1D int array (a reference to one row)
    for (int y : row)           // y is a single int
        sum += y;
```

**Good uses:** anything that must look at every element once: **searching** an unsorted array (add `break` when found), sum, average, min/max, finding duplicates. (For a sorted array, binary search is better, and that needs an indexed loop.)

**Which loop to use:**

|Situation|Best choice|
|---|---|
|Known number of iterations, counter needed|`for`|
|Visit every element, no index needed|for-each|
|Repeat until some condition, may run 0 times|`while`|
|Must run at least once (menu, input check)|`do-while`|

#### Nested loops

A loop inside a loop. The inner loop runs fully for each pass of the outer loop. Total body runs = (outer count) × (inner count) (for the usual rectangular case).

```java
for (int i = 0; i < 10; i++) {
    for (int j = i; j < 10; j++)
        System.out.print(".");
    System.out.println();          // prints a shrinking triangle of dots
}
```

### Jump statements

`break`, `continue`, and `return` transfer control to another place. (Java has no `goto`, although it is a reserved word. `break` and `continue` with labels cover its few legitimate uses.)

#### `break`: three uses

1. **End a `switch` case** (covered above).
2. **Exit a loop immediately.** Skips the rest of the body and the condition; control resumes at the first statement after the loop.
3. **Labeled `break`**: a "civilized goto".

```java
for (int i = 0; i < 100; i++) {
    if (i == 10) break;            // leave the loop when i is 10
    System.out.println("i: " + i); // prints 0 through 9
}
System.out.println("Loop complete.");
```

- Works in `for`, `while`, `do-while`, and for-each.
- **In nested loops, a plain `break` exits only the innermost loop.** The outer loop keeps going. Likewise, a `break` in a `switch` exits only the `switch`, not an enclosing loop.
- Use `break` for special situations ("found it", "error"). The loop's own condition should be the normal way to end it. Many `break`s scattered around make code hard to follow.

**Labeled `break`:** `break label;` jumps out of the **named, enclosing** block or loop, to the statement right after it.

```java
outer: for (int i = 0; i < 3; i++) {          // label the outer loop
    System.out.print("Pass " + i + ": ");
    for (int j = 0; j < 100; j++) {
        if (j == 10) break outer;             // exit BOTH loops
        System.out.print(j + " ");
    }
    System.out.println("This will not print");
}
System.out.println("Loops complete.");
// Output: Pass 0: 0 1 2 3 4 5 6 7 8 9 Loops complete.
```

- A **label** is an identifier followed by a colon, placed before a loop or block.
- It works on **any block**, not only loops. `break` jumps to the **end** of that labeled block.
- **The label must belong to a block that encloses the `break` statement.** You can't jump to an arbitrary label elsewhere (this is what makes it "civilized"). Breaking to a label on a loop that has already ended or doesn't contain the `break` is a **compile error**.
- Classic use: escaping **deeply nested loops** in one step.

#### `continue`

Skips the **rest of the current iteration** and moves on to the next one. The loop itself keeps running.

```java
for (int i = 0; i < 10; i++) {
    System.out.print(i + " ");
    if (i % 2 == 0) continue;     // even: skip the newline below
    System.out.println();         // prints a newline only after odd numbers
}
```

Where control goes after `continue`:

|Loop|`continue` jumps to|
|---|---|
|`while`, `do-while`|the **condition** test|
|`for`|the **iteration** part (`i++`), **then** the condition|

**Labeled `continue`:** `continue outer;` skips the rest of the inner loop **and** continues with the next iteration of the labeled outer loop.

```java
outer: for (int i = 0; i < 10; i++) {
    for (int j = 0; j < 10; j++) {
        if (j > i) {
            System.out.println();
            continue outer;       // abandon inner loop, next i
        }
        System.out.print(" " + (i * j));
    }
}
// prints a triangular multiplication table
```

`continue` is used rarely. Java's loops fit most needs. Use it when you need to skip an iteration cleanly.

**`break` vs `continue`:** `break` **ends** the loop. `continue` **skips to the next pass**.

#### `return`

Exits the **current method** immediately and goes back to the caller. (It can also pass a value back, covered with methods in a later chapter.) In `main()`, `return` ends the program, since the JVM called `main`.

```java
System.out.println("Before the return.");
if (t) return;                           // leaves main() here
System.out.println("This won't execute.");
```

**Unreachable code:** the compiler gives an error if code can never run, e.g. a statement right after an unconditional `return`, `break`, or `continue`. That's why the book wraps the `return` in `if (t)`: the compiler can't be sure it always runs, so the code after it isn't flagged.

## Mechanics / reference

### Quick execution-order cheat sheet

```
while:      test → body → test → body → ... (0 or more times)
do-while:   body → test → body → test → ...  (1 or more times)
for:        init (once) → test → body → update → test → body → update ...
for-each:   next element → body → next element ... (until none left)
```

### What each jump does to flow

|Statement|Inside a loop|Inside a `switch`|In a method|
|---|---|---|---|
|`break`|exits the innermost loop|exits the `switch`|n/a|
|`break label`|exits the labeled block or loop|exits the labeled block|n/a|
|`continue`|next iteration of the innermost loop|n/a (needs a loop)|n/a|
|`continue label`|next iteration of the labeled loop|n/a|n/a|
|`return`|leaves the whole method|leaves the whole method|leaves the method|

### Common patterns

```java
// 1. Linear search with early exit
boolean found = false;
for (int x : arr) { if (x == target) { found = true; break; } }

// 2. Input validation (must ask at least once)
do { /* read input */ } while (!valid);

// 3. Escape nested loops
search:
for (int[] row : grid)
    for (int v : row)
        if (v == target) break search;

// 4. Infinite loop with a controlled exit
for (;;) { if (done()) break; }
```

## Pitfalls

- **Missing braces on `if`/`else`/loops.** Only the next single statement is controlled. The second line "indented under" the `if` always runs.
- **A stray `;` after `if (...)`, `for (...)`, or `while (...)`.** It becomes an empty body, and the block after it runs unconditionally.
- **Dangling `else`.** It binds to the nearest unmatched `if` **in the same block**, not by indentation.
- **`=` instead of `==` in a condition.** A compile error for ints, but `if (flag = true)` compiles for a `boolean`.
- **Forgetting `break` in a `switch`.** Execution falls through into the next case. Comment intentional fall-through.
- **Duplicate `case` values** (compile error), or a non-constant `case` value.
- **Switching on an unsupported type** (`long`, `double`, `boolean`) or on a `null` string (runtime `NullPointerException`).
- **Off-by-one errors:** `i <= arr.length` instead of `i < arr.length` throws `ArrayIndexOutOfBoundsException`. Remember arrays start at index 0.
- **Infinite loops:** forgetting the update (`n--`, `i++`), or a condition that can never become false.
- **`continue` skipping the update in a `while`/`do-while`.** If the increment is after the `continue`, the loop can spin forever. (In a `for`, the update still runs.)
- **Modifying the for-each variable** thinking it changes the array. It doesn't. Use an indexed `for` to modify elements.
- **Changing a collection while for-each iterates over it** throws `ConcurrentModificationException`. _(Beyond the chapter.)_
- **Using the loop variable after the loop** when it was declared inside the `for`. It's out of scope.
- **Plain `break` in nested loops** only exits the inner one. Use a labeled `break` to leave both.
- **Labeled `break` to a label that doesn't enclose it.** Compile error.
- **`do-while` without the final semicolon** (`} while (cond);`).
- **Using `float`/`double` as a loop counter.** Rounding errors can skip or repeat iterations. Prefer ints. _(Beyond the chapter.)_
- **Code after `return`/`break`/`continue`** is "unreachable" and won't compile.

## Flashcards

- What are the three categories of control statements in Java? :: Selection (`if`, `switch`), iteration (`while`, `do-while`, `for`, for-each), and jump (`break`, `continue`, `return`) #card
- What must the condition of an `if` be? :: A `boolean` expression (not an int) #card
- Why should you always use braces with `if`/`else`? :: Only one statement belongs directly to each branch; indentation is ignored by the compiler, so extra lines silently run unconditionally #card
- Which `if` does an `else` belong to? :: The nearest preceding `if` in the same block that doesn't already have an `else` #card
- How does an if-else-if ladder execute? :: Top to bottom; the first true condition's statement runs and the rest are skipped; the final `else` is the default #card
- Which types can a `switch` expression have? :: byte, short, int, char, enum, and String (since Java 7). Not long, float, double, or boolean #card
- What does `switch` compare against? :: Unique constant values, for equality only. Duplicate case constants are an error #card
- What is fall-through in a `switch`? :: Without `break`, execution continues into the next case's statements until a `break` or the end of the switch #card
- When is fall-through useful? :: To let several cases share the same code (stacked case labels with no statements between them) #card
- What happens if no `case` matches and there's no `default`? :: Nothing; execution continues after the switch #card
- Why is `switch` often faster than a long if-else-if chain? :: The compiler can build a jump table since all cases are constants of one type compared for equality #card
- Can two `switch` statements, one nested in the other, use the same case values? :: Yes, each switch is its own block, so there's no conflict #card
- What's the difference between `while` and `do-while`? :: `while` tests before the body (may run 0 times); `do-while` tests after (runs at least once) #card
- When would you choose `do-while`? :: When the body must run at least once, e.g. a menu or input validation #card
- In what order do the parts of a `for` loop execute? :: Initialization once, then condition, body, iteration, condition, body, iteration, ... until the condition is false #card
- What is the scope of a variable declared in the `for` initialization? :: Only the `for` statement (including its body); it doesn't exist after the loop #card
- What does `for (;;)` do? :: An infinite loop (all three parts empty); exit with `break` or `return` #card
- Is the comma an operator in Java's `for` loop? :: No; it's a separator that lets you put several statements in the initialization and iteration parts #card
- What does the for-each loop do? :: Iterates over every element of an array or collection from first to last: `for (type x : collection)` #card
- Can you modify an array through the for-each variable? :: No. The variable is a copy; assigning to it doesn't change the array. Use an indexed `for` #card
- When can't you use for-each? :: When you need the index, need to iterate backwards or skip elements, or need to modify array elements #card
- What does for-each give you when looping over a 2D array? :: One row (a 1D array) per iteration, so the loop variable must be an array type #card
- What are the three uses of `break`? :: Ending a switch case, exiting a loop, and a labeled break (a "civilized goto") #card
- Which loop does a plain `break` exit when loops are nested? :: Only the innermost loop #card
- How do you exit multiple nested loops at once? :: Label the outer loop and use `break label;` #card
- Can a labeled `break` jump to any label? :: No; the label must be on a block/loop that encloses the `break` statement #card
- What does `continue` do? :: Skips the rest of the current iteration and proceeds to the next one #card
- Where does `continue` jump in a `for` loop vs a `while` loop? :: In `for`, to the iteration part then the condition; in `while`/`do-while`, straight to the condition #card
- What does `return` do? :: Immediately exits the current method and returns control to the caller (in `main`, ends the program) #card
- What causes an "unreachable code" error? :: A statement that can never run, such as code placed right after an unconditional `return`, `break`, or `continue` #card
- Does Java have `goto`? :: No. It's a reserved word with no function; labeled `break`/`continue` cover its legitimate uses #card

## Open questions

- [ ] What are the new arrow-style `switch` expressions (Java 14+), and how do they remove fall-through?
- [ ] How does a `String` switch work internally (hash code plus `equals`), and why is it costlier than an int switch?
- [ ] How does the enhanced `for` work on collections (`Iterable` and `Iterator`), and why does modifying a list during iteration fail?
- [ ] When should I prefer `break`/`continue` and when should I restructure the loop condition instead?
- [ ] What is the time complexity of nested loops, and how do I recognize O(n), O(n²), and O(log n) loops?

## Key terms

|Term|Definition|
|---|---|
|Control statement|A statement that changes the normal top-to-bottom flow of execution|
|Selection statement|Chooses between code paths (`if`, `switch`)|
|Iteration statement|Repeats code (`while`, `do-while`, `for`, for-each)|
|Jump statement|Transfers control elsewhere (`break`, `continue`, `return`)|
|Block|Statements grouped in `{ }` and treated as one unit|
|Nested `if`|An `if` that is the target of another `if` or `else`|
|if-else-if ladder|A chain of `if`/`else if` tests evaluated top to bottom|
|`switch`|Multiway branch comparing an expression with constant case values|
|Fall-through|Continuing into the next `case` because there was no `break`|
|Jump table|Compiler-built table that lets `switch` go straight to the matching case|
|Loop|Code that repeats until a termination condition is met|
|Loop control variable|The variable (often a counter) that governs a loop|
|Null statement|A lone `;`, which is a valid statement that does nothing|
|Infinite loop|A loop that never terminates (e.g. `for (;;)`)|
|For-each (enhanced `for`)|Loop that visits each element of an array or collection in order|
|Iteration variable|The variable in a for-each loop that receives each element|
|Label|An identifier plus colon naming a block or loop, used by `break`/`continue`|
|Labeled `break`|Exits a named enclosing block or loop|
|`continue`|Skips the rest of the current iteration|
|`return`|Exits the current method and returns to its caller|
|Unreachable code|Code that can never execute; a compile-time error|

## Related


→ Next: [[INTRODUCING CLASSES]]