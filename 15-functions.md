# Lesson 15 — Functions

> **Week 8 · Friday** · Prerequisites: [Lesson 14](14-dictionaries-and-sets.md)

## 🎯 What You'll Learn

- Writing your own functions
- Parameters, arguments, and return values
- Default values and keyword arguments
- `*args` and `**kwargs`
- Variable scope (local vs global)
- Lambda functions with `map`, `filter`, `sorted`
- Recursion
- Docstrings and type hints
- A gentle first look at decorators

**You'll build:** A reusable maths toolkit and a refactored calculator.

---

## 1️⃣ Why Functions Exist

Look at this repetition:

```python
print("=" * 30)
print("     WELCOME")
print("=" * 30)

# ... 50 lines later ...

print("=" * 30)
print("     GOODBYE")
print("=" * 30)
```

Now imagine changing the width from 30 to 40. You'd hunt through the whole file.

**A function fixes this:**

```python
def banner(text):
    print("=" * 30)
    print(f"     {text}")
    print("=" * 30)

banner("WELCOME")
banner("GOODBYE")
```

Change it once, it changes everywhere.

> 🍳 **Analogy:** A function is a **recipe**. Write it once, cook it whenever you want, with different ingredients each time.

**The three reasons to use functions:**

| Reason | Meaning |
|--------|---------|
| **DRY** | Don't Repeat Yourself — write once, use many times |
| **Readable** | `calculate_tax(salary)` explains itself; 20 lines of maths doesn't |
| **Testable** | Small pieces are easy to check and fix |

---

## 2️⃣ Defining and Calling

```python
def greet():                     # def = define
    print("Hello!")

greet()                          # call it — nothing happens without this!
```

**The anatomy:**

```
def  greet ( name ) :
│     │       │     └─ colon required
│     │       └─ parameter
│     └─ function name (snake_case)
└─ keyword

    return f"Hello, {name}!"
    └─ 4 spaces = inside the function
```

### Parameters and arguments

```python
def greet(name):                    # 'name' is a PARAMETER
    print(f"Hello, {name}!")

greet("Sese")                       # "Sese" is an ARGUMENT
greet("Jacqueline")
```

### Returning values

```python
def add(a, b):
    return a + b

result = add(5, 3)
print(result)          # 8
print(add(10, 20))     # 30
```

> 🚨 **`print` vs `return` — the classic confusion:**
> ```python
> def add_print(a, b):
>     print(a + b)         # shows it on screen, gives back nothing
>
> def add_return(a, b):
>     return a + b         # hands the value back to your code
>
> x = add_print(2, 3)      # prints 5, but x is None!
> y = add_return(2, 3)     # prints nothing, but y is 5 ✅
> ```
> **Use `return` in almost all functions.** Print at the end, in your main code.

**Returning multiple values** (it's actually a tuple):

```python
def get_stats(numbers):
    return min(numbers), max(numbers), sum(numbers) / len(numbers)

low, high, avg = get_stats([4, 8, 15, 16])
print(f"Low: {low}, High: {high}, Avg: {avg:.1f}")
```

**A function with no `return` gives back `None`:**

```python
def say_hi():
    print("hi")

x = say_hi()
print(x)          # None
```

---

## 3️⃣ Types of Arguments

### Default values

```python
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

print(greet("Sodee"))              # Hello, Sodee!
print(greet("Sodee", "Welcome"))   # Welcome, Sodee!
```

> ⚠️ **Parameters with defaults must come last:**
> ```python
> def bad(greeting="Hi", name):     # ❌ SyntaxError
> def good(name, greeting="Hi"):    # ✅
> ```

### Keyword arguments — name them explicitly

```python
def describe(name, age, city):
    return f"{name}, {age}, from {city}"

print(describe("Sese", 18, "PH"))                        # positional
print(describe(age=18, city="PH", name="Sese"))          # keyword — any order!
```

> 💡 Keyword arguments make calls **self-documenting**. Compare:
> `create_user("Ada", True, False, True)` ← what do those mean?
> `create_user("Ada", is_admin=True, is_banned=False, verified=True)` ← obvious.

### `*args` — any number of positional arguments

```python
def add_all(*numbers):
    return sum(numbers)

print(add_all(1, 2))              # 3
print(add_all(1, 2, 3, 4, 5))     # 15
print(add_all())                  # 0
```

`*numbers` collects everything into a **tuple**.

### `**kwargs` — any number of keyword arguments

```python
def make_profile(**details):
    for key, value in details.items():
        print(f"  {key}: {value}")

make_profile(name="Sese", age=18, course="Python")
```
```
  name: Sese
  age: 18
  course: Python
```

`**details` collects everything into a **dictionary**.

> 🧠 **Memory aid:** one star `*` = a tuple of values. Two stars `**` = a dictionary of key=value pairs.

---

## 4️⃣ Documentation — Docstrings & Type Hints

### Docstrings

```python
def calculate_area(width, height):
    """
    Calculate the area of a rectangle.

    Args:
        width: The width in metres
        height: The height in metres

    Returns:
        The area in square metres
    """
    return width * height

print(calculate_area.__doc__)      # read the docstring
help(calculate_area)               # nicely formatted
```

### Type hints

Tell readers (and your editor) what types you expect:

```python
def calculate_area(width: float, height: float) -> float:
    """Calculate the area of a rectangle."""
    return width * height

def greet(name: str, times: int = 1) -> str:
    return f"Hello, {name}! " * times
```

> 📌 **Python does NOT enforce type hints** — they're documentation. But VS Code uses them to catch your mistakes before you run the code, which is genuinely useful.

---

## 5️⃣ Variable Scope

**Local** variables live inside a function. **Global** variables live outside.

```python
message = "I am global"

def show():
    inside = "I am local"
    print(message)      # ✅ can READ the global
    print(inside)

show()
print(message)          # ✅ works
print(inside)           # ❌ NameError — 'inside' doesn't exist out here
```

### Changing a global needs the `global` keyword

```python
count = 0

def increment():
    global count        # "I mean the OUTER count"
    count += 1

increment()
increment()
print(count)            # 2
```

> ⚠️ **`global` is usually a sign of bad design.** Prefer passing values in and returning them out:
> ```python
> def increment(count):        # ✅ much cleaner
>     return count + 1
>
> count = increment(count)
> ```

**The LEGB lookup order** — where Python searches for a name:

```
L ocal      →  inside this function
E nclosing  →  inside an outer function
G lobal     →  the file level
B uilt-in   →  Python's own names (print, len, ...)
```

> 🚨 **Never name a variable `list`, `str`, `sum`, `max`, or `print`** — you'd hide the built-in version and break your own code later.

---

## 6️⃣ Lambda Functions — Tiny Anonymous Functions

A `lambda` is a one-line function with no name.

```python
# These are equivalent
def double(x):
    return x * 2

double = lambda x: x * 2
```

Lambdas are only worth it when passed **into** another function:

### `map()` — transform every item

```python
nums = [1, 2, 3, 4, 5]
doubled = list(map(lambda x: x * 2, nums))
print(doubled)          # [2, 4, 6, 8, 10]
```

### `filter()` — keep some items

```python
evens = list(filter(lambda x: x % 2 == 0, nums))
print(evens)            # [2, 4]
```

### `sorted(key=...)` — the most useful one by far

```python
people = [("Ada", 30), ("Bola", 25), ("Chidi", 35)]

by_age = sorted(people, key=lambda p: p[1])
print(by_age)           # [('Bola', 25), ('Ada', 30), ('Chidi', 35)]

words = ["banana", "kiwi", "apple"]
print(sorted(words, key=lambda w: len(w)))    # ['kiwi', 'apple', 'banana']
```

> 💡 **Honest advice:** list comprehensions are usually clearer than `map`/`filter`:
> ```python
> list(map(lambda x: x*2, nums))          # works
> [x * 2 for x in nums]                   # more Pythonic ✅
> ```
> But `sorted(..., key=lambda ...)` is genuinely the best tool for its job — learn that one properly.

---

## 7️⃣ Recursion — Functions That Call Themselves

A recursive function solves a problem by calling itself on a smaller version.

**Every recursive function needs two parts:**
1. A **base case** — when to stop
2. A **recursive case** — call itself with something smaller

```python
def factorial(n):
    if n <= 1:              # BASE CASE — stop here
        return 1
    return n * factorial(n - 1)     # RECURSIVE CASE

print(factorial(5))         # 120
```

**How it unrolls:**
```
factorial(5) = 5 × factorial(4)
             = 5 × 4 × factorial(3)
             = 5 × 4 × 3 × factorial(2)
             = 5 × 4 × 3 × 2 × factorial(1)
             = 5 × 4 × 3 × 2 × 1  =  120
```

**Fibonacci:**

```python
def fib(n):
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)

print([fib(i) for i in range(10)])
# [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

**Countdown:**

```python
def countdown(n):
    if n <= 0:
        print("Liftoff! 🚀")
        return
    print(n)
    countdown(n - 1)

countdown(5)
```

> 🚨 **Forget the base case and you get infinite recursion:**
> ```
> RecursionError: maximum recursion depth exceeded
> ```
> Python stops you after ~1000 levels deep.

> 🤔 **When to use recursion?** Honestly, for most beginner tasks a loop is simpler and faster. Learn recursion because it appears in interviews and because some problems (tree structures, folders inside folders) are genuinely natural to express that way.

---

## 8️⃣ A First Look at Decorators

A **decorator** wraps a function to add behaviour without changing its code.

```python
import functools

def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        import time
        start = time.time()
        result = func(*args, **kwargs)
        elapsed = time.time() - start
        print(f"⏱️  {func.__name__} took {elapsed:.4f}s")
        return result
    return wrapper


@timer                                   # ← apply the decorator
def slow_sum():
    return sum(range(1_000_000))

slow_sum()
# ⏱️  slow_sum took 0.0144s
```

**A logging decorator:**

```python
def logger(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"→ Calling {func.__name__}{args}")
        result = func(*args, **kwargs)
        print(f"← Returned {result}")
        return result
    return wrapper

@logger
def add(a, b):
    return a + b

add(2, 3)
# → Calling add(2, 3)
# ← Returned 5
```

> 📌 **Don't worry if this feels magical.** For now, just understand: `@something` above a function means "wrap this function with extra behaviour." You'll meet `@property` in Lesson 18 and it'll click.

---

## ✏️ Class Activities

### Activity 15.1 — Function Warm-Up *(10 min)*

Write these functions and test each one:

| # | Function | Does |
|---|----------|------|
| 1 | `is_even(n)` | Returns `True` if `n` is even |
| 2 | `celsius_to_f(c)` | Converts temperature |
| 3 | `initials(full_name)` | `"Sese Akhinedo"` → `"S.A."` |
| 4 | `count_vowels(text)` | Counts a, e, i, o, u |
| 5 | `apply_discount(price, pct=10)` | Default 10% discount |

<details>
<summary>💡 Solutions</summary>

```python
def is_even(n: int) -> bool:
    """Return True if n is even."""
    return n % 2 == 0

def celsius_to_f(c: float) -> float:
    """Convert Celsius to Fahrenheit."""
    return (c * 9 / 5) + 32

def initials(full_name: str) -> str:
    """Turn 'Sese Akhinedo' into 'S.A.'"""
    return ".".join(part[0].upper() for part in full_name.split()) + "."

def count_vowels(text: str) -> int:
    """Count vowels in a string."""
    return sum(1 for ch in text.lower() if ch in "aeiou")

def apply_discount(price: float, pct: float = 10) -> float:
    """Apply a percentage discount."""
    return price * (1 - pct / 100)

# Tests
print(is_even(4), is_even(7))
print(celsius_to_f(37))
print(initials("Sese Akhinedo"))
print(count_vowels("Programming"))
print(apply_discount(1000), apply_discount(1000, 25))
```
</details>

---

### Activity 15.2 — Arguments Practice *(8 min)*

1. Write `make_pizza(size, *toppings)` that prints the size and each topping
2. Write `create_user(**details)` that prints every detail
3. Write `greet(name, greeting="Hello", punctuation="!")` and call it three different ways

<details>
<summary>💡 Solutions</summary>

```python
def make_pizza(size, *toppings):
    print(f"\n🍕 A {size} pizza with:")
    if not toppings:
        print("  - just cheese")
    for t in toppings:
        print(f"  - {t}")

make_pizza("large", "pepperoni", "mushroom", "pepper")
make_pizza("small")


def create_user(**details):
    print("\n👤 New user:")
    for key, value in details.items():
        print(f"  {key:<10}: {value}")

create_user(name="Sese", age=18, role="student", city="PH")


def greet(name, greeting="Hello", punctuation="!"):
    return f"{greeting}, {name}{punctuation}"

print(greet("Ada"))
print(greet("Ada", "Welcome"))
print(greet("Ada", punctuation="!!!"))
```
</details>

---

### Activity 15.3 — Lambda & Sorting *(8 min)*

```python
students = [
    {"name": "Jacqueline", "score": 85, "age": 19},
    {"name": "Sodee",      "score": 92, "age": 18},
    {"name": "Sese",       "score": 78, "age": 20},
]
```

1. Sort by score, highest first
2. Sort by name alphabetically
3. Sort by age
4. Get just the names, in a list
5. Filter students scoring above 80

<details>
<summary>💡 Solutions</summary>

```python
print(sorted(students, key=lambda s: s["score"], reverse=True))
print(sorted(students, key=lambda s: s["name"]))
print(sorted(students, key=lambda s: s["age"]))
print([s["name"] for s in students])
print([s for s in students if s["score"] > 80])
```
</details>

---

### Activity 15.4 — Recursion Practice *(8 min)*

Write recursive versions of:

1. `sum_to(n)` — sum of 1 to n
2. `power(base, exp)` — base raised to exp
3. `reverse_string(s)` — reverse a string
4. `count_down(n)` — print n down to 1

<details>
<summary>💡 Solutions</summary>

```python
def sum_to(n):
    if n <= 0:
        return 0
    return n + sum_to(n - 1)

def power(base, exp):
    if exp == 0:
        return 1
    return base * power(base, exp - 1)

def reverse_string(s):
    if len(s) <= 1:
        return s
    return reverse_string(s[1:]) + s[0]

def count_down(n):
    if n <= 0:
        print("Done!")
        return
    print(n)
    count_down(n - 1)

print(sum_to(10))              # 55
print(power(2, 8))             # 256
print(reverse_string("Python"))# nohtyP
count_down(3)
```
</details>

---

## 🏆 Mini Project — Maths Toolkit + Refactored Calculator

Build `mathkit.py` — a collection of well-documented functions.

**Required functions:**

| Function | Does |
|----------|------|
| `add`, `subtract`, `multiply`, `divide` | Basics (divide handles ÷0) |
| `power(base, exp)` | Exponentiation |
| `factorial(n)` | Recursive factorial |
| `is_prime(n)` | Prime check |
| `fibonacci(n)` | First n Fibonacci numbers |
| `average(*numbers)` | Average of any count |
| `stats(numbers)` | Returns min, max, avg as a tuple |

Then rebuild your Lesson 11 calculator using **only these functions**.

<details>
<summary>💡 Full solution</summary>

```python
"""
mathkit.py — A collection of reusable maths functions.
Author: <your name>
"""

def add(a: float, b: float) -> float:
    """Return the sum of a and b."""
    return a + b

def subtract(a: float, b: float) -> float:
    """Return a minus b."""
    return a - b

def multiply(a: float, b: float) -> float:
    """Return a times b."""
    return a * b

def divide(a: float, b: float):
    """Return a divided by b, or None if b is zero."""
    if b == 0:
        return None
    return a / b

def power(base: float, exp: float) -> float:
    """Return base raised to exp."""
    return base ** exp

def factorial(n: int) -> int:
    """Return n! using recursion."""
    if n < 0:
        raise ValueError("Factorial is undefined for negative numbers")
    if n <= 1:
        return 1
    return n * factorial(n - 1)

def is_prime(n: int) -> bool:
    """Return True if n is a prime number."""
    if n < 2:
        return False
    for i in range(2, int(n ** 0.5) + 1):
        if n % i == 0:
            return False
    return True

def fibonacci(n: int) -> list:
    """Return the first n Fibonacci numbers."""
    seq = []
    a, b = 0, 1
    for _ in range(n):
        seq.append(a)
        a, b = b, a + b
    return seq

def average(*numbers) -> float:
    """Return the average of any number of values."""
    if not numbers:
        return 0
    return sum(numbers) / len(numbers)

def stats(numbers: list) -> tuple:
    """Return (min, max, average) of a list."""
    return min(numbers), max(numbers), average(*numbers)


# ---------- The refactored calculator ----------
if __name__ == "__main__":
    print("=" * 34)
    print("        MATHS TOOLKIT")
    print("=" * 34)

    a = float(input("First number:  "))
    b = float(input("Second number: "))

    print()
    print(f"{'Addition':<20}{add(a, b):>14.2f}")
    print(f"{'Subtraction':<20}{subtract(a, b):>14.2f}")
    print(f"{'Multiplication':<20}{multiply(a, b):>14.2f}")

    result = divide(a, b)
    if result is None:
        print(f"{'Division':<20}{'undefined':>14}")
    else:
        print(f"{'Division':<20}{result:>14.2f}")

    print(f"{'Power':<20}{power(a, b):>14.2f}")
    print(f"{'Average':<20}{average(a, b):>14.2f}")

    print()
    n = int(a)
    if 0 <= n <= 20:
        print(f"{n}! = {factorial(n)}")
    print(f"{n} is {'prime' if is_prime(n) else 'not prime'}")
    print(f"First 10 Fibonacci: {fibonacci(10)}")
```

> 📌 **What's `if __name__ == "__main__":`?** It means "only run this part if this file is run directly, not when it's imported." You'll understand it fully in Lesson 16.
</details>

---

## 📌 Lesson 15 Cheat Sheet

```python
# Defining
def name(param1, param2=default):
    """Docstring explaining what this does."""
    return value

# Type hints
def area(w: float, h: float) -> float:
    return w * h

# Argument types
def f(a, b=10):              # default
f(a=1, b=2)                  # keyword
def f(*args):                # any positional → tuple
def f(**kwargs):             # any keyword → dict

# Returning
return value
return a, b, c               # returns a tuple
x, y, z = func()             # unpack it

# Scope
global counter               # modify a global (avoid if possible)

# Lambdas
lambda x: x * 2
sorted(lst, key=lambda p: p[1])
list(map(lambda x: x*2, lst))
list(filter(lambda x: x > 5, lst))

# Recursion — always need a base case
def f(n):
    if n <= 1: return 1      # BASE CASE
    return n * f(n - 1)      # RECURSIVE CASE

# Decorators
@timer
def my_function(): ...
```

---

## ✅ Self-Check

- [ ] I can explain the difference between `print` and `return`
- [ ] I know why default parameters must come last
- [ ] I understand what `*args` and `**kwargs` collect
- [ ] I can write a docstring and type hints
- [ ] I know why `global` is usually a bad idea
- [ ] I can use `sorted(..., key=lambda ...)`
- [ ] I can write a recursive function with a base case
- [ ] My maths toolkit works and my calculator uses it

---

## 📚 Homework

1. Add `gcd(a, b)` and `lcm(a, b)` to your maths toolkit
2. Write a function that takes a list of names and returns a dictionary of `{name: length}`
3. Write a recursive function that counts how many digits are in a number
4. Write a decorator called `@announce` that prints "Starting..." before and "Finished!" after any function

---

**Next:** [Lesson 16 — Modules & Packages](16-modules-and-packages.md) →
