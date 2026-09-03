# Lesson 12 — Control Flow

> **Week 6 · Saturday** · Prerequisites: [Lesson 11](11-python-basics.md)

## 🎯 What You'll Learn

- Making decisions with `if` / `elif` / `else`
- The one-line `if` (ternary)
- `match` / `case` for menus
- `for` loops and the `range()` function
- `while` loops
- `break`, `continue`, `pass`

**You'll build:** A number guessing game and a menu-driven program.

---

## 1️⃣ Making Decisions — `if`

So far your programs run top to bottom, every line, every time. Real programs **choose**.

```python
age = 20

if age >= 18:
    print("You can vote")
```

**The anatomy:**

```
if  age >= 18  :
│   └─ condition └─ colon is REQUIRED
└─ keyword

    print("You can vote")
    └─ 4 spaces = "this belongs to the if"
```

### Adding `else`

```python
age = 15

if age >= 18:
    print("You can vote")
else:
    print("Too young to vote")
```

### Adding `elif` — many options

```python
score = 78

if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
elif score >= 60:
    grade = "D"
else:
    grade = "F"

print(f"Score {score} → Grade {grade}")     # Grade C
```

> ⚠️ **Order matters!** Python checks top to bottom and **stops at the first match**. If you put `score >= 60` first, a score of 95 would get a "D" — because 95 *is* ≥ 60.

### Nested conditions

```python
age = 20
has_id = True

if age >= 18:
    if has_id:
        print("✅ Welcome in")
    else:
        print("🪪 No ID, no entry")
else:
    print("🚫 Too young")
```

**Or combine with `and` / `or` — usually cleaner:**

```python
if age >= 18 and has_id:
    print("✅ Welcome in")
elif age >= 18:
    print("🪪 No ID, no entry")
else:
    print("🚫 Too young")
```

---

## 2️⃣ The One-Line `if` (Ternary)

When you just need to pick between two values:

```python
age = 20

# The long way
if age >= 18:
    status = "adult"
else:
    status = "minor"

# The short way — same thing
status = "adult" if age >= 18 else "minor"
```

Read it as: *"**status is** 'adult' **if** age ≥ 18 **else** 'minor'."*

```python
print(f"You are an {'adult' if age >= 18 else 'minor'}")
n = 7
print(f"{n} is {'even' if n % 2 == 0 else 'odd'}")
```

> 💡 Use this for **simple** either/or choices. If you need more than two options, use a full `if`/`elif`.

---

## 3️⃣ `match` / `case` — Clean Menus

New in Python 3.10. Perfect when checking **one variable against many values**.

```python
command = input("Enter command: ")

match command:
    case "start":
        print("Starting the server...")
    case "stop" | "halt":              # multiple options with |
        print("Stopping the server...")
    case "status":
        print("Server is running")
    case _:                            # _ means "anything else"
        print(f"Unknown command: {command}")
```

**Compare it to the `if` version:**

```python
# Same logic with if/elif — noticeably noisier
if command == "start":
    print("Starting the server...")
elif command == "stop" or command == "halt":
    print("Stopping the server...")
elif command == "status":
    print("Server is running")
else:
    print(f"Unknown command: {command}")
```

> 📌 `case _:` is the catch-all — like `else`. Always include it.

---

## 4️⃣ `for` Loops — Repeat a Known Number of Times

```python
for i in range(5):
    print(f"Iteration {i}")
```
```
Iteration 0
Iteration 1
Iteration 2
Iteration 3
Iteration 4
```

> 🚨 **Programming counts from 0.** `range(5)` gives you `0, 1, 2, 3, 4` — five numbers, but starting at zero. This trips up every beginner.

### The `range()` function

```python
range(5)           # 0 1 2 3 4              (stop)
range(2, 8)        # 2 3 4 5 6 7            (start, stop)
range(0, 20, 5)    # 0 5 10 15              (start, stop, step)
range(5, 0, -1)    # 5 4 3 2 1              (counting down)
```

```python
print(list(range(5)))          # [0, 1, 2, 3, 4]
print(list(range(1, 6)))       # [1, 2, 3, 4, 5]
print(list(range(0, 20, 5)))   # [0, 5, 10, 15]
```

> 🧠 **`range(a, b)` includes `a` but EXCLUDES `b`.** To count 1 to 10, write `range(1, 11)`.

### Looping over other things

```python
# Over a list
for fruit in ["apple", "banana", "cherry"]:
    print(fruit)

# Over a string — one character at a time
for letter in "Python":
    print(letter, end=" ")       # P y t h o n

# With a counter — enumerate()
for index, fruit in enumerate(["apple", "banana"], start=1):
    print(f"{index}. {fruit}")
```
```
1. apple
2. banana
```

---

## 5️⃣ `while` Loops — Repeat Until Something Changes

Use `for` when you know **how many times**. Use `while` when you know **the condition to stop**.

```python
count = 1
while count <= 5:
    print(f"Count is {count}")
    count = count + 1        # ← THIS LINE IS CRITICAL
```

> 🚨 **Every `while` loop needs something that changes the condition.** Forget `count = count + 1` and the loop runs forever. Press **`Ctrl` + `C`** to escape an infinite loop.

### The sentinel pattern — loop until the user says stop

```python
total = 0

while True:                                  # loop forever...
    entry = input("Enter a number (or 'done'): ")

    if entry == "done":                      # ...until this breaks out
        break

    total = total + int(entry)

print(f"Total: {total}")
```

> 🧭 "Sentinel" means a special value that signals *stop*. Here it's the word `"done"`.

---

## 6️⃣ Loop Control — `break`, `continue`, `pass`

| Keyword | Effect |
|---------|--------|
| `break` | **Leave the loop immediately** |
| `continue` | **Skip the rest of this round**, go to the next |
| `pass` | **Do nothing** — a placeholder |

```python
# break — stop when you find it
for i in range(1, 11):
    if i == 6:
        print(f"Found {i}! Stopping.")
        break
    print(i, end=" ")
# 1 2 3 4 5 Found 6! Stopping.
```

```python
# continue — skip the even numbers
for i in range(1, 11):
    if i % 2 == 0:
        continue
    print(i, end=" ")
# 1 3 5 7 9
```

```python
# pass — a placeholder so Python doesn't complain
for i in range(3):
    pass        # "I'll write this later"
```

### Bonus: `else` on a loop

Python has an unusual feature — a loop can have an `else` that runs **only if the loop finished without `break`**:

```python
for i in range(1, 6):
    if i == 99:
        break
else:
    print("Loop completed without breaking")   # this runs
```

> 💡 Useful for searches: *"if we never found it, say so."*

---

## 7️⃣ Nested Loops

A loop inside a loop.

```python
for i in range(1, 4):
    for j in range(1, 4):
        print(f"{i} x {j} = {i*j}")
    print("---")
```

### Multiplication table

```python
for i in range(1, 6):
    for j in range(1, 6):
        print(f"{i*j:4}", end="")
    print()
```
```
   1   2   3   4   5
   2   4   6   8  10
   3   6   9  12  15
   4   8  12  16  20
   5  10  15  20  25
```

### Star pyramid

```python
rows = 5
for i in range(1, rows + 1):
    print(" " * (rows - i) + "*" * (2 * i - 1))
```
```
    *
   ***
  *****
 *******
*********
```

> 🧠 **How to read nested loops:** the **inner** loop runs completely for **each single step** of the outer loop. Outer = rows, inner = columns.

---

## ✏️ Class Activities

### Activity 12.1 — The Bouncer *(8 min)*

Write `bouncer.py`. Ask for age and whether they have ID.

- Under 18 → "Too young"
- 18+ without ID → "No ID, no entry"
- 18+ with ID → "Welcome in!"
- 65+ → also print "Senior discount applied"

<details>
<summary>💡 Solution</summary>

```python
age = int(input("How old are you? "))
id_answer = input("Do you have ID? (yes/no) ").lower()

has_id = id_answer in ("yes", "y")

if age < 18:
    print("🚫 Too young")
elif not has_id:
    print("🪪 No ID, no entry")
else:
    print("✅ Welcome in!")
    if age >= 65:
        print("🎟️  Senior discount applied")
```
</details>

---

### Activity 12.2 — Loop Gauntlet *(12 min)*

Six challenges. Write each one.

| # | Challenge |
|---|-----------|
| 1 | Print numbers 1–20, but only the **even** ones |
| 2 | Print a countdown from 10 to 1, then "BLAST OFF!" |
| 3 | Print the 7 times table (7×1 to 7×12) |
| 4 | Use a `while` loop to sum 1 to 100 (answer: **5050**) |
| 5 | Print each letter of your name on its own line, numbered |
| 6 | Print FizzBuzz for 1–20 (multiples of 3 → "Fizz", 5 → "Buzz", both → "FizzBuzz") |

<details>
<summary>💡 Solutions</summary>

```python
# 1 — even numbers
for i in range(2, 21, 2):
    print(i, end=" ")
print()

# 2 — countdown
for i in range(10, 0, -1):
    print(i)
print("🚀 BLAST OFF!")

# 3 — times table
for i in range(1, 13):
    print(f"7 x {i:2} = {7*i:3}")

# 4 — sum with while
total = 0
n = 1
while n <= 100:
    total += n
    n += 1
print(f"Sum = {total}")

# 5 — numbered letters
name = "Sese"
for i, letter in enumerate(name, start=1):
    print(f"{i}. {letter}")

# 6 — FizzBuzz
for i in range(1, 21):
    if i % 15 == 0:
        print("FizzBuzz")
    elif i % 3 == 0:
        print("Fizz")
    elif i % 5 == 0:
        print("Buzz")
    else:
        print(i)
```

> 💡 **FizzBuzz tip:** check `% 15` **first**. If you check `% 3` first, 15 prints "Fizz" and never reaches the FizzBuzz case.
</details>

---

### Activity 12.3 — Pattern Printing *(10 min)*

Use nested loops to print these three patterns:

```
Pattern A       Pattern B       Pattern C
*               *****           *
**              ****                **
***             ***                   ***
****            **                      ****
*****           *                         *****
```

<details>
<summary>💡 Solutions</summary>

```python
n = 5

print("Pattern A")
for i in range(1, n + 1):
    print("*" * i)

print("\nPattern B")
for i in range(n, 0, -1):
    print("*" * i)

print("\nPattern C")
for i in range(1, n + 1):
    print(" " * (i - 1) * 2 + "*" * i)
```
</details>

---

## 🏆 Mini Project 1 — Number Guessing Game

Build `guess.py`.

**Requirements:**
1. Computer picks a random number 1–100
2. User gets 7 guesses
3. Say "Too high" or "Too low" each time
4. Congratulate on a correct guess and stop
5. Reveal the answer if they run out
6. Let them type `quit` to give up

<details>
<summary>💡 Full solution</summary>

```python
"""Number Guessing Game"""
import random

secret = random.randint(1, 100)
max_attempts = 7
attempts = 0

print("🎯 I'm thinking of a number between 1 and 100.")
print(f"   You have {max_attempts} attempts. Type 'quit' to give up.")
print()

while attempts < max_attempts:
    guess_text = input("Your guess: ")

    if guess_text.lower() == "quit":
        print(f"👋 The number was {secret}.")
        break

    if not guess_text.isdigit():
        print("   ⚠️  Please enter a number.")
        continue                       # doesn't waste an attempt

    guess = int(guess_text)
    attempts += 1
    left = max_attempts - attempts

    if guess < secret:
        print(f"   ⬆️  Too low!  ({left} attempts left)")
    elif guess > secret:
        print(f"   ⬇️  Too high! ({left} attempts left)")
    else:
        print(f"\n🎉 CORRECT! You got it in {attempts} attempts!")
        break
else:
    print(f"\n💀 Out of attempts! The number was {secret}.")
```

> 🔍 Note the `while...else` at the bottom — it only runs if the loop finished **without** a `break`, which is exactly "they ran out of guesses".
</details>

---

## 🏆 Mini Project 2 — Menu-Driven Program

Build `menu.py` — a program with a menu that keeps running until the user quits.

**Requirements:**
1. Show a menu with at least 4 options
2. Use `match`/`case` to handle the choice
3. Loop back to the menu after each action
4. Handle invalid choices gracefully
5. Exit cleanly on "quit"

<details>
<summary>💡 Full solution</summary>

```python
"""Simple Menu-Driven Calculator"""

def show_menu():
    print()
    print("=" * 30)
    print("      CALCULATOR MENU")
    print("=" * 30)
    print("  1) Add two numbers")
    print("  2) Multiply two numbers")
    print("  3) Check even or odd")
    print("  4) Times table")
    print("  5) Exit")
    print("=" * 30)

while True:
    show_menu()
    choice = input("Choose [1-5]: ")

    match choice:
        case "1":
            a = float(input("First number: "))
            b = float(input("Second number: "))
            print(f"→ {a} + {b} = {a + b}")

        case "2":
            a = float(input("First number: "))
            b = float(input("Second number: "))
            print(f"→ {a} × {b} = {a * b}")

        case "3":
            n = int(input("Enter a number: "))
            print(f"→ {n} is {'even' if n % 2 == 0 else 'odd'}")

        case "4":
            n = int(input("Which table? "))
            for i in range(1, 13):
                print(f"   {n} x {i:2} = {n*i:3}")

        case "5":
            print("👋 Goodbye!")
            break

        case _:
            print(f"❌ '{choice}' is not a valid option.")
```
</details>

---

## 📌 Lesson 12 Cheat Sheet

```python
# Decisions
if condition:
    ...
elif other_condition:
    ...
else:
    ...

value = "a" if condition else "b"        # ternary

match variable:
    case "x":  ...
    case "y" | "z":  ...
    case _:  ...                          # catch-all

# Loops
for i in range(5):        ...             # 0,1,2,3,4
for i in range(1, 6):     ...             # 1,2,3,4,5
for i in range(0, 20, 5): ...             # 0,5,10,15
for i in range(5, 0, -1): ...             # 5,4,3,2,1
for item in my_list:      ...
for ch in "text":         ...
for i, x in enumerate(lst, start=1): ...

while condition:
    ...                                   # must change the condition!

while True:
    if done: break                        # sentinel pattern

# Loop control
break        # leave the loop
continue     # skip to next iteration
pass         # do nothing (placeholder)

for ...:
    ...
else:
    ...      # runs only if NO break happened
```

---

## ✅ Self-Check

- [ ] I know why `range(5)` gives 0–4, not 1–5
- [ ] I can explain when to use `for` vs `while`
- [ ] I understand the difference between `break` and `continue`
- [ ] I can write a nested loop that prints a pattern
- [ ] My guessing game works, including the `quit` option
- [ ] My menu program loops until the user exits

---

## 📚 Homework

1. Add two more menu options to your menu program
2. Write a program that prints all prime numbers between 1 and 50
3. Write a "rock paper scissors" game against the computer (use `random.choice`)
4. Make the guessing game harder: after each wrong guess, print how many numbers are still possible

---

**Next:** [Lesson 13 — Lists & Tuples](13-lists-and-tuples.md) →
