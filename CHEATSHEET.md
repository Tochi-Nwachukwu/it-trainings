# 📌 Python Master Cheat Sheet

> Everything from Lessons 11–21 on one page. Print it. Pin it up.

---

## Basics

```python
print("text")                     # output
name = input("Prompt: ")          # input — ALWAYS returns a string
age = int(input("Age: "))         # convert it!

# f-strings
print(f"{name} is {age}")
print(f"{pi:.2f}")                # 2 decimal places
print(f"{n:,}")                   # 1,234,567
print(f"{s:>10}{s:<10}{s:^10}")   # right / left / centre

# Types
str  int  float  bool  None
type(x)
int("5")   float("5.5")   str(5)   bool(0)
round(3.14159, 2)
len(x)

# Comments
# single line
"""multi-line docstring"""
```

## Operators

```python
+  -  *  /        # / always gives a float
//                # floor division (whole number)
%                 # remainder — n % 2 == 0 tests for even
**                # power

==  !=  >  <  >=  <=
and  or  not
in   not in
is                # identity — only use with None
```

---

## Control Flow

```python
if condition:
    ...
elif other:
    ...
else:
    ...

value = "a" if condition else "b"          # ternary

match variable:
    case "x":        ...
    case "y" | "z":  ...
    case _:          ...                    # catch-all

# Loops
for i in range(5):          ...             # 0,1,2,3,4
for i in range(1, 6):       ...             # 1,2,3,4,5
for i in range(0, 20, 5):   ...             # 0,5,10,15
for i in range(5, 0, -1):   ...             # 5,4,3,2,1
for item in my_list:        ...
for ch in "text":           ...
for i, x in enumerate(lst, start=1): ...
for k, v in my_dict.items(): ...

while condition:  ...                       # must change the condition!
while True:
    if done: break                          # sentinel

break     # leave the loop
continue  # skip to the next iteration
pass      # do nothing (placeholder)
```

---

## Lists & Tuples

```python
lst = [1, 2, 3]
tup = (1, 2, 3)
one_item = (5,)                            # trailing comma required!

lst[0]   lst[-1]   lst[1:3]   lst[:2]   lst[2:]   lst[::-1]

lst.append(x)       lst.insert(i, x)      lst.extend([a,b])
lst.remove(value)   lst.pop()             lst.pop(i)
del lst[i]          lst.clear()

len(lst)  max(lst)  min(lst)  sum(lst)
lst.index(x)   lst.count(x)   x in lst

sorted(lst)                # NEW list
sorted(lst, reverse=True)
sorted(lst, key=lambda p: p[1])
lst.sort()                 # IN PLACE
lst.reverse()

# Comprehensions
[x*2 for x in lst]
[x for x in lst if x > 5]
["a" if x else "b" for x in lst]
[n for row in matrix for n in row]         # flatten

# ⚠️ Copying
b = a           # ALIAS — both change together!
b = a.copy()    # real copy

x, y = (3, 4)   # unpacking
a, b = b, a     # swap
```

---

## Dictionaries & Sets

```python
d = {"key": "value"}
d = {}                                     # empty dict

d["key"]                                   # crashes if missing
d.get("key")                               # safe → None
d.get("key", "default")

d["new"] = value
d.update({"a": 1})
del d["key"]
d.pop("key")

d.keys()   d.values()   d.items()
for k, v in d.items(): ...
"key" in d                                 # checks KEYS

# The counting idiom — memorise this
counts[item] = counts.get(item, 0) + 1

{x: x**2 for x in range(5)}                # dict comprehension

# --- SETS ---
s = {1, 2, 3}
s = set([1, 2, 2])                         # dedupe
empty = set()                              # NOT {} — that's a dict!

s.add(x)   s.discard(x)   s.remove(x)

a | b    # union         — either
a & b    # intersection  — both
a - b    # difference    — in a, not b
a ^ b    # symmetric     — one but not both

list(set(my_list))                         # remove duplicates
```

---

## Functions

```python
def name(param, default=10):
    """Docstring."""
    return value

def typed(w: float, h: float) -> float:    # type hints
    return w * h

def any_args(*args):        ...            # → tuple
def any_kwargs(**kwargs):   ...            # → dict

return a, b, c                             # returns a tuple
x, y, z = func()                           # unpack

global counter                             # modify a global (avoid)

# Lambdas
lambda x: x * 2
sorted(lst, key=lambda p: p[1])
list(map(lambda x: x*2, lst))
list(filter(lambda x: x > 5, lst))

# Recursion — ALWAYS needs a base case
def f(n):
    if n <= 1: return 1                    # BASE CASE
    return n * f(n - 1)
```

---

## Modules & Packages

```python
import math
from math import sqrt, pi
import numpy as np

if __name__ == "__main__":                 # only runs if executed directly
    ...

# Standard library
import math      # sqrt ceil floor pi gcd
import random    # randint choice shuffle sample
import os        # getcwd listdir path.exists makedirs
import sys       # argv version exit
import json      # dumps loads dump load
from datetime import datetime, timedelta
from collections import Counter
```

```bash
python3 -m venv venv
source venv/bin/activate       # Windows: venv\Scripts\activate
deactivate

pip install package
pip freeze > requirements.txt
pip install -r requirements.txt
```

---

## Files & Errors

```python
with open("file.txt") as f:                # read
    text = f.read()
    lines = f.readlines()
    for line in f: ...                     # best for big files

with open("file.txt", "w") as f:           # WRITE — erases first!
    f.write("text\n")

with open("file.txt", "a") as f:           # APPEND — safe
    f.write("more\n")

line.strip()                               # remove trailing \n

# JSON
import json
json.dump(data, file, indent=2)            # dict → file
data = json.load(file)                     # file → dict
json.dumps(data)                           # dict → string
json.loads(text)                           # string → dict

# CSV
import csv
csv.writer(f).writerows(rows)
for row in csv.DictReader(f): row["col"]

# Errors
try:
    risky()
except ValueError as e:
    print(e)
except (TypeError, KeyError):
    ...
else:
    ...                                    # no error happened
finally:
    ...                                    # always runs

raise ValueError("message")

class MyError(Exception):
    pass

# Common exceptions
ValueError  TypeError  ZeroDivisionError
FileNotFoundError  KeyError  IndexError  PermissionError
```

---

## Classes

```python
class ClassName:
    class_var = "shared"                   # class variable

    def __init__(self, arg):
        self.instance_var = arg            # instance variable
        self._protected = arg              # "internal"

    def method(self):                      # instance method
        return self.instance_var

    @classmethod
    def cls_method(cls): ...               # class method

    @staticmethod
    def util(): ...                        # static method

    @property
    def value(self):                       # getter — no brackets to use
        return self._protected

    @value.setter
    def value(self, new):                  # setter — with validation
        self._protected = new

    def __str__(self):  return "friendly"  # print(obj)
    def __repr__(self): return "Class()"   # debugging


# Inheritance
class Child(Parent):
    def __init__(self, a, b):
        super().__init__(a)                # parent's setup
        self.b = b

    def method(self):
        return super().method() + " extra"

isinstance(obj, Class)
issubclass(Child, Parent)

# Dunder methods
__str__  __repr__  __len__  __eq__  __lt__
__add__  __sub__  __mul__   __getitem__  __contains__
```

---

## Useful Tools

```python
# Generators
def gen(n):
    for i in range(n):
        yield i                            # pause and hand back one

(x**2 for x in range(10))                  # generator expression
# single-use · memory-efficient · can't index

# Decorators
import functools
def my_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        # before
        result = func(*args, **kwargs)
        # after
        return result
    return wrapper

@my_decorator
def my_function(): ...

# Regex
import re
re.search(r"pat", text)       # first match or None
re.findall(r"pat", text)      # all matches as a list
re.sub(r"pat", "new", text)   # replace
re.match(r"pat", text)        # match at the START

\d digit   \w word   \s space   . any
+ one-plus   * zero-plus   ? optional
{3} exactly   [abc] any of   [^abc] none of
^ start   $ end   | or   ( ) capture

# Datetime
from datetime import datetime, timedelta
now = datetime.now()
now.strftime("%Y-%m-%d %H:%M")             # datetime → string
datetime.strptime("2026-01-01", "%Y-%m-%d")# string → datetime
now + timedelta(days=7)
(date_a - date_b).days

%Y year  %m month  %d day  %B month-name
%A weekday  %H hour  %M minute  %S second
```

---

## Algorithms

```python
# Big O
O(1)        dict/set lookup, list index
O(log n)    binary search
O(n)        single loop
O(n log n)  sorted()
O(n²)       nested loops — usually too slow

# Number
is_prime:  check divisors up to int(n**0.5) + 1
fibonacci: a, b = b, a + b
gcd:       while b: a, b = b, a % b

# Search
linear:  loop through — any list, O(n)
binary:  halve each time — SORTED list only, O(log n)

# Strings
s[::-1]                                    # reverse
sorted(a) == sorted(b)                     # anagram
"".join(c for c in s if c.isalnum())       # strip punctuation

# Lists
set(lst)                                   # dedupe
n*(n+1)//2 - sum(lst)                      # find missing number
seen = {}                                  # "remember what you've seen"
```

---

## 🩺 Common Errors & Fixes

| Error | Usual cause | Fix |
|-------|-------------|-----|
| `IndentationError` | Wrong spacing | Use exactly 4 spaces, never mix tabs |
| `SyntaxError` | Missing `:` or unbalanced `( )` | Check the line **above** the one reported |
| `NameError` | Typo, or used before defining | Check spelling and order |
| `TypeError` | Mixing types, e.g. `"5" + 5` | Convert with `int()` / `str()` |
| `ValueError` | `int("abc")` | Validate input before converting |
| `IndexError` | List index too big | Check `len()` first |
| `KeyError` | Dict key missing | Use `.get()` |
| `AttributeError` | Method doesn't exist | Check spelling; check the type |
| `ZeroDivisionError` | Divided by 0 | Check the denominator first |
| `FileNotFoundError` | Wrong path | Use `os.path.exists()` first |

> 💡 **How to read a traceback:** start at the **bottom**. The last line names the error and the line above it shows where it happened.

---

*Schull AI Academy · Python Phase 2*
