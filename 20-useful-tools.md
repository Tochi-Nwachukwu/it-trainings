# Lesson 20 — Useful Python Tools

> **Week 10 · Saturday** · Prerequisites: [Lesson 19](19-classes-part2.md)

## 🎯 What You'll Learn

- Generators — handling huge data without huge memory
- Decorators — adding behaviour to functions
- Regular expressions — finding patterns in text
- Dates and times, properly

**You'll build:** A log file parser.

> 📌 These four tools are what turn "someone who knows Python" into "someone who automates things." They show up constantly in DevOps work.

---

## 1️⃣ Generators — Memory-Efficient Loops

### The problem

```python
def get_numbers(n):
    result = []
    for i in range(n):
        result.append(i)
    return result

numbers = get_numbers(10_000_000)     # 😱 ~400MB of RAM
```

You built a list of 10 million items in memory, just to loop over it once.

### The generator solution

Swap `return` for **`yield`**:

```python
def get_numbers(n):
    for i in range(n):
        yield i                        # hand back ONE item, then pause

numbers = get_numbers(10_000_000)      # 😌 almost no memory used
```

> 🚰 **Analogy:** A list is a **bucket** — you fill the whole thing first. A generator is a **tap** — water comes out one cup at a time, only when you ask.

### How `yield` works

```python
def countdown(n):
    print("  Starting...")
    while n > 0:
        yield n            # PAUSE here and hand back n
        n -= 1             # resume here on the next request
    print("  Done!")

gen = countdown(3)
print(gen)                 # <generator object countdown at 0x...>

print(next(gen))           # Starting...  then  3
print(next(gen))           # 2
print(next(gen))           # 1
```

Usually you just loop:

```python
for number in countdown(5):
    print(number, end=" ")     # 5 4 3 2 1
```

### Generator expressions

Like a list comprehension, but with **round brackets**:

```python
squares_list = [x**2 for x in range(1000000)]     # builds all 1M — slow, big
squares_gen  = (x**2 for x in range(1000000))     # builds none yet — instant

print(sum(squares_gen))     # computed one at a time
```

### The real-world use — reading huge files

```python
def read_large_file(path):
    """Yield one line at a time — works on a 50GB file."""
    with open(path) as f:
        for line in f:
            yield line.strip()

# Only one line is in memory at any moment
for line in read_large_file("huge.log"):
    if "ERROR" in line:
        print(line)
```

| | List | Generator |
|---|------|-----------|
| Memory | Holds everything | One item at a time |
| Speed to create | Slow for big data | Instant |
| Can reuse? | ✅ Yes, many times | ❌ Once only |
| Can index? | ✅ `lst[5]` | ❌ No |
| Best for | Small data you'll reuse | Big data you'll loop once |

> ⚠️ **Generators are single-use.** Loop over one twice and the second loop gets nothing.

---

## 2️⃣ Decorators — Wrapping Functions

A decorator adds behaviour **around** a function without editing it.

```python
import functools

def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        import time
        start = time.time()
        result = func(*args, **kwargs)
        print(f"⏱️  {func.__name__} took {time.time() - start:.4f}s")
        return result
    return wrapper


@timer
def slow_function():
    return sum(range(5_000_000))

slow_function()
# ⏱️  slow_function took 0.1421s
```

**What `@timer` actually means:**

```python
@timer
def my_func(): ...

# is exactly the same as:
def my_func(): ...
my_func = timer(my_func)
```

### A logging decorator

```python
def logger(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"→ {func.__name__}({args})")
        result = func(*args, **kwargs)
        print(f"← returned {result}")
        return result
    return wrapper

@logger
def add(a, b):
    return a + b

add(3, 4)
# → add((3, 4))
# ← returned 7
```

### A retry decorator — genuinely useful

```python
def retry(times=3):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, times + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    print(f"  Attempt {attempt} failed: {e}")
                    if attempt == times:
                        raise
        return wrapper
    return decorator


@retry(times=3)
def unreliable():
    import random
    if random.random() < 0.7:
        raise ConnectionError("Network glitch")
    return "Success!"
```

> 🎁 **Why `@functools.wraps(func)`?** Without it, your decorated function forgets its own name and docstring. Always include it — it's one line.

**You can stack decorators:**

```python
@timer
@logger
def process():
    ...
```

---

## 3️⃣ Regular Expressions — Pattern Matching

Regex finds patterns in text: emails, dates, IP addresses, phone numbers.

```python
import re

log = "2026-08-28 10:15:02 ERROR Database timeout from 192.168.1.50"
```

### The four functions you need

```python
# 1. search — find the FIRST match
match = re.search(r"\d{4}-\d{2}-\d{2}", log)
print(match.group())            # 2026-08-28

# 2. findall — find ALL matches as a list
print(re.findall(r"\d+\.\d+\.\d+\.\d+", log))
# ['192.168.1.50']

# 3. sub — find and replace
print(re.sub(r"\d+\.\d+\.\d+\.\d+", "[REDACTED]", log))
# 2026-08-28 10:15:02 ERROR Database timeout from [REDACTED]

# 4. match — does it start with this pattern?
print(bool(re.match(r"\d{4}", log)))    # True
```

### The pattern language

| Pattern | Matches | Example |
|---------|---------|---------|
| `\d` | Any digit | `\d{4}` → 2026 |
| `\w` | Letter, digit, or `_` | `\w+` → hello_1 |
| `\s` | Whitespace | |
| `.` | Any character | |
| `+` | One or more | `\d+` → 123 |
| `*` | Zero or more | |
| `?` | Zero or one (optional) | |
| `{3}` | Exactly 3 | `\d{3}` → 555 |
| `{2,4}` | Between 2 and 4 | |
| `[abc]` | Any of a, b, c | `[aeiou]` → vowels |
| `[^abc]` | NOT a, b, or c | |
| `^` | Start of string | `^ERROR` |
| `$` | End of string | `done$` |
| `\|` | Or | `cat\|dog` |
| `( )` | Capture group | |

> 🔑 **Always use raw strings: `r"pattern"`.** Without the `r`, Python eats the backslashes before regex sees them.

### Capture groups — extracting parts

```python
log = "2026-08-28 10:15:02 ERROR Database timeout"

pattern = r"(\d{4}-\d{2}-\d{2}) (\d{2}:\d{2}:\d{2}) (\w+) (.*)"
m = re.match(pattern, log)

if m:
    date, time, level, message = m.groups()
    print(f"Date:    {date}")
    print(f"Time:    {time}")
    print(f"Level:   {level}")
    print(f"Message: {message}")
```
```
Date:    2026-08-28
Time:    10:15:02
Level:   ERROR
Message: Database timeout
```

### Common practical patterns

```python
EMAIL  = r"^[\w.+-]+@[\w-]+\.[\w.]+$"
IP     = r"\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b"
DATE   = r"\d{4}-\d{2}-\d{2}"
PHONE  = r"^\+?\d{10,14}$"

print(bool(re.match(EMAIL, "sese@schull.io")))    # True
print(bool(re.match(EMAIL, "not-an-email")))      # False
```

> 🧪 **Tip:** use **regex101.com** to build and test patterns visually. It explains every piece of your pattern as you type.

---

## 4️⃣ Dates and Times

```python
from datetime import datetime, timedelta, date

now = datetime.now()
print(now)                                  # 2026-08-28 14:30:00.123456
print(now.year, now.month, now.day)
print(now.hour, now.minute)
```

### Formatting — `strftime` (datetime → string)

```python
print(now.strftime("%Y-%m-%d"))             # 2026-08-28
print(now.strftime("%d/%m/%Y"))             # 28/08/2026
print(now.strftime("%A, %d %B %Y"))         # Friday, 28 August 2026
print(now.strftime("%H:%M:%S"))             # 14:30:00
print(now.strftime("%I:%M %p"))             # 02:30 PM
```

### Parsing — `strptime` (string → datetime)

```python
d = datetime.strptime("2026-12-25", "%Y-%m-%d")
print(d.strftime("%A"))                     # Friday
```

> 🧠 **Remembering which is which:**
> `strf`ormat = **f**ormat it *out* to a string
> `strp`arse = **p**arse it *in* from a string

### Date maths with `timedelta`

```python
tomorrow = now + timedelta(days=1)
last_week = now - timedelta(weeks=1)
in_2_hours = now + timedelta(hours=2)

# How long between two dates?
new_year = datetime(2026, 1, 1)
diff = now - new_year
print(f"{diff.days} days since New Year")
```

### Practical examples

```python
# Age calculator
def calculate_age(birth_year, birth_month, birth_day):
    born = date(birth_year, birth_month, birth_day)
    today = date.today()
    age = today.year - born.year
    if (today.month, today.day) < (born.month, born.day):
        age -= 1
    return age

print(calculate_age(2007, 5, 15))

# Days until an event
exam = datetime(2026, 12, 1)
print(f"{(exam - datetime.now()).days} days until the exam")

# Timestamped log entry
def log(message):
    stamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    print(f"[{stamp}] {message}")

log("Server started")
```

---

## ✏️ Class Activities

### Activity 20.1 — Generator Practice *(10 min)*

Write these generators:

1. `evens(n)` — yields even numbers up to n
2. `fibonacci_gen(n)` — yields the first n Fibonacci numbers
3. `chunk(lst, size)` — yields the list in chunks of `size`
4. Compare the memory of a list vs a generator using `sys.getsizeof`

<details>
<summary>💡 Solutions</summary>

```python
import sys

def evens(n):
    for i in range(0, n + 1, 2):
        yield i

def fibonacci_gen(n):
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b

def chunk(lst, size):
    for i in range(0, len(lst), size):
        yield lst[i:i + size]


print(list(evens(10)))                    # [0, 2, 4, 6, 8, 10]
print(list(fibonacci_gen(10)))            # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
print(list(chunk([1,2,3,4,5,6,7], 3)))    # [[1,2,3], [4,5,6], [7]]

# Memory comparison
lst = [x**2 for x in range(100000)]
gen = (x**2 for x in range(100000))
print(f"\nList:      {sys.getsizeof(lst):>10,} bytes")
print(f"Generator: {sys.getsizeof(gen):>10,} bytes")
```
</details>

---

### Activity 20.2 — Build Decorators *(12 min)*

Write these decorators:

1. `@announce` — prints "Starting X..." before and "Finished X!" after
2. `@count_calls` — tracks how many times a function has been called
3. `@validate_positive` — raises `ValueError` if any argument is negative

<details>
<summary>💡 Solutions</summary>

```python
import functools

def announce(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"▶️  Starting {func.__name__}...")
        result = func(*args, **kwargs)
        print(f"✅ Finished {func.__name__}!")
        return result
    return wrapper


def count_calls(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        wrapper.calls += 1
        print(f"  ({func.__name__} has been called {wrapper.calls} time(s))")
        return func(*args, **kwargs)
    wrapper.calls = 0
    return wrapper


def validate_positive(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        for a in args:
            if isinstance(a, (int, float)) and a < 0:
                raise ValueError(f"Negative argument not allowed: {a}")
        return func(*args, **kwargs)
    return wrapper


@announce
@count_calls
def process_data(n):
    return n * 2

@validate_positive
def area(w, h):
    return w * h

process_data(5)
process_data(10)

print(area(3, 4))
try:
    area(-3, 4)
except ValueError as e:
    print(f"❌ {e}")
```
</details>

---

### Activity 20.3 — Regex Hunt *(12 min)*

Given this text, extract each item with regex:

```python
text = """
Contact Sese at sese@schull.io or call +2348012345678.
Server 192.168.1.50 went down on 2026-08-28 at 14:30.
Backup server 10.0.0.15 took over. Ticket #4471 was opened.
Email admin@example.com for details.
"""
```

Extract: all emails · all IP addresses · all dates · the ticket number · all times

<details>
<summary>💡 Solution</summary>

```python
import re

text = """
Contact Sese at sese@schull.io or call +2348012345678.
Server 192.168.1.50 went down on 2026-08-28 at 14:30.
Backup server 10.0.0.15 took over. Ticket #4471 was opened.
Email admin@example.com for details.
"""

print("Emails:  ", re.findall(r"[\w.+-]+@[\w-]+\.[\w.]+", text))
print("IPs:     ", re.findall(r"\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b", text))
print("Dates:   ", re.findall(r"\d{4}-\d{2}-\d{2}", text))
print("Ticket:  ", re.findall(r"#(\d+)", text))
print("Times:   ", re.findall(r"\b\d{2}:\d{2}\b", text))
print("Phone:   ", re.findall(r"\+\d{10,14}", text))
```
</details>

---

### Activity 20.4 — Date Practice *(8 min)*

1. Print today's date as "Friday, 28 August 2026"
2. Print the date 100 days from now
3. Calculate how many days until 31 December this year
4. Given `"25/12/2026"`, parse it and print which weekday it falls on
5. Print a timestamp in the format `[2026-08-28 14:30:00]`

<details>
<summary>💡 Solution</summary>

```python
from datetime import datetime, timedelta

now = datetime.now()

print(now.strftime("%A, %d %B %Y"))
print((now + timedelta(days=100)).strftime("%Y-%m-%d"))

end_of_year = datetime(now.year, 12, 31)
print(f"{(end_of_year - now).days} days until 31 December")

xmas = datetime.strptime("25/12/2026", "%d/%m/%Y")
print(f"Christmas 2026 is a {xmas.strftime('%A')}")

print(f"[{now.strftime('%Y-%m-%d %H:%M:%S')}]")
```
</details>

---

## 🏆 Mini Project — Log File Parser

Build `logparser.py` — combining **generators**, **regex**, **decorators**, and **datetime**.

**Requirements:**
1. Generate a sample log file if one doesn't exist
2. Read it with a **generator** (memory-safe)
3. Parse each line with **regex** into structured data
4. Time the parsing with a **decorator**
5. Report: total lines, count by level, all unique IPs, errors per hour

<details>
<summary>💡 Full solution</summary>

```python
"""
Log File Parser
Combines generators, regex, decorators, and datetime.
"""

import re
import os
import random
import functools
import time
from datetime import datetime, timedelta
from collections import Counter

LOGFILE = "server.log"

# ---------- DECORATOR ----------
def timer(func):
    """Print how long a function takes."""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"⏱️  {func.__name__} took {time.time() - start:.4f}s")
        return result
    return wrapper


# ---------- SAMPLE DATA ----------
def create_sample_log(path, lines=300):
    """Create a fake log file for testing."""
    levels = ["INFO"] * 12 + ["WARNING"] * 4 + ["ERROR"] * 3
    messages = {
        "INFO":    ["User logged in", "Request served", "Cache refreshed", "Health check OK"],
        "WARNING": ["High memory usage", "Slow query detected", "Disk at 80%"],
        "ERROR":   ["Database timeout", "Connection refused", "Auth failed"],
    }
    ips = ["192.168.1.50", "10.0.0.15", "172.16.4.2", "203.0.113.9"]

    start = datetime(2026, 8, 28, 0, 0, 0)
    with open(path, "w") as f:
        for i in range(lines):
            stamp = start + timedelta(seconds=i * 137)
            level = random.choice(levels)
            msg = random.choice(messages[level])
            ip = random.choice(ips)
            f.write(f"{stamp.strftime('%Y-%m-%d %H:%M:%S')} {level} {msg} from {ip}\n")


# ---------- GENERATOR ----------
def read_lines(path):
    """Yield one line at a time — memory-safe for huge files."""
    with open(path) as f:
        for line in f:
            line = line.strip()
            if line:
                yield line


def parse_lines(lines):
    """Yield each line parsed into a dictionary."""
    pattern = re.compile(
        r"(?P<date>\d{4}-\d{2}-\d{2}) "
        r"(?P<time>\d{2}:\d{2}:\d{2}) "
        r"(?P<level>\w+) "
        r"(?P<message>.+?) from "
        r"(?P<ip>\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})"
    )
    for line in lines:
        match = pattern.match(line)
        if match:
            yield match.groupdict()


# ---------- ANALYSIS ----------
@timer
def analyse(path):
    """Parse the log and return statistics."""
    levels = Counter()
    ips = Counter()
    errors_by_hour = Counter()
    messages = Counter()
    total = 0

    for entry in parse_lines(read_lines(path)):
        total += 1
        levels[entry["level"]] += 1
        ips[entry["ip"]] += 1
        messages[entry["message"]] += 1
        if entry["level"] == "ERROR":
            hour = entry["time"][:2]
            errors_by_hour[hour] += 1

    return {
        "total": total,
        "levels": levels,
        "ips": ips,
        "errors_by_hour": errors_by_hour,
        "messages": messages,
    }


def report(stats):
    print("\n" + "=" * 52)
    print("            LOG ANALYSIS REPORT")
    print("=" * 52)
    print(f"  Total entries: {stats['total']}")

    print("\n  BY LEVEL")
    print("  " + "-" * 40)
    for level, count in stats["levels"].most_common():
        pct = count / stats["total"] * 100
        bar = "█" * int(pct / 3)
        print(f"  {level:<10}{count:>5} ({pct:>5.1f}%)  {bar}")

    print("\n  TOP SOURCE IPs")
    print("  " + "-" * 40)
    for ip, count in stats["ips"].most_common(5):
        print(f"  {ip:<18}{count:>5}")

    print("\n  TOP MESSAGES")
    print("  " + "-" * 40)
    for msg, count in stats["messages"].most_common(5):
        print(f"  {msg:<30}{count:>5}")

    if stats["errors_by_hour"]:
        print("\n  ERRORS BY HOUR")
        print("  " + "-" * 40)
        for hour in sorted(stats["errors_by_hour"]):
            count = stats["errors_by_hour"][hour]
            print(f"  {hour}:00{'':<12}{count:>5}  {'▓' * count}")

    print("=" * 52)


# ---------- MAIN ----------
if __name__ == "__main__":
    if not os.path.exists(LOGFILE):
        print(f"📝 Creating sample log: {LOGFILE}")
        create_sample_log(LOGFILE)

    stats = analyse(LOGFILE)
    report(stats)
```

**Sample output:**
```
⏱️  analyse took 0.0038s

====================================================
            LOG ANALYSIS REPORT
====================================================
  Total entries: 300

  BY LEVEL
  ----------------------------------------
  INFO        189 ( 63.0%)  ████████████████████
  WARNING      66 ( 22.0%)  ███████
  ERROR        45 ( 15.0%)  █████

  TOP SOURCE IPs
  ----------------------------------------
  192.168.1.50         84
  10.0.0.15            79
  ...
```
</details>

---

## 📌 Lesson 20 Cheat Sheet

```python
# --- GENERATORS ---
def gen(n):
    for i in range(n):
        yield i                        # pause and hand back

(x**2 for x in range(10))              # generator expression
next(g)                                # get the next item
# single-use, memory-efficient, can't index

# --- DECORATORS ---
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

# --- REGEX ---
import re
re.search(r"pat", text)        # first match (or None)
re.findall(r"pat", text)       # all matches as a list
re.sub(r"pat", "new", text)    # replace
re.match(r"pat", text)         # match at the START only
m.group()   m.groups()   m.groupdict()

\d digit   \w word char   \s space   . any
+ one-plus   * zero-plus   ? optional
{3} exactly 3   {2,4} range
[abc] any of   [^abc] none of
^ start   $ end   | or   ( ) capture

# --- DATETIME ---
from datetime import datetime, timedelta, date
now = datetime.now()
now.strftime("%Y-%m-%d %H:%M")       # datetime → string
datetime.strptime("2026-01-01", "%Y-%m-%d")   # string → datetime
now + timedelta(days=7, hours=2)
(date_a - date_b).days

%Y year  %m month  %d day  %B month name
%A weekday  %H hour  %M minute  %S second
```

---

## ✅ Self-Check

- [ ] I can explain when a generator beats a list
- [ ] I know why generators can only be used once
- [ ] I can write a decorator with `*args, **kwargs`
- [ ] I know why `@functools.wraps` matters
- [ ] I can write a regex to find an email or an IP address
- [ ] I know the difference between `strftime` and `strptime`
- [ ] My log parser runs and produces a report

---

## 📚 Homework

1. Add a `--level ERROR` filter option to your log parser
2. Write a generator that yields only prime numbers, infinitely
3. Write a `@cache` decorator that remembers results of previous calls
4. Write a regex that validates a Nigerian phone number (`+234` followed by 10 digits)
5. Write a script that reports how many days you've been alive

---

**Next:** [Lesson 21 — Algorithm Challenges](21-algorithms.md) →
