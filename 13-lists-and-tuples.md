# Lesson 13 — Lists & Tuples

> **Week 7 · Friday** · Prerequisites: [Lesson 12](12-control-flow.md)

## 🎯 What You'll Learn

- Lists — storing many items in one variable
- Indexing and slicing
- The list methods you'll actually use
- List comprehensions (Python's superpower)
- Nested lists and grids
- Tuples and when to use them
- The copying trap that catches everyone

**You'll build:** A to-do list application.

---

## 1️⃣ Why Lists Exist

Imagine storing 5 student names:

```python
student1 = "Jacqueline"
student2 = "Sodee"
student3 = "Sese"
student4 = "Ada"
student5 = "Chidi"
```

Horrible. Now imagine 500. A **list** holds them all in one place:

```python
students = ["Jacqueline", "Sodee", "Sese", "Ada", "Chidi"]
```

> 📦 **Analogy:** A list is an **egg carton**. One container, numbered slots, each holding one item.

```python
print(students)          # ['Jacqueline', 'Sodee', 'Sese', 'Ada', 'Chidi']
print(len(students))     # 5
print(type(students))    # <class 'list'>
```

A list can hold **anything**, even mixed types:

```python
mixed = ["text", 42, 3.14, True, None]
empty = []
```

---

## 2️⃣ Indexing — Getting One Item

```python
fruits = ["apple", "banana", "cherry", "date"]
#           0         1         2        3      ← positive index
#          -4        -3        -2       -1      ← negative index
```

```python
print(fruits[0])     # apple    ← FIRST item is index 0
print(fruits[2])     # cherry
print(fruits[-1])    # date     ← LAST item
print(fruits[-2])    # cherry
```

> 🚨 **Counting starts at 0.** The 1st item is `[0]`, the 2nd is `[1]`.
> **Negative indexes count from the end.** `[-1]` is always the last item — very handy.

**Changing an item:**

```python
fruits[1] = "blueberry"
print(fruits)     # ['apple', 'blueberry', 'cherry', 'date']
```

**Going too far gives an error:**

```python
print(fruits[10])     # IndexError: list index out of range
```

---

## 3️⃣ Slicing — Getting a Section

```python
fruits = ["apple", "banana", "cherry", "date", "elderberry"]

print(fruits[1:3])     # ['banana', 'cherry']       from 1, up to (not including) 3
print(fruits[:2])      # ['apple', 'banana']        from the start
print(fruits[2:])      # ['cherry', 'date', 'elderberry']   to the end
print(fruits[:])       # a full copy
print(fruits[::2])     # every 2nd item
print(fruits[::-1])    # reversed!
```

> 🧠 **The rule: `list[start:stop]` includes `start`, excludes `stop`.**
> Think of it as *"from here, up to but not including there."*

> 💡 **`[::-1]` reverses anything** — lists, strings, tuples. Memorise it.

---

## 4️⃣ List Methods — The Ones You'll Actually Use

### Adding items

```python
fruits = ["apple", "banana"]

fruits.append("cherry")            # add ONE item to the end
print(fruits)                      # ['apple', 'banana', 'cherry']

fruits.insert(1, "blueberry")      # insert at position 1
print(fruits)                      # ['apple', 'blueberry', 'banana', 'cherry']

fruits.extend(["date", "fig"])     # add MULTIPLE items
print(fruits)                      # [..., 'date', 'fig']
```

> ⚠️ **`append` vs `extend`:**
> ```python
> a = [1, 2]; a.append([3, 4]); print(a)    # [1, 2, [3, 4]]  ← a list INSIDE
> b = [1, 2]; b.extend([3, 4]); print(b)    # [1, 2, 3, 4]    ← merged
> ```

### Removing items

```python
fruits = ["apple", "banana", "cherry", "date"]

fruits.remove("banana")       # remove by VALUE
last = fruits.pop()           # remove & return the LAST item
first = fruits.pop(0)         # remove & return item at index 0
del fruits[0]                 # delete by index
fruits.clear()                # empty the whole list
```

### Searching & counting

```python
nums = [5, 2, 9, 1, 7, 2]

print(nums.index(9))     # 2      ← position of first 9
print(nums.count(2))     # 2      ← how many 2s
print(9 in nums)         # True
print(len(nums))         # 6
print(max(nums))         # 9
print(min(nums))         # 1
print(sum(nums))         # 26
```

### Sorting

```python
nums = [5, 2, 9, 1, 7]

print(sorted(nums))                  # [1, 2, 5, 7, 9]  ← NEW list
print(sorted(nums, reverse=True))    # [9, 7, 5, 2, 1]
print(nums)                          # [5, 2, 9, 1, 7]  ← original unchanged!

nums.sort()                          # sorts IN PLACE
print(nums)                          # [1, 2, 5, 7, 9]  ← original changed

nums.reverse()
print(nums)                          # [9, 7, 5, 2, 1]
```

> 🔑 **`sorted(x)` returns a new list. `x.sort()` changes the original.**
> Use `sorted()` when you need to keep the original order too.

---

## 5️⃣ List Comprehensions — Python's Superpower

This is the feature that makes Python programmers happy.

**The long way:**
```python
squares = []
for x in range(1, 6):
    squares.append(x ** 2)
print(squares)          # [1, 4, 9, 16, 25]
```

**The Python way — one line:**
```python
squares = [x ** 2 for x in range(1, 6)]
print(squares)          # [1, 4, 9, 16, 25]
```

**The pattern:**
```
[  expression   for  item  in  collection  ]
   └─ what to      └─ the loop
      put in
```

### With a condition

```python
# Only multiples of 3
print([x for x in range(1, 21) if x % 3 == 0])
# [3, 6, 9, 12, 15, 18]

# Uppercase the names
names = ["ada", "bola", "chidi"]
print([n.upper() for n in names])
# ['ADA', 'BOLA', 'CHIDI']

# Only long words
words = ["hi", "hello", "hey", "greetings"]
print([w for w in words if len(w) > 3])
# ['hello', 'greetings']
```

### With if/else

```python
nums = [1, 2, 3, 4, 5, 6]
print(["even" if n % 2 == 0 else "odd" for n in nums])
# ['odd', 'even', 'odd', 'even', 'odd', 'even']
```

> 📌 **Position matters:**
> - Filtering (`if` only) goes at the **end**: `[x for x in list if cond]`
> - Choosing (`if/else`) goes at the **front**: `[a if cond else b for x in list]`

---

## 6️⃣ Nested Lists — Grids and Tables

A list can contain lists. That gives you a **grid**.

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

print(matrix[0])        # [1, 2, 3]   ← first row
print(matrix[1][2])     # 6           ← row 1, column 2
```

**Looping over a grid:**

```python
for row in matrix:
    for value in row:
        print(f"{value:4}", end="")
    print()
```
```
   1   2   3
   4   5   6
   7   8   9
```

**Flattening it:**

```python
flat = [n for row in matrix for n in row]
print(flat)     # [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

**Real-world grid — student records:**

```python
students = [
    ["Jacqueline", 85, 92],
    ["Sodee", 78, 88],
    ["Sese", 91, 95],
]

for name, test1, test2 in students:
    average = (test1 + test2) / 2
    print(f"{name:<12} avg: {average:.1f}")
```

---

## 7️⃣ Tuples — Lists That Can't Change

A **tuple** is a list you cannot modify. Round brackets instead of square.

```python
point = (3, 4)
colours = ("red", "green", "blue")

print(point[0])       # 3       ← indexing works
print(len(colours))   # 3       ← len works
print("red" in colours)  # True ← membership works

point[0] = 99         # ❌ TypeError: 'tuple' object does not support item assignment
```

> 🔒 **Analogy:** A list is a **whiteboard** — you can erase and rewrite. A tuple is **printed paper** — permanent.

### Why would you want that?

| Use a tuple when… | Example |
|-------------------|---------|
| The data must never change | `RGB_RED = (255, 0, 0)` |
| It represents one "thing" with fixed parts | `coordinates = (lat, lon)` |
| You want to protect against bugs | Config values |
| You need a dictionary key | Lists can't be keys, tuples can |

### Tuple unpacking — very useful

```python
point = (3, 4)
x, y = point            # unpack into two variables
print(x, y)             # 3 4

# Swap two variables in one line!
a, b = 1, 2
a, b = b, a
print(a, b)             # 2 1
```

**Functions often return tuples:**

```python
def get_stats(numbers):
    return min(numbers), max(numbers), sum(numbers) / len(numbers)

low, high, avg = get_stats([4, 8, 15, 16])
print(f"Low: {low}, High: {high}, Avg: {avg:.1f}")
```

> ⚠️ **A one-item tuple needs a trailing comma:**
> ```python
> not_a_tuple = (5)      # this is just the number 5
> real_tuple  = (5,)     # THIS is a tuple
> ```

---

## 8️⃣ ⚠️ The Copying Trap

This bug catches **every** beginner. Read carefully.

```python
a = [1, 2, 3]
b = a              # ⚠️ this does NOT make a copy!
b.append(4)

print(a)     # [1, 2, 3, 4]   ← a changed too!
print(b)     # [1, 2, 3, 4]
```

**Why?** `b = a` doesn't copy the list — it makes `b` a **second label for the same list**.

```
    a ──┐
        ├──► [1, 2, 3, 4]
    b ──┘
```

**The fix — make a real copy:**

```python
a = [1, 2, 3]
b = a.copy()       # ✅ or list(a)  or  a[:]
b.append(4)

print(a)     # [1, 2, 3]      ← safe!
print(b)     # [1, 2, 3, 4]
```

> 🚨 **Second trap: don't modify a list while looping over it.**
> ```python
> nums = [1, 2, 3, 4]
> for n in nums:
>     if n % 2 == 0:
>         nums.remove(n)     # ❌ skips items unpredictably
> ```
> **Do this instead:**
> ```python
> nums = [n for n in nums if n % 2 != 0]     # ✅ build a new list
> ```

---

## ✏️ Class Activities

### Activity 13.1 — List Workout *(10 min)*

Start with `scores = [72, 88, 95, 61, 79, 88, 54]` and answer using code:

1. How many scores are there?
2. What's the highest? Lowest? Average (2 dp)?
3. Print the scores sorted highest-first
4. How many students scored 88?
5. Add a new score of 90 to the end
6. Remove the lowest score
7. Print only the scores above 75
8. Print the first 3 and the last 2 scores

<details>
<summary>💡 Solution</summary>

```python
scores = [72, 88, 95, 61, 79, 88, 54]

print(f"1. Count:   {len(scores)}")
print(f"2. Max:     {max(scores)}")
print(f"   Min:     {min(scores)}")
print(f"   Average: {sum(scores)/len(scores):.2f}")
print(f"3. Sorted:  {sorted(scores, reverse=True)}")
print(f"4. Count of 88: {scores.count(88)}")

scores.append(90)
print(f"5. After append: {scores}")

scores.remove(min(scores))
print(f"6. After removing lowest: {scores}")

print(f"7. Above 75: {[s for s in scores if s > 75]}")
print(f"8. First 3: {scores[:3]}, Last 2: {scores[-2:]}")
```
</details>

---

### Activity 13.2 — Comprehension Challenge *(10 min)*

Write each as a **one-line list comprehension**:

| # | Task |
|---|------|
| 1 | Cubes of 1–10 |
| 2 | All even numbers from 1–50 |
| 3 | Lengths of each word in `["python", "is", "fun"]` |
| 4 | Only names starting with "S" from `["Sese", "Ada", "Sodee", "Bola"]` |
| 5 | Convert `["1", "2", "3"]` into actual numbers |
| 6 | Label each number 1–10 as "big" (>5) or "small" |

<details>
<summary>💡 Solutions</summary>

```python
print([x**3 for x in range(1, 11)])
print([x for x in range(1, 51) if x % 2 == 0])
print([len(w) for w in ["python", "is", "fun"]])
print([n for n in ["Sese", "Ada", "Sodee", "Bola"] if n.startswith("S")])
print([int(s) for s in ["1", "2", "3"]])
print(["big" if x > 5 else "small" for x in range(1, 11)])
```
</details>

---

### Activity 13.3 — Grade Book Grid *(10 min)*

Given this nested list, print a formatted report showing each student's average and grade.

```python
gradebook = [
    ["Jacqueline", 85, 92, 78],
    ["Sodee",      70, 65, 80],
    ["Sese",       95, 88, 91],
    ["Ada",        55, 61, 48],
]
```

**Expected output:**
```
NAME          T1   T2   T3    AVG  GRADE
Jacqueline    85   92   78   85.0    B
Sodee         70   65   80   71.7    C
Sese          95   88   91   91.3    A
Ada           55   61   48   54.7    F
```

<details>
<summary>💡 Solution</summary>

```python
gradebook = [
    ["Jacqueline", 85, 92, 78],
    ["Sodee",      70, 65, 80],
    ["Sese",       95, 88, 91],
    ["Ada",        55, 61, 48],
]

print(f"{'NAME':<12}{'T1':>4}{'T2':>5}{'T3':>5}{'AVG':>7}{'GRADE':>7}")

for row in gradebook:
    name = row[0]
    tests = row[1:]
    avg = sum(tests) / len(tests)

    if avg >= 90:   grade = "A"
    elif avg >= 80: grade = "B"
    elif avg >= 70: grade = "C"
    elif avg >= 60: grade = "D"
    else:           grade = "F"

    print(f"{name:<12}{tests[0]:>4}{tests[1]:>5}{tests[2]:>5}{avg:>7.1f}{grade:>7}")
```
</details>

---

### Activity 13.4 — Spot the Bug *(5 min)*

What's wrong with each snippet? Fix them.

```python
# A
shopping = ["milk", "bread"]
backup = shopping
backup.append("eggs")
print(shopping)         # why does this show eggs?

# B
nums = [1, 2, 3, 4, 5, 6]
for n in nums:
    if n % 2 == 0:
        nums.remove(n)
print(nums)             # why isn't this [1, 3, 5]?

# C
t = (5)
print(type(t))          # why isn't this a tuple?
```

<details>
<summary>💡 Answers</summary>

```python
# A — b = a doesn't copy, it aliases. Fix:
backup = shopping.copy()

# B — modifying while iterating skips items. Fix:
nums = [n for n in nums if n % 2 != 0]

# C — (5) is just 5 in brackets. A one-item tuple needs a comma. Fix:
t = (5,)
```
</details>

---

## 🏆 Mini Project — To-Do List App

Build `todo.py` — a working task manager.

**Requirements:**
1. Menu: add task, view tasks, mark done, delete task, quit
2. Store tasks as a list of lists: `[task_text, is_done]`
3. Show tasks numbered, with `[✓]` or `[ ]`
4. Handle invalid input without crashing
5. Show a count of completed vs total

<details>
<summary>💡 Full solution</summary>

```python
"""To-Do List Application"""

tasks = []       # each item: [description, done?]

def show_tasks():
    if not tasks:
        print("\n📭 No tasks yet!")
        return
    print("\n📋 YOUR TASKS")
    print("-" * 40)
    for i, (text, done) in enumerate(tasks, start=1):
        mark = "✓" if done else " "
        print(f"  {i}. [{mark}] {text}")
    completed = sum(1 for _, done in tasks if done)
    print("-" * 40)
    print(f"  {completed}/{len(tasks)} completed")

while True:
    print("\n" + "=" * 40)
    print("        TO-DO LIST MANAGER")
    print("=" * 40)
    print("  1) Add a task")
    print("  2) View tasks")
    print("  3) Mark a task done")
    print("  4) Delete a task")
    print("  5) Quit")

    choice = input("Choose [1-5]: ")

    match choice:
        case "1":
            text = input("Task description: ")
            if text.strip():
                tasks.append([text, False])
                print(f"✅ Added: {text}")
            else:
                print("❌ Task cannot be empty")

        case "2":
            show_tasks()

        case "3":
            show_tasks()
            if tasks:
                num = input("Which task number? ")
                if num.isdigit() and 1 <= int(num) <= len(tasks):
                    tasks[int(num) - 1][1] = True
                    print("✅ Marked done!")
                else:
                    print("❌ Invalid task number")

        case "4":
            show_tasks()
            if tasks:
                num = input("Delete which number? ")
                if num.isdigit() and 1 <= int(num) <= len(tasks):
                    removed = tasks.pop(int(num) - 1)
                    print(f"🗑️  Deleted: {removed[0]}")
                else:
                    print("❌ Invalid task number")

        case "5":
            print("👋 Goodbye!")
            break

        case _:
            print("❌ Invalid choice")
```
</details>

---

## 📌 Lesson 13 Cheat Sheet

```python
# Creating
lst = [1, 2, 3]
empty = []
tup = (1, 2, 3)
one_item_tuple = (5,)

# Access
lst[0]      lst[-1]      lst[1:3]     lst[:2]     lst[2:]    lst[::-1]

# Adding
lst.append(x)         # one item at the end
lst.insert(i, x)      # at position i
lst.extend([a, b])    # merge another list

# Removing
lst.remove(value)     # by value
lst.pop()             # last item (returns it)
lst.pop(i)            # item at index i
del lst[i]
lst.clear()

# Info
len(lst)   max(lst)   min(lst)   sum(lst)
lst.index(x)   lst.count(x)   x in lst

# Sorting
sorted(lst)                 # NEW list
sorted(lst, reverse=True)
lst.sort()                  # in place
lst.reverse()

# Comprehensions
[x*2 for x in lst]
[x for x in lst if x > 5]
["a" if x else "b" for x in lst]
[n for row in matrix for n in row]      # flatten

# Copying — IMPORTANT
b = a           # ❌ alias, not a copy
b = a.copy()    # ✅ real copy

# Tuple unpacking
x, y = (3, 4)
a, b = b, a     # swap
```

---

## ✅ Self-Check

- [ ] I know why `fruits[0]` is the first item, not `fruits[1]`
- [ ] I can slice a list to get any section
- [ ] I know the difference between `append` and `extend`
- [ ] I know the difference between `sorted(x)` and `x.sort()`
- [ ] I can write a list comprehension with a condition
- [ ] I understand why `b = a` is dangerous
- [ ] I know when to use a tuple instead of a list
- [ ] My to-do app runs and handles bad input

---

## 📚 Homework

1. Add a "clear all completed tasks" option to your to-do app
2. Write a program that finds the second-largest number in a list without using `sort()`
3. Given a list of names, print only those with more than 5 letters, in alphabetical order
4. Build a 5×5 multiplication grid using nested lists, then print it neatly

---

**Next:** [Lesson 14 — Dictionaries & Sets](14-dictionaries-and-sets.md) →
