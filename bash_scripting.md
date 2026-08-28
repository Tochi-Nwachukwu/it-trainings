# 🐚 Bash Scripting — Variables, Conditionals & Loops

### Class 7 · Week 4 · Friday
**Schull AI Academy — AWS Cloud DevOps & Linux Training**

**Duration:** 2 hours 30 minutes
**Prerequisites:** Classes 1–6 (Linux navigation, text processing, process management)
**Environment:** Ubuntu 24.04 LTS · Bash 5.x

---

## 🎯 Why This Class Matters

Everything you've learned so far — `grep`, `find`, `ps`, `df`, `ssh` — you've typed **one command at a time**.

Today that changes. Today you learn to **teach the computer to type for you**.

> A script is just a list of commands in a file. But once you add *variables*, *decisions*, and *repetition*, that list becomes a **program** — and you stop being someone who uses Linux and start being someone who **automates** it.

Every DevOps pipeline, every deployment, every server backup you'll ever build starts here.

---

## 📋 Lesson Roadmap

| # | Topic | Time |
|---|-------|------|
| 0 | Setup & Your First Script | 10 min |
| 1 | Script Structure — Shebang, Comments, Best Practice | 15 min |
| 2 | Variables — The Boxes That Hold Your Data | 20 min |
| 3 | Command Substitution — Capturing Output | 15 min |
| 4 | Arithmetic — Making Bash Do Maths | 20 min |
| 5 | Conditionals — Teaching Scripts to Decide | 25 min |
| 6 | Test Operators — The Questions You Can Ask | 20 min |
| 7 | Loops — Doing Things Over and Over | 25 min |
| 8 | Loop Control — break, continue, exit, sleep | 15 min |
| 9 | 🏆 Class Activity: System Health Check Script | 20 min |
| 10 | 🚀 Mini Project: Server Sentinel | 25 min |

---

# 0️⃣ Setup & Your First Script

## The Three-Step Ritual

Every single script you ever write follows the same three steps. Burn this into your memory:

```
1. WRITE it       nano myscript.sh
2. PERMIT it      chmod +x myscript.sh
3. RUN it         ./myscript.sh
```

### 🔬 Hands-On: Hello, Scripting World

```bash
mkdir -p ~/bash-class && cd ~/bash-class
nano hello.sh
```

Type this in:

```bash
#!/bin/bash
echo "Hello from my first Bash script!"
echo "I am running as: $(whoami)"
echo "The time is: $(date '+%H:%M:%S')"
```

Save with `Ctrl+O`, `Enter`, exit with `Ctrl+X`. Then:

```bash
chmod +x hello.sh
./hello.sh
```
```
Hello from my first Bash script!
I am running as: student
The time is: 19:42:11
```

🎉 **You just wrote a program.**

### ⚠️ The Four Ways Beginners Get Stuck

| Error you see | What it means | The fix |
|---------------|---------------|---------|
| `Permission denied` | You forgot step 2 | `chmod +x script.sh` |
| `command not found` | You typed `script.sh` instead of `./script.sh` | Bash won't look in the current folder unless you say `./` |
| `bad interpreter: ^M` | You wrote the file on Windows | `dos2unix script.sh` |
| Nothing happens | Your file has no `#!` line | Add `#!/bin/bash` as line 1 |

> 💡 **Why `./` ?** When you type a command, Bash searches the folders in `$PATH`. Your current folder is deliberately **not** in `$PATH` (a security decision — otherwise a malicious `ls` in a downloads folder could hijack the real one). `./` means *"look right here, in this exact directory."*

---

# 1️⃣ Script Structure

## 🧠 The Shebang — `#!/bin/bash`

The very first line of every script is the **shebang** (sharp + bang: `#!`).

```bash
#!/bin/bash
```

It is **not** a comment, even though it starts with `#`. It's an instruction to the operating system that says:

> *"When someone runs this file, feed it to the program at `/bin/bash`."*

**Different shebangs for different languages:**

| Shebang | Runs the file with |
|---------|--------------------|
| `#!/bin/bash` | Bash (what we use) |
| `#!/bin/sh` | The basic POSIX shell — fewer features |
| `#!/usr/bin/env bash` | Finds bash wherever it lives — more portable |
| `#!/usr/bin/python3` | Python |

> 🧭 **Rule for this course:** always use `#!/bin/bash`. Use `#!/usr/bin/env bash` when writing scripts that might run on macOS too.

---

## 📝 Comments — Notes to Future You

Anything after `#` on a line is ignored by Bash.

```bash
# This whole line is a comment

echo "Hello"   # This part after the command is also a comment
```

> 🕰️ **The golden rule of comments:** Don't explain *what* the code does — the code already says that. Explain **why**.
>
> ❌ `count=$((count + 1))   # add one to count`
> ✅ `count=$((count + 1))   # track retries so we can give up after 5`

---

## 🏗️ A Professional Script Skeleton

This is the shape every serious script should take. Copy it as your starting template.

```bash
#!/bin/bash
# ==============================================================
#  Script Name : backup.sh
#  Description : Backs up the /var/www folder to /backups
#  Author      : Nnamdi Anaba
#  Created     : 2026-08-28
#  Usage       : ./backup.sh
# ==============================================================

# ---------- CONFIGURATION ----------
SOURCE_DIR="/var/www"
BACKUP_DIR="/backups"
MAX_BACKUPS=7

# ---------- MAIN LOGIC ----------
echo "Starting backup..."
# ... actual work goes here ...
echo "Backup complete."

# ---------- EXIT ----------
exit 0
```

**Why this structure wins:**

| Section | Purpose |
|---------|---------|
| Header block | Anyone (including you in 6 months) knows instantly what this does |
| Configuration | All the values that might change are at the **top**, not buried in line 84 |
| Main logic | The actual work |
| `exit 0` | Explicitly reports success to whatever called this script |

---

## ✏️ **CLASS ACTIVITY 1.1 — Your Script Template** *(5 min)*

1. Create `~/bash-class/template.sh` using the professional skeleton above
2. Change the header to describe a script that "reports system information"
3. In the main section, add three commands: `hostname`, `uptime`, and `df -h /`
4. Make it executable and run it
5. **Challenge:** Add a comment explaining *why* you'd want this script, not what it does

---

# 2️⃣ Variables — The Boxes That Hold Your Data

## 🧠 What Is a Variable?

A variable is a **labelled box**. You put something in it, and later you get it back by name.

```
   name  ────►  ┌──────────┐
                │ "Nnamdi" │
                └──────────┘
```

## The Two Golden Rules

```bash
name="Nnamdi"      # ASSIGN: no $, NO SPACES around =
echo "$name"       # ACCESS: with $
```

> 🚨 **THE #1 BEGINNER ERROR — SPACES AROUND `=`**
>
> ```bash
> name = "Nnamdi"     # ❌ BROKEN
> ```
> ```
> name: command not found
> ```
> Bash sees a space and thinks `name` is a **command** you're trying to run, with `=` and `"Nnamdi"` as its arguments. There must be **zero spaces** on either side of `=`.

---

## 🔬 Hands-On: Variables in Action

```bash
nano vars.sh
```

```bash
#!/bin/bash

# Assignment — no spaces around =
name="Nnamdi"
course='Bash Scripting'
year=2026

# Access — with $
echo "Hello, $name!"
echo "Welcome to $course"
echo "The year is $year"

# Curly braces {} when the variable touches other text
echo "You are a ${course}er"        # without braces this breaks
echo "Next year is $((year + 1))"

# Length of a variable
echo "Your name has ${#name} letters"

# Read-only (constant) — cannot be changed later
readonly ACADEMY="Schull AI Academy"
echo "Studying at $ACADEMY"
```

```bash
chmod +x vars.sh && ./vars.sh
```
```
Hello, Nnamdi!
Welcome to Bash Scripting
The year is 2026
You are a Bash Scriptinger
Next year is 2027
Your name has 6 letters
Studying at Schull AI Academy
```

---

## 💬 Quotes Matter — A Lot

This is the single most confusing thing for beginners. Here's the whole story:

| Quote style | Variables expand? | Use when |
|-------------|-------------------|----------|
| `"double"` | ✅ Yes | **Almost always** — this is your default |
| `'single'` | ❌ No — prints literally | You want the text exactly as typed |
| `none` | ✅ Yes, but **breaks on spaces** | Almost never — this is a bug waiting to happen |

### 🔬 See the Difference

```bash
name="Nnamdi Anaba"

echo "Hello $name"     # Hello Nnamdi Anaba     ← correct
echo 'Hello $name'     # Hello $name            ← literal, no expansion
echo Hello $name       # Hello Nnamdi Anaba     ← works here, but...
```

Now watch it break:

```bash
file="my report.txt"
touch "$file"          # ✅ creates ONE file: "my report.txt"
touch $file            # ❌ creates TWO files: "my" and "report.txt"
```

> 🛡️ **The rule that will save you hours:** **Always wrap variables in double quotes** — `"$var"` — unless you have a specific reason not to. This one habit prevents more bugs than any other in Bash.

---

## 🌍 Environment vs Local Variables

| | Local variable | Environment variable |
|---|----------------|----------------------|
| Visible to | Only the current script/shell | The script **and every program it launches** |
| How to create | `name="value"` | `export name="value"` |
| Naming style | `lowercase` | `UPPERCASE` |
| Example | `counter=5` | `PATH`, `HOME`, `USER` |

### 🔬 Hands-On: Prove the Difference

```bash
# Look at environment variables you already have
echo "User:  $USER"
echo "Home:  $HOME"
echo "Shell: $SHELL"
echo "Path:  $PATH"

# See them all
env | head -10
```

Now demonstrate scope:

```bash
nano scope.sh
```
```bash
#!/bin/bash
local_var="I am local"
export shared_var="I am exported"

echo "Inside parent script:"
echo "  local_var  = $local_var"
echo "  shared_var = $shared_var"

# Launch a CHILD script
bash -c 'echo "Inside child script:"; echo "  local_var  = [$local_var]"; echo "  shared_var = [$shared_var]"'
```
```bash
chmod +x scope.sh && ./scope.sh
```
```
Inside parent script:
  local_var  = I am local
  shared_var = I am exported
Inside child script:
  local_var  = []                 ← EMPTY! Not inherited
  shared_var = I am exported      ← Inherited because it was exported
```

> 🧠 **Mental model:** A normal variable is a note on **your** desk. An exported variable is a note you **photocopy and hand to everyone you send out on an errand**.

---

## ⌨️ Reading Input from the User

```bash
#!/bin/bash
echo -n "What is your name? "
read -r name
echo "Nice to meet you, $name!"

# Prompt built into read
read -rp "What is your favourite command? " cmd
echo "Ah, $cmd is a good one."

# Hidden input (for passwords)
read -rsp "Enter a secret: " secret
echo ""
echo "Your secret is ${#secret} characters long."
```

> 💡 **Always use `read -r`.** Without `-r`, backslashes in the user's input get mangled. There is essentially no situation where you want that.

---

## ✏️ **CLASS ACTIVITY 2.1 — The Interview Bot** *(8 min)*

Write `interview.sh` that:

1. Asks for the user's **name**
2. Asks for their **age**
3. Asks for their **favourite Linux command**
4. Prints a formatted summary using all three
5. Also prints how many letters are in their name
6. Prints what year they were born (current year − age)

**Expected output:**
```
=== INTERVIEW SUMMARY ===
Name:      Sodee Peterside  (15 letters)
Age:       19  (born around 2007)
Favourite: grep

Nice to meet you, Sodee Peterside!
```

<details>
<summary>💡 Solution</summary>

```bash
#!/bin/bash
read -rp "What is your name? " name
read -rp "How old are you? " age
read -rp "Favourite Linux command? " cmd

current_year=$(date +%Y)
birth_year=$(( current_year - age ))

echo ""
echo "=== INTERVIEW SUMMARY ==="
echo "Name:      $name  (${#name} letters)"
echo "Age:       $age  (born around $birth_year)"
echo "Favourite: $cmd"
echo ""
echo "Nice to meet you, $name!"
```
</details>

---

# 3️⃣ Command Substitution — Capturing Output

## 🧠 The Concept

Normally a command **prints** its output to the screen. Command substitution **catches** that output and stores it in a variable instead.

```bash
today=$(date)
```

```
   date  ──runs──►  "Fri 28 Aug 2026"  ──captured into──►  today
```

## Two Syntaxes — Use the Modern One

```bash
files=$(ls | wc -l)      # ✅ MODERN — use this
files=`ls | wc -l`       # ⚠️ OLD backticks — still works, but avoid
```

**Why `$( )` wins:**

| | `$( )` | Backticks |
|---|--------|-----------|
| Readable | ✅ | ❌ (` and ' look alike) |
| Nestable | ✅ `$(echo $(date))` | ❌ Needs ugly escaping |
| Modern standard | ✅ | ❌ Legacy |

---

## 🔬 Hands-On: Capturing Everything

```bash
nano capture.sh
```
```bash
#!/bin/bash

# Capture simple output
today=$(date '+%A, %d %B %Y')
user=$(whoami)
host=$(hostname)

# Capture the result of a pipeline
file_count=$(ls -1 | wc -l)
disk_used=$(df -h / | awk 'NR==2 {print $5}')
mem_free=$(free -h | awk '/Mem:/ {print $7}')

# Nesting works!
uptime_msg="System has been up: $(uptime -p)"

echo "===== SYSTEM SNAPSHOT ====="
echo "Date:       $today"
echo "User:       $user@$host"
echo "Files here: $file_count"
echo "Disk used:  $disk_used"
echo "Mem free:   $mem_free"
echo "$uptime_msg"
```
```bash
chmod +x capture.sh && ./capture.sh
```
```
===== SYSTEM SNAPSHOT =====
Date:       Friday, 28 August 2026
User:       student@ubuntu
Files here: 6
Disk used:  47%
Mem free:   3.7Gi
System has been up: up 2 minutes
```

> 🔥 **This is the technique that makes scripts *powerful*.** Any command you've learned in the last 6 weeks — `grep`, `awk`, `df`, `ps` — can now feed its answer directly into your script's logic.

---

## ✏️ **CLASS ACTIVITY 3.1 — The Status Card** *(7 min)*

Write `statuscard.sh` that captures and prints a neat status card containing:

| Item | Hint |
|------|------|
| Hostname | `hostname` |
| Current user | `whoami` |
| Kernel version | `uname -r` |
| Number of processes | `ps aux --no-heading \| wc -l` |
| Disk usage % of `/` | `df -h / \| awk 'NR==2 {print $5}'` |
| Number of logged-in users | `who \| wc -l` |
| Your public IP | `curl -s ifconfig.me` |

Use `printf` for neat alignment:
```bash
printf "%-20s %s\n" "Hostname:" "$hostname"
```

---

# 4️⃣ Arithmetic — Making Bash Do Maths

Bash has **four** ways to do maths. You need to know when to use which.

## 1. `$(( ))` — The Modern Standard ⭐

```bash
a=17
b=5

echo "Sum:       $(( a + b ))"      # 22
echo "Diff:      $(( a - b ))"      # 12
echo "Product:   $(( a * b ))"      # 85
echo "Quotient:  $(( a / b ))"      # 3   ← integer only!
echo "Remainder: $(( a % b ))"      # 2
echo "Power:     $(( a ** 2 ))"     # 289
```

Notice you **don't need `$`** on variables inside `$(( ))`.

**Increment shortcuts:**

```bash
count=10
(( count++ ))       # count is now 11
(( count += 5 ))    # count is now 16
(( count-- ))       # count is now 15
count=$((count+1))  # also works — and it's the SAFEST (see gotcha below)
```

---

## 2. `let` — Works, But Dated

```bash
let result=17*5
echo $result        # 85
```

## 3. `expr` — Ancient, Avoid

```bash
expr 17 + 5         # 22
expr 17 \* 5        # 85   ← must escape the *
```
> Requires spaces around every operator and escaping for `*`. You'll see it in old scripts; don't write new ones with it.

---

## 4. `bc` — For Decimals 🔢

**Here's the big limitation:** Bash arithmetic is **integer-only**. Decimals are silently thrown away.

```bash
echo $(( 17 / 5 ))     # 3    ← the .4 vanished!
echo $(( 10 / 3 ))     # 3    ← should be 3.333
```

For real maths, pipe to `bc` and set the `scale` (decimal places):

```bash
echo "scale=2; 17 / 5" | bc         # 3.40
echo "scale=4; 22 / 7" | bc         # 3.1428
```

### 🔬 Hands-On: Temperature Converter

```bash
nano temp.sh
```
```bash
#!/bin/bash
read -rp "Enter temperature in Celsius: " c

# Integer maths would lose precision, so use bc
f=$(echo "scale=1; ($c * 9 / 5) + 32" | bc)
k=$(echo "scale=2; $c + 273.15" | bc)

echo ""
echo "${c}°C = ${f}°F = ${k}K"

# A little bonus judgement
if (( $(echo "$c > 35" | bc -l) )); then
    echo "🔥 That's seriously hot!"
elif (( $(echo "$c < 10" | bc -l) )); then
    echo "🥶 Bring a jacket!"
else
    echo "😎 Pleasant weather."
fi
```
```bash
chmod +x temp.sh && ./temp.sh
```
```
Enter temperature in Celsius: 37

37°C = 98.6°F = 310.15K
🔥 That's seriously hot!
```

---

## 🚨 The `(( i++ ))` Trap — Read This Twice

This bug catches **everyone**, including experienced people. Here it is:

```bash
#!/bin/bash
x=0
if (( x++ )); then
    echo "TRUE"
else
    echo "FALSE"     # ← this runs!
fi
echo "x is now $x"   # ← but x DID become 1
```
```
FALSE
x is now 1
```

**Why?** `(( x++ ))` is *post*-increment. It returns the **old** value (`0`) as its exit status. In Bash arithmetic, `0` means **false**. So the increment happens, but the test fails.

**Where this bites you:** counting things with `&&`.

```bash
# ❌ BROKEN — the || branch fires when score is 0
[ -f myfile ] && (( score++ )) || echo "file missing"
```

**The three safe fixes:**

```bash
score=$((score + 1))     # ✅ SAFEST — assignment always succeeds
(( ++score ))            # ✅ pre-increment returns the NEW value
(( score++ )) || true    # ✅ explicitly swallow the exit code
```

> 📌 **House rule for this class:** when counting, always write `count=$((count + 1))`. It's two extra characters and it will never surprise you.

---

## ✏️ **CLASS ACTIVITY 4.1 — The Calculator Challenge** *(8 min)*

Write `calc.sh` that:

1. Asks for two numbers
2. Prints the sum, difference, product
3. Prints the division to **2 decimal places** (use `bc`!)
4. Prints the remainder
5. Prints which number is larger
6. **Bonus:** Print the average to 2 decimal places

<details>
<summary>💡 Solution</summary>

```bash
#!/bin/bash
read -rp "First number:  " a
read -rp "Second number: " b

echo ""
echo "$a + $b = $(( a + b ))"
echo "$a - $b = $(( a - b ))"
echo "$a × $b = $(( a * b ))"
echo "$a ÷ $b = $(echo "scale=2; $a / $b" | bc)"
echo "$a % $b = $(( a % b ))"
echo "Average = $(echo "scale=2; ($a + $b) / 2" | bc)"

if [ "$a" -gt "$b" ]; then
    echo "$a is larger"
elif [ "$b" -gt "$a" ]; then
    echo "$b is larger"
else
    echo "They are equal"
fi
```
</details>

---

# 5️⃣ Conditionals — Teaching Scripts to Decide

## 🧠 The Shape of an `if`

```bash
if [ condition ]; then
    # do this when true
fi
```

Read it out loud: *"**if** condition, **then** do this, **fi**nish."*

> 🤓 **Why `fi`?** It's `if` spelled backwards. Bash uses this pattern elsewhere too: `case` ends with `esac`.

### The Spacing Rules Are Strict

```bash
if [ "$age" -ge 18 ]; then     # ✅ correct
if ["$age" -ge 18]; then       # ❌ no space after [ or before ]
if [ "$age"-ge 18 ]; then      # ❌ no space around the operator
```

> The `[` is actually a **command**, not punctuation! That's why it needs spaces around it — just like any other command needs a space before its arguments.

---

## 🪜 The Full Ladder: `if` / `elif` / `else`

```bash
#!/bin/bash
read -rp "Enter your score (0-100): " score

if [ "$score" -ge 90 ]; then
    grade="A"; comment="Outstanding! 🌟"
elif [ "$score" -ge 80 ]; then
    grade="B"; comment="Great work! 👏"
elif [ "$score" -ge 70 ]; then
    grade="C"; comment="Solid effort. 👍"
elif [ "$score" -ge 60 ]; then
    grade="D"; comment="You passed, but let's push higher."
else
    grade="F"; comment="Let's book some extra practice time."
fi

echo "Score: $score → Grade: $grade"
echo "$comment"
```

> ⚠️ **Order matters!** Bash checks top to bottom and stops at the first match. If you put `-ge 60` first, a score of 95 would be graded "D" — because 95 *is* ≥ 60.

---

## 🔗 Combining Conditions — AND / OR / NOT

| Operator | Meaning | Example |
|----------|---------|---------|
| `&&` | AND — both must be true | `[ "$a" -gt 5 ] && [ "$b" -lt 10 ]` |
| `\|\|` | OR — either can be true | `[ "$a" -eq 0 ] \|\| [ "$b" -eq 0 ]` |
| `!` | NOT — inverts the result | `[ ! -f "myfile.txt" ]` |

```bash
#!/bin/bash
age=20
has_id="yes"

# AND — both must pass
if [ "$age" -ge 18 ] && [ "$has_id" = "yes" ]; then
    echo "✅ Entry granted"
else
    echo "❌ Entry denied"
fi

# OR — either qualifies for a discount
if [ "$age" -lt 13 ] || [ "$age" -gt 65 ]; then
    echo "🎟️  Discount applies"
else
    echo "💵 Full price"
fi

# NOT — check something is missing
if [ ! -f "config.txt" ]; then
    echo "⚠️  config.txt not found — creating it"
    touch config.txt
fi
```

---

## 🎛️ `case` — When You Have Many Options

When you're checking **one variable against many possible values**, `case` is far cleaner than a wall of `elif`.

```bash
case "$variable" in
    pattern1)  commands ;;
    pattern2)  commands ;;
    *)         default  ;;
esac
```

> Each branch ends with `;;` — and the whole block ends with `esac` (`case` backwards).

### 🔬 Hands-On: The Menu

```bash
nano menu.sh
```
```bash
#!/bin/bash
echo "╔════════════════════════════╗"
echo "║   SYSTEM INFO MENU         ║"
echo "╠════════════════════════════╣"
echo "║  1) Disk usage             ║"
echo "║  2) Memory usage           ║"
echo "║  3) Logged-in users        ║"
echo "║  4) System uptime          ║"
echo "║  q) Quit                   ║"
echo "╚════════════════════════════╝"
read -rp "Choose an option: " choice

case "$choice" in
    1)
        echo "--- Disk Usage ---"
        df -h /
        ;;
    2)
        echo "--- Memory Usage ---"
        free -h
        ;;
    3)
        echo "--- Logged-in Users ---"
        who
        ;;
    4)
        echo "--- Uptime ---"
        uptime -p
        ;;
    q|Q|quit|exit)                 # multiple patterns with |
        echo "Goodbye! 👋"
        exit 0
        ;;
    *)                             # the catch-all
        echo "❌ '$choice' is not a valid option"
        exit 1
        ;;
esac
```

**Pattern matching in `case` is powerful:**

```bash
case "$filename" in
    *.txt)        echo "Text file"       ;;
    *.jpg|*.png)  echo "Image file"      ;;
    *.sh)         echo "Shell script"    ;;
    backup_*)     echo "A backup file"   ;;
    [0-9]*)       echo "Starts with a digit" ;;
    *)            echo "Unknown type"    ;;
esac
```

---

## ✏️ **CLASS ACTIVITY 5.1 — The Bouncer** *(10 min)*

Write `bouncer.sh` — a nightclub door policy script.

**Rules:**
- Ask for **age** and whether they have **ID** (yes/no)
- Under 18 → *"Sorry, you're too young."*
- 18+ but no ID → *"No ID, no entry."*
- 18+ with ID → *"Welcome in!"*
- **Bonus:** 65 or over → also print *"Senior discount applied 🎟️"*
- **Bonus:** Use a `case` statement to accept `yes`, `y`, `Yes`, `Y` all as valid ID confirmation

<details>
<summary>💡 Solution</summary>

```bash
#!/bin/bash
read -rp "How old are you? " age
read -rp "Do you have ID? (yes/no) " id_answer

# Normalise the ID answer using case
case "$id_answer" in
    yes|y|Yes|Y|YES) has_id="yes" ;;
    *)               has_id="no"  ;;
esac

if [ "$age" -lt 18 ]; then
    echo "🚫 Sorry, you're too young."
elif [ "$has_id" = "no" ]; then
    echo "🪪 No ID, no entry."
else
    echo "✅ Welcome in!"
    if [ "$age" -ge 65 ]; then
        echo "🎟️  Senior discount applied"
    fi
fi
```
</details>

---

# 6️⃣ Test Operators — The Questions You Can Ask

The `[ ]` brackets let you ask three families of questions.

## 📁 File Tests

| Operator | True when… | Example |
|----------|-----------|---------|
| `-e` | File **e**xists (any type) | `[ -e "/etc/passwd" ]` |
| `-f` | Is a regular **f**ile | `[ -f "script.sh" ]` |
| `-d` | Is a **d**irectory | `[ -d "/home/student" ]` |
| `-s` | Exists and is **s**ize > 0 (not empty) | `[ -s "log.txt" ]` |
| `-r` | Is **r**eadable | `[ -r "config.ini" ]` |
| `-w` | Is **w**ritable | `[ -w "/tmp" ]` |
| `-x` | Is e**x**ecutable | `[ -x "script.sh" ]` |
| `-L` | Is a symbolic **l**ink | `[ -L "/bin/sh" ]` |
| `f1 -nt f2` | f1 is **n**ewer **t**han f2 | `[ "a.txt" -nt "b.txt" ]` |

### 🔬 Hands-On: The File Inspector

```bash
nano inspect.sh
```
```bash
#!/bin/bash
read -rp "Enter a path to inspect: " target

echo ""
echo "=== INSPECTING: $target ==="

if [ ! -e "$target" ]; then
    echo "❌ Does not exist."
    exit 1
fi

echo "✅ Exists"

if   [ -f "$target" ]; then echo "📄 Type: regular file"
elif [ -d "$target" ]; then echo "📁 Type: directory"
elif [ -L "$target" ]; then echo "🔗 Type: symbolic link"
else                        echo "❓ Type: something else"
fi

[ -r "$target" ] && echo "👁️  Readable"    || echo "🚫 Not readable"
[ -w "$target" ] && echo "✏️  Writable"    || echo "🚫 Not writable"
[ -x "$target" ] && echo "⚡ Executable"   || echo "🚫 Not executable"

if [ -f "$target" ]; then
    if [ -s "$target" ]; then
        echo "📏 Size: $(du -h "$target" | cut -f1) — has content"
    else
        echo "📏 Size: EMPTY"
    fi
fi
```

Try it on `/etc/passwd`, `/home`, and a file that doesn't exist.

---

## 🔤 String Tests

| Operator | True when… |
|----------|-----------|
| `-z "$s"` | String is **z**ero-length (empty) |
| `-n "$s"` | String is **n**on-empty |
| `"$a" = "$b"` | Strings are identical |
| `"$a" != "$b"` | Strings differ |
| `"$a" < "$b"` | `$a` sorts before `$b` alphabetically |

```bash
username=""
if [ -z "$username" ]; then
    echo "⚠️  No username provided — using 'guest'"
    username="guest"
fi
echo "Hello, $username"
```

> 🛡️ **Always quote strings in tests.** If `$name` is empty and unquoted, `[ $name = "x" ]` becomes `[ = "x" ]` — a syntax error. With quotes, `[ "" = "x" ]` works fine.

---

## 🔢 Numeric Tests

Here's where beginners get burned: **numbers and strings use different operators.**

| Numeric | Meaning | String equivalent |
|---------|---------|-------------------|
| `-eq` | **eq**ual | `=` |
| `-ne` | **n**ot **e**qual | `!=` |
| `-gt` | **g**reater **t**han | — |
| `-ge` | **g**reater or **e**qual | — |
| `-lt` | **l**ess **t**han | — |
| `-le` | **l**ess or **e**qual | — |

```bash
# ✅ Numbers
[ "$count" -gt 10 ]

# ✅ Strings
[ "$name" = "Nnamdi" ]

# ⚠️ Watch this trap:
[ "10" = "10.0" ]     # FALSE — different strings
[ "10" -eq "10" ]     # TRUE  — same number
[ "09" -eq "9" ]      # TRUE  — numerically equal
[ "09" = "9" ]        # FALSE — different text
```

---

## ✏️ **CLASS ACTIVITY 6.1 — The Safety Checker** *(10 min)*

Write `safecheck.sh` that takes a filename and safely reports on it.

Requirements:
1. If **no filename is given**, print usage and exit with code 1
2. If the file **doesn't exist**, offer to create it
3. If it exists but is **empty**, warn the user
4. If it has content, show the **line count** and the **first 3 lines**
5. Report whether it's readable, writable, executable

> 💡 The first argument passed to a script is available as `$1`. The number of arguments is `$#`.

<details>
<summary>💡 Solution</summary>

```bash
#!/bin/bash

if [ $# -eq 0 ]; then
    echo "Usage: $0 <filename>"
    exit 1
fi

file="$1"

if [ ! -e "$file" ]; then
    read -rp "'$file' doesn't exist. Create it? (y/n) " ans
    if [ "$ans" = "y" ]; then
        touch "$file"
        echo "✅ Created $file"
    else
        echo "Aborted."
        exit 1
    fi
fi

if [ ! -f "$file" ]; then
    echo "⚠️  '$file' is not a regular file."
    exit 1
fi

echo ""
echo "=== REPORT: $file ==="
[ -r "$file" ] && echo "Readable:   yes" || echo "Readable:   no"
[ -w "$file" ] && echo "Writable:   yes" || echo "Writable:   no"
[ -x "$file" ] && echo "Executable: yes" || echo "Executable: no"

if [ -s "$file" ]; then
    echo "Lines:      $(wc -l < "$file")"
    echo ""
    echo "First 3 lines:"
    head -3 "$file" | sed 's/^/  /'
else
    echo "⚠️  File is empty."
fi
```

Run it: `./safecheck.sh /etc/hostname`
</details>

---

# 7️⃣ Loops — Doing Things Over and Over

## 🔁 `for` Loops — When You Know the List

### Style 1: Number Ranges

```bash
for i in {1..5}; do
    echo "Iteration $i"
done
```

**With a step:**
```bash
for i in {0..20..5}; do echo -n "$i "; done   # 0 5 10 15 20
```

**Counting down:**
```bash
for i in {5..1}; do echo -n "$i "; done       # 5 4 3 2 1
```

### Style 2: Lists of Items

```bash
for lang in Bash Python Go Rust; do
    echo "Learning $lang"
done

for service in ssh cron nginx; do
    echo "Checking $service..."
done
```

### Style 3: Over Files

```bash
for file in *.txt; do
    echo "Found: $file ($(wc -l < "$file") lines)"
done
```

### Style 4: C-Style

```bash
for (( i=1; i<=5; i++ )); do
    echo "Count: $i"
done
```

Read it as: *start at 1 · keep going while i ≤ 5 · add 1 each time*.

### Style 5: Over Command Output

```bash
for user in $(cut -d: -f1 /etc/passwd | head -5); do
    echo "User: $user"
done
```

---

## 🔬 Hands-On: Fun with `for`

```bash
nano forfun.sh
```
```bash
#!/bin/bash

echo "=== 🚀 Rocket Launch ==="
for i in {5..1}; do
    echo "T-minus $i..."
    sleep 0.5
done
echo "🚀 LIFTOFF!"

echo ""
echo "=== 🎲 Rolling 5 dice ==="
for roll in {1..5}; do
    die=$(( RANDOM % 6 + 1 ))
    echo -n "$die "
done
echo ""

echo ""
echo "=== ⭐ Star Pyramid ==="
rows=5
for (( i=1; i<=rows; i++ )); do
    for (( s=i; s<rows; s++ )); do echo -n " "; done
    for (( a=1; a<=2*i-1; a++ )); do echo -n "*"; done
    echo ""
done

echo ""
echo "=== ✖️  Times Tables ==="
for i in {1..5}; do
    for j in {1..5}; do
        printf "%4d" $(( i * j ))
    done
    echo ""
done
```
```bash
chmod +x forfun.sh && ./forfun.sh
```
```
=== 🚀 Rocket Launch ===
T-minus 5...
...
🚀 LIFTOFF!

=== 🎲 Rolling 5 dice ===
4 4 2 2 6

=== ⭐ Star Pyramid ===
    *
   ***
  *****
 *******
*********

=== ✖️  Times Tables ===
   1   2   3   4   5
   2   4   6   8  10
   3   6   9  12  15
   4   8  12  16  20
   5  10  15  20  25
```

> 🎲 **`$RANDOM`** is a magic Bash variable that gives a random number 0–32767 every time you read it. `$(( RANDOM % 6 + 1 ))` squeezes that into 1–6.

---

## ⏳ `while` Loops — Repeat *While* Something Is True

```bash
while [ condition ]; do
    commands
done
```

```bash
count=1
while [ "$count" -le 5 ]; do
    echo "Count is $count"
    count=$(( count + 1 ))
done
```

> 🚨 **Every `while` loop needs an exit path.** If nothing inside changes the condition, you get an **infinite loop**. Press `Ctrl+C` to escape one.

### The Killer Use Case: Reading a File Line by Line

```bash
nano grades.sh
```
```bash
#!/bin/bash

# Build sample data
printf "alice,85\nbob,62\ncarol,94\ndave,45\n" > grades.csv

echo "=== STUDENT RESULTS ==="
while IFS=',' read -r name score; do
    if   [ "$score" -ge 80 ]; then status="🌟 DISTINCTION"
    elif [ "$score" -ge 50 ]; then status="✅ PASS"
    else                           status="❌ FAIL"
    fi
    printf "  %-8s %3d  %s\n" "$name" "$score" "$status"
done < grades.csv
```
```
=== STUDENT RESULTS ===
  alice     85  🌟 DISTINCTION
  bob       62  ✅ PASS
  carol     94  🌟 DISTINCTION
  dave      45  ❌ FAIL
```

> 🧩 **Decoding `while IFS=',' read -r name score; do ... done < file`:**
> - `< file` feeds the file into the loop
> - `read -r name score` grabs one line and splits it into two variables
> - `IFS=','` tells `read` to split on commas instead of spaces
>
> This four-word incantation is how Bash processes CSV, logs, and config files. **Memorise it.**

---

## ⏱️ `until` Loops — Repeat *Until* Something Becomes True

`until` is `while`'s mirror image: it loops **while the condition is FALSE**.

```bash
n=1
until [ "$n" -gt 5 ]; do
    echo "n is $n"
    n=$(( n + 1 ))
done
```

**When to actually use it — waiting for something:**

```bash
#!/bin/bash
echo "Waiting for /tmp/ready.flag to appear..."

tries=0
until [ -f /tmp/ready.flag ]; do
    tries=$(( tries + 1 ))
    echo "  still waiting... (check $tries)"
    sleep 1

    if [ "$tries" -ge 10 ]; then
        echo "❌ Gave up after 10 tries."
        exit 1
    fi
done

echo "✅ File appeared after $tries checks!"
```

> 💡 **`while` vs `until`:** Use whichever makes the sentence read naturally.
> *"**While** the server is down, keep retrying"* vs *"**Until** the server is up, keep retrying."* Same logic, different phrasing.

---

## ✏️ **CLASS ACTIVITY 7.1 — Loop Gauntlet** *(12 min)*

Complete all six. Write each as a separate small script or all in one file.

| # | Challenge |
|---|-----------|
| 1 | Print the numbers 1–20, but only the **even** ones |
| 2 | Print a countdown from 10 to 1, then "BLAST OFF!" |
| 3 | Print the 7 times table (7×1 through 7×12) |
| 4 | Use a `while` loop to sum the numbers 1–100 (answer should be **5050**) |
| 5 | Create 5 files named `report_1.txt` … `report_5.txt`, each containing its own number |
| 6 | Loop through `/etc/passwd` and print just the usernames of the first 5 entries |

<details>
<summary>💡 Solutions</summary>

```bash
# 1 — even numbers
for i in {1..20}; do
    if [ $(( i % 2 )) -eq 0 ]; then echo -n "$i "; fi
done; echo

# 2 — countdown
for i in {10..1}; do echo "$i..."; sleep 0.3; done
echo "🚀 BLAST OFF!"

# 3 — times table
for i in {1..12}; do
    echo "7 x $i = $(( 7 * i ))"
done

# 4 — sum 1..100
sum=0; n=1
while [ $n -le 100 ]; do
    sum=$(( sum + n ))
    n=$(( n + 1 ))
done
echo "Sum = $sum"

# 5 — create files
for i in {1..5}; do
    echo "This is report number $i" > "report_$i.txt"
done
ls report_*.txt

# 6 — usernames
while IFS=':' read -r user _; do
    echo "User: $user"
done < <(head -5 /etc/passwd)
```
</details>

---

# 8️⃣ Loop Control — break, continue, exit, sleep

| Command | Effect |
|---------|--------|
| `break` | **Leave the loop entirely**, right now |
| `continue` | **Skip the rest of this iteration**, go to the next one |
| `exit N` | **Quit the whole script** with exit code N |
| `sleep N` | **Pause** for N seconds (accepts decimals: `sleep 0.5`) |

## 🛑 `break` — Stop Searching Once You've Found It

```bash
#!/bin/bash
echo "Searching for the number 6..."

for i in {1..10}; do
    echo "  checking $i"
    if [ "$i" -eq 6 ]; then
        echo "  🎯 Found it! Stopping."
        break
    fi
done
echo "Search complete."
```

## ⏭️ `continue` — Skip This One, Keep Going

```bash
#!/bin/bash
echo "Processing files, skipping backups..."

for file in report.txt backup.bak notes.txt old.bak data.csv; do
    case "$file" in
        *.bak)
            echo "  ⏭️  Skipping backup: $file"
            continue
            ;;
    esac
    echo "  ✅ Processing: $file"
done
```

## 🚪 `exit` — Exit Codes Are How Scripts Talk

```bash
exit 0    # SUCCESS — everything worked
exit 1    # FAILURE — something went wrong
exit 2    # Misuse — e.g. wrong arguments given
```

Check the last exit code with `$?`:

```bash
./myscript.sh
echo "That script exited with code: $?"
```

> 🔗 **Why this matters for DevOps:** Exit codes are how scripts chain together. `./deploy.sh && ./notify.sh` only runs `notify.sh` if `deploy.sh` exits **0**. Your CI/CD pipeline lives or dies by correct exit codes.

## 😴 `sleep` — Controlling Time

```bash
sleep 5       # 5 seconds
sleep 0.5     # half a second
sleep 2m      # 2 minutes
sleep 1h      # 1 hour
```

### 🔬 Hands-On: A Real Progress Bar

```bash
nano progress.sh
```
```bash
#!/bin/bash
total=30

echo "Installing awesome software..."
for (( i=0; i<=total; i++ )); do
    pct=$(( i * 100 / total ))

    printf "\r["
    for (( j=0; j<i; j++ ));      do printf "█"; done
    for (( j=i; j<total; j++ ));  do printf "░"; done
    printf "] %3d%%" "$pct"

    sleep 0.05
done
echo ""
echo "✅ Installation complete!"
```

> 🎨 **The trick:** `\r` is a **carriage return** — it moves the cursor back to the start of the line **without** making a new line. So each redraw overwrites the previous bar, creating animation.

---

## ✏️ **CLASS ACTIVITY 8.1 — Number Guessing Game** *(10 min)*

Write `guess.sh` — a complete game.

**Requirements:**
1. Pick a random number 1–100 using `$RANDOM`
2. Loop, asking the user to guess
3. Tell them "Too high" or "Too low"
4. Count their attempts
5. `break` when they get it right
6. Give up after **7 attempts** and reveal the answer
7. **Bonus:** let them type `quit` to give up early

<details>
<summary>💡 Solution</summary>

```bash
#!/bin/bash
secret=$(( RANDOM % 100 + 1 ))
attempts=0
max_attempts=7

echo "🎯 I'm thinking of a number between 1 and 100."
echo "   You have $max_attempts attempts. (Type 'quit' to give up)"
echo ""

while [ "$attempts" -lt "$max_attempts" ]; do
    read -rp "Your guess: " guess

    if [ "$guess" = "quit" ]; then
        echo "👋 Giving up? The number was $secret."
        exit 0
    fi

    # Reject non-numbers
    case "$guess" in
        ''|*[!0-9]*)
            echo "   ⚠️  Please enter a number."
            continue
            ;;
    esac

    attempts=$(( attempts + 1 ))
    remaining=$(( max_attempts - attempts ))

    if [ "$guess" -lt "$secret" ]; then
        echo "   ⬆️  Too low!  ($remaining left)"
    elif [ "$guess" -gt "$secret" ]; then
        echo "   ⬇️  Too high! ($remaining left)"
    else
        echo ""
        echo "🎉 CORRECT! You got it in $attempts attempts!"
        exit 0
    fi
done

echo ""
echo "💀 Out of attempts! The number was $secret."
exit 1
```

> Note how `continue` is used so a bad input doesn't waste an attempt.
</details>

---

# 9️⃣ 🏆 CLASS ACTIVITY — System Health Check Script

**Time:** 20 minutes · **This is the curriculum's required activity**

## The Brief

Build `health-check.sh` — a script that inspects the system and reports its health with **colour-coded** warnings, using **conditionals** and **loops**.

## Requirements

| # | Feature | Technique needed |
|---|---------|------------------|
| 1 | Report disk, memory, load, process count | Command substitution |
| 2 | Colour-code each: green OK / yellow WARNING / red CRITICAL | `if`/`elif`/`else` |
| 3 | Loop through a list of services and check each | `for` loop |
| 4 | Count total issues found | Counter variable |
| 5 | Exit `0` if healthy, `1` if any issues | `exit` codes |

## Full Working Solution

```bash
nano health-check.sh
```

```bash
#!/bin/bash
# ==============================================================
#  Script      : health-check.sh
#  Description : Reports system health with colour-coded status
#  Author      : <your name>
#  Usage       : ./health-check.sh
# ==============================================================

# ---------- COLOURS ----------
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
CYAN='\033[0;36m'
BOLD='\033[1m'
NC='\033[0m'                    # No Colour — resets

# ---------- CONFIGURATION ----------
DISK_WARN=70
DISK_CRIT=85
MEM_WARN=70
MEM_CRIT=85
SERVICES="cron ssh systemd-journald"

issues=0

# ---------- HEADER ----------
echo -e "${BOLD}${CYAN}╔══════════════════════════════════════╗${NC}"
echo -e "${BOLD}${CYAN}║      SYSTEM HEALTH CHECK REPORT      ║${NC}"
echo -e "${BOLD}${CYAN}╚══════════════════════════════════════╝${NC}"
echo "  Host : $(hostname)"
echo "  Date : $(date '+%Y-%m-%d %H:%M:%S')"
echo "  User : $(whoami)"
echo ""

# ---------- DISK CHECK ----------
disk=$(df -h / | awk 'NR==2 {print $5}' | tr -d '%')
printf "  %-16s %3s%%  " "Disk usage:" "$disk"
if [ "$disk" -ge "$DISK_CRIT" ]; then
    echo -e "${RED}[CRITICAL]${NC}"
    issues=$(( issues + 1 ))
elif [ "$disk" -ge "$DISK_WARN" ]; then
    echo -e "${YELLOW}[WARNING]${NC}"
    issues=$(( issues + 1 ))
else
    echo -e "${GREEN}[OK]${NC}"
fi

# ---------- MEMORY CHECK ----------
mem=$(free | awk '/Mem:/ {printf "%.0f", $3/$2*100}')
printf "  %-16s %3s%%  " "Memory usage:" "$mem"
if [ "$mem" -ge "$MEM_CRIT" ]; then
    echo -e "${RED}[CRITICAL]${NC}"
    issues=$(( issues + 1 ))
elif [ "$mem" -ge "$MEM_WARN" ]; then
    echo -e "${YELLOW}[WARNING]${NC}"
    issues=$(( issues + 1 ))
else
    echo -e "${GREEN}[OK]${NC}"
fi

# ---------- LOAD AVERAGE ----------
cores=$(nproc)
load=$(uptime | awk -F'load average:' '{print $2}' | cut -d, -f1 | tr -d ' ')
printf "  %-16s %s  (cores: %s)\n" "Load average:" "$load" "$cores"

# ---------- PROCESS COUNT ----------
procs=$(ps aux --no-heading | wc -l)
printf "  %-16s %s\n" "Processes:" "$procs"

# ---------- SERVICE LOOP ----------
echo ""
echo -e "  ${BOLD}Service Status:${NC}"
for svc in $SERVICES; do
    printf "    %-22s" "$svc"
    if pgrep -x "$svc" > /dev/null 2>&1; then
        echo -e "${GREEN}running${NC}"
    else
        echo -e "${YELLOW}not running${NC}"
        issues=$(( issues + 1 ))
    fi
done

# ---------- SUMMARY ----------
echo ""
echo "  ────────────────────────────────────"
if [ "$issues" -eq 0 ]; then
    echo -e "  ${GREEN}${BOLD}✅ ALL SYSTEMS HEALTHY${NC}"
    exit 0
else
    echo -e "  ${YELLOW}${BOLD}⚠️  $issues ISSUE(S) DETECTED${NC}"
    exit 1
fi
```

```bash
chmod +x health-check.sh
./health-check.sh
echo "Exit code was: $?"
```

**Sample output:**
```
╔══════════════════════════════════════╗
║      SYSTEM HEALTH CHECK REPORT      ║
╚══════════════════════════════════════╝
  Host : ubuntu
  Date : 2026-08-28 19:38:24
  User : student

  Disk usage:       47%  [OK]
  Memory usage:      6%  [OK]
  Load average:    0.00  (cores: 1)
  Processes:         53

  Service Status:
    cron                  running
    ssh                   running
    systemd-journald      running

  ────────────────────────────────────
  ✅ ALL SYSTEMS HEALTHY
```

### 🎨 Understanding the Colour Codes

```bash
RED='\033[0;31m'      # \033[ = escape sequence, 0;31 = red, m = end
NC='\033[0m'          # reset to default
echo -e "${RED}Danger!${NC} Back to normal."
```

> ⚠️ **You must use `echo -e`** for colours to work — `-e` tells echo to interpret escape sequences. **Always end with `${NC}`** or your whole terminal stays coloured!

### 🧪 Test Your Thresholds

Temporarily lower a threshold to prove your warning logic works:

```bash
# Edit the script and set DISK_WARN=10, then run again.
# You should see [WARNING] and exit code 1.
```

---

## 🎯 Extension Challenges

Once your script works, try these:

1. Add a **network check** — ping `8.8.8.8` and report reachable/unreachable
2. Write the report to a **log file** as well as the screen
3. Add a **top 3 memory consumers** section using `ps` and a loop
4. Accept a `--quiet` flag that only prints if there are problems

---

# 🔟 🚀 MINI PROJECT — "Server Sentinel"

**Time:** 25 minutes in class + homework · **This is your portfolio piece**

## 📖 The Brief

You've built a health check that runs once. Now build **Server Sentinel** — an interactive, menu-driven monitoring tool that a real sysadmin would actually keep on their server.

This project combines **every single thing** from today:

| Concept | Where it's used |
|---------|-----------------|
| Variables & constants | Thresholds, colours, log path |
| Command substitution | Every metric gathered |
| Arithmetic | Percentage & progress-bar maths |
| Conditionals | Status determination |
| `case` statements | The menu system |
| File tests | Log file existence checks |
| `while` loop | The main menu loop |
| `for` loop | Progress bars, scan steps |
| `break` / `exit` | Quitting cleanly |
| `sleep` | Scan pacing, live monitor |

---

## 🎯 Requirements

Your `sentinel.sh` must have:

1. **A menu** that keeps reappearing until the user quits (`while` + `case`)
2. **A dashboard** with visual progress bars for disk and memory
3. **A full scan** that loops through checks and reports pass/fail
4. **A logging system** that appends timestamped events to a file
5. **Colour-coded output** — green/yellow/red by severity
6. **Graceful handling** of invalid menu choices
7. **A clean exit** with a goodbye message

---

## 💻 The Complete Solution

```bash
nano sentinel.sh
```

```bash
#!/bin/bash
# ==============================================================
#  Script      : sentinel.sh
#  Description : Interactive server monitoring dashboard
#  Author      : <your name>
#  Version     : 1.0
#  Usage       : ./sentinel.sh
# ==============================================================

# ---------- COLOURS ----------
RED='\033[0;31m';    GREEN='\033[0;32m'
YELLOW='\033[1;33m'; BLUE='\033[0;34m'
CYAN='\033[0;36m';   BOLD='\033[1m'
NC='\033[0m'

# ---------- CONFIGURATION ----------
LOGFILE="$HOME/sentinel.log"
DISK_THRESHOLD=80
MEM_THRESHOLD=80
BAR_WIDTH=25

# ==============================================================
#  HELPER FUNCTIONS
# ==============================================================

log_event() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOGFILE"
}

get_disk()  { df -h / | awk 'NR==2 {print $5}' | tr -d '%'; }
get_mem()   { free | awk '/Mem:/ {printf "%.0f", $3/$2*100}'; }
get_procs() { ps aux --no-heading | wc -l; }

# Decide OK / WARNING / CRITICAL from a value and threshold
status_of() {
    local value=$1 threshold=$2
    if   [ "$value" -ge "$threshold" ]; then
        echo "CRITICAL"
    elif [ "$value" -ge $(( threshold - 20 )) ]; then
        echo "WARNING"
    else
        echo "OK"
    fi
}

colour_for() {
    case "$1" in
        OK)       echo "$GREEN"  ;;
        WARNING)  echo "$YELLOW" ;;
        CRITICAL) echo "$RED"    ;;
    esac
}

# Draw a text progress bar for a percentage
draw_bar() {
    local pct=$1
    local filled=$(( pct * BAR_WIDTH / 100 ))
    local empty=$(( BAR_WIDTH - filled ))
    printf "["
    for (( i=0; i<filled; i++ )); do printf "█"; done
    for (( i=0; i<empty;  i++ )); do printf "░"; done
    printf "] %3d%%" "$pct"
}

# ==============================================================
#  MENU ACTIONS
# ==============================================================

show_dashboard() {
    local disk mem procs dstat mstat
    disk=$(get_disk)
    mem=$(get_mem)
    procs=$(get_procs)
    dstat=$(status_of "$disk" "$DISK_THRESHOLD")
    mstat=$(status_of "$mem"  "$MEM_THRESHOLD")

    echo -e "${BOLD}${CYAN}╔════════════════════════════════════════════╗${NC}"
    echo -e "${BOLD}${CYAN}║         SERVER SENTINEL DASHBOARD          ║${NC}"
    echo -e "${BOLD}${CYAN}╚════════════════════════════════════════════╝${NC}"
    printf "  Host      : %s\n" "$(hostname)"
    printf "  Uptime    : %s\n" "$(uptime -p 2>/dev/null || echo 'n/a')"
    printf "  Time      : %s\n" "$(date '+%H:%M:%S')"
    echo ""
    printf "  Disk   "; draw_bar "$disk"
    echo -e "  $(colour_for "$dstat")[$dstat]${NC}"
    printf "  Memory "; draw_bar "$mem"
    echo -e "  $(colour_for "$mstat")[$mstat]${NC}"
    echo ""
    printf "  Processes : %s\n" "$procs"

    log_event "Dashboard viewed — disk:${disk}% mem:${mem}%"
}

top_consumers() {
    echo -e "${BOLD}Top 5 processes by memory:${NC}"
    printf "  %-10s %-8s %-6s %s\n" "USER" "PID" "%MEM" "COMMAND"
    ps aux --sort=-%mem --no-heading | head -5 | \
        awk '{printf "  %-10s %-8s %-6s %s\n", $1, $2, $4, $11}'
    log_event "Viewed top memory consumers"
}

disk_report() {
    echo -e "${BOLD}Filesystem usage:${NC}"
    df -h | awk 'NR==1 || /^\/dev/ {printf "  %-22s %-8s %-8s %s\n", $1, $2, $5, $6}'
    log_event "Viewed disk report"
}

run_scan() {
    local alerts=0 value
    echo -e "${BOLD}Running full system scan...${NC}"
    echo ""

    for check in disk memory processes; do
        printf "  Checking %-14s" "${check}..."
        sleep 0.3

        case "$check" in
            disk)
                value=$(get_disk)
                if [ "$value" -ge "$DISK_THRESHOLD" ]; then
                    echo -e "${RED}FAIL (${value}%)${NC}"
                    alerts=$(( alerts + 1 ))
                    log_event "ALERT: disk at ${value}%"
                else
                    echo -e "${GREEN}PASS (${value}%)${NC}"
                fi
                ;;
            memory)
                value=$(get_mem)
                if [ "$value" -ge "$MEM_THRESHOLD" ]; then
                    echo -e "${RED}FAIL (${value}%)${NC}"
                    alerts=$(( alerts + 1 ))
                    log_event "ALERT: memory at ${value}%"
                else
                    echo -e "${GREEN}PASS (${value}%)${NC}"
                fi
                ;;
            processes)
                value=$(get_procs)
                if [ "$value" -gt 500 ]; then
                    echo -e "${YELLOW}HIGH ($value)${NC}"
                    alerts=$(( alerts + 1 ))
                    log_event "ALERT: $value processes running"
                else
                    echo -e "${GREEN}PASS ($value)${NC}"
                fi
                ;;
        esac
    done

    echo ""
    if [ "$alerts" -eq 0 ]; then
        echo -e "  ${GREEN}${BOLD}✅ Scan complete — no alerts${NC}"
    else
        echo -e "  ${RED}${BOLD}⚠️  Scan complete — $alerts alert(s)${NC}"
    fi
    log_event "Scan finished with $alerts alert(s)"
}

view_log() {
    if [ -f "$LOGFILE" ] && [ -s "$LOGFILE" ]; then
        echo -e "${BOLD}Last 10 log entries:${NC}"
        tail -10 "$LOGFILE" | sed 's/^/  /'
    else
        echo -e "  ${YELLOW}No log entries yet.${NC}"
    fi
}

live_monitor() {
    echo -e "${BOLD}Live monitor — press Ctrl+C to stop${NC}"
    echo ""
    local rounds=0
    while [ "$rounds" -lt 10 ]; do
        rounds=$(( rounds + 1 ))
        printf "\r  [%02d] Disk: %3s%%   Mem: %3s%%   Procs: %-5s" \
               "$rounds" "$(get_disk)" "$(get_mem)" "$(get_procs)"
        sleep 1
    done
    echo ""
    echo "  Monitor finished after $rounds readings."
}

show_menu() {
    echo ""
    echo -e "${BOLD}${BLUE}══════ SERVER SENTINEL ══════${NC}"
    echo "  1) Dashboard"
    echo "  2) Top memory consumers"
    echo "  3) Disk report"
    echo "  4) Run full scan"
    echo "  5) Live monitor (10s)"
    echo "  6) View log"
    echo "  7) Exit"
    echo -n "  Choose [1-7]: "
}

# ==============================================================
#  MAIN PROGRAM LOOP
# ==============================================================

log_event "Session started"

while true; do
    show_menu
    read -r choice
    echo ""

    case "$choice" in
        1) show_dashboard ;;
        2) top_consumers  ;;
        3) disk_report    ;;
        4) run_scan       ;;
        5) live_monitor   ;;
        6) view_log       ;;
        7)
            echo -e "  ${CYAN}Goodbye! 👋${NC}"
            log_event "Session ended"
            exit 0
            ;;
        *)
            echo -e "  ${RED}❌ '$choice' is not a valid option.${NC}"
            ;;
    esac
done
```

```bash
chmod +x sentinel.sh
./sentinel.sh
```

**Sample dashboard output:**
```
╔════════════════════════════════════════════╗
║         SERVER SENTINEL DASHBOARD          ║
╚════════════════════════════════════════════╝
  Host      : ubuntu
  Uptime    : up 2 minutes
  Time      : 19:38:54

  Disk   [███████████░░░░░░░░░░░░░░]  47%  [OK]
  Memory [█░░░░░░░░░░░░░░░░░░░░░░░░]   6%  [OK]

  Processes : 53
```

---

## 🏅 Grading Rubric (20 points)

| Criterion | Points |
|-----------|--------|
| Script runs without errors, has shebang + header comments | 3 |
| Menu loops correctly and handles invalid input | 3 |
| Dashboard displays with working progress bars | 3 |
| Full scan uses a loop and reports pass/fail per check | 3 |
| Conditionals correctly classify OK / WARNING / CRITICAL | 3 |
| Logging works — events appended with timestamps | 2 |
| Colour output used appropriately and reset with `${NC}` | 2 |
| Exits cleanly with a goodbye message | 1 |

## 🌟 Bonus Extensions (+5 each)

1. **Threshold editor** — a menu option that lets the user change `DISK_THRESHOLD` at runtime
2. **Report export** — save a full report to a timestamped `.txt` file
3. **Network check** — add a menu option that pings hosts in a loop and reports which are reachable
4. **Log rotation** — if the log exceeds 100 lines, keep only the last 50
5. **Startup argument** — support `./sentinel.sh scan` to run a scan directly without the menu

---

# 📌 Class 7 Cheat Sheet

### Script Basics
```bash
#!/bin/bash              # shebang — always line 1
# comment                # explain WHY, not what
chmod +x script.sh       # make executable
./script.sh              # run it
exit 0                   # success  |  exit 1 = failure
echo $?                  # exit code of last command
```

### Variables
```bash
name="value"             # NO SPACES around =
echo "$name"             # access with $, wrap in "quotes"
echo "${name}s"          # braces when touching other text
echo "${#name}"          # length
readonly CONST="x"       # constant
export SHARED="x"        # visible to child processes
read -rp "Prompt: " var  # get user input
```

### Command Substitution
```bash
today=$(date)            # ✅ modern
files=$(ls | wc -l)      # captures pipeline output
```

### Arithmetic
```bash
$(( a + b ))             # ✅ modern, integer only
count=$((count + 1))     # ✅ safest increment
echo "scale=2; 22/7"|bc  # decimals
```

### Conditionals
```bash
if [ cond ]; then ... elif [ cond ]; then ... else ... fi
[ "$a" -gt 5 ] && [ "$b" -lt 10 ]      # AND
[ "$a" -eq 0 ] || [ "$b" -eq 0 ]       # OR
[ ! -f file ]                          # NOT

case "$x" in
    a|b)  cmd ;;
    *.txt) cmd ;;
    *)    default ;;
esac
```

### Test Operators
```bash
# Files
-e exists   -f regular file   -d directory   -s not empty
-r readable -w writable       -x executable  -L symlink

# Strings
-z empty    -n not empty      =  equal       != not equal

# Numbers
-eq  -ne  -gt  -ge  -lt  -le
```

### Loops
```bash
for i in {1..5}; do ... done            # range
for i in {0..20..5}; do ... done        # with step
for x in a b c; do ... done             # list
for f in *.txt; do ... done             # files
for ((i=1; i<=5; i++)); do ... done     # C-style

while [ cond ]; do ... done             # while true
until [ cond ]; do ... done             # until true

while IFS=',' read -r a b; do           # read a file
    ...
done < file.csv
```

### Loop Control
```bash
break        # leave the loop
continue     # skip to next iteration
exit 1       # quit the script
sleep 0.5    # pause (accepts decimals)
```

### Colours
```bash
RED='\033[0;31m'; GREEN='\033[0;32m'
YELLOW='\033[1;33m'; NC='\033[0m'
echo -e "${GREEN}OK${NC}"      # -e is REQUIRED
```

---

# 📚 Homework

1. **Fix the bugs** — I'll give you a broken script in class. Find and fix all 5 errors.
2. **Extend Server Sentinel** — implement at least **two** bonus extensions.
3. **Backup script** — write `backup.sh` that copies a folder to `~/backups/` with a timestamped name, checks the source exists first, and reports how many files were copied.
4. **Multiplication quiz** — write a script that asks 5 random multiplication questions, tracks the score, and gives a grade at the end.
5. **Reading** — run `help test`, `help for`, `help case` in your terminal. Find one operator or feature we didn't cover and write down what it does.

---

# 🚀 Next Class Preview

**Class 8 — Bash Functions, Arguments & Advanced Scripting**
Functions · `$1 $2 $@` arguments · return values · arrays · `getopts` · error handling with `set -euo pipefail` · debugging with `bash -x`

---

*Schull AI Academy · AWS Cloud DevOps & Linux Training · Class 7, Week 4*
