# Lesson 21 — Algorithm Challenges

> **Week 11 · Friday** · Prerequisites: [Lesson 20](20-useful-tools.md)

## 🎯 What You'll Learn

- Classic number algorithms (factorial, Fibonacci, primes, GCD)
- Searching: linear vs binary search
- Sorting: bubble, selection, insertion
- String algorithms: palindromes, anagrams, reversal
- List algorithms: duplicates, missing numbers
- Big O notation — how to talk about speed

**You'll solve:** 10 classic interview problems.

> 🎯 **Why this lesson matters:** these exact problems appear in technical interviews at almost every company. More importantly, solving them trains you to *think* like a programmer.

---

## 1️⃣ Big O — Talking About Speed

Big O describes **how an algorithm slows down as the data grows**.

| Notation | Name | 10 items | 1,000 items | Example |
|----------|------|----------|-------------|---------|
| **O(1)** | Constant | 1 step | 1 step | `my_list[5]`, `my_dict["key"]` |
| **O(log n)** | Logarithmic | ~3 steps | ~10 steps | Binary search |
| **O(n)** | Linear | 10 steps | 1,000 steps | Looping through a list |
| **O(n log n)** | Linearithmic | ~33 steps | ~10,000 | Good sorting (`sorted()`) |
| **O(n²)** | Quadratic | 100 steps | 1,000,000 | Nested loops, bubble sort |

> 🏃 **Analogy:** Looking for a name in a phone book.
> - Checking every page in order = **O(n)** — slow
> - Opening in the middle and halving each time = **O(log n)** — fast
> - Comparing every name against every other name = **O(n²)** — painful

**How to spot it in your code:**

```python
# O(1) — no loop, size doesn't matter
def first_item(lst):
    return lst[0]

# O(n) — one loop over the data
def find_max(lst):
    biggest = lst[0]
    for x in lst:
        if x > biggest:
            biggest = x
    return biggest

# O(n²) — a loop inside a loop
def has_duplicate(lst):
    for i in range(len(lst)):
        for j in range(i + 1, len(lst)):
            if lst[i] == lst[j]:
                return True
    return False
```

> 💡 **The practical rule:** nested loops over the same data = O(n²) = probably too slow for big inputs. Look for a way to use a `set` or `dict` instead — those give O(1) lookups.

---

## 2️⃣ Number Algorithms

### Factorial

```python
# Iterative — usually better
def factorial(n):
    result = 1
    for i in range(2, n + 1):
        result *= i
    return result

# Recursive — more elegant, uses more memory
def factorial_rec(n):
    if n <= 1:
        return 1
    return n * factorial_rec(n - 1)

print(factorial(5))       # 120
```
**Complexity:** O(n)

---

### Fibonacci

```python
def fibonacci(n):
    """Return the first n Fibonacci numbers."""
    seq = []
    a, b = 0, 1
    for _ in range(n):
        seq.append(a)
        a, b = b, a + b
    return seq

print(fibonacci(10))
# [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```
**Complexity:** O(n)

> ⚠️ **The naive recursive version is a trap:**
> ```python
> def fib_slow(n):
>     if n < 2: return n
>     return fib_slow(n-1) + fib_slow(n-2)     # O(2ⁿ) — catastrophic
> ```
> `fib_slow(40)` takes about a minute because it recalculates the same values millions of times. The loop version is instant.

---

### Prime numbers

```python
def is_prime(n):
    """Return True if n is prime."""
    if n < 2:
        return False
    if n == 2:
        return True
    if n % 2 == 0:
        return False
    # Only check odd divisors up to √n
    for i in range(3, int(n ** 0.5) + 1, 2):
        if n % i == 0:
            return False
    return True

print([n for n in range(2, 30) if is_prime(n)])
# [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]
```
**Complexity:** O(√n)

> 🔑 **Why only check up to √n?** If `n = a × b`, one of them must be ≤ √n. So if nothing up to √n divides it, nothing will.

---

### GCD and LCM

```python
def gcd(a, b):
    """Greatest Common Divisor — Euclid's algorithm."""
    while b:
        a, b = b, a % b
    return a

def lcm(a, b):
    """Lowest Common Multiple."""
    return a * b // gcd(a, b)

print(gcd(48, 18))     # 6
print(lcm(4, 6))       # 12
```
**Complexity:** O(log n) — remarkably fast

---

## 3️⃣ Searching

### Linear search — check every item

```python
def linear_search(arr, target):
    """Return the index of target, or -1."""
    for i, value in enumerate(arr):
        if value == target:
            return i
    return -1

print(linear_search([5, 2, 9, 1, 7], 9))     # 2
```
**Complexity:** O(n) · Works on **any** list

---

### Binary search — halve it every time

> ⚠️ **Only works on a SORTED list.**

```python
def binary_search(arr, target):
    """Return the index of target in a SORTED list, or -1."""
    low, high = 0, len(arr) - 1

    while low <= high:
        mid = (low + high) // 2

        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            low = mid + 1          # search the right half
        else:
            high = mid - 1         # search the left half

    return -1

print(binary_search([1, 3, 5, 7, 9, 11], 7))    # 3
```
**Complexity:** O(log n)

**How much faster?** For 1,000,000 items:
- Linear search: up to **1,000,000** checks
- Binary search: at most **20** checks

> 🎯 **This is the classic demonstration of why algorithms matter.** Same problem, 50,000× fewer steps.

---

## 4️⃣ Sorting

You'd use `sorted()` in real code — but implementing these teaches you how algorithms work.

### Bubble sort — simplest, slowest

Repeatedly swap adjacent items that are in the wrong order.

```python
def bubble_sort(arr):
    a = arr.copy()
    n = len(a)
    for i in range(n):
        swapped = False
        for j in range(n - i - 1):
            if a[j] > a[j + 1]:
                a[j], a[j + 1] = a[j + 1], a[j]
                swapped = True
        if not swapped:        # already sorted — stop early
            break
    return a

print(bubble_sort([5, 2, 9, 1, 7]))     # [1, 2, 5, 7, 9]
```
**Complexity:** O(n²)

---

### Selection sort — find the smallest, move it to the front

```python
def selection_sort(arr):
    a = arr.copy()
    for i in range(len(a)):
        smallest = i
        for j in range(i + 1, len(a)):
            if a[j] < a[smallest]:
                smallest = j
        a[i], a[smallest] = a[smallest], a[i]
    return a
```
**Complexity:** O(n²)

---

### Insertion sort — like sorting playing cards in your hand

```python
def insertion_sort(arr):
    a = arr.copy()
    for i in range(1, len(a)):
        current = a[i]
        j = i - 1
        while j >= 0 and a[j] > current:
            a[j + 1] = a[j]
            j -= 1
        a[j + 1] = current
    return a
```
**Complexity:** O(n²), but **O(n)** if the list is nearly sorted — genuinely useful for that case.

> 🐍 **In real code, always use `sorted()`.** Python's built-in sort is Timsort — O(n log n) and written in C. You will never beat it.

---

## 5️⃣ String Algorithms

### Palindrome check

```python
def is_palindrome(text):
    """Ignore case, spaces, and punctuation."""
    clean = "".join(c.lower() for c in text if c.isalnum())
    return clean == clean[::-1]

print(is_palindrome("A man, a plan, a canal: Panama"))   # True
print(is_palindrome("hello"))                            # False
```
**Complexity:** O(n)

---

### Anagram detection

```python
def is_anagram(a, b):
    """Do both strings use exactly the same letters?"""
    a = "".join(a.lower().split())
    b = "".join(b.lower().split())
    return sorted(a) == sorted(b)

print(is_anagram("listen", "silent"))          # True
print(is_anagram("hello", "world"))            # False
```
**Complexity:** O(n log n) because of the sort

**Faster version using a dictionary — O(n):**

```python
from collections import Counter

def is_anagram_fast(a, b):
    return Counter(a.lower().replace(" ", "")) == Counter(b.lower().replace(" ", ""))
```

---

### String reversal

```python
def reverse(text):
    return text[::-1]

def reverse_words(sentence):
    return " ".join(sentence.split()[::-1])

print(reverse("Python"))                       # nohtyP
print(reverse_words("the cat sat"))            # sat cat the
```

---

### Count characters

```python
def char_frequency(text):
    counts = {}
    for ch in text.lower():
        if ch.isalpha():
            counts[ch] = counts.get(ch, 0) + 1
    return counts

print(char_frequency("hello"))     # {'h': 1, 'e': 1, 'l': 2, 'o': 1}
```

---

### First non-repeating character

```python
def first_unique(text):
    counts = {}
    for ch in text:
        counts[ch] = counts.get(ch, 0) + 1
    for ch in text:
        if counts[ch] == 1:
            return ch
    return None

print(first_unique("swiss"))     # w
```
**Complexity:** O(n) — two passes, still linear

---

## 6️⃣ List Algorithms

### Find duplicates

```python
def find_duplicates(lst):
    seen = set()
    duplicates = set()
    for item in lst:
        if item in seen:
            duplicates.add(item)
        seen.add(item)
    return sorted(duplicates)

print(find_duplicates([1, 2, 3, 2, 4, 1, 5]))    # [1, 2]
```
**Complexity:** O(n) — the `set` gives O(1) lookups

---

### Find the missing number

Given numbers 1 to n with exactly one missing:

```python
def find_missing(lst, n):
    """Use the sum formula — no loop needed!"""
    expected = n * (n + 1) // 2
    return expected - sum(lst)

print(find_missing([1, 2, 3, 4, 6, 7, 8, 9, 10], 10))    # 5
```
**Complexity:** O(n) · **Beautifully simple** — no searching required

---

### Second largest

```python
def second_largest(lst):
    unique = sorted(set(lst), reverse=True)
    return unique[1] if len(unique) > 1 else None

print(second_largest([5, 2, 9, 9, 1, 7]))    # 7
```

---

### Two sum — a famous interview question

Find two numbers that add up to a target.

```python
def two_sum(nums, target):
    """Return the indexes of two numbers that sum to target."""
    seen = {}                        # value → index
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
    return None

print(two_sum([2, 7, 11, 15], 9))    # [0, 1]
```
**Complexity:** O(n) — the naive nested-loop version is O(n²)

> 🏆 **This is the #1 most-asked coding interview question.** The trick is the dictionary: instead of searching for a partner, you *remember* what you've seen.

---

## ✏️ Class Activities

### Activity 21.1 — Five Number Challenges *(20 min)*

| # | Problem |
|---|---------|
| 1 | Write `sum_digits(n)` — sum the digits of a number (`1234` → `10`) |
| 2 | Write `is_perfect(n)` — a number equal to the sum of its divisors (`6 = 1+2+3`) |
| 3 | Write `count_primes(n)` — how many primes below n |
| 4 | Write `reverse_number(n)` — `1234` → `4321` |
| 5 | Write `is_armstrong(n)` — `153 = 1³+5³+3³` ✓ |

<details>
<summary>💡 Solutions</summary>

```python
def sum_digits(n):
    return sum(int(d) for d in str(abs(n)))

def is_perfect(n):
    if n < 2:
        return False
    divisors = [i for i in range(1, n) if n % i == 0]
    return sum(divisors) == n

def is_prime(n):
    if n < 2: return False
    for i in range(2, int(n**0.5) + 1):
        if n % i == 0: return False
    return True

def count_primes(n):
    return sum(1 for i in range(2, n) if is_prime(i))

def reverse_number(n):
    return int(str(abs(n))[::-1]) * (1 if n >= 0 else -1)

def is_armstrong(n):
    digits = str(n)
    power = len(digits)
    return n == sum(int(d) ** power for d in digits)


print(sum_digits(1234))          # 10
print(is_perfect(6))             # True
print(is_perfect(28))            # True
print(count_primes(30))          # 10
print(reverse_number(1234))      # 4321
print(is_armstrong(153))         # True
print([n for n in range(1, 1000) if is_armstrong(n)])
```
</details>

---

### Activity 21.2 — Five String Challenges *(20 min)*

| # | Problem |
|---|---------|
| 1 | `count_words(text)` — return `{word: count}` |
| 2 | `longest_word(text)` — the longest word |
| 3 | `capitalise_words(text)` — capitalise every word (no `.title()`!) |
| 4 | `remove_duplicates(text)` — keep only the first occurrence of each character |
| 5 | `is_pangram(text)` — does it use every letter a–z? |

<details>
<summary>💡 Solutions</summary>

```python
def count_words(text):
    counts = {}
    for word in text.lower().split():
        word = word.strip(".,!?;:")
        if word:
            counts[word] = counts.get(word, 0) + 1
    return counts

def longest_word(text):
    return max(text.split(), key=len)

def capitalise_words(text):
    return " ".join(w[0].upper() + w[1:] if w else w for w in text.split())

def remove_duplicates(text):
    seen = set()
    result = []
    for ch in text:
        if ch not in seen:
            seen.add(ch)
            result.append(ch)
    return "".join(result)

def is_pangram(text):
    return set("abcdefghijklmnopqrstuvwxyz") <= set(text.lower())


print(count_words("the cat the dog the bird"))
print(longest_word("Python programming is wonderful"))
print(capitalise_words("hello world from python"))
print(remove_duplicates("programming"))
print(is_pangram("The quick brown fox jumps over the lazy dog"))    # True
```
</details>

---

### Activity 21.3 — Search & Sort Race *(12 min)*

1. Implement `linear_search` and `binary_search`
2. Build a sorted list of 100,000 numbers
3. Search for a number near the end with **both**
4. Count how many comparisons each one made
5. Time both using `time.time()`

<details>
<summary>💡 Solution</summary>

```python
import time

def linear_search_counted(arr, target):
    comparisons = 0
    for i, v in enumerate(arr):
        comparisons += 1
        if v == target:
            return i, comparisons
    return -1, comparisons

def binary_search_counted(arr, target):
    comparisons = 0
    low, high = 0, len(arr) - 1
    while low <= high:
        comparisons += 1
        mid = (low + high) // 2
        if arr[mid] == target:
            return mid, comparisons
        elif arr[mid] < target:
            low = mid + 1
        else:
            high = mid - 1
    return -1, comparisons


data = list(range(100_000))
target = 99_998

start = time.time()
idx, comps = linear_search_counted(data, target)
t1 = time.time() - start
print(f"Linear:  found at {idx}, {comps:>6,} comparisons, {t1:.6f}s")

start = time.time()
idx, comps = binary_search_counted(data, target)
t2 = time.time() - start
print(f"Binary:  found at {idx}, {comps:>6,} comparisons, {t2:.6f}s")

print(f"\nBinary made {comps} comparisons instead of ~100,000!")
```

**Typical output:**
```
Linear:  found at 99998, 99,999 comparisons, 0.004521s
Binary:  found at 99998,     17 comparisons, 0.000012s

Binary made 17 comparisons instead of ~100,000!
```
</details>

---

## 🏆 Final Challenge Set — 10 Problems

Solve all ten. These are real interview questions.

| # | Problem | Difficulty |
|---|---------|------------|
| 1 | FizzBuzz for 1–100 | ⭐ |
| 2 | Reverse a string without using `[::-1]` | ⭐ |
| 3 | Find the largest number in a list without `max()` | ⭐ |
| 4 | Check if two strings are anagrams | ⭐⭐ |
| 5 | Count vowels and consonants in a sentence | ⭐⭐ |
| 6 | Find all pairs in a list that sum to a target | ⭐⭐ |
| 7 | Remove duplicates from a list, preserving order | ⭐⭐ |
| 8 | Find the first non-repeating character | ⭐⭐ |
| 9 | Merge two sorted lists into one sorted list | ⭐⭐⭐ |
| 10 | Find the longest word in a sentence, ignoring punctuation | ⭐⭐⭐ |

<details>
<summary>💡 All 10 solutions</summary>

```python
# 1 — FizzBuzz
def fizzbuzz(n=100):
    for i in range(1, n + 1):
        if i % 15 == 0:   print("FizzBuzz")
        elif i % 3 == 0:  print("Fizz")
        elif i % 5 == 0:  print("Buzz")
        else:             print(i)


# 2 — Reverse without slicing
def reverse_string(s):
    result = ""
    for ch in s:
        result = ch + result        # prepend each character
    return result


# 3 — Largest without max()
def find_largest(lst):
    if not lst:
        return None
    biggest = lst[0]
    for x in lst[1:]:
        if x > biggest:
            biggest = x
    return biggest


# 4 — Anagram
def is_anagram(a, b):
    a = "".join(a.lower().split())
    b = "".join(b.lower().split())
    return sorted(a) == sorted(b)


# 5 — Vowels and consonants
def count_letters(text):
    vowels = consonants = 0
    for ch in text.lower():
        if ch.isalpha():
            if ch in "aeiou":
                vowels += 1
            else:
                consonants += 1
    return vowels, consonants


# 6 — All pairs summing to target — O(n)
def find_pairs(nums, target):
    seen = set()
    pairs = set()
    for n in nums:
        complement = target - n
        if complement in seen:
            pairs.add(tuple(sorted((n, complement))))
        seen.add(n)
    return sorted(pairs)


# 7 — Remove duplicates, keep order
def dedupe(lst):
    seen = set()
    result = []
    for item in lst:
        if item not in seen:
            seen.add(item)
            result.append(item)
    return result


# 8 — First non-repeating character
def first_unique(text):
    counts = {}
    for ch in text:
        counts[ch] = counts.get(ch, 0) + 1
    for ch in text:
        if counts[ch] == 1:
            return ch
    return None


# 9 — Merge two sorted lists
def merge_sorted(a, b):
    result = []
    i = j = 0
    while i < len(a) and j < len(b):
        if a[i] <= b[j]:
            result.append(a[i]); i += 1
        else:
            result.append(b[j]); j += 1
    result.extend(a[i:])
    result.extend(b[j:])
    return result


# 10 — Longest word ignoring punctuation
def longest_word(sentence):
    words = ["".join(c for c in w if c.isalnum()) for w in sentence.split()]
    words = [w for w in words if w]
    return max(words, key=len) if words else None


# ---------- TESTS ----------
print(reverse_string("Python"))                          # nohtyP
print(find_largest([5, 2, 9, 1, 7]))                     # 9
print(is_anagram("listen", "silent"))                    # True
print(count_letters("Hello World"))                      # (3, 7)
print(find_pairs([1, 2, 3, 4, 5, 6], 7))                 # [(1,6),(2,5),(3,4)]
print(dedupe([1, 2, 2, 3, 1, 4]))                        # [1, 2, 3, 4]
print(first_unique("swiss"))                             # w
print(merge_sorted([1, 3, 5], [2, 4, 6]))                # [1,2,3,4,5,6]
print(longest_word("Python, programming: wonderful!"))   # programming
```
</details>

---

## 📌 Lesson 21 Cheat Sheet

```python
# --- BIG O ---
O(1)        dict/set lookup, list index
O(log n)    binary search
O(n)        single loop
O(n log n)  sorted()
O(n²)       nested loops over same data

# --- NUMBERS ---
factorial: loop multiply, or recursion with base case
fibonacci: a, b = b, a + b            (never use naive recursion)
is_prime:  check divisors up to int(n**0.5) + 1
gcd:       while b: a, b = b, a % b
lcm:       a * b // gcd(a, b)

# --- SEARCH ---
linear:  loop, O(n),     works on any list
binary:  halve, O(log n), needs a SORTED list

# --- SORT ---
bubble / selection / insertion:  O(n²) — for learning
sorted(lst):                     O(n log n) — for real code

# --- STRINGS ---
s[::-1]                          reverse
sorted(a) == sorted(b)           anagram
"".join(c for c in s if c.isalnum())    strip punctuation
s.lower()  s.split()  s.strip()

# --- LISTS ---
set(lst)                         dedupe (loses order)
n*(n+1)//2 - sum(lst)            find missing number
seen = {}                        the "remember what you've seen" trick
```

---

## ✅ Self-Check

- [ ] I can explain what O(n) and O(n²) mean
- [ ] I can write `is_prime` and explain the √n optimisation
- [ ] I can implement binary search from scratch
- [ ] I know why binary search needs a sorted list
- [ ] I can check for palindromes and anagrams
- [ ] I can solve Two Sum in O(n) using a dictionary
- [ ] I solved all 10 final challenges

---

## 📚 Homework

1. Solve 5 problems on **HackerRank** or **LeetCode** (Easy difficulty)
2. Implement merge sort (research it first — it's O(n log n))
3. Write a function that finds the **third** largest number in a list
4. Given a list of words, group the anagrams together
5. Time `bubble_sort` vs `sorted()` on 5,000 random numbers — how big is the gap?

---

## 🎓 Phase 2 Complete!

You started at "what is a variable?" and you can now:

- ✅ Write, run, and debug Python programs
- ✅ Use every core data structure: lists, tuples, dicts, sets
- ✅ Write reusable functions and organise code into packages
- ✅ Read and write files, and handle errors properly
- ✅ Design your own classes with inheritance
- ✅ Use generators, decorators, regex, and datetime
- ✅ Solve classic algorithm problems

**Next up:** Phase 3 — applying Python to DevOps automation.

---

← [Back to the course home](../README.md)
