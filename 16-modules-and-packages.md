# Lesson 16 — Modules, Packages & Virtual Environments

> **Week 8 · Saturday** · Prerequisites: [Lesson 15](15-functions.md)

## 🎯 What You'll Learn

- Importing code from other files
- Building your own modules and packages
- The standard library essentials (`os`, `datetime`, `json`, `random`, `math`)
- Virtual environments — and why every project needs one
- Installing packages with `pip` and `requirements.txt`
- How to structure a real Python project

**You'll build:** Your own installable Python package.

---

## 1️⃣ What Is a Module?

**A module is just a `.py` file.** That's it.

Any file you write can be imported into another file. That's how you split a big program into manageable pieces.

> 🧰 **Analogy:** Your kitchen doesn't keep everything in one drawer. Knives in one, spoons in another. Modules are drawers for your code.

### Three ways to import

```python
# 1. Import the whole module
import math
print(math.sqrt(16))          # 4.0
print(math.pi)                # 3.141592653589793

# 2. Import specific things
from math import sqrt, pi
print(sqrt(16))               # no "math." prefix needed
print(pi)

# 3. Import with a nickname
import math as m
print(m.sqrt(16))
```

> ⚠️ **Never do `from math import *`.** It dumps every name into your file and you lose track of where things came from — plus it can silently overwrite your own variables.

---

## 2️⃣ Making Your Own Module

Create `greetings.py`:

```python
"""A module of greeting functions."""

def hello(name):
    return f"Hello, {name}!"

def goodbye(name):
    return f"Goodbye, {name}!"

LANGUAGE = "English"
```

Now in `main.py` (same folder):

```python
import greetings

print(greetings.hello("Sese"))
print(greetings.LANGUAGE)

# or
from greetings import hello
print(hello("Sodee"))
```

### The `if __name__ == "__main__":` guard

Every module has a hidden variable `__name__`:
- When you **run** a file directly → `__name__` is `"__main__"`
- When you **import** it → `__name__` is the module's name

```python
# calculator.py

def add(a, b):
    return a + b

print("This runs on import — annoying!")

if __name__ == "__main__":
    print("This ONLY runs when you run calculator.py directly")
    print(add(2, 3))
```

> 🔑 **Always wrap your test/demo code in this guard.** Otherwise importing your module triggers all its side effects.

---

## 3️⃣ Packages — Folders of Modules

A **package** is a folder containing modules, with a special `__init__.py` file.

```
mytoolkit/
├── __init__.py         ← marks this folder as a package
├── maths.py
├── strings.py
└── files.py
```

**Building it:**

```bash
mkdir mytoolkit
cd mytoolkit
touch __init__.py maths.py strings.py
```

`mytoolkit/maths.py`:
```python
def add(a, b):
    return a + b

def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(n ** 0.5) + 1):
        if n % i == 0:
            return False
    return True
```

`mytoolkit/strings.py`:
```python
def reverse(text):
    return text[::-1]

def is_palindrome(text):
    clean = "".join(c.lower() for c in text if c.isalnum())
    return clean == clean[::-1]
```

`mytoolkit/__init__.py`:
```python
"""My personal toolkit package."""

from .maths import add, is_prime
from .strings import reverse, is_palindrome

__version__ = "1.0.0"
```

**Using it** (from the folder *above* `mytoolkit`):

```python
import mytoolkit

print(mytoolkit.add(2, 3))
print(mytoolkit.is_palindrome("racecar"))
print(mytoolkit.__version__)

# Or import a specific module
from mytoolkit import maths
print(maths.is_prime(17))
```

> 📌 **What does `__init__.py` do?** It tells Python "this folder is a package." It also lets you decide what gets exposed when someone imports your package. It can be completely empty and still work.

---

## 4️⃣ The Standard Library — Batteries Included

Python ships with hundreds of ready-made modules. Here are the ones you'll actually use.

### `math` — maths functions

```python
import math

print(math.sqrt(16))        # 4.0
print(math.ceil(4.1))       # 5    round up
print(math.floor(4.9))      # 4    round down
print(math.pi)              # 3.14159...
print(math.pow(2, 10))      # 1024.0
print(math.gcd(48, 18))     # 6
```

### `random` — randomness

```python
import random

print(random.randint(1, 6))                    # random int 1-6 (inclusive)
print(random.choice(["rock", "paper", "scissors"]))
print(random.random())                          # float between 0 and 1

deck = [1, 2, 3, 4, 5]
random.shuffle(deck)                            # shuffles IN PLACE
print(deck)

print(random.sample(range(1, 50), 6))           # 6 unique lottery numbers
```

### `datetime` — dates and times

```python
from datetime import datetime, timedelta

now = datetime.now()
print(now)                                      # 2026-08-28 14:30:00.123456
print(now.strftime("%Y-%m-%d"))                 # 2026-08-28
print(now.strftime("%A, %d %B %Y"))             # Friday, 28 August 2026
print(now.strftime("%H:%M"))                    # 14:30

# Date maths
next_week = now + timedelta(days=7)
print(next_week.strftime("%Y-%m-%d"))

# Difference between dates
new_year = datetime(2026, 1, 1)
print((now - new_year).days, "days since New Year")

# Parse a string into a date
d = datetime.strptime("2026-12-25", "%Y-%m-%d")
print(d.strftime("%d/%m/%Y"))                   # 25/12/2026
```

**Common format codes:**

| Code | Means | Example |
|------|-------|---------|
| `%Y` | 4-digit year | 2026 |
| `%m` | Month number | 08 |
| `%d` | Day | 28 |
| `%B` | Month name | August |
| `%A` | Weekday name | Friday |
| `%H:%M:%S` | Time | 14:30:00 |

### `os` and `sys` — talking to the system

```python
import os, sys

print(os.getcwd())                    # current folder
print(os.listdir("."))                # files here
print(os.path.exists("data.txt"))     # does it exist?
print(os.path.join("folder", "file.txt"))    # safe path building

os.makedirs("output", exist_ok=True)  # create folder if missing

print(sys.argv)                       # command-line arguments
print(sys.version)                    # Python version
```

### `json` — the universal data format

```python
import json

data = {"name": "Sese", "scores": [90, 85], "active": True}

text = json.dumps(data)              # dict → string
print(text)                          # {"name": "Sese", ...}

back = json.loads(text)              # string → dict
print(back["name"])
```

> 📌 We use `json` heavily in **Lesson 17** for saving data to files.

---

## 5️⃣ Virtual Environments

### The problem they solve

Project A needs `requests` version 1.0. Project B needs version 2.0. Install one and you break the other.

> 🧺 **Analogy:** Two chefs sharing one spice rack. Chef A relabels the paprika and Chef B's dish is ruined. A virtual environment gives each project **its own spice rack**.

### Creating and using one

```bash
# Create it (do this inside your project folder)
python3 -m venv venv

# Activate it
source venv/bin/activate         # Linux / macOS
venv\Scripts\activate            # Windows

# Your prompt changes:
(venv) student@ubuntu:~/myproject$

# Now anything you install stays in THIS project
pip install requests

# Leave the environment
deactivate
```

> 🚨 **Always activate your venv before `pip install`.** If you forget, packages get installed system-wide and you're back to the clashing-spice-rack problem.

---

## 6️⃣ `pip` — Installing Packages

```bash
pip install requests               # install
pip install requests==2.31.0       # a specific version
pip install --upgrade requests     # upgrade
pip uninstall requests             # remove
pip list                           # what's installed
pip show requests                  # details about a package
```

### `requirements.txt` — sharing your dependencies

```bash
# Save what your project needs
pip freeze > requirements.txt
```

Produces something like:
```
certifi==2024.2.2
charset-normalizer==3.3.2
idna==3.6
requests==2.31.0
urllib3==2.2.1
```

Now anyone can recreate your exact setup:

```bash
pip install -r requirements.txt
```

> 🔑 **This is how professional Python projects work.** Clone a repo, make a venv, `pip install -r requirements.txt`, and you're running.

### Trying a third-party package

```bash
pip install requests
```

```python
import requests

response = requests.get("https://api.github.com")
print(response.status_code)          # 200
print(response.headers["content-type"])

# Fetching JSON data
r = requests.get("https://api.github.com/users/torvalds")
if r.status_code == 200:
    data = r.json()
    print(data["name"], "-", data["public_repos"], "repos")
```

---

## 7️⃣ Project Structure

Here's how a real, tidy Python project looks:

```
my-project/
├── README.md              ← what this project is
├── requirements.txt       ← dependencies
├── .gitignore             ← files git should ignore
├── venv/                  ← virtual environment (NEVER commit this)
│
├── src/                   ← your actual code
│   ├── __init__.py
│   ├── main.py
│   └── utils/
│       ├── __init__.py
│       └── helpers.py
│
├── tests/                 ← tests for your code
│   └── test_helpers.py
│
└── data/                  ← input/output files
    └── sample.csv
```

**A good `.gitignore` for Python:**

```gitignore
venv/
__pycache__/
*.pyc
.env
.vscode/
.DS_Store
```

> 🗑️ **What is `__pycache__`?** Python's compiled cache. It's generated automatically. Never commit it.

---

## ✏️ Class Activities

### Activity 16.1 — Standard Library Tour *(10 min)*

Write one script that uses **all five** of these modules:

1. `math` — print the square root of 144
2. `random` — simulate rolling two dice
3. `datetime` — print today's date as "Friday, 28 August 2026"
4. `os` — print the current directory and list its files
5. `json` — convert a dictionary to a JSON string and back

<details>
<summary>💡 Solution</summary>

```python
import math, random, os, json
from datetime import datetime

print("--- math ---")
print(f"√144 = {math.sqrt(144)}")

print("\n--- random ---")
d1, d2 = random.randint(1, 6), random.randint(1, 6)
print(f"Dice: {d1} + {d2} = {d1 + d2}")

print("\n--- datetime ---")
print(datetime.now().strftime("%A, %d %B %Y"))

print("\n--- os ---")
print(f"Current folder: {os.getcwd()}")
print(f"Files: {os.listdir('.')[:5]}")

print("\n--- json ---")
data = {"name": "Sese", "score": 91}
text = json.dumps(data)
print(f"As JSON string: {text}")
print(f"Back to dict:   {json.loads(text)['name']}")
```
</details>

---

### Activity 16.2 — Build a Module *(10 min)*

Create `textkit.py` with these functions, then import and test it from `main.py`:

| Function | Does |
|----------|------|
| `word_count(text)` | Number of words |
| `char_count(text, spaces=False)` | Characters, optionally excluding spaces |
| `title_case(text)` | Capitalise each word |
| `reverse(text)` | Reverse the string |
| `is_palindrome(text)` | Ignore case and punctuation |

Include a docstring for each and an `if __name__ == "__main__":` test block.

<details>
<summary>💡 Solution</summary>

```python
# textkit.py
"""A toolkit of text-processing functions."""

def word_count(text: str) -> int:
    """Return the number of words in text."""
    return len(text.split())

def char_count(text: str, spaces: bool = False) -> int:
    """Return the character count, optionally excluding spaces."""
    return len(text) if spaces else len(text.replace(" ", ""))

def title_case(text: str) -> str:
    """Capitalise the first letter of every word."""
    return text.title()

def reverse(text: str) -> str:
    """Return text reversed."""
    return text[::-1]

def is_palindrome(text: str) -> bool:
    """Return True if text reads the same backwards, ignoring case/punctuation."""
    clean = "".join(c.lower() for c in text if c.isalnum())
    return clean == clean[::-1]


if __name__ == "__main__":
    sample = "A man, a plan, a canal: Panama"
    print(f"Text:       {sample}")
    print(f"Words:      {word_count(sample)}")
    print(f"Chars:      {char_count(sample)}")
    print(f"Title:      {title_case(sample)}")
    print(f"Reversed:   {reverse(sample)}")
    print(f"Palindrome? {is_palindrome(sample)}")
```

```python
# main.py
import textkit

print(textkit.word_count("hello world"))
print(textkit.is_palindrome("racecar"))
```
</details>

---

### Activity 16.3 — Virtual Environment Practice *(8 min)*

Do this in your terminal and record each command:

1. Create a folder called `venv-practice` and enter it
2. Create a virtual environment named `venv`
3. Activate it — confirm your prompt changed
4. Run `pip list` — note how few packages there are
5. Install `requests`
6. Run `pip list` again — see how many appeared
7. Save to `requirements.txt`
8. Deactivate
9. Run `pip list` again — is `requests` still there?

<details>
<summary>💡 Commands</summary>

```bash
mkdir venv-practice && cd venv-practice
python3 -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate
pip list
pip install requests
pip list
pip freeze > requirements.txt
cat requirements.txt
deactivate
pip list                          # requests is gone from the system view
```
</details>

---

## 🏆 Mini Project — Build a Real Package

Create a complete, properly structured package called **`studentkit`**.

**Required structure:**

```
studentkit-project/
├── README.md
├── requirements.txt
├── .gitignore
├── main.py
└── studentkit/
    ├── __init__.py
    ├── grades.py
    └── reports.py
```

**`grades.py` must contain:** `calculate_average`, `letter_grade`, `has_passed`
**`reports.py` must contain:** `format_student_row`, `print_report`

<details>
<summary>💡 Full solution</summary>

**`studentkit/grades.py`**
```python
"""Functions for calculating student grades."""

def calculate_average(scores: list) -> float:
    """Return the average of a list of scores."""
    if not scores:
        return 0.0
    return sum(scores) / len(scores)

def letter_grade(average: float) -> str:
    """Convert a numeric average to a letter grade."""
    if average >= 90:  return "A"
    if average >= 80:  return "B"
    if average >= 70:  return "C"
    if average >= 60:  return "D"
    return "F"

def has_passed(average: float, pass_mark: float = 50) -> bool:
    """Return True if the average meets the pass mark."""
    return average >= pass_mark
```

**`studentkit/reports.py`**
```python
"""Functions for formatting student reports."""

from .grades import calculate_average, letter_grade, has_passed

def format_student_row(name: str, scores: list) -> str:
    """Return one formatted line for a student."""
    avg = calculate_average(scores)
    grade = letter_grade(avg)
    status = "PASS" if has_passed(avg) else "FAIL"
    return f"{name:<14}{avg:>7.1f}{grade:>4}{status:>7}"

def print_report(students: dict) -> None:
    """Print a full class report."""
    print("=" * 40)
    print("           CLASS REPORT")
    print("=" * 40)
    print(f"{'NAME':<14}{'AVG':>7}{'GR':>4}{'STATUS':>7}")
    print("-" * 40)

    for name, scores in students.items():
        print(format_student_row(name, scores))

    all_avgs = [calculate_average(s) for s in students.values()]
    print("-" * 40)
    print(f"{'CLASS AVERAGE':<14}{calculate_average(all_avgs):>7.1f}")
    passed = sum(1 for a in all_avgs if has_passed(a))
    print(f"{'PASS RATE':<14}{passed}/{len(all_avgs)}")
    print("=" * 40)
```

**`studentkit/__init__.py`**
```python
"""studentkit — tools for managing student grades and reports."""

from .grades import calculate_average, letter_grade, has_passed
from .reports import format_student_row, print_report

__version__ = "1.0.0"
__author__ = "<your name>"
```

**`main.py`**
```python
"""Demo of the studentkit package."""

import studentkit

students = {
    "Jacqueline": [85, 92, 78],
    "Sodee":      [70, 65, 80],
    "Sese":       [95, 88, 91],
    "Ada":        [45, 51, 48],
}

studentkit.print_report(students)

print(f"\nUsing studentkit v{studentkit.__version__}")
print(f"Sese's average: {studentkit.calculate_average(students['Sese']):.1f}")
```

**`.gitignore`**
```gitignore
venv/
__pycache__/
*.pyc
.env
```

**Run it:**
```bash
python3 main.py
```
```
========================================
           CLASS REPORT
========================================
NAME              AVG  GR STATUS
----------------------------------------
Jacqueline       85.0   B   PASS
Sodee            71.7   C   PASS
Sese             91.3   A   PASS
Ada              48.0   F   FAIL
----------------------------------------
CLASS AVERAGE    74.0
PASS RATE     3/4
========================================
```
</details>

---

## 📌 Lesson 16 Cheat Sheet

```python
# Importing
import math
from math import sqrt, pi
import numpy as np
from .mymodule import func         # relative (inside a package)

# The main guard
if __name__ == "__main__":
    # only runs when executed directly
    ...

# Package structure
mypackage/
├── __init__.py
└── module.py

# Standard library
import math       # sqrt, ceil, floor, pi, gcd
import random     # randint, choice, shuffle, sample
import os         # getcwd, listdir, path.exists, makedirs
import sys        # argv, version, exit
import json       # dumps, loads, dump, load
from datetime import datetime, timedelta
```

```bash
# Virtual environments
python3 -m venv venv
source venv/bin/activate        # Linux/Mac
venv\Scripts\activate           # Windows
deactivate

# pip
pip install package
pip install package==1.2.3
pip uninstall package
pip list
pip freeze > requirements.txt
pip install -r requirements.txt
```

---

## ✅ Self-Check

- [ ] I can import from another file I wrote
- [ ] I know what `if __name__ == "__main__":` does and why
- [ ] I can create a package folder with `__init__.py`
- [ ] I can name 4 standard library modules and what they do
- [ ] I can create, activate, and deactivate a virtual environment
- [ ] I can generate and use a `requirements.txt`
- [ ] My `studentkit` package runs from `main.py`

---

## 📚 Homework

1. Add a `statistics.py` module to `studentkit` with `highest_scorer` and `lowest_scorer`
2. Write a script using `random` and `datetime` that generates 10 fake login records with random times
3. Create a venv for your studentkit project and generate a `requirements.txt`
4. Use `requests` to fetch `https://api.github.com/users/<your-username>` and print your repo count

---

**Next:** [Lesson 17 — Files & Errors](17-files-and-errors.md) →
