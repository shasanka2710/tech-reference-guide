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
