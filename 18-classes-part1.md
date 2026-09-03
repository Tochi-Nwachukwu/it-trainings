# Lesson 18 — Classes, Part 1

> **Week 9 · Saturday** · Prerequisites: [Lesson 17](17-files-and-errors.md)

## 🎯 What You'll Learn

- What objects are and why they exist
- Writing your first class
- `__init__` and `self` (demystified)
- Instance vs class variables
- Instance, class, and static methods
- Keeping data safe (encapsulation)
- `@property` for controlled access
- `__str__` and `__repr__`

**You'll build:** A bank account system.

---

## 1️⃣ The Problem Classes Solve

Say you're tracking bank accounts. With what you know now:

```python
owner1 = "Jacqueline"
balance1 = 5000

owner2 = "Sodee"
balance2 = 3000

def deposit(balance, amount):
    return balance + amount

balance1 = deposit(balance1, 1000)
```

Now add 100 accounts. Add transaction history, interest rates, account numbers. It becomes unmanageable — the **data** and the **functions that work on it** are drifting apart.

**A class bundles them together:**

```python
account1 = BankAccount("Jacqueline", 5000)
account1.deposit(1000)
print(account1.balance)
```

Clean. Each account carries its own data *and* knows how to behave.

---

## 2️⃣ Classes vs Objects

> 🍪 **Analogy:** A **class** is a cookie cutter. An **object** is a cookie.
>
> One cutter (class) → unlimited cookies (objects), each separate.

```python
class Dog:                 # ← the cutter (blueprint)
    pass

buddy = Dog()              # ← a cookie (object/instance)
rex = Dog()                # ← another, completely separate
```

| Term | Meaning |
|------|---------|
| **Class** | The blueprint / template |
| **Object** (instance) | A specific thing built from the blueprint |
| **Attribute** | Data the object holds (`account.balance`) |
| **Method** | A function the object can do (`account.deposit()`) |

---

## 3️⃣ Your First Class

```python
class Dog:
    def __init__(self, name, breed):
        self.name = name
        self.breed = breed

    def bark(self):
        return f"{self.name} says Woof!"


buddy = Dog("Buddy", "Labrador")
rex = Dog("Rex", "Poodle")

print(buddy.name)      # Buddy
print(rex.breed)       # Poodle
print(buddy.bark())    # Buddy says Woof!
```

### Understanding `__init__`

`__init__` runs **automatically** when you create an object. It's the setup function.

```python
buddy = Dog("Buddy", "Labrador")
#             │        │
#             └────────┴──► passed into __init__
```

> 🏗️ Think of `__init__` as the **assembly line**: "when you build a Dog, here's what you need and here's how to set it up."

### Understanding `self` — the part that confuses everyone

`self` means **"this particular object."**

```python
class Dog:
    def __init__(self, name):
        self.name = name        # store name ON THIS OBJECT

    def bark(self):
        return f"{self.name} says Woof!"    # read from THIS OBJECT
```

When you write `buddy.bark()`, Python secretly calls `Dog.bark(buddy)` — passing the object in as `self`.

```
buddy.bark()      →      Dog.bark(buddy)
                                   └─ becomes 'self'
```

**Rules:**
- `self` is **always** the first parameter of an instance method
- You **never** pass it manually — Python does it
- `self.something` = data belonging to this object
- `something` (no `self`) = a temporary local variable, gone when the method ends

```python
class Counter:
    def __init__(self):
        self.count = 0          # ✅ survives — belongs to the object
        temp = 99               # ❌ vanishes when __init__ finishes

    def increment(self):
        self.count += 1         # ✅ works
        # print(temp)           # ❌ NameError
```

---

## 4️⃣ Instance vs Class Variables

```python
class Student:
    school = "Schull AI Academy"      # CLASS variable — shared by ALL
    total_students = 0

    def __init__(self, name):
        self.name = name              # INSTANCE variable — unique per object
        Student.total_students += 1


a = Student("Sese")
b = Student("Sodee")

print(a.name, b.name)                 # Sese Sodee     ← different
print(a.school, b.school)             # same for both  ← shared
print(Student.total_students)         # 2
```

| | Instance variable | Class variable |
|---|-------------------|----------------|
| Defined | Inside `__init__` with `self.` | At the top of the class |
| Belongs to | One object | The class (all objects share it) |
| Example | `self.name` | `school`, `total_students` |

---

## 5️⃣ Three Kinds of Methods

```python
class Circle:
    PI = 3.14159                       # class variable
    count = 0

    def __init__(self, radius):
        self.radius = radius
        Circle.count += 1

    # 1. INSTANCE method — works with THIS object's data
    def area(self):
        return Circle.PI * self.radius ** 2

    # 2. CLASS method — works with the class itself
    @classmethod
    def how_many(cls):
        return cls.count

    # 3. STATIC method — just a utility, no object or class needed
    @staticmethod
    def is_valid_radius(r):
        return isinstance(r, (int, float)) and r > 0


c = Circle(5)
print(c.area())                     # 78.53975   ← needs an object
print(Circle.how_many())            # 1          ← called on the class
print(Circle.is_valid_radius(-2))   # False      ← pure utility
```

| Method type | First parameter | Use when |
|-------------|-----------------|----------|
| Instance | `self` | You need this object's data (most common) |
| Class | `cls` | You need class-level data |
| Static | *(none)* | It's a helper that just happens to belong here |

---

## 6️⃣ Encapsulation — Protecting Your Data

Without protection, anyone can break your object:

```python
class BankAccount:
    def __init__(self, balance):
        self.balance = balance

acc = BankAccount(1000)
acc.balance = -999999           # 😱 nothing stopped this!
```

Python uses **naming conventions** to signal intent:

| Name | Meaning | Enforced? |
|------|---------|-----------|
| `self.name` | Public — use freely | — |
| `self._name` | Protected — "internal, please don't touch" | No, just convention |
| `self.__name` | Private — Python renames it to hide it | Partially |

```python
class BankAccount:
    def __init__(self, balance):
        self._balance = balance          # "don't touch directly"

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("Deposit must be positive")
        self._balance += amount
        return self._balance

    def get_balance(self):
        return self._balance


acc = BankAccount(1000)
acc.deposit(500)
print(acc.get_balance())        # 1500
acc.deposit(-100)               # ❌ ValueError — protected!
```

> 🧠 **Python's philosophy:** "We're all consenting adults here." Python doesn't *force* privacy like Java does — the underscore is a polite sign saying *"this is internal, touch at your own risk."*

---

## 7️⃣ `@property` — The Elegant Way

Writing `get_balance()` and `set_balance()` everywhere is clunky. `@property` makes a method **look like** an attribute.

```python
class BankAccount:
    def __init__(self, balance=0):
        self._balance = balance

    @property
    def balance(self):
        """Read the balance."""
        return self._balance

    @balance.setter
    def balance(self, value):
        """Set the balance, with validation."""
        if value < 0:
            raise ValueError("Balance cannot be negative")
        self._balance = value


acc = BankAccount(1000)
print(acc.balance)          # 1000  ← looks like an attribute, runs a method!
acc.balance = 2000          # ✅ goes through the setter
acc.balance = -50           # ❌ ValueError
```

**A computed property — no stored value at all:**

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    @property
    def area(self):
        return self.width * self.height


r = Rectangle(5, 3)
print(r.area)         # 15   ← no brackets! It's a property
r.width = 10
print(r.area)         # 30   ← recalculates automatically
```

---

## 8️⃣ `__str__` and `__repr__`

By default, printing an object is ugly:

```python
print(acc)      # <__main__.BankAccount object at 0x7f8b...>
```

Fix it with `__str__`:

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self._balance = balance

    def __str__(self):
        """Friendly version — for users."""
        return f"{self.owner}: ₦{self._balance:,.2f}"

    def __repr__(self):
        """Technical version — for developers/debugging."""
        return f"BankAccount('{self.owner}', {self._balance})"


acc = BankAccount("Jacqueline", 6500)
print(acc)          # Jacqueline: ₦6,500.00        ← uses __str__
print(repr(acc))    # BankAccount('Jacqueline', 6500)  ← uses __repr__
print([acc])        # [BankAccount('Jacqueline', 6500)] ← lists use __repr__
```

| Method | Audience | Goal |
|--------|----------|------|
| `__str__` | Users | Readable and friendly |
| `__repr__` | Developers | Unambiguous, ideally code that recreates it |

> 💡 **If you only write one, write `__repr__`** — Python falls back to it when `__str__` is missing.

---

## ✏️ Class Activities

### Activity 18.1 — Your First Class *(10 min)*

Create a `Student` class with:
- `name`, `age`, and `scores` (a list)
- A method `average()` returning the mean score
- A method `add_score(score)` that validates 0–100
- A `grade` property returning A/B/C/D/F based on the average
- `__str__` that prints nicely

<details>
<summary>💡 Solution</summary>

```python
class Student:
    def __init__(self, name, age):
        self.name = name
        self.age = age
        self.scores = []

    def add_score(self, score):
        """Add a score, must be 0-100."""
        if not 0 <= score <= 100:
            raise ValueError(f"Score must be 0-100, got {score}")
        self.scores.append(score)

    def average(self):
        """Return the mean score, or 0 if none."""
        if not self.scores:
            return 0
        return sum(self.scores) / len(self.scores)

    @property
    def grade(self):
        avg = self.average()
        if avg >= 90: return "A"
        if avg >= 80: return "B"
        if avg >= 70: return "C"
        if avg >= 60: return "D"
        return "F"

    def __str__(self):
        return f"{self.name} ({self.age}) — avg {self.average():.1f}, grade {self.grade}"


s = Student("Sese", 18)
s.add_score(95)
s.add_score(88)
s.add_score(91)
print(s)                        # Sese (18) — avg 91.3, grade A
print(f"Grade: {s.grade}")

try:
    s.add_score(150)
except ValueError as e:
    print(f"Error: {e}")
```
</details>

---

### Activity 18.2 — Class vs Instance Variables *(8 min)*

Create a `Book` class where:
- Every book has its own `title` and `author`
- A **class variable** `library_name` shared by all
- A **class variable** `total_books` that counts how many exist
- A `@classmethod` `count()` returning the total
- A `@staticmethod` `is_valid_isbn(isbn)` checking it's 13 digits

<details>
<summary>💡 Solution</summary>

```python
class Book:
    library_name = "Schull Library"
    total_books = 0

    def __init__(self, title, author):
        self.title = title
        self.author = author
        Book.total_books += 1

    @classmethod
    def count(cls):
        return cls.total_books

    @staticmethod
    def is_valid_isbn(isbn):
        return isbn.isdigit() and len(isbn) == 13

    def __str__(self):
        return f"'{self.title}' by {self.author}"


b1 = Book("Python Crash Course", "Eric Matthes")
b2 = Book("Automate the Boring Stuff", "Al Sweigart")

print(b1)
print(b2)
print(f"Library: {Book.library_name}")
print(f"Total books: {Book.count()}")
print(f"Valid ISBN? {Book.is_valid_isbn('9781593279288')}")
print(f"Valid ISBN? {Book.is_valid_isbn('123')}")
```
</details>

---

### Activity 18.3 — Properties *(10 min)*

Create a `Temperature` class that stores Celsius internally but exposes:
- `celsius` — a property with a setter that rejects below −273.15
- `fahrenheit` — a computed property (getter **and** setter)
- `kelvin` — a read-only computed property

<details>
<summary>💡 Solution</summary>

```python
class Temperature:
    def __init__(self, celsius=0):
        self._celsius = 0
        self.celsius = celsius        # goes through the setter for validation

    @property
    def celsius(self):
        return self._celsius

    @celsius.setter
    def celsius(self, value):
        if value < -273.15:
            raise ValueError("Below absolute zero!")
        self._celsius = value

    @property
    def fahrenheit(self):
        return (self._celsius * 9 / 5) + 32

    @fahrenheit.setter
    def fahrenheit(self, value):
        self.celsius = (value - 32) * 5 / 9

    @property
    def kelvin(self):
        return self._celsius + 273.15      # read-only, no setter

    def __str__(self):
        return f"{self._celsius:.1f}°C = {self.fahrenheit:.1f}°F = {self.kelvin:.2f}K"


t = Temperature(25)
print(t)

t.fahrenheit = 100          # set in F, stored as C
print(t)

try:
    t.celsius = -300
except ValueError as e:
    print(f"Error: {e}")

try:
    t.kelvin = 300          # no setter!
except AttributeError:
    print("Error: kelvin is read-only")
```
</details>

---

## 🏆 Mini Project — Bank Account System

Build `bank.py`.

**Requirements:**

| Feature | Detail |
|---------|--------|
| `__init__` | owner, starting balance (default 0), auto account number |
| `deposit(amount)` | Must be positive |
| `withdraw(amount)` | Must be positive and ≤ balance |
| `balance` | Read-only `@property` |
| Transaction history | Every deposit/withdrawal logged |
| `statement()` | Print all transactions |
| Class variables | Bank name + total accounts |
| `__str__` / `__repr__` | Both implemented |
| Custom exception | `InsufficientFundsError` |

<details>
<summary>💡 Full solution</summary>

```python
"""
Bank Account System
Demonstrates classes, properties, encapsulation, and custom exceptions.
"""

from datetime import datetime


class InsufficientFundsError(Exception):
    """Raised when a withdrawal exceeds the available balance."""
    pass


class BankAccount:
    # ---- Class variables (shared by all accounts) ----
    bank_name = "Schull Bank"
    total_accounts = 0

    def __init__(self, owner, balance=0):
        if balance < 0:
            raise ValueError("Opening balance cannot be negative")

        self.owner = owner
        self._balance = balance
        self._transactions = []

        BankAccount.total_accounts += 1
        self.account_number = f"SB{BankAccount.total_accounts:05d}"

        if balance > 0:
            self._log("OPENING", balance)

    # ---- Private helper ----
    def _log(self, kind, amount):
        self._transactions.append({
            "type": kind,
            "amount": amount,
            "balance": self._balance,
            "time": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
        })

    # ---- Property ----
    @property
    def balance(self):
        """Read-only access to the balance."""
        return self._balance

    # ---- Instance methods ----
    def deposit(self, amount):
        """Add money to the account."""
        if amount <= 0:
            raise ValueError("Deposit must be positive")
        self._balance += amount
        self._log("DEPOSIT", amount)
        return self._balance

    def withdraw(self, amount):
        """Take money out, if there's enough."""
        if amount <= 0:
            raise ValueError("Withdrawal must be positive")
        if amount > self._balance:
            raise InsufficientFundsError(
                f"Cannot withdraw ₦{amount:,.2f} — balance is ₦{self._balance:,.2f}"
            )
        self._balance -= amount
        self._log("WITHDRAWAL", amount)
        return self._balance

    def statement(self):
        """Print a full transaction statement."""
        print("\n" + "=" * 58)
        print(f"  {BankAccount.bank_name.upper()} — STATEMENT")
        print("=" * 58)
        print(f"  Account : {self.account_number}")
        print(f"  Owner   : {self.owner}")
        print("-" * 58)

        if not self._transactions:
            print("  No transactions yet.")
        else:
            print(f"  {'TYPE':<12}{'AMOUNT':>14}{'BALANCE':>14}  {'TIME':<10}")
            print("-" * 58)
            for t in self._transactions:
                print(f"  {t['type']:<12}{t['amount']:>14,.2f}"
                      f"{t['balance']:>14,.2f}  {t['time'][11:]}")

        print("-" * 58)
        print(f"  {'CURRENT BALANCE':<12}{self._balance:>28,.2f}")
        print("=" * 58)

    # ---- Class & static methods ----
    @classmethod
    def account_count(cls):
        return cls.total_accounts

    @staticmethod
    def is_valid_amount(amount):
        return isinstance(amount, (int, float)) and amount > 0

    # ---- String representations ----
    def __str__(self):
        return f"{self.account_number} | {self.owner}: ₦{self._balance:,.2f}"

    def __repr__(self):
        return f"BankAccount('{self.owner}', {self._balance})"


# ================= DEMO =================
if __name__ == "__main__":
    acc1 = BankAccount("Jacqueline Anywanwu", 50000)
    acc2 = BankAccount("Sodee Peterside", 25000)

    acc1.deposit(15000)
    acc1.withdraw(8000)
    acc1.deposit(3500)

    acc1.statement()

    print(f"\nAll accounts:")
    for acc in (acc1, acc2):
        print(f"  {acc}")

    print(f"\nTotal accounts at {BankAccount.bank_name}: {BankAccount.account_count()}")

    # Error handling demos
    print("\n--- Testing error handling ---")
    try:
        acc2.withdraw(999999)
    except InsufficientFundsError as e:
        print(f"❌ {e}")

    try:
        acc2.deposit(-100)
    except ValueError as e:
        print(f"❌ {e}")

    try:
        acc2.balance = 1_000_000
    except AttributeError:
        print("❌ Balance is read-only — must use deposit/withdraw")
```

**Output:**
```
==========================================================
  SCHULL BANK — STATEMENT
==========================================================
  Account : SB00001
  Owner   : Jacqueline Anywanwu
----------------------------------------------------------
  TYPE                AMOUNT       BALANCE  TIME
----------------------------------------------------------
  OPENING          50,000.00     50,000.00  14:22:01
  DEPOSIT          15,000.00     65,000.00  14:22:01
  WITHDRAWAL        8,000.00     57,000.00  14:22:01
  DEPOSIT           3,500.00     60,500.00  14:22:01
----------------------------------------------------------
  CURRENT BALANCE                  60,500.00
==========================================================
```
</details>

---

## 📌 Lesson 18 Cheat Sheet

```python
class ClassName:
    class_var = "shared by all"          # class variable

    def __init__(self, arg):             # constructor
        self.instance_var = arg          # instance variable
        self._protected = arg            # "internal, don't touch"

    def method(self):                    # instance method
        return self.instance_var

    @classmethod
    def cls_method(cls):                 # class method
        return cls.class_var

    @staticmethod
    def util():                          # static method — no self/cls
        return "helper"

    @property
    def value(self):                     # getter — access without ()
        return self._protected

    @value.setter
    def value(self, new):                # setter — with validation
        self._protected = new

    def __str__(self):                   # for print()
        return "friendly text"

    def __repr__(self):                  # for debugging
        return "ClassName(arg)"


obj = ClassName("hello")     # create an instance
obj.method()                 # call a method
obj.value                    # property — no brackets!
ClassName.cls_method()       # call on the class
```

---

## ✅ Self-Check

- [ ] I can explain the difference between a class and an object
- [ ] I understand what `self` is and why it's the first parameter
- [ ] I know when `__init__` runs
- [ ] I can tell instance variables from class variables
- [ ] I know what `@property` does and why it's better than `get_x()`
- [ ] I can explain the difference between `__str__` and `__repr__`
- [ ] My bank account project runs and handles all its errors

---

## 📚 Homework

1. Add a `transfer(other_account, amount)` method to `BankAccount`
2. Add a `SavingsAccount` idea: same as `BankAccount` but with an `add_interest()` method
3. Create a `Rectangle` class with `area` and `perimeter` properties and a `is_square` property
4. Save your bank accounts to JSON when the program exits (combine with Lesson 17!)

---

**Next:** [Lesson 19 — Classes Part 2](19-classes-part2.md) →
