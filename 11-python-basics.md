# Lesson 11 — Python Basics

> **Week 6 · Friday** · Prerequisites: [SETUP.md](../SETUP.md) completed

## 🎯 What You'll Learn

- Where Python came from and why it's everywhere
- How to write and run your first real program
- Variables — labelled boxes that hold data
- The 5 data types you'll use every day
- Converting between types
- Getting input from a user and printing nicely
- All the operators (`+`, `>`, `and`, `in`…)

**You'll build:** A working calculator.

---

## 📖 A Very Short History

In **1989**, a Dutch programmer named **Guido van Rossum** was bored over Christmas. He decided to build a programming language that was *readable* — one that looked almost like English.

He named it **Python**, not after the snake, but after the British comedy show *Monty Python's Flying Circus*. That's why Python documentation is full of "spam" and "eggs" jokes.

**Why it took over the world:**

| Reason | What it means for you |
|--------|----------------------|
| Reads like English | `if age > 18:` — you can guess what it does |
| No semicolons or braces | Less punctuation to get wrong |
| Batteries included | Huge library of ready-made tools |
| Used everywhere | AI, web, DevOps, data science, automation |

> 📌 **Python 2 vs Python 3:** Python 2 is dead (retired in 2020). We use **Python 3** exclusively. If you find a tutorial using `print "hello"` without brackets, it's ancient — skip it.

---

## 1️⃣ Your First Program

Create `hello.py`:

```python
print("Hello, world!")
print("I am learning Python")
```

Run it:
```bash
python3 hello.py
```

`print()` is a **function** — it displays whatever you put in the brackets.

---

## 2️⃣ Comments — Notes for Humans

Python ignores anything after `#`.

```python
# This is a comment. Python skips it entirely.

print("This runs")     # This part is also ignored

# Comments explain WHY, not WHAT.
# ❌ Bad:  x = x + 1   # add 1 to x
# ✅ Good: x = x + 1   # count this login attempt
```

**Multi-line notes (docstrings)** use triple quotes:

```python
"""
This program calculates a student's average score.
Written by: Sese Akhinedo
Date: Week 6
"""
```

---

## 3️⃣ Indentation — Python's Big Rule

Most languages use `{ }` to group code. **Python uses spaces.**

```python
if 5 > 3:
    print("This is inside the if")     # 4 spaces = inside
    print("So is this")
print("This is outside")               # no spaces = outside
```

> 🚨 **The #1 beginner error.** Get the spacing wrong and Python refuses to run:
> ```
> IndentationError: expected an indented block
> ```
>
> **The rule: always use exactly 4 spaces. Never mix tabs and spaces.**
> In VS Code, press Tab — it inserts 4 spaces automatically.

---

## 4️⃣ Variables — Labelled Boxes

A variable is a name that holds a value.

```python
name = "Nnamdi"
age = 19
```

```
    name  ──►  ┌──────────┐        age  ──►  ┌────┐
               │ "Nnamdi" │                  │ 19 │
               └──────────┘                  └────┘
```

**Rules for names:**

| ✅ Legal | ❌ Illegal | Why |
|---------|-----------|-----|
| `name` | `2name` | Can't start with a number |
| `student_age` | `student age` | No spaces |
| `total_1` | `total-1` | No hyphens (that's minus!) |
| `_private` | `class` | Can't use Python keywords |

**Naming style (PEP 8 — the official style guide):**

```python
student_name = "Sese"        # ✅ snake_case for variables
MAX_ATTEMPTS = 3             # ✅ ALL_CAPS for constants
studentName = "Sese"         # ❌ camelCase — that's JavaScript style
```

> 🐍 **PEP 8** is Python's official style guide. Following it makes your code look professional and readable to other Python programmers.

**Python is dynamically typed** — you don't declare the type, and a variable can change type:

```python
x = 5           # x is now an integer
x = "hello"     # x is now a string — perfectly legal!
```

---

## 5️⃣ The Five Data Types You Need

```python
name = "Sodee"          # str    — text
age = 19                # int    — whole number
height = 1.75           # float  — decimal number
is_student = True       # bool   — True or False
nothing = None          # None   — "no value at all"

print(type(name))       # <class 'str'>
print(type(age))        # <class 'int'>
print(type(height))     # <class 'float'>
print(type(is_student)) # <class 'bool'>
print(type(nothing))    # <class 'NoneType'>
```

| Type | Holds | Examples |
|------|-------|----------|
| `str` | Text | `"hello"`, `'Python'`, `"123"` |
| `int` | Whole numbers | `42`, `-7`, `0` |
| `float` | Decimals | `3.14`, `-0.5`, `2.0` |
| `bool` | True/False | `True`, `False` |
| `NoneType` | Nothing | `None` |

> ⚠️ **`"123"` is NOT `123`.** One is text, one is a number. `"5" + "5"` gives `"55"`, but `5 + 5` gives `10`.

---

## 6️⃣ Type Conversion

Convert between types with `int()`, `float()`, `str()`, `bool()`:

```python
print(int("42") + 8)        # 50    — text to number
print(float("3.5") * 2)     # 7.0
print(str(99) + "!")        # "99!" — number to text
print(int(9.99))            # 9     — chops off decimals, NO rounding
print(round(9.99))          # 10    — this rounds properly
print(round(3.14159, 2))    # 3.14  — round to 2 decimal places
```

**What counts as `False`?**

```python
print(bool(0))        # False
print(bool(""))       # False — empty string
print(bool([]))       # False — empty list
print(bool(None))     # False
print(bool("hi"))     # True  — anything else
print(bool(42))       # True
```

> 🧠 **Rule of thumb:** empty or zero → `False`. Everything else → `True`.

---

## 7️⃣ Input and Output

### Getting input

```python
name = input("What is your name? ")
print(f"Hello, {name}!")
```

> 🚨 **`input()` ALWAYS returns a string** — even if the user types numbers.

```python
age = input("Your age? ")      # user types 19
print(age + 1)                 # ❌ TypeError! "19" + 1 makes no sense
```

**Fix it by converting:**

```python
age = int(input("Your age? "))
print(age + 1)                 # ✅ 20
```

### f-strings — the modern way to print

Put an `f` before the quotes, then use `{}` to drop in variables:

```python
name = "Jacqueline"
age = 19
height = 1.68

print(f"{name} is {age} years old")
print(f"Next year she'll be {age + 1}")        # maths inside!
print(f"Height: {height:.1f}m")                # 1 decimal place
print(f"Pi is roughly {3.14159:.2f}")          # 3.14
```

**Formatting tricks:**

```python
print(f"|{name:>15}|")     # right-aligned  |     Jacqueline|
print(f"|{name:<15}|")     # left-aligned   |Jacqueline     |
print(f"|{name:^15}|")     # centred        |  Jacqueline   |
print(f"{1234567:,}")      # 1,234,567 — thousands separator
```

---

## 8️⃣ Operators

### Arithmetic

```python
a, b = 17, 5

print(a + b)     # 22   addition
print(a - b)     # 12   subtraction
print(a * b)     # 85   multiplication
print(a / b)     # 3.4  division (always gives a float)
print(a // b)    # 3    floor division (throws away the decimal)
print(a % b)     # 2    modulo (the remainder)
print(a ** 2)    # 289  power
```

> 🔑 **`%` (modulo) is more useful than it looks.** `n % 2 == 0` tells you if `n` is even. It's used constantly.

### Comparison — always gives `True` or `False`

```python
print(17 > 5)      # True
print(17 == 5)     # False   ← two equals signs to COMPARE
print(17 != 5)     # True    ← "not equal"
print(17 >= 17)    # True
```

> 🚨 **`=` vs `==`** — the classic mistake.
> `=` **assigns**: `age = 19` (put 19 in the box)
> `==` **compares**: `age == 19` (is the box holding 19?)

### Logical

```python
print(True and False)    # False — both must be true
print(True or False)     # True  — either one
print(not True)          # False — flips it
```

### Membership — `in`

```python
print("th" in "python")           # True
print("z" not in "python")        # True
print(3 in [1, 2, 3])             # True
```

### Identity — `is` vs `==`

```python
x = [1, 2]
y = [1, 2]
print(x == y)    # True  — same CONTENTS
print(x is y)    # False — different BOXES in memory
```

> 🧠 **Analogy:** Two identical twins `==` each other (same appearance), but they're not `is` each other (not the same person).
> Use `==` for values. Only use `is` with `None`: `if result is None:`

---

## ✏️ Class Activities

### Activity 11.1 — Variable Practice *(5 min)*

Create `about_me.py` that stores your name, age, height, favourite subject, and whether you like coding. Print each one on its own line **with its type**.

<details>
<summary>💡 Solution</summary>

```python
name = "Sodee Peterside"
age = 19
height = 1.72
subject = "Python"
likes_coding = True

print(f"Name: {name} ({type(name)})")
print(f"Age: {age} ({type(age)})")
print(f"Height: {height} ({type(height)})")
print(f"Subject: {subject} ({type(subject)})")
print(f"Likes coding: {likes_coding} ({type(likes_coding)})")
```
</details>

---

### Activity 11.2 — The Interview Bot *(8 min)*

Write `interview.py` that asks for name, age, and city, then prints a formatted card.

**Expected output:**
```
=== PROFILE CARD ===
Name       : Sese Akhinedo
Age        : 18 (born around 2008)
City       : Port Harcourt
Name length: 14 characters
```

<details>
<summary>💡 Solution</summary>

```python
name = input("Your name? ")
age = int(input("Your age? "))
city = input("Your city? ")

birth_year = 2026 - age

print()
print("=== PROFILE CARD ===")
print(f"Name       : {name}")
print(f"Age        : {age} (born around {birth_year})")
print(f"City       : {city}")
print(f"Name length: {len(name)} characters")
```
</details>

---

### Activity 11.3 — Type Detective *(5 min)*

Predict what each line prints. Then run it and check.

```python
print("5" + "5")
print(5 + 5)
print("5" * 3)
print(5 * 3)
print(int("5") + 5)
print(10 / 3)
print(10 // 3)
print(10 % 3)
print(bool(0), bool(""), bool("False"))
```

<details>
<summary>💡 Answers</summary>

```
55          ← string concatenation, not maths!
10
555         ← string repeated 3 times
15
10
3.3333333333333335
3
1
False False True    ← "False" is a non-empty string, so it's True!
```
The last one catches everyone. `bool("False")` is `True` because the *string* isn't empty.
</details>

---

### Activity 11.4 — Unit Converter *(8 min)*

Write `converter.py` that asks for a temperature in Celsius and prints it in Fahrenheit and Kelvin, each to **1 decimal place**.

Formulas: `F = (C × 9/5) + 32` · `K = C + 273.15`

<details>
<summary>💡 Solution</summary>

```python
celsius = float(input("Temperature in Celsius: "))

fahrenheit = (celsius * 9 / 5) + 32
kelvin = celsius + 273.15

print()
print(f"{celsius}°C = {fahrenheit:.1f}°F = {kelvin:.1f}K")
```
</details>

---

## 🏆 Mini Project — The Calculator

Build `calculator.py`.

**Requirements:**
1. Ask for two numbers
2. Show sum, difference, product, quotient (2 dp), floor division, remainder, and power
3. Print results in a neat table
4. Handle the division-by-zero case gracefully

<details>
<summary>💡 Full solution</summary>

```python
"""
Simple Calculator
Takes two numbers and performs all basic operations.
"""

print("=" * 32)
print("       SIMPLE CALCULATOR")
print("=" * 32)

a = float(input("First number:  "))
b = float(input("Second number: "))

print()
print(f"{'Operation':<20}{'Result':>12}")
print("-" * 32)
print(f"{'Addition':<20}{a + b:>12.2f}")
print(f"{'Subtraction':<20}{a - b:>12.2f}")
print(f"{'Multiplication':<20}{a * b:>12.2f}")

if b != 0:
    print(f"{'Division':<20}{a / b:>12.2f}")
    print(f"{'Floor division':<20}{a // b:>12.2f}")
    print(f"{'Remainder':<20}{a % b:>12.2f}")
else:
    print(f"{'Division':<20}{'undefined':>12}")

print(f"{'Power (a^b)':<20}{a ** b:>12.2f}")
print("-" * 32)
```

**Sample run:**
```
================================
       SIMPLE CALCULATOR
================================
First number:  17
Second number: 5

Operation                 Result
--------------------------------
Addition                   22.00
Subtraction                12.00
Multiplication             85.00
Division                    3.40
Floor division              3.00
Remainder                   2.00
Power (a^b)           1419857.00
--------------------------------
```
</details>

---

## 📌 Lesson 11 Cheat Sheet

```python
# Printing & input
print("text")
name = input("Prompt: ")           # ALWAYS returns a string
age  = int(input("Age: "))         # convert it!

# f-strings
print(f"{name} is {age}")
print(f"{pi:.2f}")                 # 2 decimal places
print(f"{n:,}")                    # thousands separator
print(f"{s:>10}{s:<10}{s:^10}")    # right / left / centre

# Types
str  int  float  bool  None
type(x)                            # check a type
int("5")  float("5.5")  str(5)  bool(0)
round(3.14159, 2)

# Operators
+  -  *  /  //  %  **               # arithmetic
== != >  <  >= <=                   # comparison
and  or  not                        # logical
in  not in                          # membership
is                                  # identity (use only with None)

# Style
snake_case = "variables"
CONSTANT_NAME = "constants"
# 4 spaces for indentation, always
```

---

## ✅ Self-Check

- [ ] I can create a `.py` file and run it from the terminal
- [ ] I know why `input()` needs `int()` around it for numbers
- [ ] I can explain the difference between `=` and `==`
- [ ] I can name all 5 basic data types
- [ ] I can use f-strings with formatting like `:.2f`
- [ ] I know what `%` (modulo) does and why it's useful
- [ ] My calculator project runs without errors

---

## 📚 Homework

1. Extend the calculator to also print the **average** of the two numbers
2. Write a script that asks for a price and a discount %, then prints the final price
3. Write a "seconds converter": ask for a number of seconds, print it as hours, minutes, seconds (hint: use `//` and `%`)
4. Deliberately cause three different errors, and write down what each error message said

---

**Next:** [Lesson 12 — Control Flow](12-control-flow.md) →
