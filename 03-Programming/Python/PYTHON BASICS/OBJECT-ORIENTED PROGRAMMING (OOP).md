---
title: OBJECT ORIENTED PROGRAMMING
date: 2026-05-19
phase: phase-1
tags:
  - concept
  - networking
links: []
status: learning
---
# OBJECT-ORIENTED PROGRAMMING (OOP)

## One-Line Summary

**OOP** is a way to _bundle data (attributes) and behavior (methods) together_ into a **class** — a reusable template — and then create as many **objects** (instances) from it as you need, each carrying its own independent copy of the data.

---

## PART 1 — CORE CONCEPTS

### Why OOP Exists

Before OOP, large programs became unmanageable walls of code. As programs grow to millions of lines, you can never hold the whole thing in your head. OOP solves this by letting you:

- **Zoom in** on 50 relevant lines while ignoring the other 999,950.
- **Subdivide** a problem into zones, where each zone (object) handles its own data and behaviour.
- **Reuse** — write `Dog` once, create a thousand dog objects.

Think of it this way: a program without OOP is like one enormous kitchen where everything is spread out on one table. OOP gives every chef their own station with their own tools and ingredients.

> **Java comparison:** OOP in Python feels similar to Java — classes, objects, `this` (called `self` in Python), constructors, inheritance — but Python's syntax is far less ceremonial and doesn't require access modifiers (`public`, `private`, etc.).

---

### The 4 Key Terms You Must Know

|Term|Plain-English Meaning|Analogy|
|---|---|---|
|**Class**|The blueprint / template|Cookie cutter|
|**Object / Instance**|One thing _made from_ the blueprint|An actual cookie|
|**Attribute**|A variable that _belongs to_ an object|The frosting colour on _this_ cookie|
|**Method**|A function that _belongs to_ a class|"Apply frosting" — the action|

---

## PART 2 — CREATING CLASSES

### Basic Class Structure

```python
class Dog():                          # Python — class names use CamelCaps!
    """A simple attempt to model a dog."""   # docstring — ALWAYS write one

    def __init__(self, name, age):    # constructor: runs automatically on creation
        """Initialize name and age attributes."""
        self.name = name              # self.name = instance attribute
        self.age = age

    def sit(self):                    # method — always takes self as first param
        """Simulate a dog sitting."""
        print(self.name.title() + " is now sitting.")

    def roll_over(self):
        """Simulate rolling over."""
        print(self.name.title() + " rolled over!")
```

---

### The `__init__()` Method — The Constructor

`__init__()` is **automatically called** by Python the moment you create an instance. Think of it as the "setup crew" that runs before you get to use the object.

```python
# What Python does internally when you write:
my_dog = Dog('willie', 6)   # Python

# 1. Allocates memory for a new Dog object
# 2. Calls Dog.__init__(my_dog, 'willie', 6)
# 3. __init__ stores: my_dog.name = 'willie', my_dog.age = 6
# 4. Returns the object and assigns it to my_dog
```

**Rules for `__init__()`:**

- The first parameter **must** be `self` — it is a reference to the instance being created.
- `self` is passed automatically — you never pass it yourself.
- Variables set with `self.` become attributes accessible throughout the entire class.
- It has **no explicit `return`** — Python returns the new instance automatically.

> **Java comparison:** `__init__` = constructor. `self` = `this`. Python just makes `self` visible.

```python
# Java                                    # Python equivalent
class Dog {                               class Dog():
    String name;                              def __init__(self, name, age):
    int age;                                      self.name = name
    Dog(String name, int age) {                   self.age = age
        this.name = name;
        this.age = age;
    }
}
```

---

### Setting a Default Value for an Attribute

If an attribute always starts with the same value, set it inside `__init__()` without adding it as a parameter — this makes it _optional_ to provide:

```python
class Car():   # Python
    def __init__(self, make, model, year):
        self.make = make
        self.model = model
        self.year = year
        self.odometer_reading = 0     # ← default value, NOT a parameter

    def get_descriptive_name(self):
        long_name = str(self.year) + ' ' + self.make + ' ' + self.model
        return long_name.title()

    def read_odometer(self):
        print("This car has " + str(self.odometer_reading) + " miles on it.")

my_new_car = Car('audi', 'a4', 2016)
print(my_new_car.get_descriptive_name())   # 2016 Audi A4
my_new_car.read_odometer()                  # This car has 0 miles on it.
```

---

## PART 3 — WORKING WITH INSTANCES

### Creating an Instance

Creating an instance is called **instantiation**. It looks like a function call to the class itself:

```python
# Creating instances   # Python
my_dog = Dog('willie', 6)
your_dog = Dog('lucy', 3)

# Each is a completely independent object with its own data
print(my_dog.name)     # willie
print(your_dog.name)   # lucy
# They don't share data — changing one doesn't affect the other
```

---

### Accessing Attributes — Dot Notation

Use `instance.attribute` to read a value:

```python
my_dog = Dog('willie', 6)   # Python

print(my_dog.name)    # willie
print(my_dog.age)     # 6

# Python's lookup order:
# 1. Look at the instance my_dog
# 2. Find the attribute 'name' that belongs to it
# 3. Return its value
```

---

### Calling Methods

Use `instance.method()` — dot notation again:

```python
my_dog = Dog('willie', 6)   # Python

my_dog.sit()        # Willie is now sitting.
my_dog.roll_over()  # Willie rolled over!

# You NEVER pass self manually when calling — Python does it:
# my_dog.sit()  is equivalent to  Dog.sit(my_dog)
```

---

### Modifying Attributes — Three Ways

#### 1. Directly Through the Instance

The simplest — access and overwrite:

```python
my_new_car = Car('audi', 'a4', 2016)   # Python
my_new_car.odometer_reading = 23        # direct assignment
my_new_car.read_odometer()              # This car has 23 miles on it.
```

#### 2. Through a Method (Setter)

Better practice — the method can validate the input before accepting it:

```python
class Car():   # Python
    # ...
    def update_odometer(self, mileage):
        """Set odometer, reject rollbacks."""
        if mileage >= self.odometer_reading:
            self.odometer_reading = mileage
        else:
            print("You can't roll back an odometer!")

my_new_car.update_odometer(23)    # ✅ sets to 23
my_new_car.update_odometer(10)    # ❌ "You can't roll back an odometer!"
```

#### 3. Incrementing (Adding) Through a Method

For values you want to _add_ to rather than replace:

```python
class Car():   # Python
    # ...
    def increment_odometer(self, miles):
        """Add the given amount to the odometer reading."""
        self.odometer_reading += miles

my_used_car = Car('subaru', 'outback', 2013)
my_used_car.update_odometer(23500)      # start at 23500
my_used_car.increment_odometer(100)     # add 100 more
my_used_car.read_odometer()             # This car has 23600 miles on it.
```

> ⚠️ **Security note:** Anyone with access to the object can still set `my_car.odometer_reading = 0` directly. Python has no true `private` keyword. Use naming conventions: `_attribute` (protected by convention) or `__attribute` (name-mangled, harder to access).

---

### Multiple Instances — Each is Independent

```python
s = PartyAnimal('Sally')   # Python
j = PartyAnimal('Jim')

s.party()   # Sally party count 1
j.party()   # Jim party count 1
s.party()   # Sally party count 2   ← s and j have INDEPENDENT x counters
```

Each object has its own copy of all instance attributes. `s.x` and `j.x` are completely separate variables.

---

### Inspecting an Object with `dir()` and `type()`

```python
an = PartyAnimal()   # Python

print(type(an))        # <class '__main__.PartyAnimal'>
print(type(an.x))      # <class 'int'>
print(type(an.party))  # <class 'method'>
print(dir(an))         # lists ALL attributes and methods, incl. dunder ones
```

---

## PART 4 — OBJECT LIFECYCLE

### Constructor — `__init__()`

Called automatically when the object is **created**. Use it to set initial state:

```python
class PartyAnimal:   # Python
    def __init__(self):
        self.x = 0
        print('I am constructed')
```

### Destructor — `__del__()`

Called automatically when the object is **destroyed** (when the variable is reassigned or goes out of scope). Rarely needed — use for cleanup (closing files, releasing resources):

```python
class PartyAnimal:   # Python
    def __init__(self):
        self.x = 0
        print('I am constructed')

    def party(self):
        self.x += 1
        print('So far', self.x)

    def __del__(self):
        print('I am destructed', self.x)

an = PartyAnimal()   # Output: I am constructed
an.party()           # So far 1
an.party()           # So far 2
an = 42              # Python destroys the object here → I am destructed 2
print('an contains', an)   # an contains 42
```

> **Lifecycle summary:** `__init__` → (object lives and does things) → `__del__` → object gone.

---

## PART 5 — INHERITANCE

### Core Concept

Inheritance lets a **child class** automatically get all the attributes and methods of a **parent class**, then add or change things on top.

Think of it like biological inheritance: a `CricketFan` is a `PartyAnimal` who also knows cricket — they have all the PartyAnimal behaviours PLUS cricket-specific ones.

```
PartyAnimal (parent / superclass)
    └── CricketFan (child / subclass) — inherits everything, adds six()
```

---

### Basic Inheritance — `super()`

```python
from party import PartyAnimal   # Python — parent in its own file

class CricketFan(PartyAnimal):  # CricketFan inherits from PartyAnimal
    def __init__(self, nam):
        super().__init__(nam)   # ← calls PartyAnimal's __init__ first!
        self.points = 0         # CricketFan-specific attribute

    def six(self):
        self.points += 6
        self.party()            # can call parent methods directly!
        print(self.name, "points", self.points)

s = PartyAnimal("Sally")
j = CricketFan("Jim")
s.party()   # Sally party count 1
j.party()   # Jim party count 1
j.six()     # Jim party count 2 / Jim points 6
```

**The `super()` function:** tells Python to call `__init__` from the parent class. This sets up all parent attributes (`self.name`, `self.x`) before CricketFan adds its own (`self.points`). Without this, `CricketFan` instances would be missing parent attributes.

---

### Real-World Inheritance Example — ElectricCar

```python
class Car():   # Python — parent class MUST appear BEFORE child class in the file
    """A simple attempt to represent a car."""

    def __init__(self, make, model, year):
        self.make = make
        self.model = model
        self.year = year
        self.odometer_reading = 0

    def get_descriptive_name(self):
        long_name = str(self.year) + ' ' + self.make + ' ' + self.model
        return long_name.title()

    def read_odometer(self):
        print("This car has " + str(self.odometer_reading) + " miles on it.")

    def update_odometer(self, mileage):
        if mileage >= self.odometer_reading:
            self.odometer_reading = mileage
        else:
            print("You can't roll back an odometer!")

    def increment_odometer(self, miles):
        self.odometer_reading += miles


class ElectricCar(Car):           # ← Car is in parentheses = inheriting
    """Models aspects specific to electric vehicles."""

    def __init__(self, make, model, year):
        """
        Initialize attributes of the parent class.
        Then initialize attributes specific to an electric car.
        """
        super().__init__(make, model, year)  # parent setup
        self.battery_size = 70               # child-only attribute

    def describe_battery(self):              # child-only method
        print("This car has a " + str(self.battery_size) + "-kWh battery.")


my_tesla = ElectricCar('tesla', 'model s', 2016)
print(my_tesla.get_descriptive_name())   # 2016 Tesla Model S  ← inherited method
my_tesla.describe_battery()              # This car has a 70-kWh battery.
```

---

### Overriding Parent Methods

Sometimes a method from the parent class doesn't make sense for the child. Redefine it with the **same name** and Python will always use the child's version:

```python
class ElectricCar(Car):   # Python
    # ...
    def fill_gas_tank(self):        # Car had a fill_gas_tank() method
        """Electric cars don't have gas tanks."""
        print("This car doesn't need a gas tank!")

# Now calling my_tesla.fill_gas_tank() uses ElectricCar's version, not Car's
```

> **Rule:** Python looks for the method in the instance's class first, then walks up to the parent. Define the same method name in the child = override.

---

### Instances as Attributes — Composition

When a class starts growing too many attributes/methods, break it into smaller classes and make instances of those smaller classes _attributes_ of the bigger class. This is called **composition** (or "has-a" relationship vs inheritance's "is-a" relationship):

```python
class Battery():   # Python — standalone class, no inheritance
    """A simple attempt to model a battery for an electric car."""

    def __init__(self, battery_size=70):   # default parameter!
        """Initialize the battery's attributes."""
        self.battery_size = battery_size

    def describe_battery(self):
        print("This car has a " + str(self.battery_size) + "-kWh battery.")

    def get_range(self):
        if self.battery_size == 70:
            range = 240
        elif self.battery_size == 85:
            range = 270
        message = "This car can go approximately " + str(range)
        message += " miles on a full charge."
        print(message)


class ElectricCar(Car):
    def __init__(self, make, model, year):
        super().__init__(make, model, year)
        self.battery = Battery()    # ← Battery INSTANCE stored as attribute!

my_tesla = ElectricCar('tesla', 'model s', 2016)
my_tesla.battery.describe_battery()   # chain dot notation to reach nested obj
my_tesla.battery.get_range()
# 2016 Tesla Model S
# This car has a 70-kWh battery.
# This car can go approximately 240 miles on a full charge.
```

> **When to use composition vs inheritance:**
> 
> - "ElectricCar **IS A** Car" → use inheritance
> - "ElectricCar **HAS A** Battery" → use composition

---

## PART 6 — IMPORTING CLASSES

As projects grow, move classes to separate files (modules) and import them. This keeps each file clean and focused.

### Import a Single Class

```python
# car.py — the module (just contains the Car class)
"""A class that can be used to represent a car."""

class Car():
    # ... full class definition


# my_car.py — the program that uses it
from car import Car    # Python — from <module_name> import <ClassName>

my_new_car = Car('audi', 'a4', 2016)
print(my_new_car.get_descriptive_name())
my_new_car.odometer_reading = 23
my_new_car.read_odometer()
```

---

### Import Multiple Classes from a Module

```python
from car import Car, ElectricCar   # Python — comma-separate class names

my_beetle = Car('volkswagen', 'beetle', 2016)
my_tesla  = ElectricCar('tesla', 'roadster', 2016)

print(my_beetle.get_descriptive_name())   # 2016 Volkswagen Beetle
print(my_tesla.get_descriptive_name())    # 2016 Tesla Roadster
```

---

### Import the Entire Module

Avoids naming conflicts — every class access includes the module name:

```python
import car   # Python

my_beetle = car.Car('volkswagen', 'beetle', 2016)      # module.ClassName()
my_tesla  = car.ElectricCar('tesla', 'roadster', 2016)
```

---

### Import All Classes — ⚠️ Avoid This

```python
from module_name import *   # Python — NOT recommended!
# ❌ Makes it unclear where each class came from
# ❌ Can cause naming conflicts that are hard to debug
# ✅ Use full module import instead when you need many classes
```

---

### Import a Module into a Module

When `ElectricCar` lives in `electric_car.py` but still needs the `Car` class from `car.py`:

```python
# electric_car.py   # Python
"""A set of classes that can be used to represent electric cars."""

from car import Car    # ← import Car from the other module

class Battery():
    # ...

class ElectricCar(Car):
    # ...


# my_cars.py
from car import Car
from electric_car import ElectricCar

my_beetle = Car('volkswagen', 'beetle', 2016)
my_tesla  = ElectricCar('tesla', 'roadster', 2016)
```

---

## PART 7 — THE PYTHON STANDARD LIBRARY

Python ships with a rich standard library of pre-built classes and functions. Now that you understand classes, you can use these powerfully.

### `OrderedDict` from `collections`

A regular dict doesn't guarantee insertion order (pre-Python 3.7). `OrderedDict` always remembers the order keys were added, even in older Python:

```python
from collections import OrderedDict   # Python

favorite_languages = OrderedDict()   # no curly braces — use constructor

favorite_languages['jen']    = 'python'
favorite_languages['sarah']  = 'c'
favorite_languages['edward'] = 'ruby'
favorite_languages['phil']   = 'python'

for name, language in favorite_languages.items():
    print(name.title() + "'s favorite language is " + language.title() + ".")
# Jen's favorite language is Python.
# Sarah's favorite language is C.
# Edward's favorite language is Ruby.
# Phil's favorite language is Python.
# (always in insertion order!)
```

> **Note:** In Python 3.7+, regular `dict` also maintains insertion order. But `OrderedDict` is still useful when you need the ordered-dict type to be explicit in your code's intent, or for older codebases.

---

## PART 8 — OOP PATTERNS AND BEST PRACTICES

### The `PartyAnimal` Pattern — Minimal Complete Class

The simplest complete class that demonstrates all key ideas at once:

```python
class PartyAnimal:   # Python

    def __init__(self, nam):
        self.x = 0
        self.name = nam
        print(self.name, 'constructed')

    def party(self):
        self.x += 1
        print(self.name, 'party count', self.x)

    def __del__(self):
        print(self.name, 'destructed, x =', self.x)


s = PartyAnimal('Sally')   # Sally constructed
j = PartyAnimal('Jim')     # Jim constructed
s.party()                  # Sally party count 1
j.party()                  # Jim party count 1
j.party()                  # Jim party count 2
# When the program ends:   # Sally destructed, x = 1
                           # Jim destructed, x = 2
```

---

### OOP as a Network of Cooperating Objects

A well-designed OOP program is a **network of objects** — each handling its own data, communicating with the others:

```
BeautifulSoup scraper example:
Input → String Object → Urllib Object → Socket Object
                     → BeautifulSoup Object → html.parser Object → Dictionary Object → String Object → Output
```

Each object hides its complexity. You use `BeautifulSoup` without knowing or caring how it handles HTTP or HTML parsing internally. This is **encapsulation**.

---

## Glossary

|Term|Definition|
|---|---|
|**class**|A template that defines attributes and methods for objects of that type|
|**object / instance**|A constructed instance of a class — has its own copy of all attributes|
|**attribute**|A variable that is part of an object — accessed via `self.attr` or `instance.attr`|
|**method**|A function defined inside a class — always takes `self` as first parameter|
|**`self`**|A reference to the current instance — Python's equivalent of Java's `this`|
|**constructor**|The `__init__()` method — runs automatically on object creation to set initial state|
|**destructor**|The `__del__()` method — runs just before object is destroyed|
|**instantiation**|The act of creating an object from a class|
|**inheritance**|A child class inheriting all attributes and methods from a parent class|
|**parent / superclass**|The class being inherited from|
|**child / subclass**|The class doing the inheriting — extends the parent|
|**`super()`**|Calls the parent class's method — used in child's `__init__` to run parent setup|
|**override**|Redefining a parent's method in the child class — child's version takes priority|
|**composition**|Storing an instance of one class as an attribute of another — "has-a" relationship|
|**encapsulation**|Bundling data + methods together and hiding internal complexity|
|**module**|A `.py` file containing class definitions that can be imported|
|**dot notation**|`object.attribute` or `object.method()` — how you access things inside objects|
|**`dir()`**|Built-in function that lists all attributes and methods of an object|
|**`type()`**|Built-in function that returns the class/type of a variable|
|**docstring**|A string literal right after `class` or `def` — describes what the class/method does|
|**CamelCaps**|Naming convention for class names: `MyClassName` (each word capitalized, no underscores)|

---

## Java vs Python — OOP

|Concept|Java|Python|
|---|---|---|
|Define a class|`class Dog { }`|`class Dog():`|
|Constructor|`Dog(String name) { this.name = name; }`|`def __init__(self, name): self.name = name`|
|Instance variable|`this.name = name;`|`self.name = name`|
|Create instance|`Dog myDog = new Dog("Willie", 6);`|`my_dog = Dog('willie', 6)`|
|Access attribute|`myDog.name`|`my_dog.name`|
|Call method|`myDog.sit();`|`my_dog.sit()`|
|Method definition|`public void sit() { }`|`def sit(self):`|
|Inheritance|`class ElectricCar extends Car { }`|`class ElectricCar(Car):`|
|Call parent constructor|`super(make, model, year);`|`super().__init__(make, model, year)`|
|Override method|Redefine with same signature|Redefine with same name|
|`this` keyword|`this`|`self` (by convention — could technically be any name)|
|`private` attribute|`private int x;`|No true private; use `_x` (convention) or `__x` (name mangling)|
|Import class|`import com.package.ClassName;`|`from module import ClassName`|
|Destructor|`finalize()` (deprecated)|`__del__()`|
|List all methods|Reflection API|`dir(obj)`|
|Check type|`obj instanceof Dog`|`type(obj)` or `isinstance(obj, Dog)`|

---

## Quick Reference

```python
# ══ DEFINING A CLASS ══════════════════════════════════════════════   # Python

class MyClass():
    """Docstring — always include this."""

    def __init__(self, param1, param2, default_param=0):
        self.attr1 = param1          # instance attribute — each object owns its own
        self.attr2 = param2
        self.default_attr = default_param   # default attribute

    def my_method(self):             # regular instance method
        print(self.attr1)

    def update_attr(self, new_val):  # setter method
        self.attr1 = new_val

    def __del__(self):               # destructor — optional, rarely needed
        print("Object destroyed")


# ══ CREATING AND USING INSTANCES ══════════════════════════════════

obj = MyClass('hello', 42)          # instantiation — calls __init__ automatically
obj.my_method()                     # call a method
print(obj.attr1)                    # access attribute
obj.attr1 = 'new value'             # modify attribute directly
obj.update_attr('via method')       # modify via setter

print(type(obj))                    # <class '__main__.MyClass'>
print(dir(obj))                     # list all attributes and methods


# ══ INHERITANCE ═══════════════════════════════════════════════════

class ChildClass(ParentClass):
    def __init__(self, param1, extra_param):
        super().__init__(param1)            # run parent's __init__ FIRST
        self.extra = extra_param            # child-only attribute

    def extra_method(self):                 # child-only method
        pass

    def parent_method_override(self):       # overrides parent version
        print("I do something different now")


# ══ COMPOSITION ═══════════════════════════════════════════════════

class Engine():
    def __init__(self, cylinders=4):
        self.cylinders = cylinders
    def start(self):
        print(f"Engine with {self.cylinders} cylinders started")

class Car():
    def __init__(self, make):
        self.make = make
        self.engine = Engine()              # ← Engine INSTANCE as attribute

my_car = Car('toyota')
my_car.engine.start()                       # chain dot notation


# ══ IMPORTING ════════════════════════════════════════════════════

from module_name import ClassName            # import one class
from module_name import ClassA, ClassB       # import multiple classes
import module_name                           # import whole module
# then use: module_name.ClassName()

# from module_name import *  ← AVOID — unclear and risky


# ══ STANDARD LIBRARY ═════════════════════════════════════════════

from collections import OrderedDict
od = OrderedDict()
od['a'] = 1
od['b'] = 2
# keys always returned in insertion order

from random import randint
x = randint(1, 6)                           # random int between 1 and 6 inclusive
```

---

## Debugging Common Bugs

**1. Forgetting `self` as first method parameter**

```python
class Dog():
    def sit():           # ❌ TypeError when called — missing self!
        print("Sitting")

    def sit(self):       # ✅ Always include self
        print("Sitting")
```

**2. Calling `super()` without arguments (Python 2 vs Python 3)**

```python
# Python 3 (what you should use)
super().__init__(make, model, year)   # ✅

# Python 2 (old — you might see this)
super(ElectricCar, self).__init__(make, model, year)   # still works in Py3 too
```

**3. Modifying an attribute that doesn't exist**

```python
class Dog():
    def __init__(self, name):
        self.name = name

d = Dog('willie')
print(d.color)   # ❌ AttributeError — 'color' was never set in __init__
```

**4. Child class not calling `super().__init__()`**

```python
class ElectricCar(Car):
    def __init__(self, make, model, year):
        # ❌ forgot super().__init__() — self.make, self.model, self.year are MISSING
        self.battery_size = 70

    def __init__(self, make, model, year):
        super().__init__(make, model, year)   # ✅ parent attributes now exist
        self.battery_size = 70
```

**5. Mixing up class attribute vs instance attribute**

```python
class Dog():
    species = 'mammal'    # class attribute — SHARED by all instances!

    def __init__(self, name):
        self.name = name  # instance attribute — unique to each object

d1 = Dog('willie')
d2 = Dog('lucy')
Dog.species = 'canine'    # changes it for ALL Dog instances!
print(d1.species)   # canine  ← both changed
print(d2.species)   # canine
```

**6. Using a mutable default argument in `__init__`**

```python
class Cart():
    def __init__(self, items=[]):   # ❌ THE LIST IS SHARED ACROSS ALL INSTANCES!
        self.items = items

    def __init__(self, items=None):  # ✅ use None, create fresh list inside
        if items is None:
            self.items = []
        else:
            self.items = items
```

---

## Practice Exercises

**Basic Classes:**

- [ ] **Restaurant** — Create a `Restaurant` class with `restaurant_name` and `cuisine_type` attributes. Methods: `describe_restaurant()` and `open_restaurant()`. Create an instance and call both methods.
- [ ] **Three Restaurants** — Create 3 instances of Restaurant. Call `describe_restaurant()` on each.
- [ ] **Users** — Class `User` with `first_name`, `last_name`, and more profile attributes. Methods: `describe_user()` and `greet_user()`. Create several instances.

**Working with Instances:**

- [ ] **Number Served** — Add `number_served = 0` as default attribute to Restaurant. Print it, change it directly, then add a `set_number_served()` setter and `increment_number_served()` incrementer method.
- [ ] **Login Attempts** — Add `login_attempts` to a User class. Write `increment_login_attempts()` and `reset_login_attempts()`. Test both.

**Inheritance:**

- [ ] **Ice Cream Stand** — `IceCreamStand` inherits from `Restaurant`. Adds a `flavors` list attribute and a method to display flavors.
- [ ] **Admin** — `Admin` inherits from `User`. Adds a `privileges` list. Method `show_privileges()` lists them.
- [ ] **Privileges** — Move `privileges` and `show_privileges()` into their own `Privileges` class. Make a `Privileges` instance as an attribute of `Admin`.
- [ ] **Battery Upgrade** — Add `upgrade_battery()` to `Battery` — if size isn't 85, set it to 85. Call `get_range()` before and after to see the difference.

**Importing:**

- [ ] **Imported Restaurant** — Store your Restaurant class in a module. Create a separate file that imports and uses it.
- [ ] **Multiple Modules** — Store `User` in one module, `Privileges` and `Admin` in another. Import from each in a main file.

---

## Questions I Still Have

_Write your open questions here. Return later and answer them._

---

## Related Notes

- [[DICTIONARIES, TUPLES & SETS]]
- [[FUNCTIONS]]
- [[CONDITIONAL STATEMENT]]
- [[ITERATIVE STATEMENTS]]
- [[LIST]]