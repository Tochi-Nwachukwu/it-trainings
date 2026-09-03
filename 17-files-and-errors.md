# Lesson 17 — Files & Error Handling

> **Week 9 · Friday** · Prerequisites: [Lesson 16](16-modules-and-packages.md)

## 🎯 What You'll Learn

- Reading and writing files safely
- The `with` statement (context managers)
- Working with text, JSON, and CSV
- Handling errors with `try` / `except`
- The exceptions you'll actually meet
- Creating your own exceptions

**You'll build:** A contact manager that **remembers your data** between runs.

---

## 1️⃣ Why Files Matter

Every program you've written so far has **amnesia**. Close it, and everything is gone.

Files fix that. Files are how programs **remember**.

---

## 2️⃣ Reading Files — Always Use `with`

First, create a file to practise on. Run this once:

```python
with open("notes.txt", "w") as f:
    f.write("line one\nline two\nline three\n")
```

Now read it back:

```python
with open("notes.txt") as f:
    content = f.read()
    print(content)
```

> 🚪 **Why `with`?** It automatically closes the file when the block ends — even if your code crashes. Without it you must remember `f.close()`, and forgetting causes real bugs (data not saved, files locked).

**The old, error-prone way — don't do this:**
```python
f = open("notes.txt")
content = f.read()
f.close()                # if anything above crashes, this never runs ❌
```

### Four ways to read

```python
# 1. Everything as one string
with open("notes.txt") as f:
    text = f.read()

# 2. One line at a time
with open("notes.txt") as f:
    first = f.readline()

# 3. All lines as a list
with open("notes.txt") as f:
    lines = f.readlines()      # ['line one\n', 'line two\n']

# 4. Loop over it — BEST for big files (memory-efficient)
with open("notes.txt") as f:
    for line_number, line in enumerate(f, start=1):
        print(f"{line_number}: {line.strip()}")
```

> 💡 **`.strip()`** removes the invisible `\n` newline at the end of each line. You almost always want it.

---

## 3️⃣ Writing Files

```python
# "w" = write — ⚠️ WIPES the file first!
with open("output.txt", "w") as f:
    f.write("First line\n")
    f.write("Second line\n")

# "a" = append — adds to the end, keeps existing content
with open("output.txt", "a") as f:
    f.write("Third line\n")
```

**File modes:**

| Mode | Meaning | If file exists | If it doesn't |
|------|---------|----------------|---------------|
| `"r"` | Read (default) | Reads it | ❌ Error |
| `"w"` | Write | **Erases everything!** | Creates it |
| `"a"` | Append | Adds to the end | Creates it |
| `"x"` | Exclusive create | ❌ Error | Creates it |

> 🚨 **`"w"` is destructive.** Opening a file in `"w"` mode deletes its contents *immediately*, before you write anything. Use `"a"` when you want to add.

**Writing multiple lines:**

```python
lines = ["apple\n", "banana\n", "cherry\n"]
with open("fruits.txt", "w") as f:
    f.writelines(lines)

# Or build it yourself
fruits = ["apple", "banana", "cherry"]
with open("fruits.txt", "w") as f:
    for fruit in fruits:
        f.write(fruit + "\n")
```

---

## 4️⃣ JSON — Saving Real Data Structures

Text files are fine for words. But how do you save a **dictionary**?

**JSON** is the answer — a universal format that every language understands.

```python
import json

data = {
    "name": "Sese",
    "age": 18,
    "scores": [90, 85, 91],
    "active": True
}

# SAVE to a file
with open("student.json", "w") as f:
    json.dump(data, f, indent=2)          # indent=2 makes it readable

# LOAD from a file
with open("student.json") as f:
    loaded = json.load(f)

print(loaded["name"])          # Sese
print(loaded["scores"][0])     # 90
```

**The file looks like this:**
```json
{
  "name": "Sese",
  "age": 18,
  "scores": [90, 85, 91],
  "active": true
}
```

> 🧠 **Four functions, easy to mix up:**
> | Function | Direction | Works with |
> |----------|-----------|------------|
> | `json.dump(data, file)` | dict → file | **file** |
> | `json.load(file)` | file → dict | **file** |
> | `json.dumps(data)` | dict → string | **string** (s = string) |
> | `json.loads(text)` | string → dict | **string** |

---

## 5️⃣ CSV — Spreadsheet Data

```python
import csv

# WRITING
rows = [
    ["name", "score", "grade"],
    ["Jacqueline", 85, "B"],
    ["Sodee", 92, "A"],
]
with open("grades.csv", "w", newline="") as f:
    csv.writer(f).writerows(rows)

# READING — as lists
with open("grades.csv") as f:
    for row in csv.reader(f):
        print(row)          # ['name', 'score', 'grade']

# READING — as dictionaries (much nicer!)
with open("grades.csv") as f:
    for row in csv.DictReader(f):
        print(f"{row['name']}: {row['score']}")
```

> ⚠️ **Always pass `newline=""` when writing CSV**, or you get blank rows between entries on Windows.

---

## 6️⃣ Errors — When Things Go Wrong

Your program **will** hit problems: missing files, bad input, wrong types. Without handling, it crashes.

```python
number = int(input("Enter a number: "))     # user types "abc"
```
```
ValueError: invalid literal for int() with base 10: 'abc'
```

Program dead. Users hate this.

### `try` / `except`

```python
try:
    number = int(input("Enter a number: "))
    print(f"You entered {number}")
except ValueError:
    print("❌ That's not a valid number!")
```

Now it fails **gracefully**.

### The full structure

```python
try:
    result = 10 / int(input("Divide 10 by: "))
except ZeroDivisionError:
    print("Can't divide by zero!")
except ValueError:
    print("That's not a number!")
else:
    print(f"Result: {result}")      # runs only if NO error
finally:
    print("Done.")                  # ALWAYS runs, error or not
```

| Block | When it runs |
|-------|-------------|
| `try` | Always — this is the risky code |
| `except` | Only if a matching error happened |
| `else` | Only if **no** error happened |
| `finally` | **Always**, no matter what |

---

## 7️⃣ The Exceptions You'll Actually Meet

| Exception | Happens when | Example |
|-----------|-------------|---------|
| `ValueError` | Right type, wrong value | `int("abc")` |
| `TypeError` | Wrong type entirely | `"5" + 5` |
| `ZeroDivisionError` | Dividing by zero | `10 / 0` |
| `FileNotFoundError` | File doesn't exist | `open("nope.txt")` |
| `KeyError` | Dictionary key missing | `{"a":1}["z"]` |
| `IndexError` | List index too big | `[1,2][9]` |
| `AttributeError` | Method doesn't exist | `"text".push()` |
| `PermissionError` | No permission | Writing to a locked file |

```python
# Catching several at once
try:
    risky_operation()
except (ValueError, TypeError) as e:
    print(f"Input problem: {e}")

# Getting the error message
try:
    int("abc")
except ValueError as e:
    print(f"Details: {e}")
    # invalid literal for int() with base 10: 'abc'
```

### 🚨 The Cardinal Sin

```python
# ❌ NEVER DO THIS
try:
    do_everything()
except:
    pass                # silently swallows EVERY error
```

This hides bugs. Your program breaks and you have **no idea why**.

```python
# ✅ Catch specific errors, and say something
try:
    do_everything()
except FileNotFoundError:
    print("Config file missing — using defaults")
except ValueError as e:
    print(f"Bad data: {e}")
```

> 🎯 **Rule: catch only what you can actually handle.** If you don't know what to do about an error, let it crash — a loud crash is better than silent wrong behaviour.

---

## 8️⃣ Raising Your Own Errors

```python
def set_age(age):
    if age < 0:
        raise ValueError("Age cannot be negative")
    if age > 150:
        raise ValueError("Age seems unrealistic")
    return age

try:
    set_age(-5)
except ValueError as e:
    print(f"Error: {e}")        # Error: Age cannot be negative
```

### Custom exception classes

```python
class InsufficientFundsError(Exception):
    """Raised when a withdrawal exceeds the balance."""
    pass

def withdraw(balance, amount):
    if amount > balance:
        raise InsufficientFundsError(
            f"Cannot withdraw ₦{amount:,}. Balance is only ₦{balance:,}"
        )
    return balance - amount

try:
    withdraw(1000, 5000)
except InsufficientFundsError as e:
    print(f"❌ {e}")
```

> 💡 **Why bother?** A custom exception name tells you *exactly* what went wrong when reading a crash report. `InsufficientFundsError` is far clearer than a generic `ValueError`.

---

## ✏️ Class Activities

### Activity 17.1 — File Basics *(10 min)*

1. Write a file `diary.txt` with 3 lines about your day
2. Read it back and print with line numbers
3. **Append** a 4th line
4. Print how many lines and how many words the file has
5. Print only lines containing the letter "a"

<details>
<summary>💡 Solution</summary>

```python
# 1. Write
with open("diary.txt", "w") as f:
    f.write("Woke up early today\n")
    f.write("Learned Python file handling\n")
    f.write("Feeling accomplished\n")

# 2. Read with line numbers
print("--- DIARY ---")
with open("diary.txt") as f:
    for i, line in enumerate(f, start=1):
        print(f"{i}: {line.strip()}")

# 3. Append
with open("diary.txt", "a") as f:
    f.write("Wrote my first file program\n")

# 4. Stats
with open("diary.txt") as f:
    lines = f.readlines()
print(f"\nLines: {len(lines)}")
print(f"Words: {sum(len(l.split()) for l in lines)}")

# 5. Filter
print("\nLines containing 'a':")
for line in lines:
    if "a" in line.lower():
        print(f"  {line.strip()}")
```
</details>

---

### Activity 17.2 — Safe Input *(8 min)*

Write a function `get_number(prompt)` that:
- Keeps asking until the user enters a valid number
- Handles `ValueError` gracefully
- Lets the user type `quit` to give up (return `None`)

<details>
<summary>💡 Solution</summary>

```python
def get_number(prompt):
    """Keep asking until a valid number is entered."""
    while True:
        answer = input(prompt)

        if answer.lower() == "quit":
            return None

        try:
            return float(answer)
        except ValueError:
            print("  ❌ That's not a number. Try again (or type 'quit').")


num = get_number("Enter a number: ")
if num is None:
    print("You gave up!")
else:
    print(f"You entered {num}, doubled is {num * 2}")
```
</details>

---

### Activity 17.3 — Exception Hunt *(8 min)*

Predict which exception each line raises, then write a `try/except` for each.

```python
int("hello")
10 / 0
[1, 2, 3][10]
{"a": 1}["b"]
"text" + 5
open("does_not_exist.txt")
```

<details>
<summary>💡 Answers</summary>

```python
tests = [
    (lambda: int("hello"),                 "ValueError"),
    (lambda: 10 / 0,                       "ZeroDivisionError"),
    (lambda: [1, 2, 3][10],                "IndexError"),
    (lambda: {"a": 1}["b"],                "KeyError"),
    (lambda: "text" + 5,                   "TypeError"),
    (lambda: open("does_not_exist.txt"),   "FileNotFoundError"),
]

for func, expected in tests:
    try:
        func()
    except Exception as e:
        print(f"{type(e).__name__:<20} (expected {expected})")
        print(f"   message: {e}")
```
</details>

---

### Activity 17.4 — JSON Round Trip *(10 min)*

1. Build a dictionary of 3 students with names, ages, and score lists
2. Save it to `students.json` with `indent=2`
3. Open the file in VS Code and look at it
4. Load it back in a fresh script and print each student's average
5. Add a 4th student and save again
6. Handle the case where the file doesn't exist yet

<details>
<summary>💡 Solution</summary>

```python
import json
import os

FILENAME = "students.json"

# Load existing data, or start fresh
if os.path.exists(FILENAME):
    with open(FILENAME) as f:
        students = json.load(f)
    print(f"Loaded {len(students)} students")
else:
    students = {
        "Jacqueline": {"age": 19, "scores": [85, 92, 78]},
        "Sodee":      {"age": 18, "scores": [70, 65, 80]},
        "Sese":       {"age": 20, "scores": [95, 88, 91]},
    }
    print("Starting with default students")

# Report
print("\n--- AVERAGES ---")
for name, info in students.items():
    avg = sum(info["scores"]) / len(info["scores"])
    print(f"  {name:<12} {avg:.1f}")

# Add one
students["Ada"] = {"age": 19, "scores": [55, 61, 48]}

# Save
with open(FILENAME, "w") as f:
    json.dump(students, f, indent=2)
print(f"\n💾 Saved {len(students)} students to {FILENAME}")
```
</details>

---

## 🏆 Mini Project — Contact Manager With Persistence

Upgrade your Lesson 14 phonebook so contacts **survive between runs**.

**Requirements:**
1. Load contacts from `contacts.json` at startup (handle first-run gracefully)
2. Save after every change
3. Menu: add, view, search, delete, quit
4. Validate input — no empty names, no duplicates
5. Handle all file errors without crashing

<details>
<summary>💡 Full solution</summary>

```python
"""
Contact Manager with JSON persistence.
Contacts are saved to contacts.json and reloaded on startup.
"""

import json
import os

FILENAME = "contacts.json"


def load_contacts():
    """Load contacts from disk. Returns {} if the file is missing or broken."""
    if not os.path.exists(FILENAME):
        print("📁 No saved contacts found — starting fresh.")
        return {}

    try:
        with open(FILENAME) as f:
            data = json.load(f)
        print(f"📂 Loaded {len(data)} contact(s).")
        return data
    except json.JSONDecodeError:
        print("⚠️  contacts.json is corrupted — starting fresh.")
        return {}
    except PermissionError:
        print("⚠️  No permission to read contacts.json.")
        return {}


def save_contacts(contacts):
    """Write contacts to disk."""
    try:
        with open(FILENAME, "w") as f:
            json.dump(contacts, f, indent=2)
        return True
    except PermissionError:
        print("❌ Could not save — permission denied.")
        return False


def show_all(contacts):
    if not contacts:
        print("\n📭 No contacts yet.")
        return
    print(f"\n📒 CONTACTS ({len(contacts)})")
    print("-" * 52)
    for name in sorted(contacts):
        info = contacts[name]
        print(f"  {name:<18} {info['phone']:<15} {info['email']}")
    print("-" * 52)


# ---------- MAIN PROGRAM ----------
contacts = load_contacts()

while True:
    print("\n" + "=" * 40)
    print("        CONTACT MANAGER")
    print("=" * 40)
    print("  1) Add contact")
    print("  2) View all")
    print("  3) Search")
    print("  4) Delete")
    print("  5) Quit")

    choice = input("Choose [1-5]: ").strip()

    match choice:
        case "1":
            name = input("Name: ").strip()
            if not name:
                print("❌ Name cannot be empty.")
            elif name in contacts:
                print(f"❌ '{name}' already exists.")
            else:
                phone = input("Phone: ").strip()
                email = input("Email: ").strip()
                contacts[name] = {"phone": phone, "email": email}
                if save_contacts(contacts):
                    print(f"✅ Saved {name}")

        case "2":
            show_all(contacts)

        case "3":
            term = input("Search for: ").strip().lower()
            matches = {n: i for n, i in contacts.items() if term in n.lower()}
            if matches:
                print(f"\n🔍 {len(matches)} match(es):")
                for n, i in matches.items():
                    print(f"  {n:<18} {i['phone']:<15} {i['email']}")
            else:
                print("❌ No matches.")

        case "4":
            name = input("Delete which contact? ").strip()
            if name in contacts:
                if input(f"Really delete {name}? (y/n) ").lower() == "y":
                    del contacts[name]
                    save_contacts(contacts)
                    print(f"🗑️  Deleted {name}")
            else:
                print("❌ Not found.")

        case "5":
            save_contacts(contacts)
            print("💾 Saved. Goodbye! 👋")
            break

        case _:
            print("❌ Invalid choice.")
```

**Test it properly:** add a contact, quit, run the program again — your contact should still be there. 🎉
</details>

---

## 📌 Lesson 17 Cheat Sheet

```python
# --- FILES ---
with open("file.txt") as f:          # read (default)
    text = f.read()                  # whole file as string
    lines = f.readlines()            # list of lines
    for line in f: ...               # best for big files

with open("file.txt", "w") as f:     # WRITE — erases first!
    f.write("text\n")

with open("file.txt", "a") as f:     # APPEND — safe
    f.write("more\n")

line.strip()                         # remove trailing \n

# --- JSON ---
import json
json.dump(data, file, indent=2)      # dict → file
data = json.load(file)               # file → dict
json.dumps(data)                     # dict → string
json.loads(text)                     # string → dict

# --- CSV ---
import csv
csv.writer(f).writerows(rows)
for row in csv.reader(f): ...
for row in csv.DictReader(f):        # row["column_name"]
    ...

# --- ERRORS ---
try:
    risky()
except ValueError as e:
    print(e)
except (TypeError, KeyError):
    ...
else:
    ...          # ran with no errors
finally:
    ...          # always runs

raise ValueError("message")

class MyError(Exception):
    pass

# Common exceptions
ValueError  TypeError  ZeroDivisionError
FileNotFoundError  KeyError  IndexError  PermissionError

# Check a file exists
import os
os.path.exists("file.txt")
```

---

## ✅ Self-Check

- [ ] I always use `with open(...)` instead of bare `open()`
- [ ] I know `"w"` erases a file and `"a"` appends
- [ ] I can save and load a dictionary using JSON
- [ ] I know the difference between `json.dump` and `json.dumps`
- [ ] I can name 5 exceptions and what causes each
- [ ] I know why bare `except: pass` is dangerous
- [ ] I can raise my own exception with a helpful message
- [ ] My contact manager saves data between runs

---

## 📚 Homework

1. Add an "export to CSV" option to your contact manager
2. Write a program that reads a text file and reports the 5 most common words (combine Lesson 14's counting with file reading)
3. Write a function that safely reads a JSON config file, returning sensible defaults if it's missing or corrupt
4. Deliberately trigger 5 different exceptions and write a `try/except` for each

---

**Next:** [Lesson 18 — Classes Part 1](18-classes-part1.md) →
