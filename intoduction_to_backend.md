# Class 1 — The Backend Mental Model

### Full-Stack & AI Engineering · Phase 1 · Week 1, Session 1
**Prepared for Chinyere E.** · Schull AI Academy

**Duration:** 2 hours
**Prerequisites:** Working knowledge of React and JavaScript
**You need:** A laptop, a browser, and an internet connection. No code editor required today.

---

## 🎯 What You'll Walk Away With

By the end of this session you will be able to:

- Explain what a server actually *does*, in your own words
- Trace a request from the moment you click to the moment data appears on screen
- Say precisely **why** a React app cannot do everything on its own
- Pick the right HTTP method for any operation
- Read a status code and know instantly who caused the problem — you or the server
- Break any request into its four parts: headers, body, query params, path params
- Call a live API from the terminal without writing a single line of app code

> 🔑 **One thing to hold onto today:** you are not learning something new. You are learning the **other half** of something you already do.

---

## 🚪 Opening — You've Been Living on One Side of a Line

You've written this before. Probably many times:

```javascript
useEffect(() => {
  fetch('https://api.example.com/products')
    .then(res => res.json())
    .then(data => setProducts(data));
}, []);
```

You know what happens **after** the data arrives. You've handled the loading spinner. You've handled the error state. You've mapped over the array and rendered the cards.

But look at that URL again.

```
https://api.example.com/products
```

**Something, somewhere, answered that.** A program you didn't write, running on a computer you've never seen, received your request, decided what to send back, and sent it.

That program is the backend. And starting today, you're going to be the one who writes it.

```
        ┌──────────────────────┐        ┌──────────────────────┐
        │                      │        │                      │
        │   THE FRONTEND       │        │   THE BACKEND        │
        │   (you live here)    │───────▶│   (where we're       │
        │                      │◀───────│    going)            │
        │   React, components, │        │                      │
        │   state, JSX         │        │   ???                │
        │                      │        │                      │
        └──────────────────────┘        └──────────────────────┘
                              THE LINE
```

Today we walk across the line and look back.

---

### ✏️ ACTIVITY 1 — Draw the Line *(5 minutes)*

On paper, draw two boxes with a line between them, like the diagram above.

**In the left box**, write down 5 things you know how to do in React.
**In the right box**, write down 5 things you *suspect* happen on the server but have never seen.

Then answer out loud:

1. When you call `fetch()`, how long does your code wait before data comes back?
2. What would happen if the thing on the other side of the line simply... didn't reply?
3. Where do you think your data physically *lives* when your React app is closed?

> 🎙️ **Instructor note:** Don't correct the right-hand box. Take a photo of it. We'll come back to it in Week 4 and she'll laugh at it. Whatever she guesses wrong today becomes the most memorable lesson later.

---

## 1️⃣ What a Server Actually Does

Strip away all the jargon, and a server does exactly **three** things, forever, in a loop:

```
        ┌─────────────────────────────────────────┐
        │                                         │
        │    1.  LISTEN    ──▶  a request arrives │
        │                                         │
        │    2.  DECIDE    ──▶  what should I do  │
        │                       about this?       │
        │                                         │
        │    3.  RESPOND   ──▶  send something    │
        │                       back              │
        │                                         │
        └──────────────┬──────────────────────────┘
                       │
                       └──────▶ go back to step 1, forever
```

That's it. That's a server. Everything else — databases, authentication, AI agents, deployment — is detail hanging off those three steps.

### 🍽️ The Restaurant

This analogy will carry you through the whole course, so let's set it up properly.

| Restaurant | Software | Notes |
|---|---|---|
| **You, the customer** | The React app (the **client**) | You want something. You can't make it yourself. |
| **The menu** | The API documentation | It tells you what you're allowed to ask for |
| **The waiter** | The API / the server | Takes your order, brings back food. Doesn't cook. |
| **The kitchen** | Business logic | Where the actual work happens |
| **The pantry / fridge** | The database | Where ingredients are stored permanently |
| **Your order** | The **request** | "One jollof rice, no pepper" |
| **The food** | The **response** | What actually arrives at your table |
| **"We're out of that"** | An error response | The kitchen replying that it can't help |

**Three things this analogy gets right, and they matter:**

1. **You never walk into the kitchen.** The customer doesn't touch the pantry. Ever. This is why your React app should never talk directly to a database.
2. **The waiter decides what's a valid order.** Ask for something not on the menu and you get refused. That's request validation.
3. **The kitchen doesn't care who's asking** — until it does. Some orders need ID (alcohol). That's authentication.

> 💡 **The single most useful reframe:** a backend is not "advanced React." It's a completely different job. The frontend's job is to *present*. The backend's job is to *decide and remember*.

---

### ✏️ ACTIVITY 2 — Be the Server *(10 minutes)*

You're going to play the server. No computer. Pen and paper only.

Your instructor will read out "requests." For each one, write down:
- **What you'd send back** (the response)
- **Whether you can even answer it**, and if not, why

Here are the requests:

| # | Request |
|---|---------|
| 1 | "Give me the details of user number 7" |
| 2 | "Give me the details of user number 99999" |
| 3 | "Create a new user called Ada" |
| 4 | "Create a new user" *(no name given)* |
| 5 | "Delete every user in the system" |
| 6 | "Give me user 7's password" |
| 7 | "Blorp the flonk" |

**Then discuss:**

- Which requests could you answer, and which couldn't you?
- For the ones you refused — **whose fault was it?** Yours, or the person asking?
- What extra information did you *wish* you had before answering #5?

<details>
<summary>💡 What this activity is secretly teaching</summary>

You just invented, from scratch, the four things every backend does:

| Request | The concept you discovered |
|---|---|
| #1 | **Successful retrieval** — the happy path (this becomes `200 OK`) |
| #2 | **Resource not found** — the thing asked for doesn't exist (`404`) |
| #3 | **Creation** — making new data (`201 Created`) |
| #4 | **Validation failure** — the request itself was malformed (`400 Bad Request`) |
| #5 | **Authorisation** — *should* this person be allowed to do that? (`403 Forbidden`) |
| #6 | **Data protection** — some data must never be sent back, even to a valid request |
| #7 | **Unknown route** — no endpoint matches (`404` again) |

Notice question #5. You instinctively wanted to know *who was asking* before you obeyed. That instinct is authentication, and you had it before I taught it to you.
</details>

---

## 2️⃣ The Request/Response Lifecycle

Now let's follow one request all the way through. You type a URL and press Enter. Here is everything that happens.

```
  ①  YOU              "I want api.example.com/products"
      │
      ▼
  ②  DNS LOOKUP       "api.example.com" → 142.250.80.46
      │               (a phonebook lookup — names to numbers)
      ▼
  ③  TCP CONNECTION   Your machine dials that address
      │               and they agree to talk
      ▼
  ④  TLS HANDSHAKE    They agree on encryption (this is the 🔒)
      │
      ▼
  ⑤  HTTP REQUEST     "GET /products HTTP/1.1"
      │                Host: api.example.com
      │                Accept: application/json
      ▼
  ⑥  SERVER RECEIVES  Which route is this? /products. Got it.
      │
      ▼
  ⑦  SERVER WORKS     Query the database. Check permissions.
      │               Format the result.
      ▼
  ⑧  HTTP RESPONSE    "200 OK"
      │                Content-Type: application/json
      │                [{"id":1,"name":"Shoes"}, ...]
      ▼
  ⑨  YOUR CODE        .then(res => res.json())
      │
      ▼
  ⑩  RENDER           setProducts(data) → React re-renders → 🎉
```

**How long does all of this take?** Usually **under 200 milliseconds**. Faster than you can blink.

> 🧠 **The step nobody tells beginners about:** Step ⑦. Everything from ① to ⑥ is plumbing that mostly works automatically. Step ⑦ — *the server deciding what to do* — is the part **you** will be writing for the rest of this course. That's the job.

---

### ✏️ ACTIVITY 3 — Trace a Real Request in DevTools *(15 minutes)*

Time to see it with your own eyes. This is the most important activity in today's class.

**Steps:**

1. Open Chrome. Press **F12** (or right-click → Inspect) to open DevTools.
2. Click the **Network** tab.
3. Tick the **Preserve log** checkbox.
4. Now navigate to **`https://github.com`**
5. Watch the list explode with entries.

**Now investigate. Write down your answers:**

| # | Question |
|---|----------|
| 1 | How many requests did that single page load make? (Look at the bottom bar) |
| 2 | Click the very **first** request in the list. What is its **Status Code**? |
| 3 | In the **Headers** tab, find `Request Method`. What is it? |
| 4 | Scroll to **Response Headers**. Find `content-type`. What does it say? |
| 5 | Find a request whose Type is `png`, `jpeg` or `svg`. What is it? |
| 6 | Look at the **Time** column. Which request was the **slowest**? |
| 7 | Now filter by **Fetch/XHR** at the top. These are the API calls. How many are there? |
| 8 | Click one XHR request → **Response** tab. Is it HTML or JSON? |

**Now the good part — break something:**

9. In the address bar, go to **`https://github.com/this-page-does-not-exist-9999`**
10. Look at the first request's status code. **What number is it?**
11. Go to **`https://api.github.com/users/torvalds`**. What do you see on screen? Why does it look different from a normal website?

> 🎙️ **Instructor note:** Sit with question 11. When she sees raw JSON rendered in a browser window, the penny usually drops — *"oh, an API isn't a website, it's just data."* That realisation is the whole point of today. Let her say it out loud before you explain it.

<details>
<summary>💡 Expected answers</summary>

1. Typically 40–80+ requests for a single GitHub page load
2. `200` (OK)
3. `GET`
4. `text/html; charset=utf-8` — the browser is being sent a *page*
5. A logo, avatar, or icon — proof that **every image is its own separate request**
6. Usually the main document, or a large JS bundle
7. Varies — often 1–5 on initial load
8. **JSON** — this is the API layer talking, not the page layer
9. —
10. `404` — the server is explicitly saying "that resource doesn't exist"
11. Raw JSON text. It looks "broken" because there's no HTML and no CSS. **This is what an API returns.** The prettiness was always the frontend's job.

**The big takeaway:** one page = dozens of requests. You have only ever written the code that *makes* them. Now you'll write the code that *answers* them.
</details>

---

## 3️⃣ Why Does React Even Need a Backend?

Fair question. React is powerful. Why can't it just do everything?

Five reasons. Each one is a wall you cannot climb from the frontend.

### 🧱 Wall 1 — Persistence

```javascript
const [todos, setTodos] = useState([]);   // lives in RAM
```

Refresh the page. **Gone.** Close the tab. **Gone.** Open on your phone. **Never existed.**

State lives in the browser's memory, and memory dies. Real data has to live somewhere that survives — a database, on a server.

### 🧱 Wall 2 — Secrets

Here's a genuine, career-ending mistake:

```javascript
// ❌ NEVER. DO. THIS.
const OPENAI_KEY = "sk-proj-abc123realkey";
fetch('https://api.openai.com/v1/chat/completions', {
  headers: { Authorization: `Bearer ${OPENAI_KEY}` }
});
```

**Why it's catastrophic:** everything in your React bundle is **downloaded to the user's computer**. Anyone can open DevTools → Sources and read it. Bots scan GitHub for exactly this. People have woken up to five-figure API bills.

> 🚨 **The rule, and it is absolute:** if a value must stay secret, it lives on the server. The frontend calls *your* backend; *your* backend holds the key and calls the AI provider. We build exactly this in Phase 4.

### 🧱 Wall 3 — Trust

Never trust the client. Ever.

```javascript
// Frontend "validation"
if (price < 0) {
  alert("Invalid price!");
  return;
}
```

Lovely. Now watch me bypass it entirely from the terminal:

```bash
curl -X POST https://yourshop.com/api/orders \
  -H "Content-Type: application/json" \
  -d '{"item": "MacBook Pro", "price": -5000}'
```

Your `if` statement never ran. I didn't use your form. **Frontend validation is a courtesy to honest users; backend validation is what actually protects you.** You need both — but only one of them is security.

### 🧱 Wall 4 — Shared Truth

If your data lives in one person's browser, nobody else can see it. No comments. No messaging. No multiplayer. No "3 people are viewing this product." A backend is the one place everyone can agree on.

### 🧱 Wall 5 — Work the Browser Can't Do

Send an email. Charge a card. Resize a video. Run a nightly job at 2am. Query a database. Talk to a payment provider. The browser is sandboxed and can't do any of it.

---

### ✏️ ACTIVITY 4 — Spot the Disaster *(10 minutes)*

Four snippets. For each one: **what's wrong, what could an attacker do, and where should the logic actually live?**

```javascript
// ── A ──
const ADMIN_PASSWORD = "SuperSecret123";
function login(input) {
  if (input === ADMIN_PASSWORD) setIsAdmin(true);
}
```

```javascript
// ── B ──
const [cart, setCart] = useState([]);
const total = cart.reduce((sum, item) => sum + item.price, 0);
// ...then sends `total` to the payment provider
```

```javascript
// ── C ──
const user = { name: "Ada", isPremium: false };
if (user.isPremium) showPremiumContent();
```

```javascript
// ── D ──
fetch(`https://api.weather.com/data?apikey=a1b2c3d4e5&city=${city}`);
```

<details>
<summary>💡 Answers</summary>

**A — Password in the frontend.**
The password is sitting in plain text in the downloaded bundle. Open DevTools → Sources → search "password" → done. An attacker doesn't even need to guess.
*Correct approach:* the password never leaves the server. The client sends an attempt, the server compares against a **hashed** value and returns a token. We build this in Class 7.

**B — Price calculated client-side.**
The attacker edits `cart` in DevTools, sets every price to `0.01`, and checks out. Your server happily charges one naira for a laptop.
*Correct approach:* the client sends only **item IDs and quantities**. The server looks up real prices from its own database and computes the total. Never trust a number the client gives you about money.

**C — Permissions checked in the UI.**
`isPremium` is just a JavaScript object in the browser. Attacker flips it to `true` in the console and the premium content renders.
*Correct approach:* the UI may *hide* things for tidiness, but the **server must refuse to send** premium data to a non-premium user. Hiding is not securing.

**D — API key in the URL.**
It's in the bundle, it's in the Network tab, and it's in server logs along the way. Also fully visible to any browser extension.
*Correct approach:* React calls `yourapi.com/weather?city=Lagos`. Your server adds the key and forwards the request. The key never touches the browser.

**The pattern in all four:** the frontend can be *edited by whoever is using it*. Treat every byte from a client as a suggestion, not a fact.
</details>

---

## 4️⃣ HTTP Methods — The Verbs

Every request carries a **method**: what you intend to *do*.

| Method | Means | Example | Changes data? | Safe to repeat? |
|--------|-------|---------|:---:|:---:|
| **GET** | Read something | `GET /products` | ❌ No | ✅ Yes |
| **POST** | Create something new | `POST /products` | ✅ Yes | ❌ **No** |
| **PUT** | Replace something entirely | `PUT /products/7` | ✅ Yes | ✅ Yes |
| **PATCH** | Update part of something | `PATCH /products/7` | ✅ Yes | ✅ Yes |
| **DELETE** | Remove something | `DELETE /products/7` | ✅ Yes | ✅ Yes |

### The two that confuse everyone

**POST vs PUT:**

```
POST /products        →  "Here's a new product. You assign it an ID."
PUT  /products/7      →  "Make product 7 look exactly like this."
```

POST creates. PUT overwrites a specific thing. Call `POST` five times and you get **five products**. Call `PUT /products/7` five times and you get **one product**, set five times. That property has a name: **idempotent** — repeating it changes nothing further.

> 🛒 **Why it matters practically:** this is exactly why a shopping site warns you *"do not refresh this page"* after payment. Payment is a `POST`. Refreshing resends it. Two charges.

**PUT vs PATCH:**

```javascript
// Product 7 currently:  { name: "Shoes", price: 5000, stock: 12 }

PUT /products/7   { "price": 6000 }
// → { price: 6000 }              ← 😱 name and stock WIPED. PUT replaces everything.

PATCH /products/7 { "price": 6000 }
// → { name: "Shoes", price: 6000, stock: 12 }   ← ✅ only price changed
```

> 💡 **In practice:** most real APIs use `PATCH` for edits, because you rarely want to resend an entire object just to change one field.

---

### ✏️ ACTIVITY 5 — Verb Matching *(8 minutes)*

You're designing the API for a music app. Write the **method + path** for each action. First try alone, then compare.

| # | The action | Method + path |
|---|-----------|---------------|
| 1 | Get all playlists | |
| 2 | Get playlist number 12 | |
| 3 | Create a new playlist | |
| 4 | Rename playlist 12 | |
| 5 | Delete playlist 12 | |
| 6 | Add a song to playlist 12 | |
| 7 | Remove song 88 from playlist 12 | |
| 8 | Get all songs in playlist 12 | |
| 9 | Replace playlist 12 completely | |
| 10 | Log in | |

<details>
<summary>💡 Answers — and the reasoning</summary>

| # | Answer | Why |
|---|--------|-----|
| 1 | `GET /playlists` | Plural noun. Reading a collection. |
| 2 | `GET /playlists/12` | The ID goes in the **path** — it identifies *which one* |
| 3 | `POST /playlists` | Create → POST to the **collection**, not to an ID |
| 4 | `PATCH /playlists/12` | Partial change → PATCH |
| 5 | `DELETE /playlists/12` | Self-explanatory |
| 6 | `POST /playlists/12/songs` | Creating a thing inside a thing — **nested resource** |
| 7 | `DELETE /playlists/12/songs/88` | Nested, and identifying exactly which song |
| 8 | `GET /playlists/12/songs` | Reading a nested collection |
| 9 | `PUT /playlists/12` | Full replacement → PUT |
| 10 | `POST /auth/login` | POST, because you're *sending* credentials in the body — never in a URL |

**Two rules you just used without being told:**

1. **Paths are nouns, methods are verbs.** `GET /playlists`, never `GET /getPlaylists`. The verb is already in the method — saying it twice is redundant.
2. **Plural, always.** `/playlists/12`, not `/playlist/12`. Reads as "of all playlists, number 12."

**Trap check:** did you write `POST /createPlaylist` or `GET /deletePlaylist/12`? Almost everyone does at first. `GET` must never change data — some browsers and proxies pre-fetch `GET` URLs, which would silently delete things.
</details>

---

## 5️⃣ Status Codes — Who Broke It?

Every response carries a three-digit number. The **first digit** tells you the whole story:

```
  1xx  ℹ️   "Hold on, still working"       (rare — you can ignore these)
  2xx  ✅   "It worked"
  3xx  ↪️   "It moved, look over there"
  4xx  🙋   "YOU made a mistake"
  5xx  💥   "I made a mistake"
```

> 🔑 **The most valuable debugging habit in this entire course:**
> **4xx = your problem. 5xx = the server's problem.**
> That one distinction tells you which side of the line to start looking on. It will save you hours.

### The ones you'll actually meet

| Code | Name | What it really means |
|------|------|---------------------|
| **200** | OK | It worked. Here's your data. |
| **201** | Created | It worked *and* something new exists now. Use for `POST`. |
| **204** | No Content | It worked, there's nothing to send back. Common for `DELETE`. |
| **301** | Moved Permanently | This URL is dead, use the new one forever |
| **400** | Bad Request | Your request was malformed — missing field, bad JSON |
| **401** | Unauthorized | **"Who are you?"** You didn't log in |
| **403** | Forbidden | **"I know who you are, and no."** Logged in, not allowed |
| **404** | Not Found | That thing doesn't exist |
| **409** | Conflict | Clashes with current state — e.g. email already registered |
| **429** | Too Many Requests | Slow down. You're rate-limited. |
| **500** | Internal Server Error | The server crashed. **Your bug.** Go read the logs. |
| **503** | Service Unavailable | Server is up but can't cope — overloaded or down for maintenance |

### 401 vs 403 — worth ten seconds of your attention

This trips up professionals, so get it right now:

```
401  →  "I don't know who you are."     →  go log in
403  →  "I know exactly who you are,    →  logging in again won't help.
         and you still can't."              you lack permission.
```

A nightclub: **401** is arriving with no ID. **403** is showing valid ID that says you're 16.

---

### ✏️ ACTIVITY 6 — Status Code Diagnosis *(10 minutes)*

For each scenario, give the status code **and** say whether the client or the server is at fault.

| # | Scenario | Code | Whose fault? |
|---|----------|:---:|:---:|
| 1 | User requests `/products/99999`, which doesn't exist | | |
| 2 | Signup form submitted with no email field | | |
| 3 | New user account successfully created | | |
| 4 | Someone tries to open the admin page while logged in as a normal user | | |
| 5 | The database connection string is wrong, so the query crashes | | |
| 6 | A logged-out visitor requests their profile | | |
| 7 | Someone signs up with an email that's already taken | | |
| 8 | A bot sends 5,000 requests in one minute | | |
| 9 | A product is deleted successfully, nothing to return | | |
| 10 | Your code has a typo and throws `undefined is not a function` | | |

<details>
<summary>💡 Answers</summary>

| # | Code | Fault | Note |
|---|------|-------|------|
| 1 | **404** | Client | They asked for something that isn't there |
| 2 | **400** | Client | Malformed request — this is validation's job |
| 3 | **201** | — | Not `200`. `201` specifically signals *something now exists* |
| 4 | **403** | Client | They're identified, just not permitted |
| 5 | **500** | **Server** | Nothing the user did caused this. Go fix your config. |
| 6 | **401** | Client | No identity presented at all |
| 7 | **409** | Client | Conflict with existing state. `400` would also be accepted, but `409` is more precise |
| 8 | **429** | Client | Rate limiting — we implement this in Phase 4 |
| 9 | **204** | — | Succeeded, no body. Or `200` with a confirmation message. |
| 10 | **500** | **Server** | Your bug. Always your bug when it's a 5xx. |

**Score yourself:** 8+ is excellent for a first class. If you mixed up 401 and 403, reread the nightclub example — it comes up in Class 7 when we build real authentication.
</details>

---

## 6️⃣ The Anatomy of a Request

Every request has up to four places to carry information. Knowing which is which is genuinely half of backend work.

```
        ┌──────────────────────────────────────────────────┐
        │  POST /api/products/7/reviews?notify=true        │
        │       └─────┬─────┘ │        └──────┬──────┘     │
        │          PATH    PATH PARAM      QUERY PARAMS    │
        │                                                  │
        │  Headers:                                        │
        │    Content-Type: application/json    ← HEADERS    │
        │    Authorization: Bearer eyJhbG...               │
        │                                                  │
        │  Body:                               ← BODY      │
        │    { "rating": 5, "text": "Great!" }             │
        └──────────────────────────────────────────────────┘
```

| Part | Carries | Example | Use it for |
|------|---------|---------|-----------|
| **Path** | *Which* resource | `/products/7` | Identifying a specific thing |
| **Path param** | An ID inside the path | the `7` above | Required identifiers |
| **Query param** | Optional modifiers | `?sort=price&limit=10` | Filtering, sorting, paging, search |
| **Headers** | Metadata about the request | `Authorization`, `Content-Type` | Auth tokens, content format |
| **Body** | The actual payload | `{ "rating": 5 }` | Data you're sending in |

### The decision rule

> **Path param** → *required* and identifies the resource. `/users/7`
> **Query param** → *optional* and modifies the result. `/users?role=admin`
> **Body** → the data you're creating or updating. Only on `POST`/`PUT`/`PATCH`.
> **Header** → information *about* the request, not the request's subject.

**`GET` requests have no body.** If you're sending data with a GET, something has gone wrong in your design.

---

### ✏️ ACTIVITY 7 — Dissect the Request *(8 minutes)*

Break each of these into its parts. Name the method, path, path params, query params, and what you'd expect in the body.

```
1.  GET /api/v1/users/42/orders?status=pending&limit=5

2.  POST /api/v1/auth/login
    Content-Type: application/json
    { "email": "ada@mail.com", "password": "hunter2" }

3.  PATCH /api/v1/products/88?notify=false
    Authorization: Bearer eyJhbGciOiJIUzI1
    { "price": 7500 }

4.  DELETE /api/v1/playlists/3/songs/19
```

<details>
<summary>💡 Answers</summary>

**1.** Method `GET` · Path `/api/v1/users/42/orders` · Path param: `42` (the user) · Query params: `status=pending`, `limit=5` · Body: **none** (it's a GET)
→ *"Give me the first 5 pending orders belonging to user 42."*

**2.** Method `POST` · Path `/api/v1/auth/login` · No path params · No query params · Headers: `Content-Type` · Body: email + password
→ Note the credentials are in the **body**, never the URL. URLs get logged, cached and appear in browser history.

**3.** Method `PATCH` · Path `/api/v1/products/88` · Path param: `88` · Query param: `notify=false` · Header: `Authorization` (a token — this is a protected action) · Body: just the field being changed
→ `PATCH` + one field = classic partial update. The token proves who's allowed to do it.

**4.** Method `DELETE` · Path `/api/v1/playlists/3/songs/19` · **Two** path params: playlist `3` and song `19` · No body
→ Nested resource. Reads as "song 19, of playlist 3."
</details>

---

## 7️⃣ Talking to APIs Without a Frontend

Here's something that will change how you work: **you don't need a React app to test an API.**

Two tools. Learn both.

| | `curl` | Postman |
|---|--------|---------|
| What | Terminal command | Desktop app with a GUI |
| Best for | Quick checks, scripts, servers, sharing in chat | Exploring, saving collections, teamwork |
| Learning curve | Steeper | Gentle |
| Available on | Every machine, everywhere | Needs installing |

> 🧰 **Use Postman while you're learning. Use `curl` when you're working.** In three months you'll reach for `curl` without thinking, and you'll be able to paste a one-line reproduction of any bug into a message.

### curl — the flags that matter

```bash
# Plain GET — just the body
curl https://api.github.com/users/torvalds

# -i : show response headers AND body   ← your most-used flag
curl -i https://api.github.com/users/torvalds

# -I : headers ONLY (fastest way to check if something is alive)
curl -I https://api.github.com

# Just the status code — brilliant for quick checks
curl -s -o /dev/null -w "%{http_code}\n" https://api.github.com

# POST with a JSON body
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Ada","email":"ada@mail.com"}'

# Send an auth token
curl https://api.example.com/me \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"

# Follow redirects (3xx)
curl -L https://github.com

# Time the request
curl -s -o /dev/null -w "status=%{http_code}  time=%{time_total}s\n" https://api.github.com
```

**Flag cheat sheet:**

| Flag | Does |
|------|------|
| `-i` | Include response headers with the body |
| `-I` | Headers only, no body |
| `-X` | Set the method (`-X POST`, `-X DELETE`) |
| `-H` | Add a header |
| `-d` | Send a body |
| `-s` | Silent — hide the progress bar |
| `-L` | Follow redirects |
| `-o` | Write output to a file (`-o /dev/null` = discard) |
| `-w` | Write out a custom format after the request |

### What a real response looks like

Here's an actual `curl -i` against a small server, so you can see the shape:

```bash
curl -i http://localhost:3100/api/users/1
```
```
HTTP/1.1 200 OK
Content-Type: application/json
Date: Thu, 17 Sep 2026 19:04:43 GMT
Connection: keep-alive

{
  "id": 1,
  "name": "Chinyere",
  "role": "developer"
}
```

Look carefully. **Three parts:**
1. **Status line** — `HTTP/1.1 200 OK`
2. **Headers** — metadata, then a blank line
3. **Body** — the JSON you actually wanted

That **blank line** is the boundary between headers and body. It's part of the HTTP specification. When you write your own server in Class 10, you'll see it up close.

And a failed request:

```
HTTP/1.1 404 Not Found
Content-Type: application/json

{
  "error": "Not Found"
}
```

> 💡 **Note the error still has a proper body.** Good APIs explain *what* went wrong, not just *that* it went wrong. When you design your own error responses in Class 4, come back to this.

---

### ✏️ ACTIVITY 8 — THE MAIN LAB *(20 minutes)*

**This is today's core deliverable.** You'll call three live APIs using both tools.

#### Part A — curl in the terminal

Run each command. For every one, write down the **status code**, the **content-type**, and **one thing you found interesting**.

```bash
# 1. A simple GET
curl -i https://api.github.com/users/torvalds

# 2. Headers only — is it alive?
curl -I https://api.github.com

# 3. Status code only
curl -s -o /dev/null -w "%{http_code}\n" https://api.github.com/users/torvalds

# 4. Now force a 404 — a user that cannot exist
curl -i https://api.github.com/users/zzzznotarealuser99999

# 5. Query parameters in action
curl -i "https://api.github.com/search/repositories?q=react&per_page=3"

# 6. Time it
curl -s -o /dev/null -w "status=%{http_code}  time=%{time_total}s\n" https://api.github.com
```

#### Part B — the same thing in Postman

1. Install Postman (or use the web version at postman.com)
2. Create a new **Collection** named `Class 1 Lab`
3. Add a request: `GET https://api.github.com/users/torvalds` → **Send**
4. Explore the response panel: **Body**, **Headers**, **Status**, **Time**, **Size**
5. Add a second request that deliberately 404s
6. Add a third using query params — use the **Params** tab rather than typing them into the URL, and watch the URL build itself
7. **Save the collection**

#### Part C — Write it up

Fill in this table and bring it to the next class:

| # | Request | Status | Content-Type | Notes |
|---|---------|--------|--------------|-------|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |
| 5 | | | | |

#### Part D — Questions to answer

1. In request 1, find the header `x-ratelimit-remaining`. What is it, and why do you think GitHub sends it?
2. Request 4 returned a 404. **Did the request fail?** Careful — think about it.
3. Compare the **response size** of request 1 and request 5. Why the difference?
4. In Postman, what does the **Time** value tell you, and what would make it larger?

<details>
<summary>💡 Guidance on the questions</summary>

**1.** It tells you how many more requests you're allowed before being blocked. GitHub sends it so well-behaved clients can throttle themselves rather than get cut off. If you see `x-ratelimit-remaining: 0`, you'll get a `403` on your next call until the reset time. **You may well hit this during the lab** — that's not a mistake, it's the real world, and it's why Phase 4 covers rate limiting properly.

**2.** **No — the request succeeded perfectly.** It travelled to GitHub, was understood, and received a clear answer. The *answer* was "that doesn't exist." A failed request is one that never gets a response at all: no network, DNS failure, timeout. This distinction matters enormously when you write error handling — `fetch()` does **not** throw on a 404.

**3.** Request 5 returns three whole repository objects with dozens of fields each; request 1 returns one user. More data, bigger response. This is exactly why pagination exists, and why we build it in Class 5.

**4.** Round-trip time: your machine → GitHub → back. It grows with physical distance, server load, response size, and slow database queries on their end. When you build your own API, this number becomes *your* responsibility.
</details>

---

## 🏁 End-of-Class Challenge

*Do this without looking back through the notes. 15 minutes.*

### Part 1 — Design an API on paper *(no code)*

You're building the backend for a **student attendance app**. Write the method and path for each operation:

| # | Operation | Your answer |
|---|-----------|-------------|
| 1 | List all students | |
| 2 | Get one student's details | |
| 3 | Register a new student | |
| 4 | Update a student's phone number | |
| 5 | Remove a student | |
| 6 | List all sessions | |
| 7 | Mark a student present at a session | |
| 8 | Get a student's full attendance record | |
| 9 | Get only the sessions a student **missed** | |
| 10 | Log in as an instructor | |

### Part 2 — Assign the status codes

| # | Situation | Code |
|---|-----------|------|
| 1 | Student registered successfully | |
| 2 | Requested student ID doesn't exist | |
| 3 | Registration submitted with no name | |
| 4 | A student tries to view another student's record | |
| 5 | Nobody is logged in | |
| 6 | Server's database is unreachable | |
| 7 | Student deleted, nothing to return | |

### Part 3 — Explain it back

Answer these in **your own words**, out loud, in two sentences each:

1. What are the three things every server does?
2. Why can a React app not simply hold an API key?
3. What is the difference between a `404` and a `500`, and why does that difference help you debug?
4. Why is `POST` unsafe to repeat when `PUT` is safe?

<details>
<summary>💡 Answers</summary>

**Part 1**

| # | Answer |
|---|--------|
| 1 | `GET /students` |
| 2 | `GET /students/7` |
| 3 | `POST /students` |
| 4 | `PATCH /students/7` |
| 5 | `DELETE /students/7` |
| 6 | `GET /sessions` |
| 7 | `POST /sessions/12/attendance` — creating an attendance record inside a session |
| 8 | `GET /students/7/attendance` |
| 9 | `GET /students/7/attendance?status=absent` — **filtering is a query param**, not a new path |
| 10 | `POST /auth/login` |

Question 9 is the one that separates people. A common wrong answer is `GET /students/7/missedSessions` — but that's inventing a whole new endpoint for what is really just a *filter* on data you already expose. One endpoint plus a query param scales; a new endpoint per filter does not.

**Part 2:** 1 → `201` · 2 → `404` · 3 → `400` · 4 → `403` · 5 → `401` · 6 → `500` · 7 → `204`

**Part 3** — model answers:

1. Listen for requests, decide what to do about each one, and send a response. Then repeat, forever.
2. Because everything in a React bundle is downloaded onto the user's machine, where anyone can read it in DevTools. The key must live on a server the user can't inspect.
3. A `404` means the client asked for something that isn't there — their mistake. A `500` means the server crashed handling a valid request — your mistake. It tells you instantly which side of the line to debug, which saves you from searching the wrong codebase.
4. `POST` creates a new thing each time, so calling it three times makes three things. `PUT` sets a specific resource to a specific state, so calling it three times leaves you in exactly the same place as calling it once.
</details>

---

## 📌 Class 1 Cheat Sheet

```
WHAT A SERVER DOES
  1. Listen   2. Decide   3. Respond   → repeat forever

THE REQUEST LIFECYCLE
  You → DNS → TCP → TLS → Request → Server works → Response → Render

WHY A BACKEND EXISTS
  Persistence · Secrets · Trust · Shared truth · Work browsers can't do

HTTP METHODS
  GET     read            no body      safe to repeat
  POST    create          has body     NOT safe to repeat
  PUT     replace fully   has body     safe to repeat
  PATCH   update partly   has body     safe to repeat
  DELETE  remove          no body      safe to repeat

STATUS CODES
  2xx  it worked          200 OK · 201 Created · 204 No Content
  3xx  it moved           301 Moved Permanently
  4xx  YOUR fault         400 Bad Request · 401 Unauthorized
                          403 Forbidden · 404 Not Found
                          409 Conflict · 429 Too Many Requests
  5xx  SERVER's fault     500 Internal Error · 503 Unavailable

  401 = "who are you?"    403 = "I know you, and no"

REQUEST ANATOMY
  Path         which resource      /users/7
  Path param   required ID         the 7
  Query param  optional modifier   ?sort=name&limit=10
  Headers      metadata            Authorization, Content-Type
  Body         the payload         { "name": "Ada" }   (not on GET)

NAMING RULES
  Paths are nouns, plural:  /products/7    not  /getProduct/7
  The verb lives in the method, never in the path

CURL
  curl URL                              body only
  curl -i URL                           headers + body
  curl -I URL                           headers only
  curl -s -o /dev/null -w "%{http_code}\n" URL      status only
  curl -X POST -H "Content-Type: application/json" -d '{...}' URL
  curl -H "Authorization: Bearer TOKEN" URL
  curl -L URL                           follow redirects
```

---

## ✅ Self-Check

Tick honestly. Anything unticked, revisit before Class 2.

- [ ] I can describe what a server does without using the word "server"
- [ ] I can trace a request from click to render, naming at least 6 steps
- [ ] I can give three concrete reasons a React app needs a backend
- [ ] I know why an API key in frontend code is a serious mistake
- [ ] I can choose between `POST`, `PUT` and `PATCH` for any operation
- [ ] I know what "idempotent" means and which methods are
- [ ] I can explain `401` vs `403` using an example
- [ ] I know whether a `4xx` or a `5xx` means I should check my own server code
- [ ] I can say when to use a path param vs a query param
- [ ] I called a live API from the terminal and from Postman
- [ ] My Postman collection is saved with at least 3 requests

---

## 📚 Homework

1. **Find five status codes in the wild.** Browse normally with the Network tab open. Screenshot five *different* status codes and note what caused each. At least one must be a `4xx`.

2. **Design an API for an app you use.** Pick anything — WhatsApp, Jumia, Instagram. Write 10 endpoints with methods and paths. Don't look anything up; reason it out.

3. **The curl drill.** Using only the terminal:
   - Fetch your own GitHub profile: `curl https://api.github.com/users/YOUR_USERNAME`
   - Get just the status code of any site you like
   - Get the headers of `https://schull.io` and find the `server` header
   - Make something return a `404` on purpose

4. **Read and summarise.** Look up "HTTP status codes" on MDN. Find **two** codes not covered in class and write one sentence on each.

5. **Reflection — write this down honestly.** Look at the right-hand box from Activity 1. Which of your guesses about the backend turned out to be wrong? Keep it; we'll revisit it.

---

## 🚀 Next Class

**Class 2 — Node.js Runtime & Project Setup**

Today you looked across the line. Next session you cross it: installing Node, understanding `package.json`, working with modules, reading and writing files from code, and building your first command-line tool.

**Come prepared with:**
- Node.js installed (`node --version` should print something)
- Your saved Postman collection
- Your completed lab write-up table
- Your Activity 1 paper — keep it safe

---

*Schull AI Academy · Full-Stack & AI Engineering · Class 1 of 40*
