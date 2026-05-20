---
title: COLLECTIONS
date: 2026-05-20
phase: phase-1
tags:
  - concept
  - python
links: []
status: learning
---
# PYTHON `collections` MODULE

## One-Line Summary

**`collections`** is Python's standard library module that gives you _specialised container types_ — smarter, purpose-built replacements for the plain `dict`, `list`, `set`, and `tuple` built-ins — each designed to solve one specific problem elegantly.

---

## PART 1 — WHY `collections` EXISTS

### The Problem with Plain Built-ins

Python's built-in `dict`, `list`, and `tuple` are general-purpose — they work for everything but are optimised for nothing. When your code has specific needs — like counting items, grouping data, implementing a queue, or accessing tuple fields by name — you end up writing repetitive boilerplate:

```python
# Counting words WITHOUT collections — boilerplate-heavy
word_count = {}
for word in text.split():
    if word not in word_count:
        word_count[word] = 0     # manually handle missing key
    word_count[word] += 1

# Grouping items WITHOUT collections — same KeyError problem
groups = {}
for item in items:
    key = item[0]
    if key not in groups:
        groups[key] = []         # manually initialise every new key
    groups[key].append(item)
```

`collections` eliminates all of this. Each class is a **dict/list/tuple subclass** — fully compatible with the original — but with one extra superpower.

### The 6 Classes You Must Know

|Class|Is a|Superpower|Replace when you need to…|
|---|---|---|---|
|`Counter`|`dict`|Count things automatically|Tally items, find most common|
|`defaultdict`|`dict`|Auto-create missing keys|Group, accumulate, avoid KeyError|
|`OrderedDict`|`dict`|Remembers insertion order + extras|Reorder keys, LRU cache logic|
|`deque`|(object)|O(1) append/pop on BOTH ends|Queue, stack, sliding window|
|`namedtuple`|`tuple`|Access elements by name|Readable tuples, lightweight records|
|`ChainMap`|(MutableMapping)|Stack multiple dicts as one view|Config layering, scope simulation|

> **Java comparison:** Java has `LinkedList` (≈ deque), `LinkedHashMap` (≈ OrderedDict), no native Counter (you'd use `Map<T, Integer>` + manual logic). Python's `collections` gives you all of these in one import.

---

## PART 2 — `Counter`

### What It Is

`Counter` is a dict subclass where values are **always integer counts**. Access a missing key and you get `0` instead of a `KeyError`. It's a frequency counter / bag / multiset.

```python
from collections import Counter

# Count characters in a string
c = Counter('abracadabra')
print(c)   # Counter({'a': 5, 'b': 2, 'r': 2, 'c': 1, 'd': 1})

# Count words in a list
words = ['red', 'blue', 'red', 'green', 'blue', 'blue']
c2 = Counter(words)
print(c2)  # Counter({'blue': 3, 'red': 2, 'green': 1})

# Count from keyword arguments
c3 = Counter(cats=4, dogs=8)
print(c3)  # Counter({'dogs': 8, 'cats': 4})
```

### Creating a Counter — 4 Ways

```python
from collections import Counter

c1 = Counter()                          # empty counter
c2 = Counter('gallahad')               # from iterable (string)
c3 = Counter({'red': 4, 'blue': 2})    # from dict
c4 = Counter(cats=4, dogs=8)           # from keyword args
```

### The 3 Key Methods

**`most_common(n)`** — Top n items by count (sorted descending):

```python
c = Counter('abracadabra')
print(c.most_common(3))   # [('a', 5), ('b', 2), ('r', 2)]
print(c.most_common())    # all items, sorted by count
# To get LEAST common: reverse the full list
print(c.most_common()[:-4:-1])   # last 3 (least common)
```

**`elements()`** — Iterator that repeats each element by its count:

```python
c = Counter(a=4, b=2, c=0, d=-2)
print(list(c.elements()))   # ['a', 'a', 'a', 'a', 'b', 'b']
# Note: elements with count ≤ 0 are ignored
```

**`update()`** — Add more counts (does NOT replace, it accumulates):

```python
c = Counter({'red': 2, 'blue': 1})
c.update({'red': 1, 'green': 3})
print(c)   # Counter({'red': 3, 'green': 3, 'blue': 1})

# Also works with an iterable:
c.update(['red', 'red'])
print(c)   # Counter({'red': 5, 'green': 3, 'blue': 1})
```

**`subtract()`** — Subtract counts (allows zero and negatives, unlike `-`):

```python
c = Counter(a=4, b=2, c=0, d=-2)
d = Counter(a=1, b=2, c=3, d=4)
c.subtract(d)
print(c)   # Counter({'a': 3, 'b': 0, 'c': -3, 'd': -6})
```

**`total()`** — Sum of all counts (Python 3.10+):

```python
c = Counter('abracadabra')
print(c.total())   # 11
```

### Missing Keys Return 0 — Not KeyError

```python
c = Counter('abracadabra')
print(c['z'])   # 0   ← never raises KeyError
print(c['a'])   # 5
```

### Arithmetic on Counters

Counters support `+`, `-`, `&`, `|` operations — very powerful for set-like frequency analysis:

```python
c1 = Counter({'a': 3, 'b': 1})
c2 = Counter({'a': 1, 'b': 4})

print(c1 + c2)   # Counter({'b': 5, 'a': 4})   ← add counts
print(c1 - c2)   # Counter({'a': 2})            ← subtract, keep only positives
print(c1 & c2)   # Counter({'a': 1, 'b': 1})   ← min of each (intersection)
print(c1 | c2)   # Counter({'b': 4, 'a': 3})   ← max of each (union)
```

> ⚠️ The `-` operator drops zero and negative counts. Use `subtract()` if you need to keep them.

### Real-World Use Cases

```python
from collections import Counter

# ── Word frequency in text ─────────────────────────────────────
text = "the cat sat on the mat the cat"
word_freq = Counter(text.split())
print(word_freq.most_common(3))   # [('the', 3), ('cat', 2), ('sat', 1)]

# ── Check if two strings are anagrams ─────────────────────────
def is_anagram(s1, s2):
    return Counter(s1.lower()) == Counter(s2.lower())

print(is_anagram('listen', 'silent'))   # True
print(is_anagram('hello', 'world'))     # False

# ── Find characters in s1 NOT in s2 ───────────────────────────
diff = Counter('abcde') - Counter('ace')
print(list(diff.elements()))   # ['b', 'd']

# ── Inventory management ───────────────────────────────────────
inventory = Counter(apples=50, oranges=30, bananas=20)
sold      = Counter(apples=10, oranges=5)
remaining = inventory - sold
print(remaining)   # Counter({'apples': 40, 'oranges': 25, 'bananas': 20})
```

---

## PART 3 — `defaultdict`

### What It Is

`defaultdict` is a dict subclass that accepts a **factory function** as its first argument. When you access a **missing key**, instead of raising `KeyError`, it automatically calls that factory to create a default value and inserts it.

```python
from collections import defaultdict

# Plain dict — KeyError on missing key
d = {}
d['fruits'].append('apple')   # ❌ KeyError: 'fruits'

# defaultdict(list) — auto-creates empty list for missing keys
dd = defaultdict(list)
dd['fruits'].append('apple')   # ✅ no error — creates [] then appends
dd['fruits'].append('banana')
dd['vegs'].append('carrot')
print(dd)
# defaultdict(<class 'list'>, {'fruits': ['apple', 'banana'], 'vegs': ['carrot']})
```

### The Factory Function — What You Can Pass

The factory is called with **no arguments** to produce the default value:

```python
from collections import defaultdict

defaultdict(list)           # missing key → []
defaultdict(int)            # missing key → 0
defaultdict(set)            # missing key → set()
defaultdict(str)            # missing key → ''
defaultdict(float)          # missing key → 0.0
defaultdict(lambda: 'N/A')  # missing key → 'N/A'
defaultdict(lambda: {'count': 0, 'items': []})  # missing key → fresh dict
```

### The 3 Classic Patterns

**Pattern 1 — Counting (like Counter, but manual):**

```python
from collections import defaultdict

dd = defaultdict(int)
for ch in 'abracadabra':
    dd[ch] += 1    # auto-starts at 0, no KeyError

print(dd)
# defaultdict(<class 'int'>, {'a': 5, 'b': 2, 'r': 2, 'c': 1, 'd': 1})
```

**Pattern 2 — Grouping items:**

```python
from collections import defaultdict

words = ['apple', 'ant', 'banana', 'cherry', 'avocado', 'blueberry']
by_letter = defaultdict(list)

for w in words:
    by_letter[w[0]].append(w)

print(dict(by_letter))
# {'a': ['apple', 'ant', 'avocado'], 'b': ['banana', 'blueberry'], 'c': ['cherry']}
```

**Pattern 3 — Nested structures:**

```python
from collections import defaultdict

# Nested defaultdict — a "tree" you can write to without setup
tree = defaultdict(lambda: defaultdict(int))
tree['fruits']['apple'] += 1
tree['fruits']['banana'] += 3
tree['vegs']['carrot'] += 2

print(tree['fruits'])   # defaultdict(<class 'int'>, {'apple': 1, 'banana': 3})
print(tree['vegs'])     # defaultdict(<class 'int'>, {'carrot': 2})
```

### `defaultdict` vs `dict.get()` vs `dict.setdefault()`

```python
d = {}
key = 'x'

# Option 1: plain dict — must check manually
if key not in d:
    d[key] = []
d[key].append(1)

# Option 2: dict.get() — read only, doesn't insert the default
val = d.get(key, [])   # returns [] but doesn't store it!

# Option 3: dict.setdefault() — inserts if missing, then returns
d.setdefault(key, []).append(1)   # works but ugly and slower

# Option 4: defaultdict — cleanest, fastest for repeated access
from collections import defaultdict
dd = defaultdict(list)
dd[key].append(1)   # ✅ auto-inserts [] on first access
```

> 💡 **Rule of thumb:**
> 
> - Counting → use `Counter`
> - Grouping into lists/sets → use `defaultdict(list)` / `defaultdict(set)`
> - Accumulating numbers → use `defaultdict(int)`
> - One-off missing key handling → use `dict.get(key, default)`

### The `default_factory` Attribute

You can inspect or change the factory after creation:

```python
from collections import defaultdict

dd = defaultdict(list)
print(dd.default_factory)    # <class 'list'>
dd.default_factory = int     # change factory on the fly
print(dd['new_key'])         # 0  (now uses int)
dd.default_factory = None    # disable auto-creation (behaves like plain dict)
```

---

## PART 4 — `OrderedDict`

### What It Is (and Why It Still Matters)

`OrderedDict` is a dict subclass that **remembers insertion order** and adds extra methods for reordering. Since Python 3.7, regular `dict` also preserves insertion order — so why use `OrderedDict`?

Because `OrderedDict` has **two extra features** plain `dict` doesn't:

1. `move_to_end()` — reposition a key to the front or back
2. `popitem(last=True/False)` — pop from either end
3. **Order-sensitive equality** — two `OrderedDict`s are equal only if order matches

```python
from collections import OrderedDict

od = OrderedDict()
od['one']   = 1
od['two']   = 2
od['three'] = 3
print(od)   # OrderedDict({'one': 1, 'two': 2, 'three': 3})

# Order-sensitive equality — KEY DIFFERENCE from plain dict
od1 = OrderedDict([('a', 1), ('b', 2)])
od2 = OrderedDict([('b', 2), ('a', 1)])
print(od1 == od2)                      # False ← ORDER MATTERS
print(dict(od1) == dict(od2))          # True  ← plain dict ignores order
```

### `move_to_end()` — Reorder Keys

```python
from collections import OrderedDict

od = OrderedDict({'one': 1, 'two': 2, 'three': 3})

od.move_to_end('one')                   # move to BACK (default)
print(od)   # OrderedDict({'two': 2, 'three': 3, 'one': 1})

od.move_to_end('three', last=False)     # move to FRONT
print(od)   # OrderedDict({'three': 3, 'two': 2, 'one': 1})
```

### `popitem()` — Pop from Either End

```python
from collections import OrderedDict

od = OrderedDict({'a': 1, 'b': 2, 'c': 3})

print(od.popitem(last=True))    # ('c', 3)  ← pops LAST (default)
print(od.popitem(last=False))   # ('a', 1)  ← pops FIRST
print(od)   # OrderedDict({'b': 2})
```

### LRU Cache Pattern with OrderedDict

The classic use of `OrderedDict` — a Least Recently Used cache that evicts the oldest item when full:

```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity):
        self.cache = OrderedDict()
        self.capacity = capacity

    def get(self, key):
        if key not in self.cache:
            return -1
        self.cache.move_to_end(key)   # mark as recently used
        return self.cache[key]

    def put(self, key, value):
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.capacity:
            self.cache.popitem(last=False)   # evict the oldest (leftmost)

cache = LRUCache(3)
cache.put('a', 1)
cache.put('b', 2)
cache.put('c', 3)
cache.get('a')         # 'a' is now most recently used
cache.put('d', 4)      # evicts 'b' (least recently used)
print(list(cache.cache.keys()))   # ['c', 'a', 'd']
```

### `OrderedDict` vs Plain `dict` — Summary

|Feature|`dict` (3.7+)|`OrderedDict`|
|---|---|---|
|Remembers insertion order|✅|✅|
|`move_to_end()`|❌|✅|
|`popitem(last=False)`|❌ (only LIFO)|✅|
|Order-sensitive equality|❌|✅|
|Use for LRU cache|❌ clunky|✅ natural|
|Memory|Slightly less|Slightly more|

---

## PART 5 — `deque`

### What It Is

`deque` (pronounced "deck") stands for **double-ended queue**. It's an array-like sequence where both `append` and `pop` at **either end** are O(1) — constant time, no matter how big the deque.

Compare with a regular list:

```python
lst = list(range(10000))

# list: O(n) — must shift every element
lst.insert(0, 'new')    # SLOW — moves 10,000 elements

# deque: O(1) — pointer manipulation only
from collections import deque
d = deque(range(10000))
d.appendleft('new')     # FAST — no shifting needed
```

> **Benchmark (verified):** `deque.appendleft` is **~200× faster** than `list.insert(0)` on a 10,000-element sequence.

### Creating a Deque

```python
from collections import deque

d1 = deque()                    # empty deque
d2 = deque([1, 2, 3])           # from iterable
d3 = deque('hello')             # from string → deque(['h','e','l','l','o'])
d4 = deque(range(5), maxlen=3)  # bounded deque, keeps only last 3
print(d4)   # deque([2, 3, 4], maxlen=3)
```

### All Methods at a Glance

```python
from collections import deque
d = deque([1, 2, 3])

# ── Appending ──────────────────────────────
d.append(4)          # add to RIGHT → deque([1, 2, 3, 4])
d.appendleft(0)      # add to LEFT  → deque([0, 1, 2, 3, 4])

# ── Extending ─────────────────────────────
d.extend([5, 6])         # add multiple to RIGHT
d.extendleft([10, 20])   # add multiple to LEFT (NOTE: each is prepended, so reversed!)
# extendleft([10,20]) → first 10 goes left, then 20 goes left → [20, 10, ...]

# ── Removing ──────────────────────────────
d.pop()              # remove and return RIGHTMOST
d.popleft()          # remove and return LEFTMOST
d.remove(3)          # remove first occurrence of value 3

# ── Rotating ──────────────────────────────
d = deque([1, 2, 3, 4, 5])
d.rotate(2)          # shift right by 2 → deque([4, 5, 1, 2, 3])
d.rotate(-1)         # shift left by 1  → deque([5, 1, 2, 3, 4])

# ── Other ─────────────────────────────────
d.reverse()          # reverse in-place
len(d)               # length
d[0], d[-1]          # index access (O(1) at ends, O(n) in middle)
d.count(3)           # count occurrences
d.index(3)           # find index of value
d.clear()            # remove all elements
d.copy()             # shallow copy
```

### The 3 Classic deque Patterns

**Pattern 1 — Queue (FIFO: First In, First Out):**

```python
from collections import deque

queue = deque()
queue.append('task1')    # enqueue → right
queue.append('task2')
queue.append('task3')

print(queue.popleft())   # dequeue → left → 'task1' (first in, first out)
print(queue.popleft())   # 'task2'
```

**Pattern 2 — Stack (LIFO: Last In, First Out):**

```python
from collections import deque

stack = deque()
stack.append('page1')    # push → right
stack.append('page2')
stack.append('page3')

print(stack.pop())       # pop → right → 'page3' (last in, first out)
# Use case: browser back-button history
```

**Pattern 3 — Sliding Window / Rolling History with `maxlen`:**

```python
from collections import deque

# Keep only the last 3 sensor readings
window = deque(maxlen=3)

for reading in [10, 20, 30, 40, 50]:
    window.append(reading)
    print(list(window))
# [10]
# [10, 20]
# [10, 20, 30]
# [20, 30, 40]   ← 10 auto-dropped
# [30, 40, 50]   ← 20 auto-dropped

# Rolling average of last N values
def rolling_avg(data, n):
    window = deque(maxlen=n)
    averages = []
    for x in data:
        window.append(x)
        averages.append(sum(window) / len(window))
    return averages
```

### `maxlen` — Bounded Deque

When `maxlen` is set, appending to a full deque automatically drops items from the **opposite end**:

```python
from collections import deque

d = deque(maxlen=3)
d.append(1); d.append(2); d.append(3)
print(d)            # deque([1, 2, 3], maxlen=3)
d.append(4)         # 4 appended RIGHT → 1 dropped from LEFT
print(d)            # deque([2, 3, 4], maxlen=3)
d.appendleft(0)     # 0 appended LEFT → 4 dropped from RIGHT
print(d)            # deque([0, 2, 3], maxlen=3)

print(d.maxlen)     # 3
```

---

## PART 6 — `namedtuple`

### What It Is

`namedtuple` is a **factory function** that creates a new tuple subclass where each position has a **name**. You get both index access (`p[0]`) AND name access (`p.x`) — the tuple stays immutable and memory-efficient, but your code becomes self-documenting.

```python
# Plain tuple — what does index 0 mean? You must remember.
point = (11, 22)
print(point[0])   # 11 — but is this x? latitude? width?

# namedtuple — self-documenting
from collections import namedtuple
Point = namedtuple('Point', ['x', 'y'])
p = Point(11, 22)
print(p.x)    # 11  ← intention is crystal clear
print(p[0])   # 11  ← also works (still a tuple!)
```

### Creating a namedtuple

The first argument is the **type name** (appears in `repr`). The second argument is the **field names** — a list, a tuple, or a space/comma-separated string:

```python
from collections import namedtuple

# Three equivalent ways to define fields:
Point = namedtuple('Point', ['x', 'y'])
Point = namedtuple('Point', ('x', 'y'))
Point = namedtuple('Point', 'x y')       # space-separated string
Point = namedtuple('Point', 'x, y')      # comma-separated string

# Creating instances — positional or keyword
p1 = Point(11, 22)
p2 = Point(x=11, y=22)
p3 = Point(11, y=22)
print(p1)   # Point(x=11, y=22)
```

### Accessing Values

```python
from collections import namedtuple

Point = namedtuple('Point', ['x', 'y'])
p = Point(11, 22)

# By name (the main advantage)
print(p.x)          # 11
print(p.y)          # 22

# By index (tuple compatibility)
print(p[0])         # 11
print(p[1])         # 22

# Unpacking (works exactly like a regular tuple)
x, y = p
print(x, y)         # 11 22

# In a for loop
for val in p:
    print(val)      # 11, then 22
```

### The 4 Special Methods

All namedtuples have these underscore-prefixed methods (underscore prevents clash with field names):

**`_asdict()`** — Convert to an OrderedDict:

```python
p = Point(11, 22)
print(p._asdict())   # {'x': 11, 'y': 22}
# Useful for JSON serialisation, passing to functions as kwargs
```

**`_replace(**kwargs)`** — Create a new tuple with some fields replaced (original unchanged — tuples are immutable):

```python
p = Point(11, 22)
p2 = p._replace(x=100)
print(p2)   # Point(x=100, y=22)
print(p)    # Point(x=11, y=22)   ← original unchanged
```

**`_make(iterable)`** — Create an instance from any iterable:

```python
data = [10, 20]
p = Point._make(data)
print(p)   # Point(x=10, y=20)

# Useful when reading rows from CSV, DB, etc.
import csv
EmployeeRecord = namedtuple('EmployeeRecord', 'name, age, dept')
for row in csv.reader(open('employees.csv')):
    emp = EmployeeRecord._make(row)
    print(emp.name, emp.dept)
```

**`_fields`** — Tuple of field names (introspection):

```python
print(Point._fields)   # ('x', 'y')

# Combine two namedtuples into a new one:
Point3D = namedtuple('Point3D', Point._fields + ('z',))
print(Point3D._fields)   # ('x', 'y', 'z')
```

### Default Values (Python 3.6.1+)

```python
from collections import namedtuple

# defaults apply to the RIGHTMOST fields
Point3D = namedtuple('Point3D', ['x', 'y', 'z'], defaults=[0])
p = Point3D(1, 2)       # z gets default value 0
print(p)                # Point3D(x=1, y=2, z=0)

# Multiple defaults
Config = namedtuple('Config', ['host', 'port', 'debug'], defaults=['localhost', 8080, False])
c = Config()            # all defaults
print(c)                # Config(host='localhost', port=8080, debug=False)
c2 = Config('prod.server.com')
print(c2)               # Config(host='prod.server.com', port=8080, debug=False)
```

### `rename=True` — Handle Invalid Field Names

```python
from collections import namedtuple

# 'class' and 'return' are Python keywords — can't be field names
# rename=True automatically replaces invalid names with _0, _1, etc.
Fixed = namedtuple('Fixed', ['class', 'return', 'valid'], rename=True)
print(Fixed._fields)    # ('_0', '_1', 'valid')
```

### Real-World Examples

```python
from collections import namedtuple

# ── Database rows ──────────────────────────────────────────────
Employee = namedtuple('Employee', ['name', 'dept', 'salary'])
emp = Employee('Alice', 'Engineering', 95000)
print(f"{emp.name} in {emp.dept} earns {emp.salary}")

# ── Coordinates / geometry ─────────────────────────────────────
Rectangle = namedtuple('Rectangle', ['x', 'y', 'width', 'height'])
rect = Rectangle(10, 20, 100, 50)
area = rect.width * rect.height   # rect[2] * rect[3] would be unreadable

# ── Playing cards ──────────────────────────────────────────────
Card = namedtuple('Card', ['rank', 'suit'])
hand = [Card('King', 'Hearts'), Card('Ace', 'Spades'), Card('7', 'Clubs')]
for card in hand:
    print(f"{card.rank} of {card.suit}")

# ── Function returning multiple values, readable ───────────────
Color = namedtuple('Color', ['red', 'green', 'blue'])

def parse_hex_color(hex_str):
    r = int(hex_str[1:3], 16)
    g = int(hex_str[3:5], 16)
    b = int(hex_str[5:7], 16)
    return Color(r, g, b)

c = parse_hex_color('#FF8040')
print(c.red, c.green, c.blue)   # 255 128 64
```

### `namedtuple` vs `dict` vs `class`

|Feature|`namedtuple`|`dict`|`class`|
|---|---|---|---|
|Immutable|✅|❌|❌ (usually)|
|Access by name|✅ `.x`|✅ `['x']`|✅ `.x`|
|Access by index|✅ `[0]`|❌|❌|
|Memory|Like tuple (small)|Larger|Larger|
|Unpackable|✅|❌|❌|
|Mutable fields|❌|✅|✅|
|Methods|❌ (no custom)|❌|✅|
|Use when|Lightweight records, read-only data|Flexible mappings|Behaviour needed|

---

## PART 7 — `ChainMap`

### What It Is

`ChainMap` groups multiple dicts (or mappings) into a **single unified view**. Lookups search each dict in order — the first one to have the key wins. Writes go to the **first** dict only. The underlying dicts are not merged — they're linked.

Think of it as: **config layering** — command-line args override env vars which override defaults.

```python
from collections import ChainMap

defaults = {'color': 'blue', 'user': 'guest', 'timeout': 30}
env_vars  = {'user': 'admin', 'path': '/usr'}
cli_args  = {'path': '/home', 'debug': True}

# Priority: cli_args > env_vars > defaults
cm = ChainMap(cli_args, env_vars, defaults)

print(cm['user'])     # 'admin'   ← from env_vars (cli_args has no 'user')
print(cm['color'])    # 'blue'    ← falls through to defaults
print(cm['path'])     # '/home'   ← from cli_args (highest priority wins)
print(cm['debug'])    # True      ← from cli_args
```

### Reads vs Writes

```python
from collections import ChainMap

a = {'x': 1}
b = {'x': 10, 'y': 20}
cm = ChainMap(a, b)

# READ: searches a, then b
print(cm['x'])    # 1   ← found in a first

# WRITE: always goes to FIRST map (a)
cm['x'] = 99
print(a)          # {'x': 99}   ← a was modified
print(b)          # {'x': 10, 'y': 20}   ← b unchanged

# Adding a new key also goes to the first map
cm['new'] = 'hello'
print(a)          # {'x': 99, 'new': 'hello'}
```

### `new_child()` and `.parents`

```python
from collections import ChainMap

base = ChainMap({'a': 1, 'b': 2})

# new_child() creates a new ChainMap with a fresh empty dict prepended
child = base.new_child()
child['a'] = 999          # writes to the new front dict only
print(child['a'])         # 999   ← from new layer
print(base['a'])          # 1     ← base unchanged

# .parents strips the first layer
print(child.parents)      # ChainMap({'a': 1, 'b': 2})
```

### Simulating Nested Scopes (like Python itself)

```python
from collections import ChainMap

# Python's own scope lookup works exactly like ChainMap:
# local → enclosing → global → builtins

def simulate_scope():
    builtins = {'print': print, 'len': len}
    globals_ = {'x': 10, 'name': 'global'}
    locals_  = {'x': 99}     # shadows global 'x'

    scope = ChainMap(locals_, globals_, builtins)
    print(scope['x'])         # 99   ← local wins
    print(scope['name'])      # 'global'
    print(scope['print'])     # <built-in function print>

simulate_scope()
```

### When to Use `ChainMap`

```python
# ✅ Good use cases:
# 1. Config with priority layers (CLI > env > file > defaults)
# 2. Template contexts (local vars override global vars)
# 3. Namespace simulation in interpreters
# 4. Temporarily overriding settings without mutating originals

# ❌ When NOT to use:
# - When you just need a merged copy → use {**dict1, **dict2} instead
# - When you need mutable access across all layers → use a regular dict
```

---

## PART 8 — `UserDict`, `UserList`, `UserString`

These three are **wrapper classes** designed for one specific use case: **subclassing**. You inherit from them when you want to extend `dict`, `list`, or `str` with custom behaviour, because inheriting directly from the built-in types can cause subtle bugs when overriding methods.

```python
from collections import UserDict, UserList, UserString

# ── Custom dict that rejects int keys ─────────────────────────
class StrKeyDict(UserDict):
    def __setitem__(self, key, value):
        if not isinstance(key, str):
            raise TypeError(f"Keys must be strings, got {type(key)}")
        super().__setitem__(key, value)

d = StrKeyDict()
d['name'] = 'Alice'    # ✅
# d[42] = 'Bob'        # ❌ TypeError

# ── Custom list that only allows int items ─────────────────────
class IntList(UserList):
    def append(self, item):
        if not isinstance(item, int):
            raise TypeError("Only ints allowed")
        super().append(item)

lst = IntList()
lst.append(42)     # ✅
# lst.append('x') # ❌ TypeError

# ── Custom string that auto-censors words ──────────────────────
class CensoredString(UserString):
    BAD_WORDS = {'secret', 'classified'}
    def __init__(self, data):
        censored = ' '.join(
            '***' if w.lower() in self.BAD_WORDS else w
            for w in data.split()
        )
        super().__init__(censored)

s = CensoredString("this is classified information")
print(s)   # "this is *** information"
```

> **Why not just subclass `dict` directly?** When you subclass `dict` and override `__setitem__`, methods like `update()` and `__init__()` may bypass your override because they're implemented in C and call the C-level slot directly. `UserDict` routes everything through your Python overrides reliably.

---

## PART 9 — CHOOSING THE RIGHT COLLECTION

```
You need to COUNT items
    → Counter

You need to GROUP items into lists (or avoid KeyError on new keys)
    → defaultdict(list)

You need to ACCUMULATE numbers across keys
    → defaultdict(int)

You need a dict that remembers order AND you need move_to_end() or order-sensitive equality
    → OrderedDict

You need a dict that just preserves insertion order (no reordering needed)
    → plain dict (Python 3.7+)

You need fast append/pop from BOTH ends (queue, stack, sliding window)
    → deque

You need a tuple with named fields (readable, lightweight, immutable record)
    → namedtuple

You need to stack multiple dicts with priority lookup (config layers, scopes)
    → ChainMap

You need to extend/subclass dict, list, or str safely
    → UserDict, UserList, UserString
```

---

## PART 10 — PERFORMANCE CHEAT SHEET

|Operation|`list`|`deque`|Notes|
|---|---|---|---|
|`append` (right)|O(1) amortised|O(1)|Both fast|
|`appendleft` / `insert(0,x)`|**O(n)**|O(1)|**deque wins heavily**|
|`pop` (right)|O(1)|O(1)|Both fast|
|`popleft` / `pop(0)`|**O(n)**|O(1)|**deque wins heavily**|
|Index access `[i]`|O(1)|O(n)|**list wins for random access**|
|Search / `in`|O(n)|O(n)|Same|

|Operation|`dict`|`defaultdict`|`Counter`|
|---|---|---|---|
|Key lookup|O(1)|O(1)|O(1)|
|Missing key|`KeyError`|Auto-creates default|Returns 0|
|Counting all items|Manual loop|Manual loop|`Counter(iterable)` one-liner|
|Top N most common|Manual sort|Manual sort|`.most_common(n)`|

> 🔑 **Bottom line:** `deque` for double-ended access, `Counter` for counting, `defaultdict` for grouping. Everything else is a plain `dict` until you need a specific feature.

---

## PART 11 — ALL IMPORTS AT A GLANCE

```python
# Import everything you'll ever need:
from collections import (
    Counter,        # frequency counting
    defaultdict,    # auto-default missing keys
    OrderedDict,    # insertion-ordered dict + move_to_end
    deque,          # double-ended queue
    namedtuple,     # named tuple factory
    ChainMap,       # layered dict lookup
    UserDict,       # base class for custom dicts
    UserList,       # base class for custom lists
    UserString,     # base class for custom strings
)
```

---

## Glossary

|Term|Definition|
|---|---|
|**`Counter`**|Dict subclass for counting; missing keys return 0 instead of KeyError|
|**`most_common(n)`**|Counter method returning the n most frequent elements as [(elem, count)]|
|**`elements()`**|Counter method yielding each element repeated by its count|
|**`update()`**|Counter method that ADDS counts (not replaces)|
|**`subtract()`**|Counter method that subtracts counts (allows negatives)|
|**`defaultdict`**|Dict subclass with a factory for auto-creating values on missing-key access|
|**`default_factory`**|The callable stored on a defaultdict; called with no args to produce default values|
|**`OrderedDict`**|Dict subclass remembering insertion order; adds `move_to_end()` and order-sensitive equality|
|**`move_to_end(key, last=True)`**|Repositions a key to the back (last=True) or front (last=False)|
|**`deque`**|Double-ended queue with O(1) append/pop at both ends|
|**`appendleft()`**|Deque method to add to the LEFT end|
|**`popleft()`**|Deque method to remove and return from the LEFT end|
|**`rotate(n)`**|Deque method to shift all elements n steps right (negative = left)|
|**`maxlen`**|Optional deque limit; auto-drops from opposite end when full|
|**`namedtuple`**|Factory function creating a tuple subclass with named fields|
|**`_asdict()`**|namedtuple method converting to a dict|
|**`_replace(**kwargs)`**|namedtuple method returning a new copy with specified fields changed|
|**`_make(iterable)`**|namedtuple classmethod creating an instance from any iterable|
|**`_fields`**|namedtuple attribute: tuple of field name strings|
|**`defaults`**|namedtuple parameter assigning default values to rightmost fields|
|**`ChainMap`**|Groups multiple dicts into a single view; reads search all, writes go to first|
|**`new_child()`**|ChainMap method adding a new empty dict at the front|
|**`.parents`**|ChainMap property: new ChainMap without the first map|
|**`UserDict`**|Safe base class for creating custom dict subclasses|
|**`UserList`**|Safe base class for creating custom list subclasses|
|**`UserString`**|Safe base class for creating custom string subclasses|
|**O(1)**|Constant time — speed doesn't depend on size|
|**O(n)**|Linear time — speed scales with size|
|**factory function**|A callable that creates and returns a new object each time it's called|
|**FIFO**|First In, First Out — queue pattern|
|**LIFO**|Last In, First Out — stack pattern|

---

## Java vs Python — Collections Comparison

|Java|Python `collections`|Notes|
|---|---|---|
|`HashMap<K,V>`|`dict`|Python built-in|
|`LinkedHashMap<K,V>`|`OrderedDict`|Order-preserving dict|
|`HashMap` with manual count|`Counter`|Counter is far cleaner|
|`Map.getOrDefault()`|`defaultdict`|defaultdict is cleaner|
|`LinkedList<T>`|`deque`|Both O(1) at both ends|
|`ArrayDeque<T>`|`deque`|deque is Python's equivalent|
|`Record` / POJO with only fields|`namedtuple`|namedtuple is immutable|
|No direct equivalent|`ChainMap`|Closest: scope chain in JS|
|Extend `AbstractMap`|Extend `UserDict`|Both are safe base classes|
|`Collections.frequency()`|`Counter[key]`|Counter is much more powerful|

---

## Quick Reference

```python
from collections import Counter, defaultdict, OrderedDict, deque, namedtuple, ChainMap

# ══ COUNTER ══════════════════════════════════════════════════════

c = Counter('abracadabra')
c = Counter(['a','b','a','c'])
c = Counter(a=3, b=1)

c.most_common(3)        # [('a', 5), ('b', 2), ('r', 2)]
list(c.elements())      # ['a','a','a','a','a','b','b', ...]
c.total()               # sum of all counts (Python 3.10+)
c.update(other)         # add counts from other
c.subtract(other)       # subtract counts (allows negatives)
c['missing_key']        # 0  (no KeyError!)
c1 + c2                 # add counters
c1 - c2                 # subtract (positive counts only)
c1 & c2                 # intersection (min of each)
c1 | c2                 # union (max of each)


# ══ DEFAULTDICT ══════════════════════════════════════════════════

dd = defaultdict(list)       # missing key → []
dd = defaultdict(int)        # missing key → 0
dd = defaultdict(set)        # missing key → set()
dd = defaultdict(lambda: 'N/A')  # missing key → 'N/A'

dd['key'].append(1)          # auto-creates [] on first access
dd['count'] += 1             # auto-starts at 0


# ══ ORDEREDDICT ══════════════════════════════════════════════════

od = OrderedDict()
od['a'] = 1
od.move_to_end('a')          # move to back
od.move_to_end('a', last=False)  # move to front
od.popitem(last=True)        # pop last → (key, val)
od.popitem(last=False)       # pop first → (key, val)
# OD equality is ORDER-SENSITIVE


# ══ DEQUE ════════════════════════════════════════════════════════

d = deque([1, 2, 3])
d = deque(maxlen=5)          # bounded deque

d.append(x)                  # add right
d.appendleft(x)              # add left
d.pop()                      # remove right
d.popleft()                  # remove left
d.extend([4,5])              # add multiple right
d.extendleft([0,-1])         # add multiple left (reversed order!)
d.rotate(n)                  # shift right n steps (negative = left)
d.reverse()                  # reverse in-place
d.maxlen                     # None or the max size


# ══ NAMEDTUPLE ═══════════════════════════════════════════════════

Point = namedtuple('Point', ['x', 'y'])
Point = namedtuple('Point', 'x y')

p = Point(11, 22)
p = Point(x=11, y=22)
p.x                          # name access
p[0]                         # index access
x, y = p                     # unpacking

p._asdict()                  # → {'x': 11, 'y': 22}
p._replace(x=100)            # → new Point(x=100, y=22)
Point._make([10, 20])        # → Point(x=10, y=20)
Point._fields                # → ('x', 'y')

# With defaults (rightmost fields):
NT = namedtuple('NT', ['a','b','c'], defaults=[0, 0])


# ══ CHAINMAP ═════════════════════════════════════════════════════

cm = ChainMap(cli_args, env_vars, defaults)
cm['key']                    # searches maps in order, first found wins
cm['key'] = val              # writes to FIRST map ONLY
cm.maps                      # list of underlying dicts
cm.new_child({'k': 'v'})     # new ChainMap with fresh dict prepended
cm.parents                   # ChainMap without first map
```

---

## Debugging Common Bugs

**1. Expecting `defaultdict` to behave like a plain dict when checking membership:**

```python
dd = defaultdict(list)
# ❌ This CREATES the key with an empty list just by checking!
if dd['new_key']:
    print("has items")
# After this, dd has {'new_key': []}  — probably not what you wanted

# ✅ Use 'in' to check without creating:
if 'new_key' in dd:
    print("exists")
```

**2. `Counter` arithmetic dropping negatives:**

```python
c = Counter(a=2) - Counter(a=5)
print(c)   # Counter()  ← 'a' with count -3 is DROPPED by - operator

# ✅ Use subtract() to keep negatives:
c = Counter(a=2)
c.subtract(Counter(a=5))
print(c)   # Counter({'a': -3})  ← kept
```

**3. `deque.extendleft()` reverses order:**

```python
d = deque([3, 4, 5])
d.extendleft([0, 1, 2])   # ❌ NOT [0,1,2,3,4,5]
print(d)   # deque([2, 1, 0, 3, 4, 5])
# Because each element is individually prepended: 0 then 1 (pushes 0 right), then 2 (pushes both right)

# ✅ To prepend in original order:
d.extendleft(reversed([0, 1, 2]))
```

**4. `namedtuple` fields are immutable — use `_replace()` to "change":**

```python
Point = namedtuple('Point', ['x', 'y'])
p = Point(1, 2)
# p.x = 10  # ❌ AttributeError: can't set attribute

# ✅ Create a new namedtuple:
p = p._replace(x=10)
```

**5. `ChainMap` writes only go to the first map:**

```python
a = {'x': 1}
b = {'y': 2}
cm = ChainMap(a, b)
cm['y'] = 99   # ❌ does NOT update b!
print(b)       # {'y': 2}  — unchanged
print(a)       # {'x': 1, 'y': 99}  — written to first map!

# ✅ If you want to update b, access it directly:
cm.maps[1]['y'] = 99
```

**6. `OrderedDict` equality vs plain dict equality:**

```python
from collections import OrderedDict

od1 = OrderedDict([('a', 1), ('b', 2)])
od2 = OrderedDict([('b', 2), ('a', 1)])

od1 == od2             # False ← OrderedDict cares about order
dict(od1) == dict(od2) # True  ← plain dict doesn't care about order
```

---

## Practice Exercises

**Counter:**

- [ ] Count the frequency of each character in a string. Print the top 5 most common.
- [ ] Given two strings, use `Counter` arithmetic to find characters present in the first but not the second.
- [ ] Read a text file and print the 10 most frequently used words (case-insensitive).
- [ ] Check if two strings are anagrams using `Counter`.

**defaultdict:**

- [ ] Group a list of `(name, score)` tuples by name into `{name: [scores]}` using `defaultdict(list)`.
- [ ] Count word frequencies in a sentence using `defaultdict(int)`.
- [ ] Build a graph (adjacency list) from a list of edges using `defaultdict(set)`.
- [ ] Build a nested `defaultdict` to count 2-word phrases (bigrams).

**OrderedDict:**

- [ ] Implement a simple LRU cache using `OrderedDict`.
- [ ] Read a CSV and maintain a dict of values, then `move_to_end()` to reorder by a priority column.

**deque:**

- [ ] Implement a queue (FIFO) using `deque`. Enqueue 5 tasks, process them in order.
- [ ] Implement a stack (LIFO) with `deque` to simulate browser back-button history.
- [ ] Write a function that computes the rolling average of the last N values in a stream using `deque(maxlen=N)`.
- [ ] Use `rotate()` to implement a round-robin scheduler.

**namedtuple:**

- [ ] Create a `Student` namedtuple with fields `name`, `roll`, `grade`. Create 3 instances and print them.
- [ ] Read a CSV of employee data using `_make()`.
- [ ] Create a `Color` namedtuple with `red`, `green`, `blue` fields. Write a function that converts a hex colour string to a `Color`.
- [ ] Use `_replace()` to "update" an immutable `Config` namedtuple.

**ChainMap:**

- [ ] Simulate a config system: `defaults` → `config_file` → `env_vars` → `cli_args`. Look up various keys to see which layer wins.
- [ ] Use `new_child()` to simulate entering a new scope in a simple interpreter.

---

## Questions I Still Have

_Write your open questions here. Return later and answer them._

---

## Related Notes

- [[OBJECT-ORIENTED PROGRAMMING (OOP)]]
- [[DICTIONARIES, TUPLES & SETS]]
- [[REGEX]]
- [[FUNCTIONS]]
- [[ITERATIVE STATEMENTS]]
- [[LIST]]