# Lesson 14 — Dictionaries & Sets

> **Week 7 · Saturday** · Prerequisites: [Lesson 13](13-lists-and-tuples.md)

## 🎯 What You'll Learn

- Dictionaries — storing **labelled** data
- Dictionary methods and safe access with `.get()`
- Dictionary comprehensions
- Nested dictionaries (real-world data shapes)
- Sets — collections with no duplicates
- Set operations: union, intersection, difference

**You'll build:** A word frequency counter and a phonebook app.

---

## 1️⃣ The Problem Lists Can't Solve

With a list, you look things up **by position**:

```python
student = ["Sese", 18, "Python", 91]
print(student[1])     # 18 — but what IS 18? Age? Score? Room number?
```

You have to *remember* that index 1 means age. That's fragile.

A **dictionary** lets you look things up **by name**:

```python
student = {"name": "Sese", "age": 18, "course": "Python", "score": 91}
print(student["age"])     # 18 — obvious!
```

> 📖 **Analogy:** A dictionary is a real dictionary. You don't look up "the 4,182nd word" — you look up **"apple"** and get its definition. **Key → Value.**

---

## 2️⃣ Creating and Reading Dictionaries

```python
student = {
    "name": "Sese Akhinedo",
    "age": 18,
    "course": "Python",
    "score": 91
}

print(student["name"])       # Sese Akhinedo
print(len(student))          # 4
print(type(student))         # <class 'dict'>

empty = {}                   # empty dictionary
```

**The structure:**

```
{  "name"  :  "Sese"  ,  "age"  :  18  }
    └─key     └─value     └─key    └─value
```

### ⚠️ Missing keys crash — use `.get()`

```python
print(student["grade"])          # ❌ KeyError: 'grade'

print(student.get("grade"))              # None       — safe!
print(student.get("grade", "N/A"))       # N/A        — with a default
```

> 🛡️ **Rule:** if you're not 100% sure a key exists, use `.get()`.

---

## 3️⃣ Modifying Dictionaries

```python
student = {"name": "Sese", "age": 18}

student["course"] = "Python"      # add a new key
student["age"] = 19               # update an existing key
print(student)
# {'name': 'Sese', 'age': 19, 'course': 'Python'}

del student["course"]             # delete a key
removed = student.pop("age")      # delete AND return the value
print(removed)                    # 19

student.update({"city": "PH", "level": 2})    # add several at once
```

---

## 4️⃣ Looping Over Dictionaries

```python
student = {"name": "Sese", "age": 18, "course": "Python"}

# Keys only
for key in student:
    print(key)

# Values only
for value in student.values():
    print(value)

# BOTH — this is the one you'll use most
for key, value in student.items():
    print(f"{key}: {value}")
```
```
name: Sese
age: 18
course: Python
```

**Getting them as lists:**

```python
print(list(student.keys()))     # ['name', 'age', 'course']
print(list(student.values()))   # ['Sese', 18, 'Python']
print(list(student.items()))    # [('name', 'Sese'), ('age', 18), ...]
```

**Checking membership:**

```python
print("name" in student)        # True   ← checks KEYS
print("Sese" in student)        # False  ← not values!
print("Sese" in student.values())   # True
```

---

## 5️⃣ Counting — The Killer Use Case

Dictionaries are perfect for counting things.

```python
words = "the cat sat on the mat the cat".split()

counts = {}
for word in words:
    counts[word] = counts.get(word, 0) + 1

print(counts)
# {'the': 3, 'cat': 2, 'sat': 1, 'on': 1, 'mat': 1}
```

> 🧠 **How `counts.get(word, 0) + 1` works:**
> - First time seeing "the" → `.get()` returns the default `0` → store `0 + 1 = 1`
> - Second time → `.get()` returns `1` → store `1 + 1 = 2`
>
> This one line is a genuinely famous Python idiom. Learn it.

**Finding the most common:**

```python
most_common = max(counts, key=counts.get)
print(f"Most common: '{most_common}' ({counts[most_common]} times)")
```

**Sorting by count:**

```python
for word, count in sorted(counts.items(), key=lambda pair: pair[1], reverse=True):
    print(f"{word:<6} {count}")
```

**Or use the built-in shortcut:**

```python
from collections import Counter

counts = Counter(words)
print(counts.most_common(3))
# [('the', 3), ('cat', 2), ('sat', 1)]
```

---

## 6️⃣ Dictionary Comprehensions

Same idea as list comprehensions, but building a dictionary.

```python
squares = {x: x**2 for x in range(1, 6)}
print(squares)
# {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}

# Filter an existing dict
scores = {"Ada": 85, "Bola": 45, "Chidi": 92, "Dele": 60}
passed = {name: s for name, s in scores.items() if s >= 60}
print(passed)
# {'Ada': 85, 'Chidi': 92, 'Dele': 60}

# Transform values
doubled = {k: v * 2 for k, v in scores.items()}
```

---

## 7️⃣ Nested Dictionaries — Real-World Data

This is what real data actually looks like (JSON, APIs, config files):

```python
students = {
    "s001": {"name": "Jacqueline", "score": 85, "subjects": ["Python", "Linux"]},
    "s002": {"name": "Sodee",      "score": 78, "subjects": ["Python"]},
    "s003": {"name": "Sese",       "score": 91, "subjects": ["Python", "AWS"]},
}

# Reading deep values
print(students["s001"]["name"])              # Jacqueline
print(students["s003"]["subjects"][1])       # AWS

# Looping
for sid, info in students.items():
    subjects = ", ".join(info["subjects"])
    print(f"{sid}: {info['name']:<12} {info['score']:>3}  [{subjects}]")
```
```
s001: Jacqueline    85  [Python, Linux]
s002: Sodee         78  [Python]
s003: Sese          91  [Python, AWS]
```

**Finding the top scorer:**

```python
top = max(students.values(), key=lambda s: s["score"])
print(f"Top scorer: {top['name']} with {top['score']}")
```

---

## 8️⃣ Sets — No Duplicates Allowed

A **set** is an unordered collection where **every item is unique**.

```python
numbers = {3, 1, 2, 3, 1, 2}
print(numbers)        # {1, 2, 3}   ← duplicates vanished automatically
print(len(numbers))   # 3
```

> 🎟️ **Analogy:** A set is a **guest list**. You're either on it or you're not — you can't be on it twice, and the order doesn't matter.

### Creating sets

```python
s = {1, 2, 3}
s = set([1, 2, 2, 3])        # from a list — removes duplicates
empty = set()                # ⚠️ NOT {} — that's an empty dictionary!
```

### The #1 use — removing duplicates

```python
names = ["Ada", "Bola", "Ada", "Chidi", "Bola"]
unique = list(set(names))
print(unique)          # ['Ada', 'Bola', 'Chidi']  (order may vary)
print(len(set(names))) # 3 unique names
```

### Set methods

```python
s = {1, 2, 3}

s.add(4)              # add one item
s.discard(99)         # remove if present — no error if missing
s.remove(1)           # remove — ERROR if missing
popped = s.pop()      # remove a random item
```

---

## 9️⃣ Set Operations — The Maths Bit

```python
python_students = {"Ada", "Bola", "Chidi", "Dele"}
linux_students  = {"Chidi", "Dele", "Emeka", "Femi"}
```

| Operation | Symbol | Method | Meaning |
|-----------|--------|--------|---------|
| Union | `\|` | `.union()` | Everyone in **either** |
| Intersection | `&` | `.intersection()` | Only those in **both** |
| Difference | `-` | `.difference()` | In the first, **not** the second |
| Symmetric diff | `^` | `.symmetric_difference()` | In one but **not both** |

```python
print(python_students | linux_students)
# {'Ada', 'Bola', 'Chidi', 'Dele', 'Emeka', 'Femi'}      — all students

print(python_students & linux_students)
# {'Chidi', 'Dele'}                                       — doing BOTH courses

print(python_students - linux_students)
# {'Ada', 'Bola'}                                         — Python only

print(python_students ^ linux_students)
# {'Ada', 'Bola', 'Emeka', 'Femi'}                        — only one course
```

> 🎯 **When to use a set instead of a list:**
> | Need | Use |
> |------|-----|
> | Keep order, allow duplicates | **list** |
> | Remove duplicates | **set** |
> | Fast "is this in here?" check | **set** (much faster on big data) |
> | Compare two groups | **set** |

---

## ✏️ Class Activities

### Activity 14.1 — Dictionary Basics *(8 min)*

Create a dictionary for yourself with `name`, `age`, `city`, `hobbies` (a list). Then:

1. Print your name
2. Safely try to print a `job` key that doesn't exist (no crash!)
3. Add a `course` key
4. Update your age
5. Print all keys, then all values
6. Loop through and print every key-value pair
7. Print how many hobbies you have

<details>
<summary>💡 Solution</summary>

```python
me = {
    "name": "Sodee Peterside",
    "age": 19,
    "city": "Port Harcourt",
    "hobbies": ["football", "coding", "music"]
}

print(me["name"])
print(me.get("job", "Not specified"))

me["course"] = "Python"
me["age"] = 20

print(list(me.keys()))
print(list(me.values()))

for key, value in me.items():
    print(f"  {key}: {value}")

print(f"Hobbies: {len(me['hobbies'])}")
```
</details>

---

### Activity 14.2 — Word Frequency Counter *(12 min)*

Write `wordcount.py` that:

1. Takes a sentence from the user
2. Counts how often each word appears (case-insensitive)
3. Prints results sorted by count, highest first
4. Reports the most common word and the number of unique words

<details>
<summary>💡 Solution</summary>

```python
text = input("Enter a sentence: ")

words = text.lower().split()

counts = {}
for word in words:
    word = word.strip(".,!?;:")      # clean punctuation
    if word:
        counts[word] = counts.get(word, 0) + 1

print(f"\nTotal words:  {len(words)}")
print(f"Unique words: {len(counts)}")
print("\nWORD FREQUENCY")
print("-" * 25)

for word, count in sorted(counts.items(), key=lambda p: p[1], reverse=True):
    bar = "█" * count
    print(f"  {word:<12} {count}  {bar}")

top = max(counts, key=counts.get)
print(f"\nMost common: '{top}' ({counts[top]} times)")
```

**Sample run:**
```
Enter a sentence: the cat sat on the mat the cat ran

Total words:  9
Unique words: 6

WORD FREQUENCY
-------------------------
  the          3  ███
  cat          2  ██
  sat          1  █
  on           1  █
  mat          1  █
  ran          1  █

Most common: 'the' (3 times)
```
</details>

---

### Activity 14.3 — Set Operations *(8 min)*

```python
week1 = {"Ada", "Bola", "Chidi", "Dele", "Emeka"}
week2 = {"Chidi", "Dele", "Femi", "Grace"}
```

Answer using set operations:

1. Who attended **at least one** week?
2. Who attended **both** weeks?
3. Who attended week 1 but **not** week 2 (dropouts)?
4. Who is **new** in week 2?
5. Who attended **exactly one** week?
6. How many unique people total?

<details>
<summary>💡 Solution</summary>

```python
week1 = {"Ada", "Bola", "Chidi", "Dele", "Emeka"}
week2 = {"Chidi", "Dele", "Femi", "Grace"}

print("1. Either week: ", week1 | week2)
print("2. Both weeks:  ", week1 & week2)
print("3. Dropped out: ", week1 - week2)
print("4. New joiners: ", week2 - week1)
print("5. Only one week:", week1 ^ week2)
print("6. Total unique:", len(week1 | week2))
```
</details>

---

### Activity 14.4 — Nested Data Report *(10 min)*

Given this inventory, print a formatted stock report and calculate the total value.

```python
inventory = {
    "laptop":   {"price": 450000, "qty": 3},
    "mouse":    {"price": 5000,   "qty": 12},
    "keyboard": {"price": 12000,  "qty": 7},
    "monitor":  {"price": 85000,  "qty": 0},
}
```

Also flag anything out of stock.

<details>
<summary>💡 Solution</summary>

```python
inventory = {
    "laptop":   {"price": 450000, "qty": 3},
    "mouse":    {"price": 5000,   "qty": 12},
    "keyboard": {"price": 12000,  "qty": 7},
    "monitor":  {"price": 85000,  "qty": 0},
}

print(f"{'ITEM':<12}{'PRICE':>10}{'QTY':>6}{'VALUE':>12}  STATUS")
print("-" * 52)

total = 0
for item, info in inventory.items():
    value = info["price"] * info["qty"]
    total += value
    status = "⚠️ OUT OF STOCK" if info["qty"] == 0 else "✅ in stock"
    print(f"{item:<12}{info['price']:>10,}{info['qty']:>6}{value:>12,}  {status}")

print("-" * 52)
print(f"{'TOTAL VALUE':<28}{total:>12,}")
```
</details>

---

## 🏆 Mini Project — Phonebook Application

Build `phonebook.py`.

**Requirements:**
1. Store contacts in a dictionary: `{name: {"phone": ..., "email": ...}}`
2. Menu: add, view all, search, update, delete, quit
3. Search should be case-insensitive and match partial names
4. Prevent duplicate names
5. Show a friendly message when the phonebook is empty

<details>
<summary>💡 Full solution</summary>

```python
"""Phonebook Application"""

contacts = {}

def show_all():
    if not contacts:
        print("\n📭 Phonebook is empty.")
        return
    print(f"\n📒 CONTACTS ({len(contacts)})")
    print("-" * 50)
    for name in sorted(contacts):
        info = contacts[name]
        print(f"  {name:<18} {info['phone']:<15} {info['email']}")
    print("-" * 50)

while True:
    print("\n" + "=" * 40)
    print("         PHONEBOOK MANAGER")
    print("=" * 40)
    print("  1) Add contact")
    print("  2) View all contacts")
    print("  3) Search")
    print("  4) Update a contact")
    print("  5) Delete a contact")
    print("  6) Quit")

    choice = input("Choose [1-6]: ")

    match choice:
        case "1":
            name = input("Name: ").strip()
            if not name:
                print("❌ Name cannot be empty")
            elif name in contacts:
                print(f"❌ '{name}' already exists")
            else:
                phone = input("Phone: ").strip()
                email = input("Email: ").strip()
                contacts[name] = {"phone": phone, "email": email}
                print(f"✅ Added {name}")

        case "2":
            show_all()

        case "3":
            term = input("Search for: ").lower()
            found = {n: i for n, i in contacts.items() if term in n.lower()}
            if found:
                print(f"\n🔍 {len(found)} match(es):")
                for n, i in found.items():
                    print(f"  {n:<18} {i['phone']:<15} {i['email']}")
            else:
                print("❌ No matches found")

        case "4":
            name = input("Which contact? ").strip()
            if name in contacts:
                phone = input(f"New phone [{contacts[name]['phone']}]: ").strip()
                email = input(f"New email [{contacts[name]['email']}]: ").strip()
                if phone:
                    contacts[name]["phone"] = phone
                if email:
                    contacts[name]["email"] = email
                print(f"✅ Updated {name}")
            else:
                print("❌ Contact not found")

        case "5":
            name = input("Delete which contact? ").strip()
            if name in contacts:
                confirm = input(f"Really delete {name}? (y/n) ").lower()
                if confirm == "y":
                    del contacts[name]
                    print(f"🗑️  Deleted {name}")
            else:
                print("❌ Contact not found")

        case "6":
            print("👋 Goodbye!")
            break

        case _:
            print("❌ Invalid choice")
```

> 📌 Right now the contacts disappear when the program closes. In **Lesson 17** you'll learn to save them to a file so they persist!
</details>

---

## 📌 Lesson 14 Cheat Sheet

```python
# --- DICTIONARIES ---
d = {"key": "value", "age": 19}
d = {}                                # empty

d["key"]                              # read (crashes if missing)
d.get("key")                          # safe read → None
d.get("key", "default")               # safe read with fallback

d["new"] = value                      # add / update
d.update({"a": 1, "b": 2})            # add several
del d["key"]                          # delete
d.pop("key")                          # delete and return

d.keys()   d.values()   d.items()
for k, v in d.items(): ...
"key" in d                            # checks KEYS

# Counting idiom — memorise this
counts[item] = counts.get(item, 0) + 1

# Comprehension
{x: x**2 for x in range(5)}
{k: v for k, v in d.items() if v > 10}

# Nested
data["outer"]["inner"]

# --- SETS ---
s = {1, 2, 3}
s = set([1, 2, 2])                    # dedupe a list
empty = set()                         # NOT {}

s.add(x)    s.discard(x)    s.remove(x)

a | b     # union         — either
a & b     # intersection  — both
a - b     # difference    — in a, not b
a ^ b     # symmetric     — one but not both

list(set(my_list))                    # remove duplicates
```

---

## ✅ Self-Check

- [ ] I know when a dictionary beats a list
- [ ] I use `.get()` instead of `[]` when a key might be missing
- [ ] I can write the counting idiom from memory
- [ ] I can loop with `.items()` to get keys and values together
- [ ] I can read a value from a nested dictionary
- [ ] I know all four set operations and what each returns
- [ ] I know why `empty = {}` makes a dict, not a set
- [ ] My phonebook app works

---

## 📚 Homework

1. Add a "count contacts by first letter" feature to your phonebook
2. Write a program that finds all words appearing in **both** of two sentences (use sets!)
3. Build a simple grade tracker: `{student: [score1, score2, score3]}` and print each student's average
4. Given a list with duplicates, write a one-liner that returns only the items appearing more than once

---

**Next:** [Lesson 15 — Functions](15-functions.md) →
