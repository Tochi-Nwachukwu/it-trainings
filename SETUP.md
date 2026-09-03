# ⚙️ SETUP — Getting Python Running

> Complete this **before** Lesson 11. Takes about 20 minutes.

---

## Step 1 — Check If You Already Have Python

Open a terminal (Linux/macOS) or PowerShell (Windows) and type:

```bash
python3 --version
```

If you see something like `Python 3.12.3`, you're done — **skip to Step 3**.

If you see `command not found`, continue to Step 2.

> ⚠️ **We need Python 3.10 or newer.** If you have Python 3.8 or 3.9, some features in Lesson 12 won't work. Upgrade.

---

## Step 2 — Install Python

### 🐧 Linux (Ubuntu / Debian)

Python 3 comes pre-installed on Ubuntu. If it's missing or too old:

```bash
sudo apt update
sudo apt install -y python3 python3-pip python3-venv
python3 --version
```

### 🪟 Windows

1. Go to **https://www.python.org/downloads/**
2. Click the big yellow **Download Python 3.x** button
3. Run the installer
4. ⚠️ **CRITICAL:** Tick the box that says **"Add Python to PATH"** at the bottom of the first screen. If you miss this, nothing will work.
5. Click **Install Now**
6. Open PowerShell and verify:
   ```powershell
   python --version
   ```

> On Windows the command is `python`, not `python3`. Everywhere this guide says `python3`, use `python` instead.

### 🍎 macOS

macOS ships with an old Python. Install a modern one with Homebrew:

```bash
# Install Homebrew first if you don't have it
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Then install Python
brew install python

python3 --version
```

---

## Step 3 — Install VS Code (Your Code Editor)

You *can* write Python in Notepad, but VS Code makes it far easier — it colours your code, spots mistakes, and runs programs with one click.

1. Download from **https://code.visualstudio.com/**
2. Install it
3. Open VS Code
4. Click the **Extensions** icon in the left sidebar (four squares)
5. Search for and install:

| Extension | Publisher | What it does |
|-----------|-----------|--------------|
| **Python** | Microsoft | Syntax highlighting, running code, IntelliSense |
| **Pylance** | Microsoft | Smart autocomplete and error detection |
| **Code Runner** | Jun Han | Run any file with one click |

---

## Step 4 — Your First Program

1. In VS Code: **File → Open Folder** → create a folder called `python-work`
2. **File → New File** → save it as `hello.py`
3. Type this (don't copy-paste — type it):

```python
print("Hello, Python!")
print("My name is <put your name here>")
```

4. Press **`Ctrl` + `` ` ``** (backtick) to open the terminal inside VS Code
5. Run it:

```bash
python3 hello.py
```

You should see:
```
Hello, Python!
My name is Nnamdi
```

🎉 **You are now a Python programmer.**

---

## Step 5 — Understanding Virtual Environments

A **virtual environment** is a private box for one project's packages, so different projects don't clash.

> 🧺 **Analogy:** Imagine you and your sibling both cook. If you share one spice rack, one person's changes ruin the other's recipes. A virtual environment gives each project its **own spice rack**.

### Creating and using one

```bash
# Create a virtual environment called "venv"
python3 -m venv venv

# Activate it
source venv/bin/activate         # Linux / macOS
venv\Scripts\activate            # Windows

# Your prompt now shows (venv) at the start
# Install packages safely inside it
pip install requests

# See what's installed
pip list

# Leave the environment
deactivate
```

> 📌 We cover this properly in **Lesson 16**. For now, just know the commands exist.

---

## 🎮 The Three Ways to Run Python

| Method | Command | Best for |
|--------|---------|----------|
| **Interactive shell** | `python3` | Testing a quick idea |
| **Script file** | `python3 myfile.py` | Real programs — this is what we mostly use |
| **VS Code Run button** | ▶️ top-right | Convenience while learning |

### Try the interactive shell right now

```bash
python3
```
```
>>> 2 + 2
4
>>> name = "Sese"
>>> print(f"Hi {name}")
Hi Sese
>>> exit()
```

> 💡 The `>>>` prompt means Python is waiting for you. Type `exit()` or press `Ctrl+D` to leave.

---

## 🩺 Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| `python3: command not found` | Not installed, or not on PATH | Reinstall; on Windows tick "Add to PATH" |
| `SyntaxError: invalid syntax` | A typo — missing `:` or unbalanced `(` | Read the line number in the error |
| `IndentationError` | Wrong spacing at the start of a line | Use exactly 4 spaces, never mix tabs and spaces |
| `ModuleNotFoundError` | Package isn't installed | `pip install <package-name>` |
| `'python' is not recognized` (Windows) | PATH issue | Reinstall Python, tick "Add Python to PATH" |

---

## ✅ Setup Complete Checklist

- [ ] `python3 --version` shows 3.10 or higher
- [ ] VS Code is installed with the Python extension
- [ ] I created and ran `hello.py` successfully
- [ ] I opened the interactive shell and did some maths in it
- [ ] I know how to open the terminal inside VS Code

**All ticked?** → Go to [Lesson 11](lessons/11-python-basics.md) 🚀
