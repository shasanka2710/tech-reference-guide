# 🐍 Python Quick Reference

## Data Types

```python
# Numeric
x = 10          # int
y = 3.14        # float
z = 2 + 3j      # complex

# String
s = "hello"
s = 'world'
s = """multi
line"""

# Boolean
b = True
b = False

# None
n = None
```

## Collections

```python
# List (mutable, ordered)
lst = [1, 2, 3]
lst.append(4)
lst.extend([5, 6])
lst.pop()           # removes last
lst.remove(2)       # removes first occurrence of value
lst[1:3]            # slicing

# Tuple (immutable, ordered)
t = (1, 2, 3)

# Set (mutable, unordered, unique)
s = {1, 2, 3}
s.add(4)
s.discard(2)

# Dictionary (mutable, ordered key-value)
d = {"key": "value", "num": 42}
d["new_key"] = "new_value"
d.get("key", "default")
d.keys(), d.values(), d.items()
```

## Control Flow

```python
# if / elif / else
if x > 0:
    print("positive")
elif x == 0:
    print("zero")
else:
    print("negative")

# for loop
for i in range(5):          # 0,1,2,3,4
    print(i)

for item in lst:
    print(item)

for i, item in enumerate(lst):
    print(i, item)

# while loop
while condition:
    do_something()

# List comprehension
squares = [x**2 for x in range(10)]
evens   = [x for x in range(20) if x % 2 == 0]

# Dictionary comprehension
d = {k: v for k, v in pairs}
```

## Functions

```python
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

# *args and **kwargs
def func(*args, **kwargs):
    print(args)    # tuple
    print(kwargs)  # dict

# Lambda
square = lambda x: x * x

# Generators
def gen():
    yield 1
    yield 2
```

## Classes & OOP

```python
class Animal:
    CLASS_VAR = "animal"

    def __init__(self, name):
        self.name = name          # instance variable

    def speak(self):
        return f"{self.name} makes a sound"

    @classmethod
    def from_string(cls, name_str):
        return cls(name_str)

    @staticmethod
    def is_alive():
        return True


class Dog(Animal):               # Inheritance
    def speak(self):             # Override
        return f"{self.name} barks"


d = Dog("Rex")
print(d.speak())
```

## Exception Handling

```python
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Error: {e}")
except (TypeError, ValueError):
    pass
else:
    print("No error occurred")
finally:
    print("Always runs")

# Raise
raise ValueError("Invalid input")
```

## File I/O

```python
# Read
with open("file.txt", "r") as f:
    content = f.read()
    lines = f.readlines()

# Write
with open("file.txt", "w") as f:
    f.write("Hello\n")

# Append
with open("file.txt", "a") as f:
    f.write("More text\n")
```

## Useful Built-ins

```python
len(x)           # length
type(x)          # type
isinstance(x, T) # type check
range(start, stop, step)
zip(a, b)        # pair iterables
map(func, iterable)
filter(func, iterable)
sorted(lst, key=lambda x: x, reverse=True)
min(lst), max(lst), sum(lst)
any(lst), all(lst)
```

## String Methods

```python
s.upper(), s.lower(), s.title()
s.strip(), s.lstrip(), s.rstrip()
s.split(",")
",".join(lst)
s.replace("old", "new")
s.startswith("pre"), s.endswith("suf")
s.find("sub")        # returns index or -1
f"Hello, {name}!"    # f-string (Python 3.6+)
```

## Modules & Packages

```python
import os
import sys
from pathlib import Path
from collections import defaultdict, Counter, deque
from itertools import product, combinations, permutations
import json, re, datetime, math, random
```

## Common Patterns

```python
# Swap
a, b = b, a

# Ternary
val = x if condition else y

# Unpacking
a, b, *rest = [1, 2, 3, 4, 5]

# Context manager
with open(...) as f:
    ...

# Decorators
def decorator(func):
    def wrapper(*args, **kwargs):
        print("before")
        result = func(*args, **kwargs)
        print("after")
        return result
    return wrapper

@decorator
def my_function():
    pass
```

---

## Iterators & Generators

```python
# Iterator protocol
class Counter:
    def __init__(self, low, high):
        self.current = low
        self.high = high

    def __iter__(self):       # makes object iterable
        return self

    def __next__(self):       # returns next value
        if self.current > self.high:
            raise StopIteration
        self.current += 1
        return self.current - 1

for n in Counter(1, 3):
    print(n)  # 1, 2, 3

# Generator function — yields values lazily
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

fib = fibonacci()
print(next(fib))  # 0
print(next(fib))  # 1
print(next(fib))  # 1

# Generator expression (lazy, memory-efficient)
gen = (x**2 for x in range(1_000_000))

# send() — coroutine-style generator
def accumulator():
    total = 0
    while True:
        value = yield total
        if value is None:
            break
        total += value

acc = accumulator()
next(acc)          # prime the generator
acc.send(10)       # total = 10
acc.send(20)       # total = 30

# yield from — delegate to sub-generator
def chain(*iterables):
    for it in iterables:
        yield from it

list(chain([1, 2], [3, 4]))  # [1, 2, 3, 4]
```

**Interview notes:**
- Generators are memory-efficient; they don't build the full list in memory.
- `__iter__` + `__next__` = iterator protocol.
- `yield` suspends execution and saves state.

---

## Decorators (Advanced)

```python
import functools

# Preserving metadata with functools.wraps
def log(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@log
def add(a, b):
    return a + b

# Decorator with arguments (factory pattern)
def repeat(n):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(n):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def hello():
    print("Hello!")

# Class-based decorator
class Timer:
    def __init__(self, func):
        functools.update_wrapper(self, func)
        self.func = func

    def __call__(self, *args, **kwargs):
        import time
        start = time.perf_counter()
        result = self.func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{self.func.__name__} took {elapsed:.4f}s")
        return result

@Timer
def slow_function():
    import time; time.sleep(0.1)

# Stacking decorators — applied bottom-up
@log
@repeat(2)
def greet(name):
    print(f"Hi {name}")
```

**Interview notes:**
- Without `@functools.wraps`, `func.__name__` and `__doc__` are lost.
- Decorators are applied bottom-up: `@A @B def f` → `A(B(f))`.

---

## Context Managers

```python
# Custom context manager via class
class ManagedFile:
    def __init__(self, path, mode="r"):
        self.path = path
        self.mode = mode

    def __enter__(self):
        self.file = open(self.path, self.mode)
        return self.file

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.file.close()
        return False   # False re-raises exceptions; True suppresses them

with ManagedFile("data.txt") as f:
    content = f.read()

# contextlib.contextmanager — generator-based
from contextlib import contextmanager

@contextmanager
def timer():
    import time
    start = time.perf_counter()
    yield
    print(f"Elapsed: {time.perf_counter() - start:.4f}s")

with timer():
    sum(range(10_000_000))

# contextlib.suppress — silently ignore specific exceptions
from contextlib import suppress

with suppress(FileNotFoundError):
    open("missing.txt")
```

**Interview notes:**
- `__exit__` receives exception info; returning `True` suppresses the exception.
- `contextlib.contextmanager` converts a generator into a context manager.

---

## OOP — Dunder / Magic Methods

```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y

    # String representations
    def __repr__(self):          # unambiguous, for devs
        return f"Vector({self.x}, {self.y})"

    def __str__(self):           # readable, for users
        return f"({self.x}, {self.y})"

    # Arithmetic operators
    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    def __mul__(self, scalar):
        return Vector(self.x * scalar, self.y * scalar)

    def __rmul__(self, scalar):  # scalar * vector
        return self.__mul__(scalar)

    # Comparison
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

    def __lt__(self, other):
        return abs(self) < abs(other)

    # Length / absolute value
    def __abs__(self):
        return (self.x**2 + self.y**2) ** 0.5

    def __len__(self):
        return 2

    # Container behaviour
    def __getitem__(self, idx):
        return (self.x, self.y)[idx]

    def __iter__(self):
        yield self.x
        yield self.y

    # Boolean
    def __bool__(self):
        return bool(self.x or self.y)

    # Callable
    def __call__(self, scale):
        return Vector(self.x * scale, self.y * scale)

v1 = Vector(1, 2)
v2 = Vector(3, 4)
print(v1 + v2)         # Vector(4, 6)
print(repr(v1))        # Vector(1, 2)
print(3 * v1)          # Vector(3, 6)
```

**Key dunder methods cheatsheet:**

| Method | Triggered by |
|--------|-------------|
| `__init__` | `MyClass()` |
| `__repr__` / `__str__` | `repr()` / `str()` |
| `__len__` | `len()` |
| `__getitem__` | `obj[key]` |
| `__setitem__` | `obj[key] = val` |
| `__contains__` | `item in obj` |
| `__enter__` / `__exit__` | `with` statement |
| `__iter__` / `__next__` | `for` loop / `next()` |
| `__call__` | `obj()` |
| `__eq__`, `__lt__`, etc. | `==`, `<`, etc. |
| `__hash__` | `hash()`, dict keys, sets |

---

## Properties & `__slots__`

```python
class Circle:
    __slots__ = ("_radius",)   # restricts instance attributes; saves memory

    def __init__(self, radius):
        self._radius = radius

    @property
    def radius(self):
        return self._radius

    @radius.setter
    def radius(self, value):
        if value < 0:
            raise ValueError("Radius cannot be negative")
        self._radius = value

    @radius.deleter
    def radius(self):
        del self._radius

    @property
    def area(self):
        import math
        return math.pi * self._radius ** 2

c = Circle(5)
print(c.radius)    # 5
c.radius = 10
print(c.area)      # 314.16...
```

**Interview notes:**
- `@property` enables attribute-style access with validation logic.
- `__slots__` prevents creation of `__dict__`, reducing memory per instance.

---

## Multiple Inheritance & MRO

```python
class A:
    def method(self): print("A")

class B(A):
    def method(self): print("B")

class C(A):
    def method(self): print("C")

class D(B, C):       # Diamond inheritance
    pass

d = D()
d.method()           # B — follows MRO

print(D.__mro__)
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)

# super() in cooperative multiple inheritance
class Base:
    def greet(self):
        print("Base")

class Left(Base):
    def greet(self):
        super().greet()
        print("Left")

class Right(Base):
    def greet(self):
        super().greet()
        print("Right")

class Child(Left, Right):
    def greet(self):
        super().greet()
        print("Child")

Child().greet()
# Prints: Base → Right → Left → Child  (MRO order)
```

**Interview notes:**
- Python uses **C3 linearisation** to compute MRO.
- `super()` follows MRO, not just the direct parent — essential for cooperative multiple inheritance.

---

## Abstract Classes

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self) -> float:
        """Return the area of the shape."""

    @abstractmethod
    def perimeter(self) -> float:
        """Return the perimeter."""

    def describe(self):           # concrete method
        return f"Area={self.area():.2f}, Perimeter={self.perimeter():.2f}"

class Rectangle(Shape):
    def __init__(self, w, h):
        self.w, self.h = w, h

    def area(self):
        return self.w * self.h

    def perimeter(self):
        return 2 * (self.w + self.h)

# Shape()         # TypeError: Can't instantiate abstract class
r = Rectangle(3, 4)
print(r.describe())   # Area=12.00, Perimeter=14.00
```

---

## Type Hints & Annotations

```python
# Built-in types
def greet(name: str) -> str:
    return f"Hello, {name}"

# from typing (Python 3.9+: use built-ins directly; 3.8- use typing)
from typing import Optional, Union, List, Dict, Tuple, Callable, Any

def find(lst: list[int], value: int) -> Optional[int]:
    try:
        return lst.index(value)
    except ValueError:
        return None

# Generic type variables
from typing import TypeVar, Generic
T = TypeVar("T")

class Stack(Generic[T]):
    def __init__(self) -> None:
        self._items: list[T] = []

    def push(self, item: T) -> None:
        self._items.append(item)

    def pop(self) -> T:
        return self._items.pop()

# Callable type
Transformer = Callable[[int], int]

def apply(fn: Transformer, value: int) -> int:
    return fn(value)

# TypedDict
from typing import TypedDict

class Movie(TypedDict):
    title: str
    year: int

# Protocol (structural subtyping / duck typing)
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None: ...

def render(obj: Drawable) -> None:
    obj.draw()
```

---

## Dataclasses

```python
from dataclasses import dataclass, field, asdict, astuple

@dataclass
class Point:
    x: float
    y: float
    label: str = "point"                    # default value
    tags: list[str] = field(default_factory=list)  # mutable default

    def distance_to_origin(self) -> float:
        return (self.x**2 + self.y**2) ** 0.5

p = Point(3.0, 4.0)
print(p)                   # Point(x=3.0, y=4.0, label='point', tags=[])
print(p.distance_to_origin())  # 5.0
print(asdict(p))           # {'x': 3.0, 'y': 4.0, 'label': 'point', 'tags': []}

# Frozen (immutable) dataclass — also hashable
@dataclass(frozen=True)
class ImmutablePoint:
    x: float
    y: float

# Order comparison
@dataclass(order=True)
class Student:
    gpa: float
    name: str = field(compare=False)
```

**Interview notes:**
- `@dataclass` auto-generates `__init__`, `__repr__`, `__eq__`.
- Use `frozen=True` for immutable / hashable instances.
- Use `field(default_factory=...)` for mutable defaults (never use mutable default directly).

---

## Functional Programming

```python
from functools import reduce, partial, lru_cache, cache

# reduce
from functools import reduce
product = reduce(lambda acc, x: acc * x, [1, 2, 3, 4, 5])  # 120

# partial — fix some arguments
def power(base, exp):
    return base ** exp

square = partial(power, exp=2)
cube   = partial(power, exp=3)
print(square(5))   # 25

# lru_cache — memoisation
@lru_cache(maxsize=None)
def fib(n):
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)

print(fib(50))     # fast!
fib.cache_info()   # CacheInfo(hits=48, misses=51, maxsize=None, currsize=51)

# cache (Python 3.9+) — equivalent to lru_cache(maxsize=None)
@cache
def factorial(n):
    return 1 if n == 0 else n * factorial(n - 1)

# map / filter / zip
nums = [1, 2, 3, 4, 5]
doubled  = list(map(lambda x: x * 2, nums))
evens    = list(filter(lambda x: x % 2 == 0, nums))
pairs    = list(zip("abc", [1, 2, 3]))   # [('a',1), ('b',2), ('c',3)]

# itertools
from itertools import chain, islice, groupby, accumulate, product

list(accumulate([1, 2, 3, 4]))   # [1, 3, 6, 10] — running totals
list(islice(range(100), 5))      # [0, 1, 2, 3, 4]

data = [{"dept": "eng", "name": "Alice"},
        {"dept": "eng", "name": "Bob"},
        {"dept": "hr",  "name": "Carol"}]
for dept, members in groupby(data, key=lambda x: x["dept"]):
    print(dept, list(members))
```

---

## Comprehensions — Full Picture

```python
# List comprehension
squares = [x**2 for x in range(10)]

# Nested list comprehension
matrix = [[i * j for j in range(1, 4)] for i in range(1, 4)]

# Set comprehension
unique_lengths = {len(word) for word in ["apple", "pear", "plum"]}

# Dictionary comprehension
inv = {v: k for k, v in {"a": 1, "b": 2}.items()}

# Generator expression (lazy — no list built)
total = sum(x**2 for x in range(1_000_000))

# Conditional comprehension
result = ["even" if x % 2 == 0 else "odd" for x in range(6)]
```

---

## Regular Expressions

```python
import re

text = "Order #1234 placed on 2024-01-15, total: $99.99"

# Search — first match anywhere
m = re.search(r"\d{4}-\d{2}-\d{2}", text)
if m:
    print(m.group())   # 2024-01-15

# Match — only at start of string
re.match(r"Order", text)

# Findall — list of all matches
re.findall(r"\$[\d.]+", text)   # ['$99.99']

# Finditer — iterator of match objects
for m in re.finditer(r"\d+", text):
    print(m.group(), m.start(), m.end())

# Sub — replace
cleaned = re.sub(r"\s+", " ", "too   many   spaces")

# Split
parts = re.split(r"[,\s]+", "one, two,three four")

# Compiled pattern (reuse for performance)
pattern = re.compile(r"(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})")
m = pattern.search(text)
print(m.group("year"), m.group("month"))   # 2024 01

# Common flags
re.IGNORECASE  # or re.I
re.MULTILINE   # ^ and $ match line boundaries
re.DOTALL      # . matches newline too
```

**Key regex tokens:**

| Token | Matches |
|-------|---------|
| `.` | Any character except newline |
| `\d` / `\D` | Digit / non-digit |
| `\w` / `\W` | Word char / non-word |
| `\s` / `\S` | Whitespace / non-whitespace |
| `^` / `$` | Start / end of string |
| `*` / `+` / `?` | 0+, 1+, 0 or 1 |
| `{m,n}` | m to n repetitions |
| `[abc]` | Character class |
| `(...)` | Capturing group |
| `(?:...)` | Non-capturing group |
| `(?P<name>...)` | Named group |

---

## Collections Module

```python
from collections import (
    defaultdict, Counter, deque,
    OrderedDict, namedtuple, ChainMap
)

# defaultdict — avoids KeyError, supplies default
word_count = defaultdict(int)
for word in "the quick brown fox".split():
    word_count[word] += 1

graph = defaultdict(list)
graph["A"].append("B")

# Counter — count hashable objects
c = Counter("mississippi")
print(c.most_common(3))   # [('s', 4), ('i', 4), ('p', 2)]
c1 = Counter("aab"); c2 = Counter("abb")
print(c1 + c2)            # Counter({'b': 3, 'a': 3})
print(c1 & c2)            # intersection: Counter({'a': 1, 'b': 1})

# deque — O(1) append/pop from both ends
dq = deque([1, 2, 3], maxlen=5)
dq.appendleft(0)   # [0, 1, 2, 3]
dq.rotate(1)       # [3, 0, 1, 2]

# namedtuple — immutable, lightweight record
Point = namedtuple("Point", ["x", "y"])
p = Point(3, 4)
print(p.x, p.y)    # 3 4
print(p._asdict()) # {'x': 3, 'y': 4}  (plain dict in Python 3.8+)

# ChainMap — combine multiple dicts, first match wins
defaults = {"color": "red", "user": "guest"}
env      = {"user": "alice"}
config   = ChainMap(env, defaults)
print(config["color"])  # red
print(config["user"])   # alice
```

---

## Concurrency

### Threading

```python
import threading

results = []
lock = threading.Lock()

def worker(n):
    with lock:
        results.append(n * n)

threads = [threading.Thread(target=worker, args=(i,)) for i in range(5)]
for t in threads: t.start()
for t in threads: t.join()

print(sorted(results))  # [0, 1, 4, 9, 16]

# Thread-safe queue
from queue import Queue

q = Queue()
q.put("task1")
item = q.get()
q.task_done()
q.join()           # block until all tasks done
```

### Multiprocessing

```python
from multiprocessing import Pool, Process, Queue as MPQueue

def square(x):
    return x * x

# Pool.map — parallel map
with Pool(processes=4) as pool:
    results = pool.map(square, range(10))

# Process
p = Process(target=square, args=(5,))
p.start()
p.join()
```

### asyncio

```python
import asyncio

async def fetch(url: str) -> str:
    await asyncio.sleep(1)   # simulate I/O
    return f"data from {url}"

async def main():
    # Run concurrently with gather
    results = await asyncio.gather(
        fetch("http://api1.com"),
        fetch("http://api2.com"),
    )
    print(results)

asyncio.run(main())

# Async context manager & iterator
class AsyncDB:
    async def __aenter__(self):
        print("connect")
        return self

    async def __aexit__(self, *args):
        print("disconnect")

    async def __aiter__(self):
        for row in range(3):
            await asyncio.sleep(0)
            yield row

async def use_db():
    async with AsyncDB() as db:
        async for row in db:
            print(row)
```

**Concurrency model comparison:**

| Model | Best for | GIL impact |
|-------|----------|------------|
| `threading` | I/O-bound tasks | GIL limits CPU parallelism |
| `multiprocessing` | CPU-bound tasks | Each process has own GIL |
| `asyncio` | Many concurrent I/O tasks | Single-threaded, cooperative |

---

## Memory Management & the GIL

```python
# Reference counting
import sys
x = [1, 2, 3]
print(sys.getrefcount(x))   # usually count + 1 (getrefcount arg)

# Garbage collector (handles cycles)
import gc
gc.collect()                # force a collection cycle
gc.get_count()              # (gen0, gen1, gen2) object counts

# id() and is vs ==
a = [1, 2, 3]
b = a           # same object
c = [1, 2, 3]  # equal but different object
print(a is b)   # True  — same identity
print(a is c)   # False — different identity
print(a == c)   # True  — same value

# Small integer caching (-5 to 256 are interned)
x = 256; y = 256
print(x is y)   # True   (cached)
x = 257; y = 257
print(x is y)   # False  (CPython implementation detail)

# String interning
a = "hello"
b = "hello"
print(a is b)   # True — short string literals are interned

# __slots__ reduces memory
import tracemalloc
tracemalloc.start()
# ... run code ...
snapshot = tracemalloc.take_snapshot()
```

**GIL (Global Interpreter Lock):**
- Only **one thread** executes Python bytecode at a time in CPython.
- Does **not** affect `multiprocessing` (separate processes).
- Does **not** block I/O-bound `threading` (GIL released during I/O).
- Alternatives: `PyPy`, `Jython`, Python 3.13 experimental no-GIL build.

---

## Python Internals & Scoping (LEGB)

```python
x = "global"

def outer():
    x = "enclosing"

    def inner():
        x = "local"
        print(x)        # local

    def inner_nonlocal():
        nonlocal x      # binds to enclosing x
        x = "modified enclosing"

    inner()
    inner_nonlocal()
    print(x)            # modified enclosing

outer()
print(x)                # global

# global keyword
count = 0
def increment():
    global count
    count += 1
```

**LEGB rule:** Python looks up names in this order:
1. **L**ocal — inside the current function
2. **E**nclosing — enclosing function scopes (closures)
3. **G**lobal — module-level
4. **B**uilt-in — `builtins` module (`len`, `print`, …)

```python
# Closures — function + enclosing scope
def make_multiplier(n):
    def multiply(x):
        return x * n     # n is a free variable from enclosing scope
    return multiply

double = make_multiplier(2)
print(double(5))   # 10

# Inspect closure
print(double.__closure__[0].cell_contents)  # 2
```

---

## Custom Exceptions

```python
class AppError(Exception):
    """Base exception for this application."""

class ValidationError(AppError):
    def __init__(self, field: str, message: str):
        self.field = field
        self.message = message
        super().__init__(f"[{field}] {message}")

class NotFoundError(AppError):
    def __init__(self, resource: str, resource_id):
        self.resource = resource
        self.resource_id = resource_id
        super().__init__(f"{resource} with id={resource_id} not found")

# Raise and chain
try:
    raise ValidationError("email", "Invalid format")
except ValidationError as e:
    print(e.field, e.message)
    raise NotFoundError("User", 42) from e   # __cause__ chaining
```

---

## Testing

```python
# unittest
import unittest

class TestMath(unittest.TestCase):
    def setUp(self):               # runs before each test
        self.data = [1, 2, 3]

    def tearDown(self):            # runs after each test
        pass

    def test_sum(self):
        self.assertEqual(sum(self.data), 6)

    def test_empty(self):
        self.assertEqual(sum([]), 0)

    def test_raises(self):
        with self.assertRaises(ZeroDivisionError):
            1 / 0

    def test_approx(self):
        self.assertAlmostEqual(3.14159, 3.14, places=2)

if __name__ == "__main__":
    unittest.main()

# pytest (preferred)
# Run: pytest -v tests/

def test_sum():
    assert sum([1, 2, 3]) == 6

def test_type():
    assert isinstance("hello", str)

# pytest fixtures
import pytest

@pytest.fixture
def sample_list():
    return [1, 2, 3, 4, 5]

def test_length(sample_list):
    assert len(sample_list) == 5

# Parametrize
@pytest.mark.parametrize("n,expected", [
    (0, 1),
    (1, 1),
    (5, 120),
])
def test_factorial(n, expected):
    from math import factorial
    assert factorial(n) == expected

# Mocking
from unittest.mock import MagicMock, patch

def test_mock():
    mock = MagicMock(return_value=42)
    assert mock() == 42

@patch("os.listdir", return_value=["a.py", "b.py"])
def test_listdir(mock_ls):
    import os
    assert len(os.listdir(".")) == 2
```

---

## Miscellaneous Advanced Topics

### Metaclasses

```python
# A metaclass controls class creation
class SingletonMeta(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=SingletonMeta):
    def __init__(self):
        self.connection = "connected"

db1 = Database()
db2 = Database()
print(db1 is db2)   # True
```

### Descriptors

```python
class Positive:
    """Descriptor that enforces positive numbers."""
    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return obj.__dict__.get(self.name)

    def __set__(self, obj, value):
        if value <= 0:
            raise ValueError(f"{self.name} must be positive")
        obj.__dict__[self.name] = value

class Product:
    price = Positive()
    quantity = Positive()

    def __init__(self, price, quantity):
        self.price = price
        self.quantity = quantity

p = Product(10.0, 5)
# p.price = -1   # raises ValueError
```

### `__init_subclass__` & Class Registration

```python
class Plugin:
    _registry: dict = {}

    def __init_subclass__(cls, plugin_name: str, **kwargs):
        super().__init_subclass__(**kwargs)
        Plugin._registry[plugin_name] = cls

class CSVPlugin(Plugin, plugin_name="csv"):
    pass

class JSONPlugin(Plugin, plugin_name="json"):
    pass

print(Plugin._registry)  # {'csv': <class 'CSVPlugin'>, 'json': <class 'JSONPlugin'>}
```

---

## Interview Quick-Reference

### Mutability

| Type | Mutable | Hashable |
|------|---------|----------|
| `int`, `float`, `bool` | ✗ | ✓ |
| `str` | ✗ | ✓ |
| `tuple` (of hashable) | ✗ | ✓ |
| `list` | ✓ | ✗ |
| `dict` | ✓ | ✗ |
| `set` | ✓ | ✗ |
| `frozenset` | ✗ | ✓ |

### Time Complexity of Built-in Operations

| Operation | list | dict / set |
|-----------|------|-----------|
| `x in c` | O(n) | O(1) average |
| `c[i]` | O(1) | O(1) average |
| `append` / `add` | O(1) amortised | O(1) amortised |
| `insert(0, x)` | O(n) | — |
| `pop()` (end) | O(1) | O(1) |
| `pop(0)` / `popleft` | O(n) / O(1)* | — |
| `sort` | O(n log n) | — |

*Use `collections.deque` for O(1) pops from the left.

### Common Gotchas

```python
# 1. Mutable default argument
def bad_append(item, lst=[]):     # lst is shared across calls!
    lst.append(item)
    return lst

def good_append(item, lst=None):  # correct pattern
    if lst is None:
        lst = []
    lst.append(item)
    return lst

# 2. Late binding in closures
fns = [lambda: i for i in range(3)]
print([f() for f in fns])   # [2, 2, 2] — all capture the same i

fns = [lambda i=i: i for i in range(3)]
print([f() for f in fns])   # [0, 1, 2] — bind at definition time

# 3. is vs == for None
x = None
if x is None:      # correct
    pass
if x == None:      # works but not idiomatic (custom __eq__ on other types can interfere)
    pass

# 4. Chained comparison
print(1 < 2 < 3)   # True — Python supports chained comparisons

# 5. Unpacking in assignments
a, *b, c = range(5)
print(a, b, c)     # 0 [1, 2, 3] 4

# 6. dict.get vs dict[]
d = {"key": "value"}
d.get("missing", "default")   # safe — returns "default"
# d["missing"]                # raises KeyError
```

### Useful One-Liners

```python
# Flatten a list
flat = [x for sub in [[1,2],[3,4],[5]] for x in sub]

# Frequency count
from collections import Counter
freq = Counter("banana")

# Transpose matrix
matrix = [[1,2,3],[4,5,6]]
transposed = list(zip(*matrix))

# Merge dicts (Python 3.9+)
merged = {**dict1, **dict2}
merged = dict1 | dict2

# All unique?
all_unique = len(lst) == len(set(lst))

# Safe division
result = numerator / denominator if denominator else 0

# Reverse a string
rev = s[::-1]

# Check palindrome
is_palindrome = s == s[::-1]
```
