# Class 3 — Express Framework Fundamentals

**Full-Stack & AI Engineering · Phase 1 · Week 2, Session 1**
**Prepared for Chinyere E. · Schull AI Academy**

---

## 🎯 What You'll Walk Away With

By the end of today you will have built a working REST API with six endpoints, and you will be able to:

- Say what a framework actually does for you, with a before-and-after you wrote yourself
- Start an Express server and understand every line of the four lines that do it
- Define routes for GET, POST, PUT, PATCH and DELETE, and explain why their order on the page matters
- Pull data out of a URL two different ways and know which way to reach for
- Read a request object and shape a response object on purpose rather than by accident
- Return the right status code without looking it up
- Keep your server restarting itself while you type
- Split your routes into their own file with `express.Router()` so your project does not turn into one 900-line monster

---

## 🚪 Opening Hook: You Already Know What a Framework Is

Open a terminal and run this. It is three lines.

```bash
mkdir express-warmup && cd express-warmup
npm init -y > /dev/null
npm install express
```

Now, before we touch Express, I want you to look at two files that do **exactly** the same job.

**File 1, the way Node does it on its own:**

```javascript
import http from 'node:http';

const server = http.createServer((req, res) => {
  if (req.url === '/notes' && req.method === 'GET') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify([{ id: 1, title: 'Hello' }]));
    return;
  }
  res.writeHead(404, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ error: 'Not found' }));
});

server.listen(4001, () => console.log('raw node on 4001'));
```

**File 2, the same job in Express:**

```javascript
import express from 'express';
const app = express();

app.get('/notes', (req, res) => {
  res.json([{ id: 1, title: 'Hello' }]);
});

app.listen(4002, () => console.log('express on 4002'));
```

Both answer `GET /notes` with the same JSON. I ran both. Here is the real output.

```
$ curl -i http://localhost:4001/notes
HTTP/1.1 200 OK
Content-Type: application/json
...
[{"id":1,"title":"Hello"}]

$ curl -i http://localhost:4002/notes
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 26
ETag: W/"1a-Eel4I7tF3UrV5lVxlZQjLjIxNGU"
...
[{"id":1,"title":"Hello"}]
```

Same answer. But count what the Express version never had to say:

| Job | Raw Node | Express |
|---|---|---|
| Check the URL | `if (req.url === '/notes')` | built into `app.get('/notes')` |
| Check the method | `&& req.method === 'GET'` | built into `.get` |
| Set Content-Type | you type it | `res.json()` sets it |
| Convert to JSON text | `JSON.stringify(...)` | `res.json()` does it |
| Set Content-Length | nobody set it, so Node chunked it | Express calculated 26 |
| Handle an unknown URL | your own `else` branch | free |

That last one is worth proving. I asked both servers for a path neither of them defines:

```
$ curl -o /dev/null -w "status=%{http_code}\n" http://localhost:4002/nope
status=404
```

I never wrote a 404 in the Express file. It came with the framework.

**Here is the honest version of what Express is:** it is a pile of `if` statements about URLs and methods that somebody already wrote, tested, and gave away. That is all a framework ever is. React is the same deal for the browser. You do not hand-write `document.createElement` and `addEventListener` and a diffing algorithm every time you need a button; you write `<button onClick={...}>` and React does the plumbing. Express is React's opposite number on the server: you describe what should happen for each URL, and it handles sockets, header formatting, parsing and routing.

Remember how, before you learned React, somebody probably showed you jQuery or vanilla DOM code and you thought "this is so much typing for so little"? File 1 is that feeling, for servers. Hold onto it. It is the whole reason the next three hours are worth your time.

---

## 1. Introduction to Express: The Problem It Solves

### 1.1 What frameworks exist to do

Every server on earth needs the same boring machinery:

- Listen on a port
- Read the incoming request and figure out what was asked for
- Find the piece of code that answers that particular question
- Turn the answer into correctly formatted HTTP text
- Send it back and close up

Only **one** of those five is about your app. The other four are identical whether you are running a bank, a bookshop or a note-taking app. A framework's whole pitch is: let me do those four, you do the one that matters.

Express makes a specific trade. It is called "unopinionated", and here is what that means in practice.

| | Express | Something like Django or Rails |
|---|---|---|
| Project layout | you decide | prescribed, with a generator |
| Database | pick any, or none | one is assumed |
| What it gives you | routing, middleware, request/response helpers | routing, ORM, admin panel, auth, templates, migrations |
| What you must add | validation, auth, database, almost everything else | less |
| Good when | you want to understand each piece as you add it | you want the whole thing running by Friday |

For learning backend from scratch, unopinionated is the right trade. Nothing is hidden from you. When you add authentication in Class 9, you will add it, and you will know exactly where it sits and why. Frameworks that give you auth for free also give you auth you cannot explain in an interview.

### 1.2 Install it and look at the damage

```bash
npm install express
```

```
$ node -e "console.log(require('./package.json').dependencies)"
{ express: '^5.2.1' }
```

Everything in these notes was built and tested on **Express 5.2.1** with **Node v22.22.0**. If an old tutorial you find online does something slightly differently, it is probably written for Express 4. I will flag the handful of places where that matters.

---

### ✏️ ACTIVITY 1 — Framework Archaeology (8 minutes)

Do not write any code for this one. Open File 1 and File 2 side by side on your screen.

1. Count the characters in each. Roughly is fine.
2. In File 1, point at every single place where a typo would break the server *silently* rather than crashing it. (Hint: I count at least three. A misspelled header name. A forgotten `return`. A `writeHead` after `end`.)
3. In File 2, how many of those three places still exist?
4. Now the real question. Write one sentence in your notes finishing this: *"A framework reduces bugs not by being cleverer than me, but by..."*

<details>
<summary>💡 What I'm fishing for</summary>

"...by removing the places where I am allowed to make the mistake at all."

That is the actual mechanism. Express is not smarter than you. It just gives you fewer opportunities to be wrong, because the error-prone steps are not steps you take any more. `res.json()` cannot forget the Content-Type header, because forgetting it is not an available option.

The three silent-failure spots in File 1:
- `'Content-Type'` spelled wrong: no crash, browser just guesses the type and may render your JSON as plain text
- the `return` on line 7 removed: Node tries to write headers twice and throws `ERR_HTTP_HEADERS_SENT`, which does not reach the browser, so the user sees a hung request
- `res.end()` left out: request hangs forever, no error anywhere

All three are gone in the Express version, not because Express checks for them, but because you no longer write those lines.
</details>

---

## 2. Creating a Server: The Four Lines

Make a real project now. We will grow this one into the lab.

```bash
mkdir notes-api && cd notes-api
npm init -y
npm install express dotenv
npm install --save-dev nodemon
```

Open `package.json` and add `"type": "module"` so your `import` statements work (same as Class 2):

```json
{
  "name": "notes-api",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "node src/server.js",
    "dev": "nodemon src/server.js"
  }
}
```

Now `src/server.js`:

```javascript
import express from 'express';   // 1. get the framework

const app = express();           // 2. make an application
const PORT = 4000;

app.get('/health', (req, res) => {   // 3. describe one route
  res.json({ status: 'ok' });
});

app.listen(PORT, () => {         // 4. open the door
  console.log(`Notes API listening on http://localhost:${PORT}`);
});
```

Run it:

```
$ node src/server.js
Notes API listening on http://localhost:4000
```

And notice something: **your terminal is stuck.** No new prompt. That is correct and it is important. In Class 2 every script you ran finished and gave the prompt back. A server does not finish. `app.listen()` tells Node "keep the event loop alive, something might call". The process will sit there until you press `Ctrl+C`.

So from now on you need **two terminals open**: one with the server running, one for `curl`. Get used to that layout today.

### 2.1 What each of the four lines really is

**`const app = express()`.** `express` is a function, and calling it hands you back a thing that is, under the hood, a request handler. It is the same shape of function Node's `http.createServer()` wants. Express does not replace Node's HTTP server; it produces a very sophisticated handler to feed it. You can see the join if you want to:

```javascript
import http from 'node:http';
import express from 'express';

const app = express();
app.get('/', (req, res) => res.send('hi'));

http.createServer(app).listen(4000);   // works identically to app.listen(4000)
```

`app.listen(4000)` is a convenience that does exactly that createServer call for you. Knowing this stops Express from feeling like magic and it will matter in Class 14 when we add WebSockets, which need the raw server object.

**`PORT`** is a number from 0 to 65535 identifying which door on the machine you are standing behind. Your React dev server usually sits on 3000 or 5173. Pick anything in the 3000–9000 range that is free. If the port is taken you get this, and it is one of the two errors you will hit most this term:

```
Error: listen EADDRINUSE: address already in use :::4000
```

Translation: something is already on 4000. Nearly always it is a server you started earlier and forgot about. Fix it:

```bash
lsof -i :4000          # macOS / Linux: see what's there
kill -9 <the PID>      # end it
```

**The callback inside `listen`** runs once, after the port is successfully open. Put your "ready" log there and nowhere else. If you `console.log` after `app.listen(...)` instead of inside it, you are printing before the port is confirmed, and you will claim to be ready while actually having crashed.

### 2.2 Read the port from the environment

Hardcoding 4000 is fine on your laptop and wrong in production, where the hosting platform picks the port and tells you via an environment variable. You already know how to handle this from Class 2:

```javascript
import 'dotenv/config';
import express from 'express';

const app = express();
const PORT = process.env.PORT ?? 4000;
```

With a `.env` file:

```
PORT=4000
```

And `.env` in `.gitignore`, with a committed `.env.example` beside it. Same discipline as last class. In Class 13, when we deploy this to a real server, the `?? 4000` is what makes it work without changing a line.

---

### ✏️ ACTIVITY 2 — Two Servers, One Port (10 minutes)

Hands-on. You are going to cause `EADDRINUSE` on purpose, because an error you have met once is an error that never scares you again.

1. Start your server: `node src/server.js`. Leave it.
2. Open a **second terminal**, go to the same folder, and run `node src/server.js` again.
3. Read the error out loud. Find the port number in it.
4. In the second terminal, change `PORT` to 4001 via the environment without editing any file:
   ```bash
   PORT=4001 node src/server.js
   ```
5. Confirm both are alive:
   ```bash
   curl localhost:4000/health
   curl localhost:4001/health
   ```
6. Now kill the first one with `Ctrl+C` and immediately `curl localhost:4000/health` again. What does curl say?

<details>
<summary>💡 Expected results</summary>

Step 3 gives you:
```
Error: listen EADDRINUSE: address already in use :::4000
```

Step 4 works, because `PORT=4001` in front of the command sets the environment variable for just that one process, and your `process.env.PORT ?? 4000` picks it up. This is exactly how hosting platforms will hand you a port in Class 13.

Step 5 gives `{"status":"ok"}` twice. Two independent servers, two ports, one machine.

Step 6 gives:
```
curl: (7) Failed to connect to localhost port 4000 after 0 ms: Connection refused
```

"Connection refused" means nothing is listening at all. Learn to tell it apart from a 404, which means something *is* listening and it answered you, it just does not have that path. Beginners conflate these two constantly and then debug the wrong half of the system. Connection refused is a server problem. 404 is a URL problem.
</details>

---

## 3. Routing: Teaching Your Server the Map

A route is a pair: **one HTTP method plus one path**, and the function to run when both match.

```javascript
app.get('/notes', handler);       // GET    /notes
app.post('/notes', handler);      // POST   /notes
app.put('/notes/:id', handler);   // PUT    /notes/42
app.patch('/notes/:id', handler); // PATCH  /notes/42
app.delete('/notes/:id', handler);// DELETE /notes/42
```

Method and path together. `GET /notes` and `POST /notes` are two different routes that happen to share a path, and they will do completely different things. This is the same idea you met in Class 1 when we talked about methods being verbs: the path says *which thing*, the method says *what to do to it*.

The handler always receives the same two arguments in the same order:

```javascript
app.get('/notes', (req, res) => { ... });
//                  ↑    ↑
//                  |    └── the response you are building
//                  └─────── the request that came in
```

`req` and `res` are conventions, not keywords. You could call them `question` and `answer`. Everyone on earth calls them `req` and `res`, so you will too.

### 3.1 Route ordering, and the bug that will bite you

Express checks routes **top to bottom, first match wins, then it stops looking.** That sentence is the single most important thing in this section, so here is what happens when you forget it.

```javascript
import express from 'express';
const app = express();

app.get('/notes/:id', (req, res) => {
  res.json({ matched: '/notes/:id', id: req.params.id });
});

app.get('/notes/archived', (req, res) => {
  res.json({ matched: '/notes/archived' });
});

app.listen(4003);
```

That looks completely reasonable. Here is what it actually does:

```
$ curl http://localhost:4003/notes/archived
{"matched":"/notes/:id","id":"archived"}

$ curl http://localhost:4003/notes/7
{"matched":"/notes/:id","id":"7"}
```

Your `/notes/archived` route is **unreachable**. It will never run, not once, ever. No error, no warning, no log line. Express reached `/notes/:id` first, `:id` happily matched the text `archived`, and Express stopped looking. The second route might as well be deleted.

Swap the two blocks and nothing else:

```javascript
app.get('/notes/archived', (req, res) => {       // specific first
  res.json({ matched: '/notes/archived' });
});

app.get('/notes/:id', (req, res) => {            // general second
  res.json({ matched: '/notes/:id', id: req.params.id });
});
```

```
$ curl http://localhost:4004/notes/archived
{"matched":"/notes/archived"}

$ curl http://localhost:4004/notes/7
{"matched":"/notes/:id","id":"7"}
```

**The rule: specific paths go above general ones.** A path with a literal word in a position goes above a path with `:param` in that position.

If you want a mental hook, think of CSS specificity, or a `switch` statement with no `break`. Order decides the winner. It is the same reason `.btn-primary` has to come after `.btn` in your stylesheets.

---

### ✏️ ACTIVITY 3 — Break It On Purpose (12 minutes)

You get far more out of an error you caused than one you stumbled into. So cause four.

Make a scratch file `order-lab.js`. Start from this, which works:

```javascript
import express from 'express';
const app = express();

app.get('/notes/archived', (req, res) => res.json({ route: 'archived' }));
app.get('/notes/count',    (req, res) => res.json({ route: 'count' }));
app.get('/notes/:id',      (req, res) => res.json({ route: 'by id', id: req.params.id }));

app.listen(4100, () => console.log('order-lab on 4100'));
```

Confirm all three work, then break it four times. Predict out loud before each `curl`:

1. Move the `:id` route to the **top**. Which of the three still work? Which are now dead?
2. Put `:id` back at the bottom. Now add `app.get('/notes/:anything', ...)` **above** `/notes/count` returning `{route: 'greedy'}`. What happens to `/notes/count`? Does the name `:anything` vs `:id` change anything?
3. Change the last route to `app.get('/notes/:id/:extra', ...)`. Does `/notes/7` still match it? Try `/notes/7/8`.
4. Delete all three routes and leave only `app.listen(...)`. `curl -i localhost:4100/notes/7`. What status comes back, and who produced it?

<details>
<summary>💡 Answers</summary>

**1.** Only `/notes/:id` runs. Both `archived` and `count` are dead, matched as ids. You will see `{"route":"by id","id":"archived"}`.

**2.** `/notes/count` is dead; the greedy route catches it. And no, the *name* of the parameter is irrelevant to matching. `:id`, `:anything`, `:banana` all match the same thing. The name only decides the key you read it back from in `req.params`. This catches people out because the name *looks* meaningful.

**3.** `/notes/7` does **not** match `/notes/:id/:extra` and you get a 404. Segment count must match exactly; `:param` matches one segment, never zero and never two. `/notes/7/8` matches and gives you `{id: '7', extra: '8'}`.

**4.** `HTTP/1.1 404 Not Found`, with an HTML body saying `Cannot GET /notes/7`. Express produced it. With no route matching, Express's built-in final handler answers. Later today you will replace that HTML with your own JSON, because an API that returns HTML errors is annoying for whoever is calling it, which in a few weeks is going to be you, from React.
</details>

---

## 4. Getting Data Out of a URL

Two mechanisms, two different jobs. Mixing them up is one of the most common API design mistakes, so get the distinction straight now.

```
http://localhost:4000/api/notes/42?tag=study&limit=5
                      └─────┬────┘ └────────┬───────┘
                       path params        query string
                      which thing          how to shape
                                           the answer
```

### 4.1 Route parameters: `req.params`

Declared with a colon in the path. They name a specific resource.

```javascript
app.get('/notes/:id', (req, res) => {
  console.log(req.params);     // { id: '42' }
});

app.get('/users/:userId/notes/:noteId', (req, res) => {
  console.log(req.params);     // { userId: '7', noteId: '42' }
});
```

Route params are **structural**. They are part of the identity of the thing you are asking for. `/notes/42` without the 42 is not a meaningful request at all.

### 4.2 Query strings: `req.query`

Everything after the `?`, as `key=value` pairs joined by `&`. Express parses them for you.

```javascript
app.get('/notes', (req, res) => {
  console.log(req.query);     // { tag: 'study', limit: '5' }
});
```

Query params are **modifiers**. They filter, sort, paginate or search. The request still makes sense without them, you just get everything instead of a slice. You have written these a hundred times in React without thinking about it, any time you did `fetch('/search?q=' + term)`.

### 4.3 The deciding question

> Does removing this piece change *which thing* I am asking for, or just *how much* of it I get back?

Changes which thing → route param. Changes how much or in what order → query string.

| Request | Why |
|---|---|
| `GET /notes/42` | one specific note, identity, param |
| `GET /notes?tag=study` | all notes, narrowed, query |
| `GET /users/7/notes` | notes belonging to a specific user, identity, param |
| `GET /notes?sort=newest&limit=10` | shaping the list, query |
| `GET /notes/42?fields=title` | both: identity in the path, shaping in the query |

### 4.4 The thing that will cost you an hour if nobody warns you

**Everything out of `req.params` and `req.query` is a string. Always. No exceptions.**

I built a route that dumps the whole request so you can see it for yourself:

```javascript
app.all('/inspect/:category/:id', (req, res) => {
  res.json({
    method: req.method,
    path:   req.path,
    params: req.params,
    query:  req.query,
    body:   req.body,
    ct:     req.get('content-type') ?? null,
    ua:     req.get('user-agent'),
    ip:     req.ip,
  });
});
```

Real output:

```
$ curl "http://localhost:4005/inspect/work/42?tag=urgent&limit=5&done=true"
{
    "method": "GET",
    "path": "/inspect/work/42",
    "params": {
        "category": "work",
        "id": "42"
    },
    "query": {
        "tag": "urgent",
        "limit": "5",
        "done": "true"
    },
    "ct": null,
    "ua": "curl/8.5.0",
    "ip": "127.0.0.1"
}
```

Look hard at `"42"`, `"5"` and `"true"`. Every one has quotes around it. `limit` is the string `"5"`, not the number 5. `done` is the string `"true"`, which in JavaScript is **truthy even when it says "false"**, because any non-empty string is truthy.

```javascript
Boolean("false")     // true  😱
"5" + 1              // "51"
Number("5") + 1      // 6
```

So convert at the boundary, the moment the value arrives:

```javascript
const id = Number(req.params.id);
if (Number.isNaN(id)) {
  return res.status(400).json({ error: 'id must be a number' });
}

const limit = Number(req.query.limit) || 10;     // || catches NaN and 0
const done  = req.query.done === 'true';         // compare to the string
```

One more oddity worth knowing before it surprises you. A repeated query key does not overwrite; it becomes an array:

```
$ curl "http://localhost:4005/inspect/a/b?tag=work&tag=urgent"
{'tag': ['work', 'urgent']}
```

So `req.query.tag` is sometimes a string and sometimes an array, depending on what the caller sent. If you call `.toLowerCase()` on it you will crash on the array case. Defend with `[].concat(req.query.tag ?? [])` when it matters.

---

### ✏️ ACTIVITY 4 — Param or Query? (10 minutes)

**Part A, on paper, 5 minutes.** Design the URL for each of these. Decide param vs query and write the full path:

1. Show me the note with id 17
2. Show me every note tagged "work"
3. Show me the 10 most recent notes
4. Show me every note belonging to user 3
5. Show me user 3's notes tagged "work", newest first
6. Show me note 17, but only its title
7. Search all notes for the word "deployment"

**Part B, at the keyboard, 5 minutes.** Write one route that proves you understand both:

```javascript
app.get('/report/:year/:month', (req, res) => {
  // TODO: return year and month as NUMBERS, not strings
  // TODO: read ?format= from the query, default to 'summary'
  // TODO: 400 if month is not between 1 and 12
});
```

Test it with `/report/2026/3`, `/report/2026/13`, `/report/2026/3?format=full`, and `/report/abc/3`.

<details>
<summary>💡 Answers</summary>

**Part A:**
1. `/notes/17`: identity
2. `/notes?tag=work`: filter
3. `/notes?sort=recent&limit=10`: shaping
4. `/users/3/notes`: identity of the owner, nested
5. `/users/3/notes?tag=work&sort=newest`: identity in path, filters in query
6. `/notes/17?fields=title`: identity in path, shaping in query
7. `/notes?q=deployment`: filter

Number 4 is the one people get wrong, usually writing `/notes?userId=3`. Both technically work, but `/users/3/notes` reads as what it is: a collection that belongs to something. Nesting the path communicates ownership. You will meet this again in Class 10 when a note belongs to a logged-in user and the server stops trusting the client to say who it is.

**Part B:**

```javascript
app.get('/report/:year/:month', (req, res) => {
  const year  = Number(req.params.year);
  const month = Number(req.params.month);
  const format = req.query.format ?? 'summary';

  if (Number.isNaN(year) || Number.isNaN(month)) {
    return res.status(400).json({ error: 'year and month must be numbers' });
  }
  if (month < 1 || month > 12) {
    return res.status(400).json({ error: 'month must be between 1 and 12' });
  }

  res.json({ year, month, format, types: { year: typeof year, month: typeof month } });
});
```

`/report/abc/3` returns 400 because `Number('abc')` is `NaN`. Note that you must use `Number.isNaN()` and not `=== NaN`, because `NaN === NaN` is `false` in JavaScript. That is not a joke; it is in the spec.
</details>

---

## 5. The Request and Response Objects

### 5.1 `req`, the parts you will actually use

| Property | What it holds | Example |
|---|---|---|
| `req.method` | the HTTP verb | `'POST'` |
| `req.path` | path without the query string | `'/api/notes/42'` |
| `req.originalUrl` | full path including query | `'/api/notes/42?tag=x'` |
| `req.params` | route parameters, all strings | `{ id: '42' }` |
| `req.query` | parsed query string, all strings | `{ tag: 'study' }` |
| `req.body` | parsed request body | `{ title: 'Buy milk' }` |
| `req.get('name')` | one header, case-insensitive | `req.get('content-type')` |
| `req.headers` | every header, as an object | |
| `req.ip` | caller's IP address | `'127.0.0.1'` |

`req` is read-only in spirit. You are inspecting what arrived. (You *can* attach your own properties to it, and in Class 9 we will, when auth middleware sets `req.user`. That is a deliberate pattern, not casual mutation.)

### 5.2 `req.body` and the bug every beginner hits once

`req.body` is **not** there by default. Express does not parse request bodies unless you tell it to. Watch what happens without the parser:

```javascript
app.post('/echo', (req, res) => {
  console.log('typeof req.body =', typeof req.body, '| value =', req.body);
  res.json({ received: req.body ?? null });
});
```

```
$ curl -X POST localhost:4006/echo -H "Content-Type: application/json" -d '{"title":"Buy milk"}'
typeof req.body = object | value = { title: 'Buy milk' }
{"received":{"title":"Buy milk"}}
```

That worked because I had this line above it:

```javascript
app.use(express.json());
```

Delete that one line and the same request gives you `undefined`, and your handler crashes on `req.body.title` with `TypeError: Cannot read properties of undefined`. This is **the** most common Express beginner bug, and the error message points at your handler rather than at the missing line, which is why it eats an hour.

`express.json()` is middleware. It sits in front of all your routes, reads the raw bytes off the socket, and if the `Content-Type` says JSON, parses them and puts the result on `req.body`. We cover middleware properly in Class 4. Today just know: **`app.use(express.json())` goes near the top, before your routes, or bodies do not exist.**

Two related traps, both real output:

```
$ curl -X POST localhost:4006/echo -d '{"title":"Buy milk"}'
typeof req.body = undefined | value = undefined
{"received":null}
```

No `Content-Type` header, so `express.json()` declined to parse it. The parser only touches bodies whose content type it recognises. When your fetch from React sends a body and the server sees nothing, this is nearly always why. (In Express 4 you would get `{}` here instead of `undefined`, which is why older tutorials check `Object.keys(req.body).length`. On Express 5, use `req.body ?? {}`.)

```
$ curl -i -X POST localhost:4006/echo -H "Content-Type: application/json" -d '{"title": }'
HTTP/1.1 400 Bad Request
```

Malformed JSON, and Express returned 400 without you writing a single line of validation. The parser threw, and Express's default error handler turned the throw into a 400. Free correctness.

### 5.3 `res`, the parts you will actually use

| Method | What it does |
|---|---|
| `res.json(obj)` | sends JSON, sets Content-Type, stringifies. Your default. |
| `res.status(code)` | sets the status, returns `res` so you can chain |
| `res.send(x)` | sends whatever: object becomes JSON, string becomes HTML |
| `res.end()` | ends with no body. For 204. |
| `res.sendStatus(code)` | sets status and sends the status name as plain text |
| `res.set('name', 'v')` | sets a response header |
| `res.redirect(url)` | 302 plus a Location header |

Here is each one, run for real:

```javascript
app.get('/a', (req, res) => res.send({ a: 1 }));
app.get('/b', (req, res) => res.send('<h1>hello</h1>'));
app.get('/c', (req, res) => res.json({ c: 3 }));
app.get('/d', (req, res) => res.status(204).end());
app.get('/e', (req, res) => res.sendStatus(403));
```

```
--- /a ---            --- /b ---                  --- /c ---
HTTP/1.1 200 OK       HTTP/1.1 200 OK             HTTP/1.1 200 OK
application/json      text/html; charset=utf-8    application/json
{"a":1}               <h1>hello</h1>              {"c":3}

--- /d ---            --- /e ---
HTTP/1.1 204          HTTP/1.1 403 Forbidden
(no body)             text/plain
                      Forbidden
```

`res.send()` guesses the type from what you hand it. For an API, always use `res.json()`: it says what you mean and it cannot guess wrong.

### 5.4 One response per request. That is the law.

An HTTP request gets exactly one response. Try to send two and Node throws. This is the second classic bug, and it has a specific shape: **a missing `return`.**

```javascript
app.post('/bug-a', (req, res) => {
  if (!req.body?.title) {
    res.status(400).json({ error: 'title required' });     // no return!
  }
  res.status(201).json({ created: req.body?.title });      // runs anyway
});
```

The request with no title goes through the `if`, sends a 400, and then **keeps going** into the 201 line. Here is exactly what happens, from the real run:

The caller sees this and nothing seems wrong:
```
HTTP/1.1 400 Bad Request
{"error":"title required"}
```

But the server terminal has this in it:
```
Error [ERR_HTTP_HEADERS_SENT]: Cannot set headers after they are sent to the client
    at ServerResponse.setHeader (node:_http_outgoing:700:11)
    at ServerResponse.header (.../express/lib/response.js:686:10)
    at ServerResponse.send (.../express/lib/response.js:163:12)
    at ServerResponse.json (.../express/lib/response.js:252:15)
```

The client got a correct-looking answer. Your logs are filling with errors. In a worse version of this bug the first response is the wrong one and the correct response is the one that gets thrown away, and you spend an afternoon confused.

The fix is one word, every time:

```javascript
if (!req.body?.title) {
  return res.status(400).json({ error: 'title required' });
}
```

`return` here is not about returning a value. Nobody reads what a route handler returns. It exists purely to stop the function. Get in the habit of writing `return res.` for every early exit and the bug cannot happen.

### 5.5 The opposite bug: sending nothing

```javascript
app.get('/bug-c', (req, res) => {
  res.status(200);        // sets the status... and then nothing
});
```

```
$ curl -m 2 localhost:4007/bug-c
curl exit code: 28
```

Exit code 28 is curl's timeout. No error on the server, no log line, nothing in the terminal at all. The request just hangs until something gives up. `res.status()` only *sets* the number; it never sends. Something must actually send: `.json()`, `.send()`, or `.end()`.

**Every path through a handler must end in exactly one send.** Not zero, not two. When you are debugging a hung request, go read your handler and find the branch with no send on it.

---

### ✏️ ACTIVITY 5 — The Request X-Ray (12 minutes)

Build the inspector yourself, then use it to answer questions you cannot answer from memory.

```javascript
import express from 'express';
const app = express();
app.use(express.json());

app.all('/x/:a/:b', (req, res) => {
  res.json({
    method: req.method,
    path: req.path,
    originalUrl: req.originalUrl,
    params: req.params,
    query: req.query,
    body: req.body ?? null,
    contentType: req.get('content-type') ?? null,
  });
});

app.listen(4200, () => console.log('x-ray on 4200'));
```

`app.all` matches every method, which is handy for a debug route and something you would never ship.

Now investigate. Run each, read the output, and write down the answer:

1. `curl "localhost:4200/x/one/two?z=9"`: what is in `body`?
2. `curl -X DELETE "localhost:4200/x/one/two"`: does `method` change? Do the params survive?
3. `curl -X POST localhost:4200/x/one/two -H "Content-Type: application/json" -d '{"n": 5}'`: is `n` the number 5 or the string "5"? Why is this different from query params?
4. `curl "localhost:4200/x/one/two?z=9&z=10"`: what type is `z` now?
5. `curl "localhost:4200/x/hello%20world/two"`: what does `params.a` contain? Did Express decode it?
6. `curl "localhost:4200/x/one/two?empty="`: is `empty` present in query? What is its value?

<details>
<summary>💡 Answers</summary>

1. `null`. GET requests normally carry no body, so `express.json()` has nothing to parse and `req.body` is `undefined`, which my `?? null` turns into null.

2. `method` is `"DELETE"`, params are unchanged. The method and the path are completely independent. This is why `app.all` can exist at all.

3. **The number 5.** This is the big one. Query and path params are always strings because a URL is text with no type information. A JSON body has real types, because JSON has numbers, booleans, null and arrays. So `req.body.n` is a number and `req.query.n` would be a string. This is a genuine argument for putting structured data in the body rather than the query string.

4. An array: `["9","10"]`. Still strings, now in a box.

5. `"hello world"`, with a real space. Express URL-decodes params for you. `%20` is how a space travels in a URL. You get the decoded value, which is almost always what you want.

6. Present, with the value `""`. Empty string. Which is falsy. So `if (req.query.empty)` is false even though the key exists. If you need to distinguish "not sent" from "sent empty", check `'empty' in req.query`.
</details>

---

## 6. Status Codes: Say What Happened

You met these in Class 1. Now you are the one choosing them. Getting them right is not pedantry: your React code, and every other client, decides what to do next based on this number alone.

```javascript
res.status(201).json(newNote);     // chain: status, then send
res.status(404).json({ error: 'No note with id 99' });
res.status(204).end();             // success, deliberately no body
```

### 6.1 The ones you need today

| Code | Name | Use it when |
|---|---|---|
| **200** | OK | GET succeeded, or PUT/PATCH succeeded and you are returning the updated thing |
| **201** | Created | POST made something new. Return the new thing, including its id. |
| **204** | No Content | DELETE succeeded. There is deliberately nothing to send. |
| **400** | Bad Request | the caller sent rubbish: missing field, wrong type, unparseable |
| **401** | Unauthorized | you are not logged in (Class 9) |
| **403** | Forbidden | you are logged in, but this is not yours (Class 9) |
| **404** | Not Found | that path or that id does not exist |
| **409** | Conflict | it clashes with what exists, like a duplicate email |
| **500** | Internal Server Error | **your** code broke, not the caller's fault |

### 6.2 The two rules that matter most

**Rule one: 4xx is the caller's fault, 5xx is yours.** Returning 500 for a missing title is a lie that sends whoever is debugging into your server logs when the problem was in their request. Returning 400 for a crash in your own code is a lie in the other direction.

**Rule two: 200 with an error message inside is the worst possible answer.**

```javascript
// Please never
res.json({ success: false, error: 'Note not found' });
```

```javascript
// Yes
res.status(404).json({ error: 'No note with id 99' });
```

Why this matters to you specifically, as someone who writes React: `fetch` does **not** throw on 404 or 500. `response.ok` is how you find out, and it is driven by the status code.

```javascript
const response = await fetch('/api/notes/999');

if (!response.ok) {                       // reads the STATUS
  console.log(response.status);           // 404
  return setError('Note not found');
}
const note = await response.json();
```

If your server answers 200 with `{success: false}`, `response.ok` is `true`, your error branch never fires, and your component tries to render `note.title` on an object with no title. You will debug the React side for an hour before the thought "maybe the server is lying about the status code" arrives. Do not build that trap for your future self.

### 6.3 201 and 204, the two that get forgotten

**201 on create**, and send back the created object. The caller needs the id you just assigned; it has no other way to learn it.

```javascript
const note = addNote({ title, body, tag, done: false });
res.status(201).json(note);       // note.id is in there
```

**204 on delete**, with no body at all.

```javascript
res.status(204).end();
```

A 204 response with a body is not a valid HTTP response. `res.json()` tries to send one, so use `.end()`.

---

### ✏️ ACTIVITY 6 — Status Code Courtroom (10 minutes)

Work in pairs if you can, or argue with yourself. For each scenario, name the status code and justify it in one sentence. Two of these are genuinely contested; find them.

1. `GET /notes/42` and note 42 exists
2. `GET /notes/42` and note 42 was deleted yesterday
3. `POST /notes` with `{"body": "no title here"}`
4. `POST /notes` with a valid note
5. `DELETE /notes/42` and it works
6. `DELETE /notes/42` and 42 was already deleted five seconds ago
7. `PATCH /notes/42` with `{}`
8. `POST /notes` with a title, but your database connection is down
9. `GET /notes/abc` where ids are always numbers
10. `POST /users` with an email that is already registered

<details>
<summary>💡 Answers, including the two arguments</summary>

1. **200**
2. **404.** It is gone; as far as the API is concerned it does not exist. (410 Gone exists for "deleted on purpose, do not ask again" but almost nobody uses it.)
3. **400.** Caller's fault, missing required field.
4. **201**, with the new note and its id in the body.
5. **204.**
6. **Argument one.** Two defensible answers. **404** says "there is nothing here to delete", which is honest and is what we build today. **204** says "the thing is not there, which is what you asked for", treating DELETE as idempotent: call it ten times, the end state is identical, so why should call two be an error? Both ship in real APIs. What matters is picking one and documenting it. I have us return 404 because it is more informative while you are learning.
7. **400.** Nothing to change. Some APIs return 200 and call it a no-op; 400 is kinder because an empty PATCH is nearly always a bug in the caller.
8. **500 or 503.** Your fault, not theirs. Never 400. 503 Service Unavailable is more precise if it is a known temporary outage.
9. **Argument two.** **400** says the id is the wrong shape, which is a malformed request. **404** says there is no note at that address. I use 400 today, because it tells the caller something actionable: the problem is your id format, not our data. Both are common.
10. **409 Conflict.** Not 400, because the request was well-formed. The conflict is with existing state. You will write this exact handler in Class 9.
</details>

---

## 7. Nodemon: Stop Restarting Your Server

Right now your loop is: edit file, `Ctrl+C`, up-arrow, Enter, test. A hundred times an hour. And when you forget the restart, you test old code and draw the wrong conclusion, which is worse than the tedium.

Your React dev server already solved this; you change a component and the page updates. Nodemon is that, for Node.

```bash
npm install --save-dev nodemon
```

`--save-dev` because this is a development tool. It belongs in `devDependencies`, never `dependencies`. Production runs `node`, not nodemon.

Add a script:

```json
"scripts": {
  "start": "node src/server.js",
  "dev": "nodemon src/server.js"
}
```

Then `npm run dev`. Real output, including a restart I triggered by saving a file:

```
[nodemon] 3.1.14
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: js,mjs,cjs,json
[nodemon] starting `node src/server.js`
Notes API listening on http://localhost:4000
[nodemon] restarting due to changes...
[nodemon] starting `node src/server.js`
Notes API listening on http://localhost:4000
```

I saved `src/routes/notes.js` and the server came back on its own. That is the whole feature.

Three things worth knowing:

**It restarts, it does not hot-reload.** React swaps a component while keeping your app state. Nodemon kills the process and starts a new one. Everything in memory is gone. Today our notes live in a plain array, so **every save wipes your notes back to the three starting ones.** That is not a bug and it is going to confuse you at least once this afternoon. It stops being an issue in Class 5 when the data lives in a database instead.

**`rs` plus Enter** forces a restart by hand, useful when a change is in a file type nodemon is not watching.

**Two scripts, two purposes.** `npm start` for running it properly, `npm run dev` for working on it. Keep both. When we deploy in Class 13, the platform runs `npm start`, and if nodemon is in there you will have a bad afternoon.

---

### ✏️ ACTIVITY 7 — Prove Nodemon Is Watching (6 minutes)

1. `npm run dev`.
2. In a second terminal: `curl localhost:4000/health`.
3. Without stopping anything, edit `/health` to also return `{ version: '2' }`. Save.
4. Watch terminal one. Read the restart lines.
5. `curl localhost:4000/health` again. Is version 2 there?
6. Now break your file on purpose. Delete a closing brace. Save. What does nodemon print, and is the server still answering?
7. Fix the brace, save, and confirm it recovers on its own.

<details>
<summary>💡 What step 6 looks like</summary>

You get a `SyntaxError` dumped in the terminal, and then:

```
[nodemon] app crashed - waiting for file changes before starting...
```

This is nodemon behaving well. It will not thrash in a restart loop. It sits and waits for you to save again, then tries once more. Meanwhile `curl` gets `Connection refused`, because there is genuinely no process listening.

The habit to build: when `curl` says connection refused, **look at the server terminal first.** The answer is nearly always already printed there, and it is nearly always a syntax error. Beginners instead start changing their curl command, which cannot help.
</details>

---

## 8. Organising Routes with `express.Router()`

By the time your API has twenty endpoints, a single `server.js` is 600 lines and finding anything means scrolling. Router fixes this, and the idea will be familiar: it is a mini-app you can mount at a path. If you have ever split a React page into `<Header />`, `<NoteList />` and `<Sidebar />`, you already have the instinct.

### 8.1 How it works

```javascript
// src/routes/notes.js
import express from 'express';
const router = express.Router();

router.get('/', (req, res) => { /* list */ });
router.post('/', (req, res) => { /* create */ });
router.get('/:id', (req, res) => { /* read one */ });

export default router;
```

```javascript
// src/server.js
import notesRouter from './routes/notes.js';

app.use('/api/notes', notesRouter);
```

The mount path and the router's own paths **join together**:

| In the router | Mounted at | Real URL |
|---|---|---|
| `router.get('/')` | `/api/notes` | `GET /api/notes` |
| `router.post('/')` | `/api/notes` | `POST /api/notes` |
| `router.get('/:id')` | `/api/notes` | `GET /api/notes/42` |

Three things you get from this:

**The prefix lives in one place.** Moving the whole API from `/api/notes` to `/api/v2/notes` is one character change in `server.js`. Nothing inside the router knows or cares where it is mounted.

**Files stay small and findable.** `src/routes/notes.js` holds note routes. When users arrive, `src/routes/users.js` holds those, and `server.js` stays the short file that wires things together.

**Middleware can be scoped.** In Class 9, `app.use('/api/notes', requireAuth, notesRouter)` protects every note route with one line, and leaves everything else public.

### 8.2 The mistake you will make once

Writing the prefix **twice**, once in the mount and once in the route:

```javascript
// WRONG
router.get('/api/notes', (req, res) => res.json({ hit: 'double prefix' }));
router.get('/',          (req, res) => res.json({ hit: 'correct' }));

app.use('/api/notes', router);
```

Real output:

```
$ curl localhost:4008/api/notes
{"hit":"correct, this is /api/notes"}

$ curl localhost:4008/api/notes/api/notes
{"hit":"double prefix"}
```

You built `/api/notes/api/notes` and did not mean to. There is no error, just a route that nothing will ever call. **Inside a router, paths are relative to the mount point.** Say the prefix once, in `app.use`.

### 8.3 Grouping by path with `router.route()`

When several methods share a path, you can chain them:

```javascript
router.route('/:id')
  .get(readOne)
  .put(replace)
  .patch(update)
  .delete(destroy);
```

Identical in behaviour to four separate `router.get`/`router.put`/... calls. Some teams find it tidier because the path appears once. Use whichever reads better to you; I will write them out separately in the lab so each endpoint is easy to point at while we build.

---

### ✏️ ACTIVITY 8 — Refactor Without Breaking Anything (12 minutes)

A refactor you cannot verify is a refactor you cannot trust. So build the verification first.

**Step 1.** In `server.js`, add three routes directly, no router:

```javascript
app.get('/api/books',     (req, res) => res.json({ action: 'list books' }));
app.get('/api/books/:id', (req, res) => res.json({ action: 'one book', id: req.params.id }));
app.post('/api/books',    (req, res) => res.status(201).json({ action: 'created' }));
```

**Step 2.** Record what they do. Save this as `verify.sh` and run it:

```bash
#!/bin/bash
echo "--- list ---";   curl -s localhost:4000/api/books
echo; echo "--- one ---";  curl -s localhost:4000/api/books/7
echo; echo "--- create ---"; curl -s -X POST localhost:4000/api/books
echo
```

**Step 3.** Move all three into `src/routes/books.js` as a Router, mount it at `/api/books`, delete the originals from `server.js`.

**Step 4.** Run `verify.sh` again. **Byte-for-byte identical output, or you broke something.**

**Step 5.** Change the mount to `/api/v2/books`. Update `verify.sh`. Confirm you only had to touch `server.js`, not `books.js`.

<details>
<summary>💡 The finished files</summary>

`src/routes/books.js`:
```javascript
import express from 'express';
const router = express.Router();

router.get('/',    (req, res) => res.json({ action: 'list books' }));
router.get('/:id', (req, res) => res.json({ action: 'one book', id: req.params.id }));
router.post('/',   (req, res) => res.status(201).json({ action: 'created' }));

export default router;
```

`server.js`:
```javascript
import booksRouter from './routes/books.js';
app.use('/api/books', booksRouter);
```

Notice what vanished: the string `/api/books` appears **once** in the whole project now, in the mount. That is the payoff, and step 5 proves it.

If step 4 gave you a 404, you almost certainly left `/api/books` inside the router paths. Check for `router.get('/api/books')` and cut it back to `router.get('/')`.
</details>

---

## 🏆 THE MAIN LAB — A Six-Endpoint Notes API

This is the real thing. Every line below was written, run and tested; the outputs in this section are copied from a live terminal, not typed from memory.

### What you are building

| # | Method | Path | Does | Success |
|---|---|---|---|---|
| 1 | GET | `/api/notes` | list, with optional filters | 200 |
| 2 | POST | `/api/notes` | create one | 201 |
| 3 | GET | `/api/notes/:id` | read one | 200 |
| 4 | PUT | `/api/notes/:id` | replace the whole note | 200 |
| 5 | PATCH | `/api/notes/:id` | change some fields | 200 |
| 6 | DELETE | `/api/notes/:id` | remove it | 204 |

### Project layout

```
notes-api/
├── package.json
├── .env
├── .env.example
├── .gitignore
└── src/
    ├── server.js            app setup, middleware, mount, listen
    ├── routes/
    │   └── notes.js         all six endpoints
    └── data/
        └── store.js         our "database" for today
```

Three files. Each has one job. This shape will carry you a long way.

### Step 1 — Setup

```bash
mkdir notes-api && cd notes-api
npm init -y
npm install express dotenv
npm install --save-dev nodemon
mkdir -p src/routes src/data
```

`package.json`:

```json
{
  "name": "notes-api",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "node src/server.js",
    "dev": "nodemon src/server.js"
  }
}
```

`.env`:
```
PORT=4000
```

`.env.example` (same content, this one gets committed):
```
PORT=4000
```

`.gitignore`:
```
node_modules
.env
```

### Step 2 — `src/data/store.js`

```javascript
// Our "database" for today: a plain array that lives in memory.
// Restart the server and everything here disappears. That is fine for now;
// Class 5 swaps this file for a real database and nothing else has to change.

let notes = [
  { id: 1, title: 'Learn Express',  body: 'Routes, params, middleware', tag: 'study', done: false },
  { id: 2, title: 'Call Mum',       body: 'Sunday evening',             tag: 'home',  done: false },
  { id: 3, title: 'Push to GitHub', body: 'The notes-api repo',         tag: 'study', done: true  },
];

let nextId = 4;

export function listNotes()  { return notes; }
export function findNote(id) { return notes.find(n => n.id === id); }

export function addNote(data) {
  const note = { id: nextId++, ...data };
  notes.push(note);
  return note;
}

export function replaceNote(id, data) {
  const index = notes.findIndex(n => n.id === id);
  if (index === -1) return undefined;
  notes[index] = { id, ...data };
  return notes[index];
}

export function patchNote(id, changes) {
  const note = findNote(id);
  if (!note) return undefined;
  Object.assign(note, changes);
  return note;
}

export function removeNote(id) {
  const index = notes.findIndex(n => n.id === id);
  if (index === -1) return false;
  notes.splice(index, 1);
  return true;
}
```

Why a separate file at all, when it is only an array? Because your route handlers should talk about **notes**, not about arrays. When Class 5 replaces the array with PostgreSQL, every function here changes and `routes/notes.js` does not change at all. That boundary is worth drawing even for an array of three objects, and drawing it now is what makes Class 5 easy.

Also notice each function returns something that tells the caller whether it worked: the object, or `undefined`, or `false`. That is what lets the route decide between 200 and 404 without the store knowing anything about HTTP.

### Step 3 — `src/routes/notes.js`

```javascript
import express from 'express';
import {
  listNotes, findNote, addNote,
  replaceNote, patchNote, removeNote,
} from '../data/store.js';

const router = express.Router();

// Every path below is relative to wherever this router gets mounted.
// server.js mounts it at /api/notes, so '/' here means '/api/notes'.

// 1. LIST  GET /api/notes?tag=study&done=false&q=express
router.get('/', (req, res) => {
  const { tag, done, q } = req.query;
  let results = listNotes();

  if (tag)  results = results.filter(n => n.tag === tag);
  if (done) results = results.filter(n => String(n.done) === done);
  if (q) {
    const needle = q.toLowerCase();
    results = results.filter(n =>
      n.title.toLowerCase().includes(needle) ||
      n.body.toLowerCase().includes(needle)
    );
  }

  res.json({ count: results.length, notes: results });
});

// 2. CREATE  POST /api/notes
router.post('/', (req, res) => {
  const { title, body = '', tag = 'general' } = req.body ?? {};

  if (!title || typeof title !== 'string') {
    return res.status(400).json({ error: 'title is required and must be a string' });
  }

  const note = addNote({ title, body, tag, done: false });
  res.status(201).json(note);
});

// 3. READ ONE  GET /api/notes/:id
router.get('/:id', (req, res) => {
  const id = Number(req.params.id);

  if (Number.isNaN(id)) {
    return res.status(400).json({ error: 'id must be a number' });
  }

  const note = findNote(id);
  if (!note) {
    return res.status(404).json({ error: `No note with id ${id}` });
  }

  res.json(note);
});

// 4. REPLACE  PUT /api/notes/:id
router.put('/:id', (req, res) => {
  const id = Number(req.params.id);
  const { title, body = '', tag = 'general', done = false } = req.body ?? {};

  if (!title) {
    return res.status(400).json({ error: 'PUT replaces the whole note, so title is required' });
  }

  const updated = replaceNote(id, { title, body, tag, done });
  if (!updated) {
    return res.status(404).json({ error: `No note with id ${id}` });
  }

  res.json(updated);
});

// 5. UPDATE PART  PATCH /api/notes/:id
router.patch('/:id', (req, res) => {
  const id = Number(req.params.id);
  const changes = req.body ?? {};

  delete changes.id;  // nobody gets to rewrite the id

  if (Object.keys(changes).length === 0) {
    return res.status(400).json({ error: 'Send at least one field to change' });
  }

  const updated = patchNote(id, changes);
  if (!updated) {
    return res.status(404).json({ error: `No note with id ${id}` });
  }

  res.json(updated);
});

// 6. DELETE  DELETE /api/notes/:id
router.delete('/:id', (req, res) => {
  const id = Number(req.params.id);
  const existed = removeNote(id);

  if (!existed) {
    return res.status(404).json({ error: `No note with id ${id}` });
  }

  res.status(204).end();
});

export default router;
```

Five things in there are worth pausing on.

**`req.body ?? {}` everywhere.** Because `req.body` is `undefined` when nothing was parsed, and destructuring `undefined` throws. The `?? {}` makes the destructure safe and your error message useful instead of a stack trace.

**`return` on every early exit.** Six of them. Section 5.4 is why.

**`String(n.done) === done`.** `n.done` is a real boolean; `done` from the query is the string `"true"` or `"false"`. Convert one side so you are comparing like with like. The alternative, `n.done === (done === 'true')`, works too and reads worse.

**PUT demands a title, PATCH does not.** That is the whole semantic difference. PUT means "here is the complete new version, replace what you have", so a missing field means the field is now empty, and a missing title means an invalid note. PATCH means "change just these", so fields you leave out stay as they were.

**`delete changes.id` in PATCH.** Without it, `PATCH /api/notes/1` with `{"id": 999}` lets a caller rewrite the primary key and corrupt your store. Never let the body set the id; the id comes from the URL. This is your first taste of a real security habit, and it is the start of a theme that runs through Class 9.

### Step 4 — `src/server.js`

```javascript
import 'dotenv/config';
import express from 'express';
import notesRouter from './routes/notes.js';

const app = express();
const PORT = process.env.PORT ?? 4000;

app.use(express.json());

// A tiny logger so you can watch requests arrive.
app.use((req, res, next) => {
  console.log(`${req.method} ${req.originalUrl}`);
  next();
});

app.get('/health', (req, res) => {
  res.json({ status: 'ok', uptime: Math.round(process.uptime()) });
});

app.use('/api/notes', notesRouter);

// Nothing above matched, so it does not exist.
app.use((req, res) => {
  res.status(404).json({ error: `Cannot ${req.method} ${req.originalUrl}` });
});

app.listen(PORT, () => {
  console.log(`Notes API listening on http://localhost:${PORT}`);
});
```

Order is load-bearing in this file, top to bottom:

1. `express.json()` first, so every route below it has `req.body`
2. the logger second, so it logs everything
3. `/health` and the notes router, the actual routes
4. the catch-all 404 **last**, because it matches everything. Move it above the router and your entire API returns 404.

That logger is your first hand-written middleware. Three arguments instead of two: `req`, `res`, `next`. Calling `next()` means "I am done, pass it along". Forget `next()` and every request hangs forever. That is Class 4's whole topic, and you have now written one.

### Step 5 — Run and test all six

```bash
npm run dev
```

**1. List everything**

```
$ curl -s localhost:4000/api/notes
{
    "count": 3,
    "notes": [
        { "id": 1, "title": "Learn Express",  "body": "Routes, params, middleware", "tag": "study", "done": false },
        { "id": 2, "title": "Call Mum",       "body": "Sunday evening",             "tag": "home",  "done": false },
        { "id": 3, "title": "Push to GitHub", "body": "The notes-api repo",         "tag": "study", "done": true  }
    ]
}
```

**1b. List, filtered by tag**

```
$ curl -s "localhost:4000/api/notes?tag=study"
count: 2
 - Learn Express
 - Push to GitHub
```

**1c. List, searched**

```
$ curl -s "localhost:4000/api/notes?q=github"
{
    "count": 1,
    "notes": [
        { "id": 3, "title": "Push to GitHub", "body": "The notes-api repo", "tag": "study", "done": true }
    ]
}
```

**2. Create**

```
$ curl -s -i -X POST localhost:4000/api/notes \
    -H "Content-Type: application/json" \
    -d '{"title":"Buy groundnut oil","body":"From Mile 3 market","tag":"home"}'

HTTP/1.1 201 Created
{"id":4,"title":"Buy groundnut oil","body":"From Mile 3 market","tag":"home","done":false}
```

201, and `id: 4` came back. The caller now knows the id without having to guess or re-fetch.

**2b. Create with no title**

```
HTTP/1.1 400 Bad Request
{"error":"title is required and must be a string"}
```

**3. Read one**

```
$ curl -s localhost:4000/api/notes/2
{
    "id": 2,
    "title": "Call Mum",
    "body": "Sunday evening",
    "tag": "home",
    "done": false
}
```

**3b. Read a note that is not there**

```
HTTP/1.1 404 Not Found
{"error":"No note with id 99"}
```

**3c. Read with a nonsense id**

```
$ curl -s -i localhost:4000/api/notes/banana
HTTP/1.1 400 Bad Request
{"error":"id must be a number"}
```

**4. Replace with PUT**

```
$ curl -s -X PUT localhost:4000/api/notes/2 \
    -H "Content-Type: application/json" \
    -d '{"title":"Call Mum on Sunday","body":"After church","tag":"family","done":true}'
{
    "id": 2,
    "title": "Call Mum on Sunday",
    "body": "After church",
    "tag": "family",
    "done": true
}
```

**4b. PUT with no title**

```
HTTP/1.1 400 Bad Request
{"error":"PUT replaces the whole note, so title is required"}
```

**5. PATCH just one field**

```
$ curl -s -X PATCH localhost:4000/api/notes/1 \
    -H "Content-Type: application/json" -d '{"done":true}'
{
    "id": 1,
    "title": "Learn Express",
    "body": "Routes, params, middleware",
    "tag": "study",
    "done": true
}
```

I sent only `done`. Title, body and tag survived untouched. Compare that with the PUT above, where I had to send all four fields or lose them. That is PUT versus PATCH in two commands.

**5b. PATCH with an empty body**

```
HTTP/1.1 400 Bad Request
{"error":"Send at least one field to change"}
```

**6. Delete**

```
$ curl -s -i -X DELETE localhost:4000/api/notes/3
HTTP/1.1 204 No Content
X-Powered-By: Express
```

204, and no body. Exactly right.

**6b. Delete the same note again**

```
HTTP/1.1 404 Not Found
{"error":"No note with id 3"}
```

**The final state after all of that**

```
$ curl -s localhost:4000/api/notes
count: 2
  1 Learn Express | done: true
  2 Call Mum on Sunday | done: true
```

Note 3 is gone. Note 1's `done` flipped. Note 2 was replaced wholesale. Every change stuck.

**And the catch-all**

```
$ curl -s -i localhost:4000/api/nope
HTTP/1.1 404 Not Found
{"error":"Cannot GET /api/nope"}
```

JSON, not HTML. Your React code can `response.json()` that without it exploding.

**Meanwhile, the server terminal**

```
Notes API listening on http://localhost:4000
GET /api/notes/2
GET /api/notes/99
GET /api/notes/banana
PUT /api/notes/2
PATCH /api/notes/1
DELETE /api/notes/3
GET /api/notes
GET /api/nope
GET /health
```

Your logger, working. Get used to watching that stream while you test. When a request does not do what you expect, the first question is always: did it even arrive, and at the path you thought?

### Step 6 — Save the whole test run

Put this in `test.sh` and `chmod +x test.sh`. Running one script beats retyping nine curls, and it means you will actually re-test after every change.

```bash
#!/bin/bash
B=http://localhost:4000/api/notes

echo "=== 1. LIST ==="            ; curl -s $B
echo; echo "=== 1b. FILTER ==="   ; curl -s "$B?tag=study"
echo; echo "=== 2. CREATE ==="    ; curl -s -X POST $B -H "Content-Type: application/json" \
                                      -d '{"title":"From the script","tag":"test"}'
echo; echo "=== 3. READ ONE ===" ; curl -s $B/1
echo; echo "=== 3b. 404 ==="      ; curl -s $B/999
echo; echo "=== 4. PUT ==="       ; curl -s -X PUT $B/1 -H "Content-Type: application/json" \
                                      -d '{"title":"Replaced","body":"all fields","tag":"test","done":true}'
echo; echo "=== 5. PATCH ==="     ; curl -s -X PATCH $B/2 -H "Content-Type: application/json" \
                                      -d '{"done":true}'
echo; echo "=== 6. DELETE ==="    ; curl -s -o /dev/null -w "status=%{http_code}\n" -X DELETE $B/3
echo "=== FINAL ==="              ; curl -s $B
echo
```

This is the beginning of a real test suite. In Class 12 we replace it with proper automated tests, and the thinking is identical: know what you expect, run it every time, notice when it changes.

---

## 🏁 End-of-Class Challenge

Three parts. Part A is code, Part B is explanation, Part C is debugging. Do all three; they test different things.

### Part A — Extend the API

Add these to your working notes-api. Test each with curl before moving on.

1. **`GET /api/notes/stats`** returning `{ total, done, pending, byTag }` where `byTag` counts notes per tag. **Think carefully about where in the file this route goes**, and be ready to say why.

2. **Pagination on the list route.** Support `?page=2&perPage=2` and return `{ page, perPage, total, totalPages, notes }`. Default to page 1, perPage 10. Reject `page=0` or a negative perPage with a 400.

3. **`POST /api/notes/:id/toggle`** that flips `done` and returns the updated note. 404 if the id does not exist.

4. **Better validation on create.** Reject a title longer than 100 characters, and reject a `tag` that is not one of `study`, `home`, `work`, `general`. Both 400, each with a message that says what was wrong and what is allowed.

5. **A `sort` query param** on the list route: `?sort=title` sorts alphabetically, `?sort=-title` reverses it. Do not mutate the stored array while sorting.

### Part B — Explain It Back

Write real sentences, not bullets. If you can write these clearly you understand the class; if you cannot, that is exactly where to re-read.

1. You are helping a classmate whose `POST /api/notes` keeps responding 400 even though they are sure they sent a title. Name the three most likely causes and the single curl command you would ask them to run first.

2. A teammate writes `app.get('/notes/:id', ...)` at line 10 and `app.get('/notes/recent', ...)` at line 40. Describe what happens when someone requests `/notes/recent`, why there is no error message anywhere, and the one-line fix.

3. In your own words: why does `PATCH` exist when `PUT` can already update things? Give a concrete scenario where using PUT instead of PATCH would destroy data.

4. Explain why `res.status(404).json({error: '...'})` is better than `res.json({success: false, error: '...'})`, with specific reference to what `fetch` does in each case.

5. Your store module returns `undefined` when a note is not found, and the route turns that into a 404. Why not have the store itself return the 404? What would go wrong in Class 12, when we test the store with no HTTP server running at all?

### Part C — Find the Bug

Five snippets. Each has exactly one bug. For each: name it, say what the caller sees, say what the server terminal shows, and write the fix. Then actually run them to check, because I built and ran all five.

**Bug 1**
```javascript
app.post('/notes', (req, res) => {
  if (!req.body.title) {
    res.status(400).json({ error: 'title required' });
  }
  const note = addNote({ title: req.body.title });
  res.status(201).json(note);
});
```

**Bug 2**
```javascript
const notes = [{ id: 1, title: 'One' }, { id: 2, title: 'Two' }];

app.get('/notes/:id', (req, res) => {
  const note = notes.find(n => n.id === req.params.id);
  if (!note) return res.status(404).json({ error: 'not found' });
  res.json(note);
});
```

**Bug 3**
```javascript
const app = express();

app.use('/api/notes', notesRouter);
app.use(express.json());

app.listen(4000);
```

**Bug 4**
```javascript
app.get('/notes/:id', (req, res) => {
  const note = findNote(Number(req.params.id));
  if (note) {
    res.json(note);
  }
  // nothing else
});
```

**Bug 5**
```javascript
// src/routes/notes.js
const router = express.Router();
router.get('/api/notes', (req, res) => res.json(listNotes()));
export default router;

// src/server.js
app.use('/api/notes', notesRouter);
```

<details>
<summary>💡 Part C answers, with the real output I got from each</summary>

**Bug 1: missing `return`.** The caller sees a perfectly normal-looking `400` with `{"error":"title required"}`. But the handler kept running and tried to send a second response. Real server output:

```
Error [ERR_HTTP_HEADERS_SENT]: Cannot set headers after they are sent to the client
    at ServerResponse.setHeader (node:_http_outgoing:700:11)
    at ServerResponse.json (.../express/lib/response.js:252:15)
```

A broken note also got added to the store on the way past. Fix: `return res.status(400)...`. (Second, smaller bug: `req.body.title` crashes outright if `express.json()` is missing, since you cannot read a property of `undefined`. `req.body?.title` is safer.)

**Bug 2: comparing a string to a number with `===`.** `req.params.id` is `"1"`, `n.id` is `1`, and `"1" === 1` is `false`. So **every** request 404s, including valid ones. I added some debug output to prove it:

```
$ curl -i localhost:4007/bug-b/1
HTTP/1.1 404 Not Found
{"error":"not found","looked_for":"1","typeof":"string"}
```

Nothing in the terminal. No error anywhere. Just a lookup that silently never matches, which is the most expensive kind of bug. Fix: `Number(req.params.id)`.

**Bug 3: `express.json()` registered after the router.** Middleware runs in registration order, so by the time a POST reaches the router the body has not been parsed and `req.body` is `undefined`. Caller gets a 400 or a 500 depending on how the route is written; server shows either nothing or a `TypeError`. Fix: move `app.use(express.json())` above the router. This is why section 8 insists the order in `server.js` is load-bearing.

**Bug 4: no response on the else path.** A valid id works fine. A missing id sends nothing at all, and the request hangs until the client gives up:

```
$ curl -m 2 localhost:4007/bug-c
curl exit code: 28
```

28 is curl's timeout code. Nothing in the server terminal, no error, no log. Fix: add the 404 branch. Then make it a habit to check that every branch ends in a send.

**Bug 5: double prefix.** The mount adds `/api/notes` and the route adds it again, so the real URL is `/api/notes/api/notes`:

```
$ curl localhost:4008/api/notes
(404)
$ curl localhost:4008/api/notes/api/notes
{"hit":"double prefix"}
```

Fix: `router.get('/')`. Say the prefix once, in `app.use`.
</details>

---

## 📌 Cheat Sheet

```
SETUP
  npm install express dotenv
  npm install --save-dev nodemon
  package.json:  "type": "module"
                 "scripts": { "start": "node src/server.js",
                              "dev":   "nodemon src/server.js" }

MINIMAL SERVER
  import express from 'express';
  const app = express();
  app.get('/health', (req, res) => res.json({ status: 'ok' }));
  app.listen(4000, () => console.log('up on 4000'));

ROUTES                             ORDER MATTERS, top to bottom, first match wins
  app.get(path, handler)           Specific paths ABOVE :param paths
  app.post(path, handler)          /notes/archived  before  /notes/:id
  app.put(path, handler)           Catch-all 404 LAST
  app.patch(path, handler)
  app.delete(path, handler)
  app.all(path, handler)           every method, debug only

READING THE REQUEST                EVERYTHING FROM URL IS A STRING
  req.params      { id: '42' }     Number(req.params.id)
  req.query       { tag: 'x' }     Number(req.query.limit) || 10
  req.body        needs express.json()   req.query.done === 'true'
  req.method      'POST'
  req.path        '/api/notes/42'
  req.originalUrl '/api/notes/42?tag=x'
  req.get('content-type')
  req.ip

  Repeated query key -> array:  ?t=a&t=b  =>  { t: ['a','b'] }
  No Content-Type on POST    ->  req.body is undefined

SENDING THE RESPONSE               ONE SEND PER REQUEST, NO MORE, NO LESS
  res.json(obj)                    default for APIs
  res.status(201).json(obj)        chainable
  res.status(204).end()            success, no body
  res.send(x)                      guesses the type, avoid in APIs
  res.sendStatus(403)              status + its name as plain text
  res.set('X-Thing', 'v')

  ALWAYS:  return res.status(400).json(...)   on early exit

STATUS CODES
  200 OK            GET ok, PUT/PATCH ok
  201 Created       POST made something. Return it with its id.
  204 No Content    DELETE ok. No body. Use .end()
  400 Bad Request   caller sent rubbish
  401 Unauthorized  not logged in
  403 Forbidden     logged in, not allowed
  404 Not Found     no such path or id
  409 Conflict      clashes with existing data
  500 Server Error  YOUR code broke
  4xx = their fault. 5xx = yours. Never 200 with an error inside.

MIDDLEWARE ORDER IN server.js
  1. app.use(express.json())       bodies first
  2. app.use(logger)               then logging
  3. app.use('/api/notes', router) then routes
  4. app.use(catchAll404)          LAST
  5. app.listen(PORT)

ROUTER
  // routes/notes.js
  const router = express.Router();
  router.get('/', h);          -> GET  /api/notes
  router.get('/:id', h);       -> GET  /api/notes/42
  export default router;

  // server.js
  app.use('/api/notes', notesRouter);

  Prefix lives in app.use ONLY. Never repeat it inside the router.

  Chained form:  router.route('/:id').get(a).put(b).patch(c).delete(d)

CURL
  curl -s URL                      quiet
  curl -i URL                      show response headers
  curl -X POST URL                 set the method
  curl -H "Content-Type: application/json"
  curl -d '{"title":"x"}'          send a body
  curl -o /dev/null -w "%{http_code}\n" URL     just the status
  curl -m 2 URL                    timeout after 2s (exit 28 = hung)

ERRORS YOU WILL MEET
  EADDRINUSE                   port taken.  lsof -i :4000 ; kill -9 PID
  Connection refused           nothing listening. Check the server terminal.
  404                          something IS listening, wrong path.
  ERR_HTTP_HEADERS_SENT        missing `return` before an early res
  Cannot read ... of undefined missing app.use(express.json())
  request hangs forever        a branch with no res send, or no next()
  route never fires            shadowed by a :param route above it
```

---

## ✅ Self-Check

Tick honestly. Anything unticked is a thing to revisit before Class 4.

- [ ] I can explain what a framework does, using File 1 and File 2 as the example
- [ ] I can write a four-line Express server from memory, no copy-paste
- [ ] I know what `EADDRINUSE` means and the two commands that fix it
- [ ] I can tell "connection refused" apart from a 404, and know which half of the system each one points at
- [ ] I can state the route-matching rule in one sentence
- [ ] I have caused the route-shadowing bug on purpose and watched it happen
- [ ] I can decide param vs query for a URL I have never seen before
- [ ] I remember that params and query values are always strings, and I convert at the boundary
- [ ] I know `req.body` does not exist without `app.use(express.json())`
- [ ] I know what happens when a POST arrives with no Content-Type header
- [ ] I write `return res.` on every early exit, automatically
- [ ] I can name the right status code for create, delete, missing thing, and bad input, without looking
- [ ] I can explain why 200-with-an-error-inside breaks React's `fetch`
- [ ] I have nodemon running and I know why my notes reset on every save
- [ ] I can move routes into a Router file and verify nothing broke
- [ ] I know why the prefix goes in `app.use` and not inside the router
- [ ] My notes-api answers all six endpoints correctly, including the error cases
- [ ] My `server.js` middleware is in the right order and I can say why each line sits where it does

---

## 📚 Homework

1. **Finish Part A.** All five extensions working, each verified with curl. Commit as you go, one commit per extension, with messages someone else could follow.

2. **Add a second resource.** `src/routes/tags.js`, mounted at `/api/tags`, with `GET /api/tags` returning every distinct tag and how many notes use it, and `GET /api/tags/:tag/notes` returning the notes with that tag. This forces you to use Router for real rather than as an exercise, and it is the moment the file-splitting starts paying for itself.

3. **Write the README.** For each of your endpoints: method, path, what it does, an example request with curl, an example success response, and the error cases. Then hand it to someone who has not seen your code and ask them to create and delete a note using only your README. Wherever they get stuck, your docs are wrong. Fix them.

4. **Grow the test script.** Extend `test.sh` to cover every endpoint including every error path, with the expected status printed next to the actual one. Aim for a single command that tells you whether the whole API still works.

5. **Read before Class 4.** The Express guide on [using middleware](https://expressjs.com/en/guide/using-middleware.html). Come with one question about it. You have already written one piece of middleware today, the logger in `server.js`, so read it with that in front of you and see what it says about `next()`.

**Stretch, if you want it:** take the React app you have already built, point a `fetch` at `http://localhost:4000/api/notes`, and try to display the list. It will probably fail with a CORS error in the browser console. Do not fix it yet. Bring the exact error message to Class 4; it is the first thing we handle, and meeting it yourself first is worth more than me describing it.

---

## 🚀 Next Class

**Class 4, Week 2 Session 2: Middleware & Error Handling**

The logger you wrote today was middleware, and so was `express.json()`. Next class we take that pattern apart: how `next()` passes control along the chain, how a single error handler can catch failures from every route at once so you stop writing the same try/catch forty times, and how CORS actually works, which is what stands between your API and the React app you want to point at it.

That CORS error from the stretch homework is the first thing on the board.

---

*Schull AI Academy · Full-Stack & AI Engineering · Class 3 of 40*
