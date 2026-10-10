---
title: CLASSES AND INTERFACES FOR ARRAYS
date: 2026-10-10
language: Java
phase: phase-1
tags:
links: []
status: learning
---
> **One-line summary:** Wrapping the array in a **class** hides it behind **methods**. A first attempt (`LowArray` with `setElem()` / `getElem()`) only mimics `[]` and forces the caller to track indices. A better **class interface** (`HighArray` with `insert()`, `find()`, `delete()`, `display()`) moves that bookkeeping inside the class, so the user focuses on **what** to do, not **how**. That separation is **abstraction**. The same pattern then stores **objects** (`Person`) instead of `long`s: compare keys with `equals()`, not `==`.

## What it is

Part 1 had one big `main()`. This note makes the program **object oriented** in two steps:

1. **Separate the storage structure (the array) from the code that uses it.** The user becomes a _user_ of a tool.
2. **Improve the communication between them** by designing a good **class interface**.

| Topic                           | Question it answers                                      |
| ------------------------------- | -------------------------------------------------------- |
| Dividing a program into classes | Why split storage from the code that uses it?            |
| `LowArray`                      | What does a first, not-so-useful split look like?        |
| Class interface                 | How does a class's public methods define how it is used? |
| `HighArray`                     | How do I design an interface that is convenient?         |
| Abstraction                     | Why is "what, not how" valuable?                         |
| Storing objects (`Person`)      | How do I store real records instead of numbers?          |

## Structure

### 1. Dividing a program into classes

`array.java` is essentially one big method. Splitting it into classes makes it easier to design, understand and (in real programs) modify and maintain.

Which classes? Two natural candidates:

- The **data storage structure** itself (the array and its operations).
- The **part of the program that uses** that structure.

Instead of treating the array as a bare language feature, we **encapsulate** it in a class and give other classes **methods** to reach it. Those methods are how the two classes communicate.

### 2. First attempt: `LowArray` and `LowArrayApp`

```java
class LowArray {
    // ref to array a (hidden: private)
    private long[] a;

    public LowArray(int size) {                // constructor
        a = new long[size];                    // create array
    }
    public void setElem(int index, long value) {   // set value
        a[index] = value;
    }
    public long getElem(int index) {           // get value
        return a[index];
    }
}
```

```java
class LowArrayApp {
    public static void main(String[] args) {
        LowArray arr;                          // reference
        // create LowArray object
        arr = new LowArray(100);
        // number of items in array (the USER tracks this)
        int nElems = 0;
        int j;                                 // loop variable

        // insert 10 items
        arr.setElem(0, 77);  arr.setElem(1, 99);
        arr.setElem(2, 44);  arr.setElem(3, 55);
        arr.setElem(4, 22);  arr.setElem(5, 88);
        arr.setElem(6, 11);  arr.setElem(7, 00);
        arr.setElem(8, 66);  arr.setElem(9, 33);
        // now 10 items in array
        nElems = 10;

        for (j = 0; j < nElems; j++)           // display items
            System.out.print(arr.getElem(j) + " ");
        System.out.println("");

        // search for data item
        int searchKey = 26;
        // for each element,
        for (j = 0; j < nElems; j++)
            if (arr.getElem(j) == searchKey)   //   found item?
                break;
        if (j == nElems)                       // no
            System.out.println("Can't find " + searchKey);
        else                                   // yes
            System.out.println("Found " + searchKey);

        // delete value 55
        for (j = 0; j < nElems; j++)           // look for it
            if (arr.getElem(j) == 55)
                break;
        // higher ones down
        for (int k = j; k < nElems; k++)
            arr.setElem(k, arr.getElem(k + 1));
        nElems--;                              // decrement size

        for (j = 0; j < nElems; j++)           // display items
            System.out.print(arr.getElem(j) + " ");
        System.out.println("");
    }
}
```

Output (searches for the missing key 26 first, then deletes 55):

```
77 99 44 55 22 88 11 0 66 33
Can't find 26
77 99 44 22 88 11 0 66 33
```

**What this version does:**

- The array is **private**, so only `LowArray` methods can touch it.
- `LowArray` has three members: a constructor (creates an empty array of a given size), `setElem()` (store) and `getElem()` (retrieve).
- `LowArrayApp` is the **user** of the tool. Two classes with clearly defined roles is a valuable first step toward OOP.
- A class whose job is to store data objects, like `LowArray`, is called a **container class**. Typically a container class also provides methods to access the data, and perhaps to sort it or perform other complex actions on it.

### 3. Class interfaces

How do classes interact? **Communication between classes and division of responsibility** are central to OOP. This matters most when a class has **many users**. For example, `LowArray` could just as well store traveler's-check serial numbers as baseball players' numbers.

> The way a class user relates to the class is called the class **interface**.

Fields are normally `private`, so "the interface" mostly means the **public methods**: what they do and what arguments they take. By calling them, a user interacts with an object of the class. A major OOP advantage is that the interface can be designed to be **as convenient and efficient as possible**.

```
+-- LowArray class -----------------+
|  private data: long[] a   (hidden)|
|    +---------------+              |
|    |   INTERFACE   |              |
|    |   setElem()   |              |
|    |   getElem()   |              |
|    +---------------+              |
+-----------------------------------+
```

### 4. Not so convenient: why `LowArray` falls short

- `setElem()` and `getElem()` work at a **low conceptual level**. They do exactly what the `[]` operator does. So the user still does the same low-level work as in `array.java` (the only difference is calling methods instead of using `[]`). It is not clear this is an improvement.
- The **user** (`main()`) must still keep track of **indices** and of **`nElems`**.
- There is no convenient way to **display** the array. `main()` uses a crude `for` loop. You could add a display method to `LowArrayApp`, but is it really that class's responsibility?

So `lowArray.java` shows _how_ to divide a program into classes, but doesn't buy much. The trick is redistributing **responsibilities** to get the real benefits of OOP.

#### Who's responsible for what?

In `LowArrayApp`, `main()` (the user) must manage indices. For some users that is fine (e.g., **sorting**, covered in the next chapter, benefits from direct, hands-on index access). But in a typical program, the user of a storage structure finds array indices neither helpful nor relevant.

### 5. `HighArray`: a better interface

Now the user no longer thinks about indices. `setElem()` / `getElem()` are **gone**, replaced by `insert()`, `find()`, `delete()` (plus `display()`). They need **no index argument** because the **class takes responsibility for index handling**, including tracking `nElems`.

```java
class HighArray {
    private long[] a;                          // ref to array a
    // number of data items
    private int nElems;

    public HighArray(int max) {                // constructor
        // create the array
        a = new long[max];
        nElems = 0;                            // no items yet
    }

    // find specified value
    public boolean find(long searchKey) {
        int j;
        // for each element,
        for (j = 0; j < nElems; j++)
            if (a[j] == searchKey)             //   found item?
                //   exit loop before end
                break;
        if (j == nElems)                       // gone to end?
            // yes, can't find it
            return false;
        else
            return true;                       // no, found it
    }

    // put element into array
    public void insert(long value) {
        a[nElems] = value;                     // insert it
        nElems++;                              // increment size
    }

    public boolean delete(long value) {
        int j;
        for (j = 0; j < nElems; j++)           // look for it
            if (value == a[j])
                break;
        if (j == nElems)                       // can't find it
            return false;
        else {                                 // found it
            // move higher ones down
            for (int k = j; k < nElems; k++)
                a[k] = a[k + 1];
            nElems--;                          // decrement size
            return true;
        }
    }

    // displays array contents
    public void display() {
        // for each element,
        for (int j = 0; j < nElems; j++)
            System.out.print(a[j] + " ");      //   display it
        System.out.println("");
    }
}
```

```java
class HighArrayApp {
    public static void main(String[] args) {
        int maxSize = 100;                     // array size
        // reference to array
        HighArray arr;
        // create the array
        arr = new HighArray(maxSize);

        // insert 10 items
        arr.insert(77);  arr.insert(99);  arr.insert(44);
        arr.insert(55);  arr.insert(22);
        arr.insert(88);  arr.insert(11);  arr.insert(00);
        arr.insert(66);  arr.insert(33);

        arr.display();                         // display items

        // search for item
        int searchKey = 35;
        if (arr.find(searchKey))
            System.out.println("Found " + searchKey);
        else
            System.out.println("Can't find " + searchKey);

        arr.delete(00);                        // delete 3 items
        arr.delete(55);
        arr.delete(99);

        // display items again
        arr.display();
    }
}
```

Output:

```
77 99 44 55 22 88 11 0 66 33
Can't find 35
77 44 22 88 11 66 33
```

**What each method does:**

|Method|Job|Returns|
|---|---|---|
|`HighArray(int max)`|Creates the array, sets `nElems = 0`|(constructor)|
|`find(long key)`|Linear search for `key`|`true` / `false`|
|`insert(long v)`|Places `v` at `a[nElems]`, increments `nElems`|nothing|
|`delete(long v)`|Finds `v`, shifts higher items down, decrements `nElems`|`true` if it was deleted, `false` if not found|
|`display()`|Prints all stored values|nothing|

Notice how short `main()` is now. The details that `main()` had to handle in `LowArrayApp` are handled by `HighArray`. Deleting is so easy that the example deletes **three** items instead of one.

### 6. The user's life made easier

- Searching took **eight lines** in `lowArray.java`'s `main()`. In `highArray.java` it takes **one** (`arr.find(searchKey)`).
- The user (`HighArrayApp`) doesn't need to know about indices or any other array detail. It doesn't even need to know **what kind of data structure** `HighArray` uses to store the data. The structure is **hidden behind the interface**. (In the next part, the _same_ interface is used with a somewhat different structure.)

### 7. Abstraction

> **Abstraction:** separating **how** an operation is done inside a class from **what** is visible to the class user.

It is an important aspect of software engineering. By abstracting class functionality, you can design a program without thinking about implementation details too early. The user says _what_ to insert, delete and find, not _how_.

**Practical payoff:** you can change the internals later (a different data structure, a faster search) without touching the code that uses the class, as long as the interface stays the same.

### 8. Storing objects: the `Person` class

So far items were `long`s. Real records have **several fields** (a personnel record: last name, first name, age, ...). In Java, a data record is usually a **class object**.

```java
class Person {
    private String lastName;
    private String firstName;
    private int age;

    // constructor
    public Person(String last, String first, int a) {
        lastName = last;
        firstName = first;
        age = a;
    }
    public void displayPerson() {
        System.out.print("   Last name: " + lastName);
        System.out.print(", First name: " + firstName);
        System.out.println(", Age: " + age);
    }
    public String getLast() {                  // get last name
        // this is the KEY field used for searches
        return lastName;
    }
}
```

#### `ClassDataArray`: `HighArray` adapted for `Person`

Only a few things change from `HighArray`:

1. The array's type becomes **`Person[]`** (not `long[]`).
2. The key (last name) is a **`String` object**, so comparisons use **`equals()`**, **not `==`** (`==` would compare references).
3. `insert()` **creates a new `Person`** and stores it, instead of storing a `long`.
4. `find()` returns the **`Person`** found (or `null`), not just `true`/`false`.

```java
class ClassDataArray {
    // reference to array
    private Person[] a;
    // number of data items
    private int nElems;

    public ClassDataArray(int max) {           // constructor
        // create the array (all elements null)
        a = new Person[max];
        nElems = 0;                            // no items yet
    }

    // find specified value
    public Person find(String searchName) {
        int j;
        // for each element,
        for (j = 0; j < nElems; j++)
            // found item?
            if (a[j].getLast().equals(searchName))
                // exit loop before end
                break;
        if (j == nElems)                       // gone to end?
            // yes, can't find it
            return null;
        else
            return a[j];                       // no, found it
    }

    // put person into array
    public void insert(String last, String first, int age) {
        a[nElems] = new Person(last, first, age);
        nElems++;                              // increment size
    }

    public boolean delete(String searchName) {   // delete person
        int j;
        for (j = 0; j < nElems; j++)           // look for it
            if (a[j].getLast().equals(searchName))
                break;
        if (j == nElems)                       // can't find it
            return false;
        else {                                 // found it
            for (int k = j; k < nElems; k++)   // shift down
                a[k] = a[k + 1];
            nElems--;                          // decrement size
            return true;
        }
    }

    // displays array contents
    public void displayA() {
        // for each element,
        for (int j = 0; j < nElems; j++)
            a[j].displayPerson();              // display it
    }
}
```

```java
class ClassDataApp {
    public static void main(String[] args) {
        int maxSize = 100;                     // array size
        // reference to array
        ClassDataArray arr;
        // create the array
        arr = new ClassDataArray(maxSize);

        // insert 10 items
        arr.insert("Evans", "Patty", 24);
        arr.insert("Smith", "Lorraine", 37);
        arr.insert("Yee", "Tom", 43);
        arr.insert("Adams", "Henry", 63);
        arr.insert("Hashimoto", "Sato", 21);
        arr.insert("Stimson", "Henry", 29);
        arr.insert("Velasquez", "Jose", 72);
        arr.insert("Lamarque", "Henry", 54);
        arr.insert("Vang", "Minh", 22);
        arr.insert("Creswell", "Lucinda", 18);

        arr.displayA();                        // display items

        // search for item
        String searchKey = "Stimson";
        Person found;
        found = arr.find(searchKey);
        if (found != null) {
            System.out.print("Found ");
            found.displayPerson();
        } else
            System.out.println("Can't find " + searchKey);

        System.out.println("Deleting Smith, Yee, and Creswell");
        arr.delete("Smith");                   // delete 3 items
        arr.delete("Yee");
        arr.delete("Creswell");

        // display items again
        arr.displayA();
    }
}
```

Output:

```
   Last name: Evans, First name: Patty, Age: 24
   Last name: Smith, First name: Lorraine, Age: 37
   Last name: Yee, First name: Tom, Age: 43
   Last name: Adams, First name: Henry, Age: 63
   Last name: Hashimoto, First name: Sato, Age: 21
   Last name: Stimson, First name: Henry, Age: 29
   Last name: Velasquez, First name: Jose, Age: 72
   Last name: Lamarque, First name: Henry, Age: 54
   Last name: Vang, First name: Minh, Age: 22
   Last name: Creswell, First name: Lucinda, Age: 18
Found    Last name: Stimson, First name: Henry, Age: 29
Deleting Smith, Yee, and Creswell
   Last name: Evans, First name: Patty, Age: 24
   Last name: Adams, First name: Henry, Age: 63
   Last name: Hashimoto, First name: Sato, Age: 21
   Last name: Stimson, First name: Henry, Age: 29
   Last name: Velasquez, First name: Jose, Age: 72
   Last name: Lamarque, First name: Henry, Age: 54
   Last name: Vang, First name: Minh, Age: 22
```

**Takeaway:** class objects are handled by data storage structures in much the same way as primitive types. (A serious program that used the last name as the key would need to handle **duplicate last names**, which complicates the code as discussed in part 1. Note that "Henry" appears three times as a first name.)

## Mechanics / reference

### `LowArray` vs. `HighArray`

| |`LowArray`|`HighArray`|
|---|---|---|
|Interface level|Low: mirrors `[]`|High: insert / find / delete|
|Who tracks indices?|The **user**|The **class**|
|Who tracks `nElems`?|The **user**|The **class** (a `private` field)|
|Display|User writes the loop|`display()` provided|
|Search in user code|~8 lines|1 line|
|User needs to know|It's an array with indices|Nothing about the internal structure|

### Return-value conventions used

|Method|Returns|Meaning|
|---|---|---|
|`HighArray.find`|`boolean`|Was it found?|
|`HighArray.delete`|`boolean`|Was something deleted?|
|`ClassDataArray.find`|`Person`|The object found, or `null`|
|`ClassDataArray.delete`|`boolean`|Was something deleted?|

### What changed going from `long` to `Person`

|Aspect|`HighArray` (`long`)|`ClassDataArray` (`Person`)|
|---|---|---|
|Array type|`long[]`|`Person[]`|
|Key comparison|`a[j] == searchKey`|`a[j].getLast().equals(searchName)`|
|`insert` argument|the value|the fields (`last, first, age`); it builds the object|
|`find` returns|`boolean`|`Person` (or `null`)|
|Elements before insertion|`0`|`null`|

### Common patterns

```java
// 1. Container skeleton: private data + count, public ops
class Container {
    private long[] a;
    private int nElems;
    public Container(int max) { a = new long[max]; nElems = 0; }
    public void insert(long v)   { a[nElems++] = v; }
    public boolean find(long k) {
        for (int j = 0; j < nElems; j++)
            if (a[j] == k) return true;
        return false;
    }
    // delete(...), display() ...
}

// 2. Search objects by a String key: ALWAYS equals(), never ==
if (a[j].getLast().equals(searchName)) { ... }

// 3. find() returning an object: check for null first
Person p = arr.find("Stimson");
if (p != null) p.displayPerson();
```

_(Beyond the chapter: this note's listing uses the same shifting loop style as part 1. `HighArray.insert` doesn't check for a full array, and the shifting loop `k < nElems` reads `a[nElems]`. That is harmless while there is spare capacity, but would go out of bounds if the array is completely full.)_

## Pitfalls

- **Leaking index handling to the user.** If every caller must track indices and `nElems`, the class isn't helping (`LowArray`).
- **Making the array `public`.** Defeats the point of encapsulation. Keep the data `private` and expose methods.
- **Comparing `String` keys with `==`.** It compares references, not contents. Use `equals()`.
- **Using the result of `find()` without a `null` check.** `ClassDataArray.find` returns `null` when nothing matches. Calling `displayPerson()` on `null` crashes.
- **Forgetting the array of objects starts full of `null`s.** You must create each `Person` (as `insert()` does).
- **Not keeping `nElems` in sync.** `insert` must increment and `delete` must decrement it, or later loops visit garbage or miss items.
- **Ignoring duplicate keys.** Last names repeat. `find()` and `delete()` act only on the **first** match.
- **No overflow guard.** `insert` on a full array throws an out-of-bounds error. _(Beyond the chapter.)_
- **Reading `delete()`'s boolean as optional.** Callers that ignore it won't know the key wasn't found.
- **Designing the interface around the implementation.** The interface should describe _what_ the user wants (insert, find, delete), not _how_ the data is stored (set element at index).

## Flashcards

- Why divide a program into classes? :: It clarifies functionality, making the program easier to design, understand, modify and maintain #card
- What is a container class? :: A class used to store data objects; it usually also provides methods to access, sort or otherwise process them #card
- What is a class interface? :: The way a class user relates to the class, mainly the class's public methods (what they do and their arguments) #card
- Why are class fields usually private? :: So users can only reach the data through the class's methods (encapsulation) #card
- What are the three members of `LowArray`? :: A constructor, `setElem()` and `getElem()` #card
- Why is the `LowArray` interface "not so convenient"? :: `setElem`/`getElem` operate at the same low level as `[]`, so the user still manages indices and the item count #card
- In `LowArrayApp`, who keeps track of the number of items? :: The user (`main()`), through its own `nElems` variable #card
- What replaces `setElem()`/`getElem()` in `HighArray`? :: `insert()`, `find()` and `delete()`, which need no index argument #card
- Who handles index numbers in `HighArray`? :: The class itself, inside its methods (it keeps the `nElems` field) #card
- What does `HighArray.find()` return? :: A `boolean`: `true` if the key was found, otherwise `false` #card
- What does `HighArray.delete()` return? :: `true` if the item was found and deleted, `false` if it wasn't found #card
- How does `insert()` in `HighArray` work? :: Puts the value at `a[nElems]` and increments `nElems` #card
- How many lines does a search need in `LowArrayApp` vs. `HighArrayApp`? :: About eight vs. one #card
- What is abstraction? :: Separating how an operation is performed inside a class from what is visible to the class user #card
- Why is abstraction useful? :: You can design without worrying about implementation details too early, and change internals without affecting the users #card
- Why does the user of `HighArray` not need to know what data structure is inside? :: The structure is hidden behind the interface #card
- When is direct index access still useful to a user? :: For tasks like sorting, which make efficient use of hands-on array access #card
- How are data records usually represented in Java? :: As class objects (e.g., `Person`) #card
- Which `Person` method returns the key field? :: `getLast()`, which returns the last name #card
- What four changes adapt `HighArray` to `Person` objects? :: Array type becomes `Person[]`; keys compared with `equals()`; `insert()` creates a `Person`; `find()` returns the `Person` (or null) #card
- Why use `equals()` instead of `==` for the last name? :: The key is a `String` object; `==` compares references, while `equals()` compares contents #card
- What does `ClassDataArray.find()` return when the name isn't present? :: `null` #card
- What does a new `Person[100]` contain? :: 100 references, all `null`; no `Person` objects exist yet #card
- What is a limitation of using last name as the key? :: Duplicates are possible, which complicates searching and deleting #card

## Open questions

- [ ] How would `insert()` reject or handle a full array?
- [ ] How could `find()` return all matches when last names repeat?
- [ ] How does a class's interface relate to the Java `interface` keyword? (Different idea: "interface" here means a class's public methods.)
- [ ] Could `delete()` reuse `find()` to avoid duplicating the search loop?
- [ ] Which other structures can sit behind this same interface (insert / find / delete)?

## Key terms

|Term|Definition|
|---|---|
|Encapsulation|Hiding data inside a class and exposing it only through methods|
|Container class|A class whose purpose is to store data items and provide access to them|
|Class interface|The public methods through which a user interacts with a class|
|Class user|Code (another class) that creates and uses objects of a class|
|`LowArray`|First array class: low-level `setElem()` / `getElem()`|
|`HighArray`|Improved array class: `insert()` / `find()` / `delete()` / `display()`|
|Abstraction|Separating _what_ a class does from _how_ it does it|
|`Person`|Example record class (last name, first name, age)|
|Key field|The field used for searching (here `lastName`)|
|`equals()`|Method comparing the contents of two objects (e.g., Strings)|
|`ClassDataArray`|`HighArray`-style container for `Person` objects|

## Related

→ Next: [[ORDERED ARRAYS AND BINARY SEARCH]]