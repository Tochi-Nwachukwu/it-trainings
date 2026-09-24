# Class 2 — Node.js Runtime & Project Setup

### Full-Stack & AI Engineering · Phase 1 · Week 1, Session 2
**Prepared for Chinyere E.** · Schull AI Academy

**Duration:** 2 hours
**Prerequisites:** [Class 1 — The Backend Mental Model](./Class01_Backend_Mental_Model.md)
**You need:** Laptop, VS Code, terminal, internet

---

## 🎯 What You'll Walk Away With

- Explain what Node.js actually *is* — and what it isn't
- Run JavaScript without a browser anywhere on your machine
- Read a `package.json` and know what every line does
- Choose between ES Modules and CommonJS, and fix the error when you pick wrong
- Read, write, and inspect files from code with `fs`
- Build file paths that don't break on someone else's machine
- Keep secrets out of your code with `.env`
- Structure a real Node project from an empty folder

**You'll build:** A working command-line file scanner that reads a folder, groups files by type, and writes a report.

> 🔑 **Last class you looked across the line. Today you step over it.**

---

## 🚪 Opening — The Thing You Already Know

Here's a question that sounds silly but isn't.

**Where does JavaScript run?**

For your entire career so far the answer has been: *in the browser*. Chrome opens your React app, reads your JavaScript, and executes it. The browser is the engine.

So — what happens if you take the engine **out** of the browser and put it on a server?

That's Node.js. That's the entire idea.

```
        BEFORE NODE (2009)                    AFTER NODE
   ┌───────────────────────────┐      ┌───────────────────────────┐
   │        BROWSER            │      │        BROWSER            │
   │  ┌─────────────────────┐  │      │  ┌─────────────────────┐  │
   │  │   V8 engine         │  │      │  │   V8 engine         │  │
   │  │   runs your JS      │  │      │  │   runs your JS      │  │
   │  └─────────────────────┘  │      │  └─────────────────────┘  │
   └───────────────────────────┘      └───────────────────────────┘
                                      
   JavaScript could ONLY               ┌───────────────────────────┐
   run inside a browser.               │      YOUR SERVER          │
                                       │  ┌─────────────────────┐  │
                                       │  │   V8 engine         │  │
                                       │  │   runs your JS      │  │
                                       │  └─────────────────────┘  │
                                       │   + file access           │
                                       │   + networking            │
                                       │   + databases             │
                                       └───────────────────────────┘
```

A developer named **Ryan Dahl** took V8 — the JavaScript engine Google built for Chrome — ripped it out, and wrapped it in a program that could read files, listen on network ports, and talk to databases.

**The consequence for you personally:** you do not need to learn a new language to write a backend. You already know the language. You're learning a new **environment**.

---

### ✏️ ACTIVITY 1 — Prove It To Yourself *(5 minutes)*

Open your terminal. Type `node` and press Enter. You'll get a `>` prompt.

Now type these, one at a time:

```javascript
2 + 2
"hello".toUpperCase()
[1,2,3].map(n => n * 2)
const name = "Chinyere"; `Hi ${name}`
typeof window
typeof document
process.version
process.platform
```

Press `Ctrl+C` twice to exit.

**Now answer:**

1. Which of those worked exactly as they would in the browser console?
2. What did `typeof window` return, and **why**?
3. What did `typeof document` return? What does that tell you about `document.getElementById` on a server?
4. `process` is brand new — it doesn't exist in browsers. What do you think it represents?

<details>
<summary>💡 Answers</summary>

1. Everything except the last four. All the core JavaScript — arrays, strings, template literals, arrow functions — is **identical**. That's the whole point.

2. `'undefined'`. `window` is the **browser's** global object. There is no window on a server — there's no screen, no tab, no user looking at anything.

3. `'undefined'`. **There is no DOM on the server.** No `document`, no `getElementById`, no `querySelector`. This is the single biggest adjustment for a frontend developer: you are not manipulating a page. You're processing data and sending it somewhere else.

4. `process` is Node's global object representing **the running program itself** — its version, its platform, its environment variables, its command-line arguments. Where browsers give you `window`, Node gives you `process`.

**The mental swap:** `window` → `process`. That one line summarises the environment change.
</details>

---

## 1️⃣ What Node.js Actually Is

> **Node.js = the V8 JavaScript engine + a library of things browsers won't let you do.**

| Browser JavaScript can... | Node.js can... |
|---|---|
| Manipulate the DOM | ❌ No DOM exists |
| Respond to clicks | ❌ Nobody's clicking |
| ❌ Read files from disk | ✅ Read and write any file |
| ❌ Listen on a network port | ✅ Run a server |
| ❌ Connect to a database | ✅ Connect to anything |
| ❌ Run shell commands | ✅ Run shell commands |

Same language. Completely different **superpowers**.

### The event loop — revisited, and why it matters more now

You've met this already in React, even if nobody named it. When you write:

```javascript
fetch('/api/data').then(data => setState(data));
```

...your code doesn't freeze while waiting. It carries on, and the callback fires later. That's the **event loop**.

On a server, that property stops being a convenience and becomes the entire architecture:

```
  A BLOCKING SERVER              NODE (non-blocking)
  ─────────────────              ───────────────────
  Request 1 arrives              Request 1 arrives
  → query database               → start DB query, move on
  → WAIT 200ms 😴                Request 2 arrives
  → respond                      → start DB query, move on
  Request 2 arrives              Request 3 arrives
  → query database               → start DB query, move on
  → WAIT 200ms 😴                DB result 1 → respond
  → respond                      DB result 2 → respond
                                 DB result 3 → respond
  Total: 400ms                   Total: ~200ms
```

Node is **single-threaded** but never sits idle. While one request waits on a database, Node serves fifty others.

> ⚠️ **The flip side, and it's important:** because there's only one thread, if *your* code does something genuinely slow and synchronous — a giant loop, a huge synchronous file read — **everything stops for everyone**. This is why you'll see `fs.readFile` (async) preferred over `fs.readFileSync` in anything that serves users.

---

## 2️⃣ Installing Node & Version Management

```bash
node --version    # should print v20.x or v22.x
npm --version     # comes bundled with Node
```

If you get "command not found," install from **nodejs.org** — take the **LTS** version (Long Term Support), not "Current."

> 📌 **LTS vs Current:** LTS is the stable, supported release used in production. Current has the newest features and more surprises. Always LTS for real work.

### Why version managers exist

One day a client's project will need Node 18 while your own project needs Node 22. Installing Node system-wide means you can only have one.

**nvm** (Node Version Manager) solves this:

```bash
# Install nvm (Linux / macOS)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash

nvm install 22        # install a version
nvm use 22            # switch to it
nvm ls                # list what you have
nvm alias default 22  # set the default for new terminals
```

On Windows, use **nvm-windows** or **fnm**.

> 💡 **Not urgent today.** Note it exists. The day you hit a version conflict, you'll remember this paragraph and it'll save you an afternoon.

---

## 3️⃣ npm and package.json

Every Node project starts the same way:

```bash
mkdir my-project && cd my-project
npm init -y
```

That `-y` accepts all defaults. Here's exactly what appears:

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

**`package.json` is your project's ID card.** Every field:

| Field | What it's for |
|-------|---------------|
| `name` | Project name. Lowercase, no spaces. |
| `version` | Your version, in semver (see below) |
| `description` | One line — shows on npm if you publish |
| `main` | The entry file when someone imports your package |
| `type` | `"module"` for ESM, absent for CommonJS ← **matters today** |
| `scripts` | Named shortcuts you run with `npm run` |
| `dependencies` | Packages your app needs to **run** |
| `devDependencies` | Packages you only need while **developing** |

### Installing packages

```bash
npm install dotenv              # a runtime dependency
npm install --save-dev nodemon  # a development-only dependency
npm install                     # install everything in package.json
npm uninstall dotenv            # remove
npm list --depth=0              # what's installed
```

After installing, your `package.json` gains:

```json
{
  "dependencies": {
    "dotenv": "^18.0.3"
  },
  "devDependencies": {
    "nodemon": "^3.1.14"
  }
}
```

### Semantic versioning — what `^18.0.3` means

```
        18   .   0   .   3
        │        │       │
     MAJOR    MINOR   PATCH
        │        │       │
        │        │       └─ bug fix, nothing breaks
        │        └───────── new feature, nothing breaks
        └────────────────── BREAKING CHANGE — your code may stop working
```

The symbol in front controls what npm is allowed to upgrade to:

| Notation | Allows | Meaning |
|----------|--------|---------|
| `^18.0.3` | 18.x.x | Minor + patch updates. **The default.** |
| `~18.0.3` | 18.0.x | Patch updates only. More cautious. |
| `18.0.3` | 18.0.3 exactly | Locked. No updates at all. |

> 🔒 **What's `package-lock.json`?** `package.json` says "I want ^18.0.3." The lock file records the *exact* version actually installed, down to every sub-dependency. **Always commit it.** It's why the project works identically on your machine and your teammate's.

### Scripts — shortcuts worth setting up

```json
"scripts": {
  "start": "node src/index.js",
  "dev": "nodemon src/index.js",
  "scan": "node src/index.js"
}
```

```bash
npm start        # 'start' is special — no 'run' needed
npm run dev
npm run scan
```

> 🗑️ **`node_modules` is never committed.** It can hold tens of thousands of files. Anyone can regenerate it with `npm install`. That's the first line of your `.gitignore`.

---

### ✏️ ACTIVITY 2 — Read a package.json Like a Detective *(8 minutes)*

Here's a real project's `package.json`. Answer the questions **without running anything**.

```json
{
  "name": "order-service",
  "version": "2.4.1",
  "type": "module",
  "main": "src/server.js",
  "scripts": {
    "start": "node src/server.js",
    "dev": "nodemon src/server.js",
    "test": "jest",
    "seed": "node scripts/seed-database.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "pg": "~8.11.0",
    "dotenv": "16.3.1"
  },
  "devDependencies": {
    "jest": "^29.7.0",
    "nodemon": "^3.0.2"
  }
}
```

| # | Question |
|---|----------|
| 1 | Will this project use `import` or `require`? How do you know? |
| 2 | Which command starts the app in production? |
| 3 | Which command do you use while developing, and what does it add? |
| 4 | `express` is `^4.18.2`. Could `npm install` give you 4.19.0? Could it give you 5.0.0? |
| 5 | `pg` is `~8.11.0`. Could that become 8.12.0? |
| 6 | `dotenv` is `16.3.1` with no symbol. What does that mean, and why might someone do it? |
| 7 | Will `jest` be installed on the production server? Should it be? |
| 8 | What do you think `npm run seed` does, and why is it a *script* rather than a dependency? |

<details>
<summary>💡 Answers</summary>

1. **`import`** — `"type": "module"` is present, which switches the project to ES Modules.
2. `npm start` → runs `node src/server.js`.
3. `npm run dev` → `nodemon` watches your files and **restarts the server automatically** every time you save. You'll wonder how you lived without it.
4. **4.19.0 yes. 5.0.0 no.** The caret allows minor and patch, never a major bump — because major means breaking changes.
5. **No.** The tilde allows patch only, so 8.11.1 yes, 8.12.0 no. Someone was being deliberately cautious with the database driver.
6. **Locked to exactly 16.3.1.** Usually done after a bad experience — a version once broke something, so they pinned it.
7. It *will* be installed by a plain `npm install`, but it **shouldn't be** in production. That's what `npm install --production` (or `npm ci --omit=dev`) is for — it skips devDependencies.
8. It fills the database with starter/test data. It's a script because it's a **one-off task you run manually**, not a package the app depends on. Scripts are for jobs; dependencies are for code.
</details>

---

## 4️⃣ ES Modules vs CommonJS

Node has **two** module systems. You need to recognise both, because the internet is full of examples in each.

### CommonJS — the old way (still everywhere)

```javascript
// math.js
function add(a, b) { return a + b; }
const PI = 3.14159;
module.exports = { add, PI };

// main.js
const { add, PI } = require('./math');
console.log(add(2, 3), PI);        // 5 3.14159
```

### ES Modules — the modern way (what you know from React)

```javascript
// math.mjs
export function add(a, b) { return a + b; }
export const PI = 3.14159;
export default function greet(n) { return `Hello ${n}`; }

// main.mjs
import greet, { add, PI } from './math.mjs';
console.log(add(2, 3), PI, greet('Chinyere'));
```

That second one should look extremely familiar — **it's the syntax you already write in React every day.**

### How Node decides which you're using

| Situation | Node treats it as |
|-----------|-------------------|
| `.js` file, no `"type"` in package.json | **CommonJS** |
| `.js` file, `"type": "module"` in package.json | **ES Modules** |
| `.mjs` extension | **Always** ES Modules |
| `.cjs` extension | **Always** CommonJS |

> 🚨 **The error you WILL hit in the next hour.** Write `import` in a `.js` file without `"type": "module"` and Node says:
>
> ```
> Warning: To load an ES module, set "type": "module" in the package.json
> ```
>
> **The fix:** add `"type": "module"` to `package.json`. That's it. When you see this error, don't debug your code — check that one line.

### The one thing you lose with ESM

CommonJS gives you `__dirname` (the current folder) for free. ESM doesn't. Recreate it:

```javascript
import path from 'node:path';
import { fileURLToPath } from 'node:url';

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

console.log(__dirname);   // /home/chinyere/my-project
```

> 📋 **Copy that snippet into a notes file.** You'll paste it into a dozen projects.

**In this course we use ES Modules throughout** — same syntax as React, one less thing to context-switch.

> 💡 **Node's built-in modules can be written two ways:** `import fs from 'fs'` or `import fs from 'node:fs'`. Both work. Prefer the `node:` prefix — it's explicit, marginally faster to resolve, and makes it obvious at a glance that it's built in rather than something from npm.

---

### ✏️ ACTIVITY 3 — Break It On Purpose *(10 minutes)*

You learn error messages by causing them deliberately.

```bash
mkdir module-lab && cd module-lab
npm init -y
```

**Step 1 — cause the error**

```bash
# create app.js
echo "import fs from 'node:fs'; console.log('it works');" > app.js
node app.js
```
👉 **Write down the exact error message.**

**Step 2 — fix it**

Open `package.json` and add `"type": "module"` after the version line. Run again.
👉 Did it work?

**Step 3 — break it the other way**

```bash
echo "const fs = require('node:fs'); console.log('cjs');" > old.js
node old.js
```
👉 What happens now, and **why**? (Hint: what did `"type": "module"` do to every `.js` file?)

**Step 4 — the escape hatch**

```bash
mv old.js old.cjs
node old.cjs
```
👉 Why does it work now?

**Step 5 — make two real modules**

Create `greet.js` that exports a `greet(name)` function and a `VERSION` constant. Import both into `app.js` and print them.

<details>
<summary>💡 What each step teaches</summary>

**Step 1:** Node defaults `.js` to CommonJS. `import` is invalid there.

**Step 3:** `"type": "module"` applies to **every `.js` file in the project**. So `require` — the CommonJS keyword — is now undefined. You'll see `ReferenceError: require is not defined`. This is the reverse of the same coin.

**Step 4:** `.cjs` **always** means CommonJS, regardless of `"type"`. It's the per-file override. (`.mjs` is the mirror image — always ESM.)

**Step 5 — model answer:**

```javascript
// greet.js
export const VERSION = '1.0.0';
export function greet(name) {
  return `Hello, ${name}! Welcome to Node.`;
}

// app.js
import { greet, VERSION } from './greet.js';
console.log(greet('Chinyere'));
console.log('Version:', VERSION);
```

⚠️ **Note the `.js` extension in the import.** In React, bundlers let you write `from './greet'`. **Node does not.** You must include the file extension in relative imports. This catches everyone exactly once.
</details>

---

## 5️⃣ The `path` Module — Build Paths Properly

Before `fs`, learn `path`. It prevents a whole class of bug.

**The problem:**

```javascript
const file = 'data' + '/' + 'reports' + '/' + 'summary.txt';
```

That works on your machine. On Windows the separator is `\`, not `/`. Your code breaks on a teammate's laptop, or on a server.

**The fix:**

```javascript
import path from 'node:path';

path.join('data', 'reports', 'summary.txt')
// → 'data/reports/summary.txt'   on Mac/Linux
// → 'data\reports\summary.txt'   on Windows
```

### The methods you'll use constantly

```javascript
import path from 'node:path';

path.join('data', 'reports', 'file.txt')   // 'data/reports/file.txt'
path.resolve('data', 'file.txt')           // '/home/you/project/data/file.txt'  (absolute)

path.basename('/home/ada/report.pdf')      // 'report.pdf'
path.extname('/home/ada/report.pdf')       // '.pdf'
path.dirname('/home/ada/report.pdf')       // '/home/ada'

path.parse('/home/ada/report.pdf')
// {
//   root: '/',
//   dir:  '/home/ada',
//   base: 'report.pdf',
//   ext:  '.pdf',
//   name: 'report'
// }

path.sep                                   // '/' on Unix, '\\' on Windows
```

| | `join` | `resolve` |
|---|--------|-----------|
| Does | Glues segments together | Produces an **absolute** path from where you are |
| Returns | Relative if inputs were relative | Always absolute |
| Use for | Building a path from parts | When you need a full, unambiguous path |

> 🔑 **Rule: never concatenate paths with `+` or template strings.** Always `path.join()`. It's one extra import and it removes an entire bug category.

---

## 6️⃣ The `fs` Module — Reading and Writing Files

This is the superpower browsers deny you.

`fs` comes in two flavours, and picking the right one matters:

```javascript
import fs from 'node:fs';            // callback + sync versions
import fsp from 'node:fs/promises';  // promise versions  ← use this one
```

> ✅ **Use `node:fs/promises`.** It gives you `async/await`, which you already know from React. The old callback style leads to nested pyramids of doom.

### Writing

```javascript
import fsp from 'node:fs/promises';

// Write (creates the file, or OVERWRITES it completely)
await fsp.writeFile('notes.txt', 'line one\nline two\n');

// Append (adds to the end — safe)
await fsp.appendFile('notes.txt', 'line three\n');

// Make a folder (recursive: true means "create parents, don't error if it exists")
await fsp.mkdir('reports/2026', { recursive: true });
```

> 🚨 **`writeFile` destroys existing content immediately.** There's no warning and no undo. When you mean "add," use `appendFile`.

### Reading

```javascript
const text = await fsp.readFile('notes.txt', 'utf-8');
console.log(text);
// line one
// line two
// line three
```

> ⚠️ **Never forget `'utf-8'`.** Leave it out and you get a raw `Buffer` — `<Buffer 6c 69 6e 65...>` — instead of a string. Everyone does this once.

### Working with JSON

```javascript
// Save an object
const data = { name: 'Chinyere', skills: ['React', 'Node'] };
await fsp.writeFile('data.json', JSON.stringify(data, null, 2));

// Load it back
const loaded = JSON.parse(await fsp.readFile('data.json', 'utf-8'));
console.log(loaded.skills[1]);   // 'Node'
```

The `null, 2` in `JSON.stringify` is what makes the file human-readable instead of one long line. Always include it for files you might open.

### Listing a folder

```javascript
const entries = await fsp.readdir('demo', { withFileTypes: true });

for (const entry of entries) {
  console.log(entry.name, entry.isDirectory() ? 'DIR' : 'FILE');
}
```

`withFileTypes: true` is what lets you tell files from folders. Without it you get plain strings and no way to know.

### File details with `stat`

```javascript
const stats = await fsp.stat('demo/notes.txt');

stats.size        // 39          — bytes
stats.mtime       // Date object — last modified
stats.birthtime   // Date object — created
stats.isFile()    // true
stats.isDirectory() // false
```

### Checking existence

```javascript
import fs from 'node:fs';

if (fs.existsSync('config.json')) { /* ... */ }    // simple, synchronous

// or the async way
try {
  await fsp.access('config.json');
} catch {
  console.log('file does not exist');
}
```

---

### ✏️ ACTIVITY 4 — Your First File Program *(12 minutes)*

Create `file-lab.js`. Build it **step by step**, running after each step so you see it grow.

**Step 1** — Create a folder called `playground` and write `hello.txt` containing three lines.

**Step 2** — Read it back and print it.

**Step 3** — Append a fourth line. Read again. Print how many lines there are now.

**Step 4** — Write an object to `profile.json` with your name, your skills as an array, and today's date. Read it back and print just the skills.

**Step 5** — List everything in `playground`. For each entry print the **name**, whether it's a **file or folder**, its **size in bytes**, and its **extension**.

**Step 6** — Print the total size of all files combined.

<details>
<summary>💡 Full solution</summary>

```javascript
import fsp from 'node:fs/promises';
import path from 'node:path';

// Step 1
await fsp.mkdir('playground', { recursive: true });
await fsp.writeFile('playground/hello.txt', 'line one\nline two\nline three\n');

// Step 2
const text = await fsp.readFile('playground/hello.txt', 'utf-8');
console.log('--- contents ---');
console.log(text);

// Step 3
await fsp.appendFile('playground/hello.txt', 'line four\n');
const updated = await fsp.readFile('playground/hello.txt', 'utf-8');
console.log('lines now:', updated.trim().split('\n').length);   // 4

// Step 4
const profile = {
  name: 'Chinyere',
  skills: ['React', 'JavaScript', 'Node.js'],
  date: new Date().toISOString().slice(0, 10),
};
await fsp.writeFile('playground/profile.json', JSON.stringify(profile, null, 2));

const loaded = JSON.parse(await fsp.readFile('playground/profile.json', 'utf-8'));
console.log('skills:', loaded.skills);

// Step 5
console.log('\n--- folder contents ---');
const entries = await fsp.readdir('playground', { withFileTypes: true });
let total = 0;

for (const entry of entries) {
  const full = path.join('playground', entry.name);
  const stats = await fsp.stat(full);
  total += stats.size;

  console.log(
    `${entry.isDirectory() ? 'DIR ' : 'FILE'} ` +
    `${entry.name.padEnd(15)} ` +
    `${String(stats.size).padStart(6)} bytes  ` +
    `ext=${path.extname(entry.name) || '(none)'}`
  );
}

// Step 6
console.log(`\nTotal size: ${total} bytes`);
```

**Expected output:**
```
--- contents ---
line one
line two
line three

lines now: 4
skills: [ 'React', 'JavaScript', 'Node.js' ]

--- folder contents ---
FILE hello.txt            39 bytes  ext=.txt
FILE profile.json         98 bytes  ext=.json

Total size: 137 bytes
```

**Notice what you just did:** you wrote a program that inspects the filesystem. Your React app could never do any of that.
</details>

---

## 7️⃣ Environment Variables & `.env`

Remember Wall 2 from last class — secrets must never live in code. Here's how you actually do it.

### The problem

```javascript
const DATABASE_PASSWORD = "myS3cretP@ss";   // ❌ now it's in Git, forever
```

### The solution

```bash
npm install dotenv
```

Create a `.env` file in your project root:

```bash
APP_NAME=FileScanner
PORT=3000
API_KEY=super-secret-do-not-commit
DEBUG=true
```

Load it at the very top of your entry file:

```javascript
import 'dotenv/config';

console.log(process.env.APP_NAME);   // 'FileScanner'
console.log(process.env.PORT);       // '3000'
```

### 🚨 Two traps that catch everyone

**Trap 1 — everything is a string.**

```javascript
console.log(typeof process.env.PORT);       // 'string'  ← not a number!
console.log(typeof process.env.DEBUG);      // 'string'

process.env.DEBUG === true      // false  😱
process.env.DEBUG === 'true'    // true   ✅

Number(process.env.PORT) + 1    // 3001   ← convert explicitly
```

**Trap 2 — `.env` must be gitignored.** The entire point is defeated if you commit it.

```gitignore
.env
```

> 🤝 **The convention that solves the obvious problem:** if `.env` isn't committed, how does a teammate know what variables to set? You commit a **`.env.example`** with the keys but fake values:
> ```bash
> APP_NAME=YourAppName
> PORT=3000
> API_KEY=get-this-from-the-team-lead
> ```
> Real values in `.env` (ignored), the shape of it in `.env.example` (committed).

### Sensible defaults

```javascript
export const config = {
  port:    process.env.PORT ?? 3000,
  appName: process.env.APP_NAME ?? 'MyApp',
  debug:   process.env.DEBUG === 'true',
};
```

The `??` (nullish coalescing) means "use this if the left side is null or undefined." Your app then runs even if someone forgets a variable.

---

### ✏️ ACTIVITY 5 — Secrets Done Right *(8 minutes)*

1. In your `module-lab` folder, install `dotenv`
2. Create a `.env` with: `APP_NAME`, `PORT`, `SECRET_KEY`, `DEBUG`
3. Create `.env.example` with the same keys but placeholder values
4. Create `.gitignore` ignoring `node_modules/` and `.env`
5. Write `config.js` that loads them, converts `PORT` to a **number** and `DEBUG` to a **boolean**, and provides defaults for all four
6. Import it into `app.js` and print the whole config object
7. **Now the test:** delete `PORT` from `.env` and run again. Does your app still work?

<details>
<summary>💡 Solution</summary>

```javascript
// config.js
import 'dotenv/config';

export const config = {
  appName:   process.env.APP_NAME   ?? 'DefaultApp',
  port:      Number(process.env.PORT ?? 3000),
  secretKey: process.env.SECRET_KEY ?? null,
  debug:     process.env.DEBUG === 'true',
};

// warn loudly if something important is missing
if (!config.secretKey) {
  console.warn('⚠️  SECRET_KEY is not set — check your .env file');
}
```

```javascript
// app.js
import { config } from './config.js';

console.log('Config loaded:', config);
console.log('port is a number?', typeof config.port === 'number');
console.log('debug is a boolean?', typeof config.debug === 'boolean');
```

**Step 7 result:** it should still run, falling back to `3000`. That's the whole purpose of defaults — a missing optional variable should never crash your app. A missing *critical* one (like a database password) should fail loudly and immediately, which is why the `SECRET_KEY` warning is there.
</details>

---

## 8️⃣ Project Structure

Everything in one `index.js` works — until it's 600 lines and you can't find anything.

**The structure we'll use all course:**

```
my-project/
├── package.json
├── package-lock.json
├── .env                  ← secrets (NEVER committed)
├── .env.example          ← the shape of .env (committed)
├── .gitignore
├── README.md
├── node_modules/         ← NEVER committed
└── src/
    ├── index.js          ← entry point — wires things together
    ├── config.js         ← all configuration in one place
    └── <feature>.js      ← one file per responsibility
```

**The principle: one file, one job.** `index.js` orchestrates. Everything else does one thing well. When a file starts doing two things, split it.

### A `.gitignore` for Node

```gitignore
node_modules/
.env
*.log
.DS_Store
dist/
coverage/
```

### A README worth writing

```markdown
# Project Name

One sentence on what this does.

## Setup
​```bash
npm install
cp .env.example .env    # then fill in real values
​```

## Usage
​```bash
npm start
​```

## Environment Variables
| Variable | Description | Default |
|----------|-------------|---------|
| PORT     | Port to run on | 3000 |
```

> 💡 **Write the README first, before the code.** If you can't explain what it does in one sentence, you don't know what you're building yet.

---

## 🏆 THE MAIN LAB — Build a File Scanner CLI

**Time: 30 minutes.** This is today's deliverable.

### The brief

A command-line tool that scans a folder, groups files by type, and writes a summary report to disk.

**Requirements:**
1. Accepts a folder path as a command-line argument
2. Lists every file, with size and extension
3. Groups by file type with counts and total sizes
4. Shows the 5 largest files
5. Writes a timestamped report to `reports/`
6. Supports `--ext=.csv` to filter by type
7. Fails gracefully if the folder doesn't exist
8. Uses `.env` for configuration
9. Split across multiple modules — not one file

### Reading command-line arguments

```javascript
console.log(process.argv);
// [ '/usr/bin/node', '/path/to/script.js', './reports', '--ext=.txt' ]
//   ↑ always node      ↑ always your file    ↑ what the user typed

const args = process.argv.slice(2);   // ← skip the first two, always
```

### Setup

```bash
mkdir file-scanner && cd file-scanner
npm init -y
npm install dotenv
mkdir -p src reports sample-files
```

Add `"type": "module"` to `package.json`, plus scripts:

```json
"type": "module",
"scripts": {
  "scan": "node src/index.js",
  "start": "node src/index.js"
}
```

Create some sample files to scan:

```bash
cd sample-files
echo "hello world" > notes.txt
echo "meeting minutes" > meeting.txt
printf 'name,score\nada,90\nbola,75\n' > data.csv
printf 'id,value\n1,100\n' > metrics.csv
echo '{"a":1}' > config.json
echo '{"b":2}' > settings.json
echo "console.log('hi')" > app.js
echo "export default {}" > utils.js
echo "# README" > README.md
head -c 3000 /dev/urandom > photo.jpg
cd ..
```

### Try it yourself first

**Work through it before opening the solution.** Build it in this order:

1. `src/config.js` — load `.env` with defaults
2. `src/scanner.js` — `scanDirectory()`, `groupByType()`, `filterByExtension()`
3. `src/report.js` — `formatBytes()`, `buildReport()`, `saveReport()`
4. `src/index.js` — read args, call everything, handle errors

<details>
<summary>💡 Full working solution</summary>

**`.env`**
```bash
SCAN_DIR=./sample-files
REPORT_DIR=./reports
REPORT_NAME=summary
VERBOSE=false
```

**`.gitignore`**
```gitignore
node_modules/
.env
reports/
*.log
.DS_Store
```

**`src/config.js`**
```javascript
import 'dotenv/config';

export const config = {
  scanDir:    process.env.SCAN_DIR    ?? './sample-files',
  reportDir:  process.env.REPORT_DIR  ?? './reports',
  reportName: process.env.REPORT_NAME ?? 'summary',
  verbose:    process.env.VERBOSE === 'true',
};
```

**`src/scanner.js`**
```javascript
import fsp from 'node:fs/promises';
import path from 'node:path';

/** Read a directory and return details for every file inside it. */
export async function scanDirectory(dir) {
  const entries = await fsp.readdir(dir, { withFileTypes: true });
  const files = [];

  for (const entry of entries) {
    if (entry.isDirectory()) continue;          // skip folders for now

    const fullPath = path.join(dir, entry.name);
    const stats = await fsp.stat(fullPath);

    files.push({
      name:     entry.name,
      ext:      path.extname(entry.name).toLowerCase() || '(none)',
      size:     stats.size,
      modified: stats.mtime,
    });
  }
  return files;
}

/** Group files by extension and total their sizes. */
export function groupByType(files) {
  const groups = {};
  for (const file of files) {
    if (!groups[file.ext]) groups[file.ext] = { count: 0, totalSize: 0, files: [] };
    groups[file.ext].count += 1;
    groups[file.ext].totalSize += file.size;
    groups[file.ext].files.push(file.name);
  }
  return groups;
}

export function filterByExtension(files, ext) {
  if (!ext) return files;
  const wanted = ext.startsWith('.') ? ext.toLowerCase() : '.' + ext.toLowerCase();
  return files.filter(f => f.ext === wanted);
}
```

**`src/report.js`**
```javascript
import fsp from 'node:fs/promises';
import path from 'node:path';

export function formatBytes(bytes) {
  if (bytes < 1024) return `${bytes} B`;
  if (bytes < 1024 * 1024) return `${(bytes / 1024).toFixed(1)} KB`;
  return `${(bytes / 1024 / 1024).toFixed(2)} MB`;
}

export function buildReport(dir, files, groups) {
  const totalSize = files.reduce((sum, f) => sum + f.size, 0);
  const lines = [];

  lines.push('='.repeat(52));
  lines.push('           FILE SCANNER REPORT');
  lines.push('='.repeat(52));
  lines.push(`Scanned folder : ${path.resolve(dir)}`);
  lines.push(`Generated      : ${new Date().toISOString()}`);
  lines.push(`Total files    : ${files.length}`);
  lines.push(`Total size     : ${formatBytes(totalSize)}`);
  lines.push('');
  lines.push('BY FILE TYPE');
  lines.push('-'.repeat(52));
  lines.push('TYPE        COUNT      SIZE   SHARE');
  lines.push('-'.repeat(52));

  const sorted = Object.entries(groups).sort((a, b) => b[1].count - a[1].count);
  for (const [ext, info] of sorted) {
    const share = totalSize ? ((info.totalSize / totalSize) * 100).toFixed(1) : '0.0';
    lines.push(
      ext.padEnd(11) +
      String(info.count).padStart(5) +
      formatBytes(info.totalSize).padStart(10) +
      `${share}%`.padStart(8)
    );
  }

  lines.push('');
  lines.push('LARGEST FILES');
  lines.push('-'.repeat(52));
  [...files].sort((a, b) => b.size - a.size).slice(0, 5)
    .forEach((f, i) => lines.push(`${i + 1}. ${f.name.padEnd(28)} ${formatBytes(f.size).padStart(10)}`));

  lines.push('');
  lines.push('='.repeat(52));
  return lines.join('\n');
}

export async function saveReport(reportDir, name, content) {
  await fsp.mkdir(reportDir, { recursive: true });
  const stamp = new Date().toISOString().slice(0, 10);
  const file = path.join(reportDir, `${name}-${stamp}.txt`);
  await fsp.writeFile(file, content, 'utf-8');
  return file;
}
```

**`src/index.js`**
```javascript
import fs from 'node:fs';
import { config } from './config.js';
import { scanDirectory, groupByType, filterByExtension } from './scanner.js';
import { buildReport, saveReport } from './report.js';

async function main() {
  const args = process.argv.slice(2);
  const dir = args.find(a => !a.startsWith('--')) ?? config.scanDir;
  const extArg = args.find(a => a.startsWith('--ext='))?.split('=')[1];

  if (!fs.existsSync(dir)) {
    console.error(`❌ Folder not found: ${dir}`);
    process.exit(1);
  }

  console.log(`🔍 Scanning ${dir} ...`);

  let files = await scanDirectory(dir);
  if (extArg) {
    files = filterByExtension(files, extArg);
    console.log(`   filtered to "${extArg}" → ${files.length} file(s)`);
  }

  if (files.length === 0) {
    console.log('📭 No files matched.');
    return;
  }

  const groups = groupByType(files);
  const report = buildReport(dir, files, groups);
  console.log('\n' + report);

  const saved = await saveReport(config.reportDir, config.reportName, report);
  console.log(`\n💾 Report saved to ${saved}`);
}

main().catch(err => {
  console.error('💥 Something went wrong:', err.message);
  process.exit(1);
});
```

### Running it

```bash
node src/index.js
```
```
🔍 Scanning ./sample-files ...

====================================================
           FILE SCANNER REPORT
====================================================
Scanned folder : /home/chinyere/file-scanner/sample-files
Generated      : 2026-09-22T18:37:01.503Z
Total files    : 10
Total size     : 3.1 KB

BY FILE TYPE
----------------------------------------------------
TYPE        COUNT      SIZE   SHARE
----------------------------------------------------
.js            2      36 B    1.1%
.json          2      22 B    0.7%
.csv           2      41 B    1.3%
.txt           2      33 B    1.1%
.md            1       9 B    0.3%
.jpg           1    2.9 KB   95.5%

LARGEST FILES
----------------------------------------------------
1. photo.jpg                        2.9 KB
2. data.csv                           26 B
3. meeting.txt                        21 B
4. app.js                             18 B
5. utils.js                           18 B

====================================================

💾 Report saved to reports/summary-2026-09-22.txt
```

**Filtering:**
```bash
node src/index.js ./sample-files --ext=.csv
```
```
🔍 Scanning ./sample-files ...
   filtered to ".csv" → 2 file(s)
...
.csv           2      41 B  100.0%
```

**Error handling:**
```bash
node src/index.js ./does-not-exist
```
```
❌ Folder not found: ./does-not-exist
```
(and it exits with code `1` — check with `echo $?`)
</details>

### ✅ Lab checklist

- [ ] `npm run scan` produces a report
- [ ] A file appears in `reports/` with today's date in the name
- [ ] `--ext=.csv` filters correctly
- [ ] A non-existent folder gives a clean error, not a stack trace
- [ ] `node_modules/` and `.env` are in `.gitignore`
- [ ] The code is split across at least three files in `src/`
- [ ] `echo $?` prints `1` after the error case

---

## 🏁 End-of-Class Challenge *(15 minutes)*

### Part 1 — Extend the scanner

Pick **two**:

1. **`--sort=size`** — sort output by size instead of count
2. **`--json`** — also write a `.json` version of the report
3. **Recursive scanning** — a `--deep` flag that descends into subfolders (hint: make `scanDirectory` call itself)
4. **Age report** — show the oldest and newest file using `stats.mtime`
5. **`--min-size=1000`** — only include files above a byte threshold

### Part 2 — Explain it back

Two sentences each, out loud:

1. What is Node.js, in terms a non-programmer would follow?
2. Why does `typeof window` return `undefined` in Node?
3. What does `"type": "module"` change, and what error appears without it?
4. Why use `path.join()` instead of string concatenation?
5. What's the difference between `dependencies` and `devDependencies`?
6. Why is `.env` gitignored, and what do you commit instead?

### Part 3 — Debug these

What's wrong with each?

```javascript
// A
import fsp from 'node:fs/promises';
const text = await fsp.readFile('notes.txt');
console.log(text.toUpperCase());
```

```javascript
// B
import { helper } from './utils';
```

```javascript
// C
const PORT = process.env.PORT;
if (PORT > 3000) { console.log('high port'); }
```

```javascript
// D
await fsp.writeFile('log.txt', 'new entry\n');
await fsp.writeFile('log.txt', 'another entry\n');
```

<details>
<summary>💡 Answers</summary>

**A — missing encoding.** Without `'utf-8'`, `readFile` returns a `Buffer`, which has no `.toUpperCase()`. Fix: `readFile('notes.txt', 'utf-8')`.

**B — missing file extension.** Node requires it in relative ESM imports. Bundlers let you omit it; Node doesn't. Fix: `from './utils.js'`.

**C — string vs number comparison.** `process.env.PORT` is always a string. `'3000' > 3000` does a coercion that will surprise you, and `'900' > 3000` behaves unexpectedly too. Fix: `const PORT = Number(process.env.PORT)`.

**D — the second write erases the first.** `writeFile` overwrites. The file ends up containing only `another entry`. Fix: use `appendFile` for the second call.
</details>

---

## 📌 Class 2 Cheat Sheet

```
WHAT NODE IS
  V8 engine + file access + networking, outside the browser
  No window. No document. No DOM.
  You get `process` instead of `window`

NPM
  npm init -y                    create package.json
  npm install <pkg>              runtime dependency
  npm install -D <pkg>           dev-only dependency
  npm install                    install everything
  npm run <script>               run a script
  npm start                      special — no 'run' needed

SEMVER
  ^18.0.3   minor + patch allowed    (default)
  ~18.0.3   patch only
   18.0.3   locked exactly
  MAJOR.MINOR.PATCH → breaking.feature.bugfix

MODULES
  "type": "module"  in package.json  → use import/export
  no "type"                          → use require
  .mjs  always ESM     .cjs  always CommonJS
  ⚠️  relative imports NEED the .js extension in Node

  // recreate __dirname in ESM
  import { fileURLToPath } from 'node:url';
  const __dirname = path.dirname(fileURLToPath(import.meta.url));

PATH
  path.join('a','b','c.txt')     safe joining      ← always use this
  path.resolve(...)              absolute path
  path.basename / extname / dirname / parse

FS  (import fsp from 'node:fs/promises')
  await fsp.readFile(f, 'utf-8')         ⚠️ encoding or you get a Buffer
  await fsp.writeFile(f, data)           ⚠️ OVERWRITES
  await fsp.appendFile(f, data)          safe add
  await fsp.mkdir(d, { recursive: true })
  await fsp.readdir(d, { withFileTypes: true })
  await fsp.stat(f)                      .size .mtime .isFile()
  fs.existsSync(f)

ENV
  import 'dotenv/config';
  process.env.ANYTHING          ⚠️ ALWAYS a string
  Number(process.env.PORT)
  process.env.DEBUG === 'true'
  .env → gitignored    .env.example → committed

CLI ARGS
  process.argv.slice(2)          ← skip node + script path

PROJECT LAYOUT
  package.json · .env · .env.example · .gitignore · README.md
  src/index.js · src/config.js · src/<feature>.js
```

---

## ✅ Self-Check

- [ ] I can explain Node without saying "it's JavaScript for the backend"
- [ ] I know why there's no `document` on a server
- [ ] I can read any `package.json` and explain every field
- [ ] I know what `^`, `~` and a bare version number allow
- [ ] I can switch a project between ESM and CommonJS
- [ ] I recognise the `"type": "module"` error instantly
- [ ] I always use `path.join()` instead of `+`
- [ ] I know `writeFile` overwrites and `appendFile` doesn't
- [ ] I never forget `'utf-8'` on `readFile`
- [ ] I know every `process.env` value is a string
- [ ] My file scanner runs, filters, and saves a report

---

## 📚 Homework

1. **Finish the scanner.** Implement at least two extensions from Part 1. Push it to GitHub with a proper README.

2. **Build a second CLI tool** — a **note-taker**:
   - `node notes.js add "Buy milk"` → appends to `notes.json`
   - `node notes.js list` → prints all notes, numbered
   - `node notes.js done 2` → marks note 2 complete
   - `node notes.js clear` → removes completed notes
   - Notes must survive between runs

3. **Explore your own machine.** Write a script that scans your Downloads folder and reports: total files, total size, the 10 largest, and how many of each type. Run it. You will almost certainly find something to delete.

4. **Read the docs.** Open the Node docs for `fs/promises`. Find **three** methods we didn't cover and write one line on each. (`rename`, `copyFile` and `rm` are good places to start.)

5. **Set up nvm** on your machine. Install two Node versions and switch between them. Verify with `node --version`.

---

## 🚀 Next Class

**Class 3 — Express Framework Fundamentals**

You can now run JavaScript on a server and touch the filesystem. Next you make it **listen** — your first HTTP server, routing, and a real API that your React app could actually call.

**Come with:**
- The file scanner working and pushed to GitHub
- Node installed and `node --version` working
- Postman still installed from Class 1

---

*Schull AI Academy · Full-Stack & AI Engineering · Class 2 of 40*
