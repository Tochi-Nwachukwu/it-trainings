# Lesson 19 — Classes, Part 2

> **Week 10 · Friday** · Prerequisites: [Lesson 18](18-classes-part1.md)

## 🎯 What You'll Learn

- Inheritance — reusing a class you already wrote
- `super()` — calling the parent's version
- Overriding methods
- Polymorphism — one interface, many behaviours
- Dunder methods (`__len__`, `__eq__`, `__add__`, `__getitem__`)
- Composition — the *other* way to reuse code

**You'll build:** An employee management system.

> 📌 **Note:** we're deliberately skipping multiple inheritance, MRO, and abstract base classes. They're real, but you don't need them yet — and they cause more beginner bugs than they solve.

---

## 1️⃣ The Repetition Problem

Imagine three classes:

```python
class Dog:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    def eat(self):
        return f"{self.name} is eating"
    def sleep(self):
        return f"{self.name} is sleeping"
    def speak(self):
        return "Woof!"

class Cat:
    def __init__(self, name, age):     # 🔁 identical
        self.name = name
        self.age = age
    def eat(self):                     # 🔁 identical
        return f"{self.name} is eating"
    def sleep(self):                   # 🔁 identical
        return f"{self.name} is sleeping"
    def speak(self):
        return "Meow!"
```

Only `speak()` differs. Everything else is copy-paste. Fix a bug in `eat()` and you must fix it in every class.

---

## 2️⃣ Inheritance — Write It Once

```python
class Animal:                          # PARENT (base class)
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def eat(self):
        return f"{self.name} is eating"

    def sleep(self):
        return f"{self.name} is sleeping"

    def speak(self):
        return "..."


class Dog(Animal):                     # CHILD — inherits everything
    def speak(self):                   # override just this
        return "Woof!"


class Cat(Animal):
    def speak(self):
        return "Meow!"


d = Dog("Buddy", 3)
print(d.eat())         # Buddy is eating   ← inherited from Animal
print(d.speak())       # Woof!             ← Dog's own version
```

> 🧬 **Analogy:** You inherit your parents' eye colour and height, but you have your own name. A child class inherits everything, then customises what it needs.

**The syntax:**
```python
class Child(Parent):
    ...
```

---

## 3️⃣ `super()` — Calling the Parent's Version

When a child needs the parent's setup **plus** extra:

```python
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary

    def describe(self):
        return f"{self.name} earns ₦{self.salary:,}"


class Manager(Employee):
    def __init__(self, name, salary, team_size):
        super().__init__(name, salary)      # ← run the parent's __init__ first
        self.team_size = team_size          # ← then add our own

    def describe(self):
        base = super().describe()           # ← get the parent's answer
        return base + f" and manages {self.team_size} people"


m = Manager("Bola", 400000, 5)
print(m.describe())
# Bola earns ₦400,000 and manages 5 people
```

> 🔑 **`super()` means "the parent version of this."**
> Use it in `__init__` so you don't repeat setup code, and in overridden methods so you can *extend* rather than *replace*.

**Checking relationships:**

```python
print(isinstance(m, Manager))      # True
print(isinstance(m, Employee))     # True  ← a Manager IS an Employee
print(issubclass(Manager, Employee))  # True
```

---

## 4️⃣ Polymorphism — Same Call, Different Result

**Poly** = many, **morph** = form. One method name, many behaviours.

```python
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary
    def describe(self):
        return f"{self.name} — General Staff"

class Manager(Employee):
    def describe(self):
        return f"{self.name} — Manager"

class Developer(Employee):
    def describe(self):
        return f"{self.name} — Developer"


staff = [
    Employee("Ada", 200000),
    Manager("Bola", 400000),
    Developer("Chidi", 300000),
]

for person in staff:
    print(person.describe())      # ← same call, different output each time
```
```
Ada — General Staff
Bola — Manager
Chidi — Developer
```

> ✨ **This is the magic.** The loop doesn't know or care what type each object is. It just calls `.describe()` and each object does the right thing.

### Duck typing

Python doesn't check types — it just tries the method.

> 🦆 *"If it walks like a duck and quacks like a duck, treat it as a duck."*

```python
class Duck:
    def speak(self): return "Quack"

class Robot:
    def speak(self): return "Beep boop"

# These are completely unrelated classes — Python doesn't care
for thing in [Duck(), Robot()]:
    print(thing.speak())
```

---

## 5️⃣ Dunder Methods — Making Objects Feel Native

"Dunder" = **d**ouble **under**score. These let your objects work with Python's built-in operators.

| Method | Enables | Example |
|--------|---------|---------|
| `__str__` | `print(obj)` | Friendly text |
| `__repr__` | `repr(obj)` | Debug text |
| `__len__` | `len(obj)` | How big is it |
| `__eq__` | `obj1 == obj2` | Equality |
| `__lt__` | `obj1 < obj2` | Less than (enables `sorted()`!) |
| `__add__` | `obj1 + obj2` | Addition |
| `__getitem__` | `obj[0]` | Indexing |
| `__contains__` | `x in obj` | Membership |

### Example — a Money class

```python
class Money:
    def __init__(self, amount):
        self.amount = amount

    def __str__(self):
        return f"₦{self.amount:,}"

    def __add__(self, other):
        return Money(self.amount + other.amount)

    def __sub__(self, other):
        return Money(self.amount - other.amount)

    def __eq__(self, other):
        return self.amount == other.amount

    def __lt__(self, other):
        return self.amount < other.amount


a = Money(1000)
b = Money(2500)

print(a + b)          # ₦3,500       ← + works!
print(b - a)          # ₦1,500
print(a == b)         # False
print(a < b)          # True
print(sorted([b, a])) # sorting works because we defined __lt__
```

### Example — a Playlist that behaves like a list

```python
class Playlist:
    def __init__(self, name):
        self.name = name
        self.songs = []

    def add(self, song):
        self.songs.append(song)
        return self

    def __len__(self):
        return len(self.songs)

    def __getitem__(self, index):
        return self.songs[index]

    def __contains__(self, song):
        return song in self.songs

    def __str__(self):
        return f"{self.name} ({len(self.songs)} songs)"


p = Playlist("Study Mix")
p.add("Song A")
p.add("Song B")
p.add("Song C")

print(len(p))               # 3          ← len() works
print(p[0])                 # Song A     ← indexing works
print("Song B" in p)        # True       ← 'in' works
for song in p:              # ← iteration works, just from __getitem__!
    print(f"  ♪ {song}")
```

> 🎁 **Notice:** we never wrote a loop method, but `for song in p` works. Python falls back to `__getitem__` with 0, 1, 2… until it runs out.

---

## 6️⃣ Composition — The Other Way

Inheritance says **"is-a"**. Composition says **"has-a"**.

```python
# Inheritance: a Manager IS-A Employee
class Manager(Employee): ...

# Composition: a Car HAS-A Engine
class Engine:
    def __init__(self, horsepower):
        self.horsepower = horsepower
    def start(self):
        return "Engine started 🔥"

class Car:
    def __init__(self, brand, horsepower):
        self.brand = brand
        self.engine = Engine(horsepower)      # ← Car CONTAINS an Engine

    def start(self):
        return f"{self.brand}: {self.engine.start()}"


c = Car("Toyota", 150)
print(c.start())                  # Toyota: Engine started 🔥
print(c.engine.horsepower)        # 150
```

**Which should you use?**

| Ask yourself | Then use |
|--------------|----------|
| "Is a Manager an Employee?" ✅ Yes | **Inheritance** |
| "Is a Car an Engine?" ❌ No, it *has* one | **Composition** |

> 💡 **Professional advice:** when in doubt, prefer composition. Deep inheritance chains become fragile and hard to follow. Inheritance is best kept to 1–2 levels.

---

## ✏️ Class Activities

### Activity 19.1 — Shape Hierarchy *(12 min)*

Build a `Shape` parent with `Circle`, `Rectangle`, and `Square` children.

Requirements:
- `Shape.__init__` takes a `name`
- Each child implements `area()` and `perimeter()`
- `Square` should inherit from `Rectangle` (a square **is a** rectangle!)
- Each has `__str__` showing name, area, perimeter
- Put them all in a list and loop — polymorphism!

<details>
<summary>💡 Solution</summary>

```python
import math

class Shape:
    def __init__(self, name):
        self.name = name

    def area(self):
        return 0

    def perimeter(self):
        return 0

    def __str__(self):
        return f"{self.name:<12} area={self.area():>8.2f}  perimeter={self.perimeter():>8.2f}"


class Circle(Shape):
    def __init__(self, radius):
        super().__init__("Circle")
        self.radius = radius

    def area(self):
        return math.pi * self.radius ** 2

    def perimeter(self):
        return 2 * math.pi * self.radius


class Rectangle(Shape):
    def __init__(self, width, height, name="Rectangle"):
        super().__init__(name)
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

    def perimeter(self):
        return 2 * (self.width + self.height)


class Square(Rectangle):              # a Square IS-A Rectangle
    def __init__(self, side):
        super().__init__(side, side, "Square")


shapes = [Circle(5), Rectangle(4, 6), Square(3)]

for s in shapes:
    print(s)

total = sum(s.area() for s in shapes)
print(f"\nTotal area: {total:.2f}")

biggest = max(shapes, key=lambda s: s.area())
print(f"Biggest: {biggest.name}")
```
</details>

---

### Activity 19.2 — Dunder Practice *(12 min)*

Create a `Vector` class representing a 2D point with `x` and `y`.

Implement: `__str__`, `__add__`, `__sub__`, `__eq__`, `__len__` (return magnitude rounded), and `__mul__` (multiply by a number).

<details>
<summary>💡 Solution</summary>

```python
import math

class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __str__(self):
        return f"Vector({self.x}, {self.y})"

    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    def __sub__(self, other):
        return Vector(self.x - other.x, self.y - other.y)

    def __mul__(self, scalar):
        return Vector(self.x * scalar, self.y * scalar)

    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

    def __len__(self):
        return round(math.sqrt(self.x ** 2 + self.y ** 2))

    def magnitude(self):
        return math.sqrt(self.x ** 2 + self.y ** 2)


a = Vector(3, 4)
b = Vector(1, 2)

print(a)                # Vector(3, 4)
print(a + b)            # Vector(4, 6)
print(a - b)            # Vector(2, 2)
print(a * 3)            # Vector(9, 12)
print(a == Vector(3,4)) # True
print(len(a))           # 5
print(f"{a.magnitude():.2f}")   # 5.00
```
</details>

---

### Activity 19.3 — Composition *(10 min)*

Build a `Computer` class made of `CPU`, `RAM`, and `Storage` objects. Each part has its own class with a `describe()` method. The `Computer` should print a full spec sheet.

<details>
<summary>💡 Solution</summary>

```python
class CPU:
    def __init__(self, brand, cores, ghz):
        self.brand, self.cores, self.ghz = brand, cores, ghz
    def describe(self):
        return f"{self.brand} — {self.cores} cores @ {self.ghz}GHz"

class RAM:
    def __init__(self, size_gb, ram_type):
        self.size_gb, self.ram_type = size_gb, ram_type
    def describe(self):
        return f"{self.size_gb}GB {self.ram_type}"

class Storage:
    def __init__(self, size_gb, kind):
        self.size_gb, self.kind = size_gb, kind
    def describe(self):
        return f"{self.size_gb}GB {self.kind}"


class Computer:
    def __init__(self, name, cpu, ram, storage):
        self.name = name
        self.cpu = cpu          # HAS-A
        self.ram = ram          # HAS-A
        self.storage = storage  # HAS-A

    def spec_sheet(self):
        print("=" * 46)
        print(f"  {self.name}")
        print("=" * 46)
        print(f"  {'CPU':<10}{self.cpu.describe()}")
        print(f"  {'RAM':<10}{self.ram.describe()}")
        print(f"  {'Storage':<10}{self.storage.describe()}")
        print("=" * 46)


pc = Computer(
    "Dev Workstation",
    CPU("Intel i7", 8, 3.6),
    RAM(16, "DDR4"),
    Storage(512, "SSD")
)
pc.spec_sheet()
```
</details>

---

## 🏆 Mini Project — Employee Management System

Build `employees.py`.

**Requirements:**

| Class | Inherits | Extra |
|-------|----------|-------|
| `Employee` | — | name, id, base salary, `annual_pay()`, `describe()` |
| `Manager` | Employee | team size, bonus in `annual_pay()` |
| `Developer` | Employee | programming language, certification bonus |
| `Intern` | Employee | duration in months, pays a stipend |

Plus a `Company` class using **composition** that holds employees and can:
- Add/remove employees
- Print a full payroll report
- Find the highest earner
- Calculate total payroll cost

<details>
<summary>💡 Full solution</summary>

```python
"""
Employee Management System
Demonstrates inheritance, polymorphism, and composition.
"""


class Employee:
    """Base class for all employees."""

    company_name = "Schull Technologies"
    employee_count = 0

    def __init__(self, name, base_salary):
        self.name = name
        self.base_salary = base_salary
        Employee.employee_count += 1
        self.employee_id = f"EMP{Employee.employee_count:04d}"

    def annual_pay(self):
        """Total pay for one year."""
        return self.base_salary * 12

    def role(self):
        return "General Staff"

    def describe(self):
        return f"{self.name} — {self.role()}"

    def __str__(self):
        return (f"{self.employee_id}  {self.name:<20}{self.role():<14}"
                f"₦{self.annual_pay():>12,.0f}")

    def __repr__(self):
        return f"{type(self).__name__}('{self.name}', {self.base_salary})"

    def __lt__(self, other):
        """Enables sorting by pay."""
        return self.annual_pay() < other.annual_pay()


class Manager(Employee):
    """A manager gets a bonus scaled by team size."""

    def __init__(self, name, base_salary, team_size):
        super().__init__(name, base_salary)
        self.team_size = team_size

    def annual_pay(self):
        bonus = self.team_size * 50_000
        return super().annual_pay() + bonus

    def role(self):
        return "Manager"

    def describe(self):
        return f"{super().describe()} of {self.team_size} people"


class Developer(Employee):
    """A developer earns extra per certification."""

    def __init__(self, name, base_salary, language, certifications=0):
        super().__init__(name, base_salary)
        self.language = language
        self.certifications = certifications

    def annual_pay(self):
        cert_bonus = self.certifications * 100_000
        return super().annual_pay() + cert_bonus

    def role(self):
        return "Developer"

    def describe(self):
        return f"{super().describe()} ({self.language}, {self.certifications} certs)"


class Intern(Employee):
    """An intern is paid a stipend for a fixed number of months."""

    def __init__(self, name, monthly_stipend, months):
        super().__init__(name, monthly_stipend)
        self.months = months

    def annual_pay(self):
        return self.base_salary * self.months

    def role(self):
        return "Intern"

    def describe(self):
        return f"{super().describe()} for {self.months} months"


class Company:
    """Holds employees — an example of COMPOSITION (has-a)."""

    def __init__(self, name):
        self.name = name
        self.employees = []

    def hire(self, employee):
        self.employees.append(employee)
        print(f"✅ Hired {employee.name} as {employee.role()}")
        return self

    def fire(self, name):
        for e in self.employees:
            if e.name == name:
                self.employees.remove(e)
                print(f"👋 {name} has left the company")
                return True
        print(f"❌ No employee named {name}")
        return False

    def total_payroll(self):
        return sum(e.annual_pay() for e in self.employees)

    def highest_earner(self):
        return max(self.employees) if self.employees else None

    def payroll_report(self):
        print("\n" + "=" * 62)
        print(f"  {self.name.upper()} — ANNUAL PAYROLL")
        print("=" * 62)
        print(f"  {'ID':<9}{'NAME':<20}{'ROLE':<14}{'ANNUAL':>14}")
        print("-" * 62)

        # sorted() works because we defined __lt__
        for e in sorted(self.employees, reverse=True):
            print(f"  {e}")

        print("-" * 62)
        print(f"  {'TOTAL':<43}₦{self.total_payroll():>12,.0f}")
        print(f"  {'HEADCOUNT':<43}{len(self.employees):>13}")
        top = self.highest_earner()
        if top:
            print(f"  {'TOP EARNER':<43}{top.name:>13}")
        print("=" * 62)

    def describe_all(self):
        print("\n--- TEAM ---")
        for e in self.employees:
            print(f"  • {e.describe()}")      # POLYMORPHISM in action


# ================= DEMO =================
if __name__ == "__main__":
    company = Company("Schull Technologies")

    company.hire(Manager("Bola Adeyemi", 400_000, 6))
    company.hire(Developer("Chidi Okafor", 300_000, "Python", 3))
    company.hire(Developer("Sese Akhinedo", 280_000, "JavaScript", 1))
    company.hire(Employee("Ada Nwosu", 200_000))
    company.hire(Intern("Sodee Peterside", 80_000, 6))

    company.describe_all()
    company.payroll_report()

    print()
    company.fire("Ada Nwosu")
    company.payroll_report()
```

**Output (abridged):**
```
--- TEAM ---
  • Bola Adeyemi — Manager of 6 people
  • Chidi Okafor — Developer (Python, 3 certs)
  • Sese Akhinedo — Developer (JavaScript, 1 certs)
  • Ada Nwosu — General Staff
  • Sodee Peterside — Intern for 6 months

==============================================================
  SCHULL TECHNOLOGIES — ANNUAL PAYROLL
==============================================================
  ID       NAME                ROLE                  ANNUAL
--------------------------------------------------------------
  EMP0001  Bola Adeyemi        Manager         ₦   5,100,000
  EMP0002  Chidi Okafor        Developer       ₦   3,900,000
  EMP0003  Sese Akhinedo       Developer       ₦   3,460,000
  EMP0004  Ada Nwosu           General Staff   ₦   2,400,000
  EMP0005  Sodee Peterside     Intern          ₦     480,000
--------------------------------------------------------------
  TOTAL                                        ₦  15,340,000
```
</details>

---

## 📌 Lesson 19 Cheat Sheet

```python
# Inheritance
class Child(Parent):
    def __init__(self, a, b):
        super().__init__(a)        # run parent's setup
        self.b = b

    def method(self):
        return super().method() + " extra"    # extend parent

# Checking types
isinstance(obj, Class)
issubclass(Child, Parent)

# Polymorphism — same call, different results
for item in mixed_list:
    item.describe()

# Dunder methods
__str__      print(obj)
__repr__     repr(obj), shown in lists
__len__      len(obj)
__eq__       obj1 == obj2
__lt__       obj1 < obj2   (enables sorted())
__add__      obj1 + obj2
__sub__      obj1 - obj2
__mul__      obj * 3
__getitem__  obj[0]  (also enables 'for x in obj')
__contains__ x in obj

# Composition — HAS-A
class Car:
    def __init__(self):
        self.engine = Engine()     # contains one

# Inheritance = IS-A     |     Composition = HAS-A
```

---

## ✅ Self-Check

- [ ] I can create a child class that inherits from a parent
- [ ] I know what `super().__init__()` does and why it matters
- [ ] I can explain polymorphism in my own words
- [ ] I can implement `__str__`, `__eq__`, and `__add__`
- [ ] I know the difference between "is-a" and "has-a"
- [ ] I can sort a list of my own objects by defining `__lt__`
- [ ] My employee system runs and prints a payroll report

---

## 📚 Homework

1. Add a `Contractor` class that's paid hourly for a fixed number of hours
2. Add a `give_raise(percent)` method to `Employee` — all children should inherit it
3. Add a `department` attribute and make `Company` able to report payroll per department
4. Add `__eq__` to `Employee` so two employees are equal if their IDs match

---

**Next:** [Lesson 20 — Useful Python Tools](20-useful-tools.md) →
