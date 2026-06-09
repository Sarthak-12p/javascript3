
# Promises & Fetch in JavaScript — In Depth

> A complete reference covering the Promise specification, every static and instance method, error handling strategies, the Fetch API, request/response lifecycle, headers, streaming, interceptors, and real-world patterns.

---

## Table of Contents

1. [Promises — The Core Concept](#1-promises--the-core-concept)
2. [Creating Promises](#2-creating-promises)
3. [Promise Instance Methods](#3-promise-instance-methods)
4. [Promise Static Methods](#4-promise-static-methods)
5. [Promise Chaining In Depth](#5-promise-chaining-in-depth)
6. [Promise States & Fate](#6-promise-states--fate)
7. [Microtasks & Scheduling](#7-microtasks--scheduling)
8. [Promise Anti-Patterns](#8-promise-anti-patterns)
9. [Fetch API — The Core Concept](#9-fetch-api--the-core-concept)
10. [fetch() Syntax & Options](#10-fetch-syntax--options)
11. [The Response Object](#11-the-response-object)
12. [The Request Object](#12-the-request-object)
13. [Headers](#13-headers)
14. [Sending Data](#14-sending-data)
15. [Authentication](#15-authentication)
16. [Streaming with Fetch](#16-streaming-with-fetch)
17. [Error Handling with Fetch](#17-error-handling-with-fetch)
18. [Cancelling Requests](#18-cancelling-requests)
19. [Interceptors & Wrappers](#19-interceptors--wrappers)
20. [Retry Logic](#20-retry-logic)
21. [Real-World Patterns](#21-real-world-patterns)
22. [Quick Reference](#22-quick-reference)

---

## 1. Promises — The Core Concept

A **Promise** is an object that represents the eventual result of an asynchronous operation. Think of it as a placeholder for a value that isn't available yet.

```
┌──────────────────────────────────────────────────────────┐
│                     Promise<T>                           │
│                                                          │
│  "I don't have the value yet, but I promise I will       │
│   either give you a T or tell you why I couldn't."       │
│                                                          │
│   State:  pending → fulfilled (value: T)                 │
│                   → rejected  (reason: Error)            │
└──────────────────────────────────────────────────────────┘
```

Before Promises (ES6 / 2015), async results were communicated purely through callbacks. Promises solve:

- **Callback hell** — no more pyramid nesting
- **Error propagation** — errors bubble through chains automatically
- **Composition** — `Promise.all`, `Promise.race`, etc.
- **Trust** — a Promise can only be resolved/rejected once, and always async

---

## 2. Creating Promises

### The Promise Constructor

```js
const promise = new Promise((resolve, reject) => {
  // This executor function runs synchronously and immediately

  const success = true;

  if (success) {
    resolve("The result value"); // Fulfills with a value
  } else {
    reject(new Error("Something went wrong")); // Rejects with a reason
  }
});
```

**Important rules about the executor:**
- Runs **synchronously** when `new Promise()` is called
- Should call `resolve` or `reject` exactly once (extra calls are ignored)
- Throwing inside the executor automatically rejects the promise

```js
const p = new Promise((resolve, reject) => {
  console.log("Executor runs now (sync)"); // 1st
  resolve(42);
  resolve(100); // Ignored — already resolved
});

p.then(v => console.log(v)); // 42 — not 100
console.log("After new Promise()");  // 2nd

// Output order: "Executor runs now", "After new Promise()", 42
```

### Wrapping Callback APIs

The most common use of `new Promise` is wrapping legacy callback-style APIs:

```js
// Wrapping Node.js fs.readFile
function readFile(path) {
  return new Promise((resolve, reject) => {
    fs.readFile(path, "utf8", (err, data) => {
      if (err) reject(err);
      else resolve(data);
    });
  });
}

// Wrapping a browser geolocation callback
function getPosition() {
  return new Promise((resolve, reject) => {
    navigator.geolocation.getCurrentPosition(resolve, reject);
  });
}

// Wrapping setTimeout into a sleep()
function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

### Promise.resolve & Promise.reject (Shorthand)

```js
// Creates an already-fulfilled promise
Promise.resolve(42);
Promise.resolve({ name: "Alice" });

// If you pass a thenable (or a Promise), it's adopted as-is
Promise.resolve(fetch("/api/data")); // Same as fetch("/api/data")

// Creates an already-rejected promise
Promise.reject(new Error("Immediate failure"));
```

---

## 3. Promise Instance Methods

### .then(onFulfilled, onRejected)

Registers callbacks for fulfillment and/or rejection. **Always returns a new Promise.**

```js
promise.then(
  value  => console.log("Fulfilled:", value),   // onFulfilled
  reason => console.error("Rejected:", reason)  // onRejected (optional)
);
```

The return value of the callback **becomes the resolution value** of the new Promise:

```js
Promise.resolve(2)
  .then(x => x * 10)      // returns 20
  .then(x => x + 5)       // returns 25
  .then(x => console.log(x)); // 25
```

If you return a **Promise** from `.then`, the chain waits for it:

```js
Promise.resolve(1)
  .then(x => Promise.resolve(x + 1)) // Waits for inner promise
  .then(x => console.log(x));        // 2
```

If you **throw** inside `.then`, the next `.catch` receives the error:

```js
Promise.resolve("data")
  .then(data => {
    throw new Error("Processing failed");
  })
  .catch(err => console.error(err.message)); // "Processing failed"
```

### .catch(onRejected)

Sugar for `.then(undefined, onRejected)`. Handles rejections:

```js
fetch("/api/data")
  .then(res => res.json())
  .catch(err => {
    // Catches fetch network errors AND json parse errors
    console.error("Failed:", err);
  });
```

`.catch` also **returns a new Promise** — you can recover and continue the chain:

```js
fetch("/api/primary")
  .catch(() => fetch("/api/fallback"))  // If primary fails, try fallback
  .then(res => res.json())
  .then(data => render(data));
```

### .finally(onFinally)

Runs a callback when the promise settles, regardless of outcome. **Does not receive any value**, and passes through the original value/reason:

```js
let loading = true;

fetch("/api/data")
  .then(res => res.json())
  .then(data => render(data))
  .catch(err => showError(err))
  .finally(() => {
    loading = false; // Always runs
    hideSpinner();
  });
```

```js
// finally passes through the original value
Promise.resolve(42)
  .finally(() => console.log("cleanup")) // runs
  .then(v => console.log(v));            // 42 — original value passed through
```

---

## 4. Promise Static Methods

### Promise.all(iterable)

Fulfills with an array of results when **all** promises fulfill. Rejects immediately if **any** rejects.

```js
const [user, posts, friends] = await Promise.all([
  fetchUser(1),
  fetchPosts(1),
  fetchFriends(1),
]);
```

**Short-circuit on failure:**

```js
Promise.all([
  Promise.resolve("a"),
  Promise.reject(new Error("b failed")),
  Promise.resolve("c"),
]).catch(err => console.error(err.message)); // "b failed"
// "a" and "c" results are discarded
```

**Empty array resolves immediately:**

```js
Promise.all([]).then(results => console.log(results)); // []
```

---

### Promise.allSettled(iterable)

Waits for **all** promises to settle. **Never rejects.** Returns an array of outcome objects:

```js
const results = await Promise.allSettled([
  fetchUser(1),
  fetchUser(999), // doesn't exist
  fetchUser(2),
]);

results.forEach((result, i) => {
  if (result.status === "fulfilled") {
    console.log(`[${i}] OK:`, result.value);
  } else {
    console.log(`[${i}] Error:`, result.reason.message);
  }
});
```

Each result object is either:
```js
{ status: "fulfilled", value: <resolved value> }
{ status: "rejected",  reason: <rejection reason> }
```

Use when you want **all results, partial failure acceptable** (e.g., sending notifications to multiple users).

---

### Promise.race(iterable)

Settles (fulfills OR rejects) as soon as the **first** promise settles:

```js
// Timeout pattern
function withTimeout(promise, ms) {
  const timeout = new Promise((_, reject) =>
    setTimeout(() => reject(new Error(`Timed out after ${ms}ms`)), ms)
  );
  return Promise.race([promise, timeout]);
}

try {
  const data = await withTimeout(fetch("/api/slow"), 5000);
} catch (err) {
  console.error(err.message); // "Timed out after 5000ms"
}
```

---

### Promise.any(iterable) — ES2021

Fulfills with the **first fulfilled** promise. Rejects only if **all** reject (with an `AggregateError`):

```js
// Try multiple mirrors, use whichever responds first
try {
  const response = await Promise.any([
    fetch("https://cdn1.example.com/resource"),
    fetch("https://cdn2.example.com/resource"),
    fetch("https://cdn3.example.com/resource"),
  ]);
  const data = await response.json();
} catch (err) {
  // err is AggregateError — contains all individual errors
  console.error("All mirrors failed:", err.errors);
}
```

---

### Comparison Table

| Method | Resolves when | Rejects when | Result |
|--------|---------------|--------------|--------|
| `Promise.all` | ALL fulfill | ANY rejects | Array of values |
| `Promise.allSettled` | ALL settle | Never | Array of `{status, value/reason}` |
| `Promise.race` | FIRST settles | FIRST rejects | Single value |
| `Promise.any` | FIRST fulfills | ALL reject | Single value |

---

## 5. Promise Chaining In Depth

Every `.then()` and `.catch()` returns a **new, distinct Promise**. This enables chaining.

### Value Transformation

```js
fetch("/api/user/1")
  .then(res  => res.json())         // Promise<Object>
  .then(user => user.profile)       // Promise<Profile>
  .then(profile => profile.avatarUrl) // Promise<string>
  .then(url  => preloadImage(url))  // Promise<HTMLImageElement>
  .then(img  => document.body.appendChild(img));
```

### Branching Chains

```js
const userPromise = fetch("/api/user/1").then(r => r.json());

// Two independent chains from the same promise
userPromise.then(user => updateProfile(user));
userPromise.then(user => logActivity(user));
// Both receive the same resolved user object
```

### Error Recovery in a Chain

```js
fetch("/api/preferred-data")
  .catch(() => fetch("/api/fallback-data"))  // Recover with fallback
  .then(res => res.json())                   // Continues after recovery
  .then(data => render(data))
  .catch(err => showErrorPage(err));         // Final safety net
```

### The "Diamond" Pattern

```js
const base = fetchBaseData();             // One source

const enrichedA = base.then(addTypeA);   // Branch A
const enrichedB = base.then(addTypeB);   // Branch B

// Merge results
Promise.all([enrichedA, enrichedB])
  .then(([a, b]) => merge(a, b))
  .then(result => display(result));
```

---

## 6. Promise States & Fate

### States

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   pending ──→ fulfilled  (resolved with a value)           │
│           └─→ rejected   (rejected with a reason)          │
│                                                             │
│   Once settled (fulfilled or rejected), state is FINAL.    │
│   The promise is now "resolved" — value never changes.     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Fate: Resolved vs. Settled

These terms are often confused:

- **Settled** — the promise is either fulfilled or rejected (no longer pending)
- **Resolved** — the promise's fate is determined; it may be resolved to another promise (still pending) or to a value (fulfilled)

```js
// p1 is "resolved" to p2, but still "pending" while p2 is pending
const p2 = new Promise(resolve => setTimeout(resolve, 1000));
const p1 = Promise.resolve(p2); // p1 follows p2's state

// p1 is neither fulfilled nor rejected until p2 settles
```

### Thenable Assimilation

If you resolve a Promise with another thenable (any object with a `.then` method), the outer promise **follows** the inner one:

```js
const p = new Promise(resolve => {
  resolve({
    then(onFulfilled) {
      onFulfilled(42); // Custom thenable
    }
  });
});

p.then(v => console.log(v)); // 42
```

---

## 7. Microtasks & Scheduling

Promise callbacks (`.then`, `.catch`, `.finally`) are always scheduled as **microtasks** — they run after the current synchronous task but before any macrotask (setTimeout, I/O).

```js
console.log("A");

setTimeout(() => console.log("B"), 0); // macrotask

Promise.resolve()
  .then(() => console.log("C"))  // microtask
  .then(() => console.log("D")); // microtask (queued after C runs)

console.log("E");

// Output: A, E, C, D, B
```

### Why This Matters

```js
let value = null;
Promise.resolve(42).then(v => (value = v));

// value is STILL null here — .then runs after current code
console.log(value); // null

// value will be 42 in a later microtask
setTimeout(() => console.log(value), 0); // 42
```

---

## 8. Promise Anti-Patterns

### ❌ Explicit Promise Construction Anti-Pattern

Wrapping an already-promise-returning function in `new Promise` is redundant and hides errors:

```js
// ❌ Bad
function getUser(id) {
  return new Promise((resolve, reject) => {
    fetch(`/api/users/${id}`)
      .then(res => res.json())
      .then(user => resolve(user))
      .catch(err => reject(err)); // Error double-wrapping
  });
}

// ✅ Good — return the promise directly
function getUser(id) {
  return fetch(`/api/users/${id}`).then(res => res.json());
}
```

### ❌ Nested .then Instead of Chaining

```js
// ❌ Bad — creates callback-like pyramid
fetch("/api/user")
  .then(res => {
    res.json().then(user => {         // Nested promise!
      fetch(`/api/posts/${user.id}`)
        .then(r => r.json())
        .then(posts => console.log(posts));
    });
  });

// ✅ Good — flat chain
fetch("/api/user")
  .then(res => res.json())
  .then(user => fetch(`/api/posts/${user.id}`))
  .then(res => res.json())
  .then(posts => console.log(posts));
```

### ❌ Forgetting to Return in .then

```js
// ❌ Bad — next .then gets undefined
fetch("/api/data")
  .then(res => {
    res.json(); // Missing return!
  })
  .then(data => console.log(data)); // undefined

// ✅ Good
fetch("/api/data")
  .then(res => res.json())          // Returned
  .then(data => console.log(data)); // Correct data
```

### ❌ Broken Promise Chain (Missing .catch)

```js
// ❌ Bad — unhandled rejection if anything fails
fetchUser().then(processUser).then(saveUser);

// ✅ Good
fetchUser().then(processUser).then(saveUser).catch(handleError);
```

### ❌ Promise in a Loop Without Parallelism

```js
// ❌ Bad — runs sequentially, slow
for (const id of ids) {
  const user = await fetchUser(id); // Waits for each one
}

// ✅ Good — runs in parallel
const users = await Promise.all(ids.map(id => fetchUser(id)));
```

---

## 9. Fetch API — The Core Concept

`fetch()` is the modern browser (and Node 18+) API for making HTTP requests. It returns a Promise that resolves to a `Response` object.

```
fetch(url, options)
       │
       └──→ Promise<Response>
                    │
                    ├── response.json()    → Promise<any>
                    ├── response.text()    → Promise<string>
                    ├── response.blob()    → Promise<Blob>
                    ├── response.arrayBuffer() → Promise<ArrayBuffer>
                    └── response.formData()    → Promise<FormData>
```

**Critical behavior: fetch only rejects on network failure.** HTTP error codes (404, 500) do NOT cause rejection — you must check `response.ok` manually.

---

## 10. fetch() Syntax & Options

### Basic GET

```js
const response = await fetch("https://api.example.com/users");
const users = await response.json();
```

### Full Options Reference

```js
const response = await fetch(url, {
  // HTTP method
  method: "GET",         // GET | POST | PUT | PATCH | DELETE | HEAD | OPTIONS

  // Request headers
  headers: {
    "Content-Type": "application/json",
    "Authorization": "Bearer <token>",
    "Accept": "application/json",
  },

  // Request body (not allowed for GET/HEAD)
  body: JSON.stringify({ name: "Alice" }),

  // Caching behavior
  mode: "cors",                 // cors | no-cors | same-origin | navigate
  cache: "no-cache",            // default | no-store | reload | no-cache | force-cache | only-if-cached
  credentials: "same-origin",   // omit | same-origin | include
  redirect: "follow",           // follow | error | manual
  referrerPolicy: "no-referrer",
  integrity: "sha256-abc123...", // Subresource Integrity hash

  // Cancel signal
  signal: abortController.signal,

  // Custom request priority (Chrome)
  priority: "high",             // high | low | auto
});
```

### Common Method Examples

```js
// POST with JSON
fetch("/api/users", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: "Alice", email: "alice@example.com" }),
});

// PUT (full update)
fetch("/api/users/1", {
  method: "PUT",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: "Alice Updated" }),
});

// PATCH (partial update)
fetch("/api/users/1", {
  method: "PATCH",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: "New Name" }),
});

// DELETE
fetch("/api/users/1", { method: "DELETE" });

// HEAD (get headers only, no body)
fetch("/api/users/1", { method: "HEAD" });
```

---

## 11. The Response Object

When a fetch resolves, you receive a `Response` object:

### Properties

```js
const response = await fetch("/api/data");

response.ok;         // true if status 200–299
response.status;     // HTTP status code (200, 404, 500, etc.)
response.statusText; // "OK", "Not Found", etc.
response.url;        // Final URL (after redirects)
response.redirected; // true if request was redirected
response.type;       // "basic" | "cors" | "opaque" | "error"
response.headers;    // Headers object
response.bodyUsed;   // true once body has been read
```

### Reading the Body

The body can only be read **once** — it's a stream. Call the appropriate method:

```js
response.json()        // Parses body as JSON   → Promise<any>
response.text()        // Reads body as text     → Promise<string>
response.blob()        // Reads body as Blob     → Promise<Blob>
response.arrayBuffer() // Raw binary             → Promise<ArrayBuffer>
response.formData()    // Parses as FormData     → Promise<FormData>
```

```js
// ❌ Bad — body already consumed
const res = await fetch("/api/data");
const text = await res.text();
const json = await res.json(); // TypeError: body already used

// ✅ If you need to read the body twice, clone it
const res = await fetch("/api/data");
const clone = res.clone();
const text = await res.text();
const json = await clone.json();
```

### Checking Status

```js
async function safeFetch(url) {
  const response = await fetch(url);

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}: ${response.statusText}`);
  }

  return response.json();
}
```

---

## 12. The Request Object

You can create a `Request` object separately and pass it to `fetch()`:

```js
const request = new Request("/api/users", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: "Alice" }),
});

const response = await fetch(request);
```

This is useful for:
- Building request factories
- Cloning requests in Service Workers
- Inspecting/modifying requests before sending

```js
// Clone a request (body can only be read once)
const original = new Request("/api/data", { method: "POST", body: "hello" });
const clone = original.clone();

// Both can now be used independently
await fetch(original);
await fetch(clone);
```

---

## 13. Headers

### Creating Headers

```js
// Object literal (simplest)
const headers = {
  "Content-Type": "application/json",
  "X-Custom-Header": "value",
};

// Headers constructor (more powerful)
const headers = new Headers({
  "Content-Type": "application/json",
});

headers.append("Authorization", "Bearer token123");
headers.set("Accept", "application/json");
headers.get("content-type"); // "application/json" (case-insensitive)
headers.has("Authorization"); // true
headers.delete("X-Custom-Header");

// Iterate headers
for (const [name, value] of headers) {
  console.log(`${name}: ${value}`);
}
```

### Reading Response Headers

```js
const response = await fetch("/api/data");

const contentType = response.headers.get("content-type");
const rateLimit = response.headers.get("x-ratelimit-remaining");
const totalCount = response.headers.get("x-total-count");

// Log all headers
response.headers.forEach((value, name) => {
  console.log(`${name}: ${value}`);
});
```

### Common Request Headers

```js
{
  // Body format
  "Content-Type": "application/json",
  "Content-Type": "application/x-www-form-urlencoded",
  "Content-Type": "multipart/form-data",           // Let browser set this (with boundary)

  // Auth
  "Authorization": "Bearer <jwt>",
  "Authorization": "Basic <base64(user:pass)>",
  "X-API-Key": "<key>",

  // Caching
  "Cache-Control": "no-cache",
  "If-None-Match": "<etag>",
  "If-Modified-Since": "<date>",

  // Content negotiation
  "Accept": "application/json",
  "Accept-Language": "en-US",
  "Accept-Encoding": "gzip, deflate",

  // Custom
  "X-Request-ID": crypto.randomUUID(),
  "X-Client-Version": "1.0.0",
}
```

---

## 14. Sending Data

### JSON

```js
async function createUser(userData) {
  const response = await fetch("/api/users", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(userData),
  });

  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  return response.json();
}

const user = await createUser({ name: "Alice", age: 30 });
```

### Form Data (Multipart — for file uploads)

```js
const formData = new FormData();
formData.append("name", "Alice");
formData.append("avatar", fileInput.files[0]); // File object
formData.append("tags", "admin");
formData.append("tags", "user"); // Multiple values for same key

// ✅ Do NOT set Content-Type manually — browser sets it with boundary
const response = await fetch("/api/users", {
  method: "POST",
  body: formData,
});
```

### URL-encoded Form Data

```js
const params = new URLSearchParams({
  name: "Alice",
  email: "alice@example.com",
});

const response = await fetch("/api/users", {
  method: "POST",
  headers: { "Content-Type": "application/x-www-form-urlencoded" },
  body: params.toString(), // "name=Alice&email=alice%40example.com"
});
```

### Uploading Binary Data

```js
// Upload raw file as binary
const file = fileInput.files[0];

const response = await fetch("/api/upload", {
  method: "PUT",
  headers: { "Content-Type": file.type },
  body: file,
});
```

### Query Parameters

```js
// Manual string construction
fetch(`/api/users?page=2&limit=20&sort=name`);

// Using URLSearchParams (handles encoding)
const params = new URLSearchParams({
  page: 2,
  limit: 20,
  sort: "name",
  filter: "active users", // Auto-encoded: "active+users"
});

fetch(`/api/users?${params}`);
```

---

## 15. Authentication

### Bearer Token (JWT)

```js
async function authFetch(url, options = {}) {
  const token = localStorage.getItem("accessToken");

  return fetch(url, {
    ...options,
    headers: {
      "Authorization": `Bearer ${token}`,
      "Content-Type": "application/json",
      ...options.headers,
    },
  });
}

const data = await authFetch("/api/profile").then(r => r.json());
```

### Cookies (Session-based Auth)

```js
// credentials: "include" sends cookies cross-origin
fetch("https://api.example.com/profile", {
  credentials: "include",
});

// credentials: "same-origin" (default) only sends cookies to same origin
fetch("/api/profile", {
  credentials: "same-origin",
});
```

### Basic Auth

```js
const credentials = btoa("username:password"); // Base64 encode

fetch("/api/admin", {
  headers: {
    "Authorization": `Basic ${credentials}`,
  },
});
```

### API Key

```js
fetch(`/api/data?apikey=${API_KEY}`); // In query param

fetch("/api/data", {
  headers: { "X-API-Key": API_KEY }, // In header (more secure)
});
```

---

## 16. Streaming with Fetch

For large responses, you can read the body incrementally instead of buffering it all.

### Reading a Stream

```js
async function downloadWithProgress(url, onProgress) {
  const response = await fetch(url);
  const contentLength = response.headers.get("content-length");
  const total = parseInt(contentLength, 10);

  const reader = response.body.getReader();
  const chunks = [];
  let received = 0;

  while (true) {
    const { done, value } = await reader.read();

    if (done) break;

    chunks.push(value);
    received += value.length;

    if (total) {
      onProgress(Math.round((received / total) * 100));
    }
  }

  // Combine chunks into a single Uint8Array
  const allChunks = new Uint8Array(received);
  let position = 0;
  for (const chunk of chunks) {
    allChunks.set(chunk, position);
    position += chunk.length;
  }

  return new TextDecoder().decode(allChunks);
}
```

### Server-Sent Events (SSE) / AI Streaming

```js
async function streamCompletion(prompt) {
  const response = await fetch("/api/ai/stream", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ prompt }),
  });

  const reader = response.body.getReader();
  const decoder = new TextDecoder();
  let buffer = "";

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    buffer += decoder.decode(value, { stream: true });

    // Parse SSE lines
    const lines = buffer.split("\n");
    buffer = lines.pop(); // Keep incomplete last line

    for (const line of lines) {
      if (line.startsWith("data: ")) {
        const data = line.slice(6);
        if (data === "[DONE]") return;
        const parsed = JSON.parse(data);
        process(parsed);
      }
    }
  }
}
```

### Streaming Upload

```js
// Stream a large file upload with TransformStream
async function uploadStream(file) {
  const { readable, writable } = new TransformStream();

  // Write to writable stream in background
  const writer = writable.getWriter();
  writer.write(await file.arrayBuffer());
  writer.close();

  return fetch("/api/upload", {
    method: "POST",
    body: readable,
    duplex: "half", // Required for streaming request body
  });
}
```

---

## 17. Error Handling with Fetch

### The Two Types of Fetch Errors

```
┌──────────────────────────────────────────────────────────────┐
│  Type 1: Network Error                                       │
│  • No internet connection                                    │
│  • DNS resolution failure                                    │
│  • CORS violation                                            │
│  • Request aborted                                           │
│  → fetch() Promise REJECTS with TypeError                    │
├──────────────────────────────────────────────────────────────┤
│  Type 2: HTTP Error                                          │
│  • 404 Not Found                                             │
│  • 401 Unauthorized                                          │
│  • 500 Internal Server Error                                 │
│  → fetch() Promise RESOLVES — you must check response.ok    │
└──────────────────────────────────────────────────────────────┘
```

### Robust Error Handling

```js
class HttpError extends Error {
  constructor(response) {
    super(`HTTP ${response.status}: ${response.statusText}`);
    this.name = "HttpError";
    this.status = response.status;
    this.response = response;
  }
}

async function apiFetch(url, options = {}) {
  let response;

  try {
    response = await fetch(url, options);
  } catch (networkError) {
    // Network failure, CORS, DNS, etc.
    throw new Error(`Network error: ${networkError.message}`);
  }

  if (!response.ok) {
    // Try to get error details from response body
    let errorBody;
    try {
      errorBody = await response.json();
    } catch {
      errorBody = await response.text();
    }

    const error = new HttpError(response);
    error.body = errorBody;
    throw error;
  }

  // Handle empty responses (204 No Content, 304)
  const contentType = response.headers.get("content-type") || "";
  if (!contentType.includes("application/json")) {
    return response.text();
  }

  return response.json();
}
```

### Handling Specific Status Codes

```js
async function getUser(id) {
  try {
    return await apiFetch(`/api/users/${id}`);
  } catch (err) {
    if (err.status === 404) {
      return null; // User doesn't exist
    }
    if (err.status === 401) {
      redirectToLogin();
      return;
    }
    if (err.status === 429) {
      const retryAfter = err.response.headers.get("retry-after");
      throw new Error(`Rate limited. Retry after ${retryAfter}s`);
    }
    throw err; // Re-throw unexpected errors
  }
}
```

---

## 18. Cancelling Requests

### AbortController

```js
const controller = new AbortController();

// Start the request
const fetchPromise = fetch("/api/data", {
  signal: controller.signal,
});

// Cancel after 5 seconds
const timeout = setTimeout(() => controller.abort(), 5000);

try {
  const response = await fetchPromise;
  clearTimeout(timeout);
  return response.json();
} catch (err) {
  if (err.name === "AbortError") {
    console.log("Request was cancelled");
  } else {
    throw err;
  }
}
```

### Timeout with AbortSignal.timeout() (Modern)

```js
// Cleaner timeout syntax (Node 17.3+, modern browsers)
const response = await fetch("/api/data", {
  signal: AbortSignal.timeout(5000), // Auto-abort after 5s
});
```

### Cancelling Previous Requests (Search-as-you-type)

```js
let currentController = null;

async function search(query) {
  // Cancel any in-flight request
  if (currentController) {
    currentController.abort();
  }

  currentController = new AbortController();

  try {
    const response = await fetch(`/api/search?q=${query}`, {
      signal: currentController.signal,
    });
    const results = await response.json();
    renderResults(results);
  } catch (err) {
    if (err.name !== "AbortError") {
      console.error("Search failed:", err);
    }
  }
}

// Hook up to input
searchInput.addEventListener("input", e => search(e.target.value));
```

---

## 19. Interceptors & Wrappers

### A Complete fetch Wrapper

```js
class ApiClient {
  constructor(baseUrl, defaultOptions = {}) {
    this.baseUrl = baseUrl;
    this.defaultOptions = defaultOptions;
    this.interceptors = { request: [], response: [] };
  }

  // Add request interceptor
  useRequest(fn) {
    this.interceptors.request.push(fn);
    return this;
  }

  // Add response interceptor
  useResponse(fn) {
    this.interceptors.response.push(fn);
    return this;
  }

  async request(path, options = {}) {
    let mergedOptions = {
      ...this.defaultOptions,
      ...options,
      headers: {
        "Content-Type": "application/json",
        ...this.defaultOptions.headers,
        ...options.headers,
      },
    };

    // Run request interceptors
    for (const interceptor of this.interceptors.request) {
      mergedOptions = await interceptor(mergedOptions);
    }

    let response = await fetch(`${this.baseUrl}${path}`, mergedOptions);

    // Run response interceptors
    for (const interceptor of this.interceptors.response) {
      response = await interceptor(response);
    }

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }

    return response.json();
  }

  get(path, options)         { return this.request(path, { ...options, method: "GET" }); }
  post(path, body, options)  { return this.request(path, { ...options, method: "POST",  body: JSON.stringify(body) }); }
  put(path, body, options)   { return this.request(path, { ...options, method: "PUT",   body: JSON.stringify(body) }); }
  patch(path, body, options) { return this.request(path, { ...options, method: "PATCH", body: JSON.stringify(body) }); }
  delete(path, options)      { return this.request(path, { ...options, method: "DELETE" }); }
}

// Usage
const api = new ApiClient("https://api.example.com");

// Add auth interceptor
api.useRequest(options => ({
  ...options,
  headers: {
    ...options.headers,
    Authorization: `Bearer ${getToken()}`,
  },
}));

// Add logging interceptor
api.useResponse(response => {
  console.log(`[${response.status}] ${response.url}`);
  return response;
});

const user = await api.get("/users/1");
await api.post("/users", { name: "Alice" });
```

---

## 20. Retry Logic

### Exponential Backoff with Retry

```js
async function fetchWithRetry(url, options = {}, retries = 3) {
  const { retryDelay = 300, retryOn = [429, 503, 504], ...fetchOptions } = options;

  for (let attempt = 0; attempt <= retries; attempt++) {
    try {
      const response = await fetch(url, fetchOptions);

      // Retry on specific status codes
      if (retryOn.includes(response.status) && attempt < retries) {
        const delay = retryDelay * Math.pow(2, attempt); // Exponential backoff
        const jitter = Math.random() * 100;              // Add jitter to avoid thundering herd
        console.warn(`Attempt ${attempt + 1} failed (${response.status}). Retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay + jitter));
        continue;
      }

      return response;
    } catch (networkError) {
      if (attempt === retries) throw networkError;

      const delay = retryDelay * Math.pow(2, attempt);
      console.warn(`Network error on attempt ${attempt + 1}. Retrying in ${delay}ms...`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// Usage
const response = await fetchWithRetry("/api/data", { retries: 3, retryDelay: 500 });
```

---

## 21. Real-World Patterns

### Paginated API — Load All Pages

```js
async function* paginate(baseUrl) {
  let url = baseUrl;

  while (url) {
    const response = await fetch(url);
    const { data, links } = await response.json();

    yield data;

    url = links?.next || null; // Move to next page
  }
}

async function loadAllUsers() {
  const allUsers = [];
  for await (const page of paginate("/api/users?limit=100")) {
    allUsers.push(...page);
  }
  return allUsers;
}
```

### Request Deduplication

```js
const cache = new Map();

async function deduplicatedFetch(url) {
  if (cache.has(url)) {
    return cache.get(url); // Return the same in-flight Promise
  }

  const promise = fetch(url)
    .then(r => r.json())
    .finally(() => cache.delete(url)); // Clean up after resolving

  cache.set(url, promise);
  return promise;
}

// Multiple calls to the same URL share one request
const [a, b, c] = await Promise.all([
  deduplicatedFetch("/api/config"),
  deduplicatedFetch("/api/config"), // Same URL — reuses the first request
  deduplicatedFetch("/api/config"),
]);
```

### Caching with Expiry

```js
const responseCache = new Map();

async function cachedFetch(url, ttlMs = 60_000) {
  const cached = responseCache.get(url);

  if (cached && Date.now() < cached.expiresAt) {
    return cached.data;
  }

  const response = await fetch(url);
  const data = await response.json();

  responseCache.set(url, {
    data,
    expiresAt: Date.now() + ttlMs,
  });

  return data;
}
```

### Polling Until Condition

```js
async function pollUntil(url, isComplete, intervalMs = 2000, maxAttempts = 30) {
  for (let i = 0; i < maxAttempts; i++) {
    const response = await fetch(url);
    const data = await response.json();

    if (isComplete(data)) return data;

    await new Promise(resolve => setTimeout(resolve, intervalMs));
  }

  throw new Error("Polling timed out");
}

// Poll a job status until done
const result = await pollUntil(
  "/api/jobs/123",
  job => job.status === "completed",
  3000,
  20
);
```

### Parallel Requests with Concurrency Limit

```js
async function fetchWithConcurrency(urls, limit = 5) {
  const results = [];
  const queue = [...urls];

  async function worker() {
    while (queue.length) {
      const url = queue.shift();
      const data = await fetch(url).then(r => r.json());
      results.push(data);
    }
  }

  // Run `limit` workers in parallel
  await Promise.all(
    Array.from({ length: Math.min(limit, urls.length) }, worker)
  );

  return results;
}

const results = await fetchWithConcurrency(hundredsOfUrls, 5);
```

---

## 22. Quick Reference

### Promise Methods

```js
// Create
new Promise((resolve, reject) => { ... })
Promise.resolve(value)
Promise.reject(reason)

// Instance
promise.then(onFulfilled, onRejected) → Promise
promise.catch(onRejected)             → Promise
promise.finally(onFinally)            → Promise

// Static — concurrency
Promise.all([p1, p2])         // All succeed or first failure
Promise.allSettled([p1, p2])  // All outcomes
Promise.race([p1, p2])        // First to settle
Promise.any([p1, p2])         // First to succeed
```

### fetch Cheat Sheet

```js
// GET
const data = await fetch(url).then(r => r.json());

// POST JSON
const res = await fetch(url, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify(payload),
});

// Check status
if (!res.ok) throw new Error(`HTTP ${res.status}`);

// With timeout
const res = await fetch(url, { signal: AbortSignal.timeout(5000) });

// With auth
const res = await fetch(url, {
  headers: { Authorization: `Bearer ${token}` },
  credentials: "include",
});

// Upload file
const form = new FormData();
form.append("file", fileInput.files[0]);
const res = await fetch(url, { method: "POST", body: form });

// Cancel request
const ctrl = new AbortController();
const res = await fetch(url, { signal: ctrl.signal });
ctrl.abort(); // Cancel
```

### Choosing the Right Approach

| Need | Use |
|------|-----|
| Single request | `fetch` + `async/await` |
| Multiple independent requests | `Promise.all([fetch(), fetch()])` |
| Request that can fail partially | `Promise.allSettled` |
| Timeout a request | `AbortSignal.timeout(ms)` |
| Cancel previous request | `AbortController` |
| Upload files | `FormData` body |
| Stream large responses | `response.body.getReader()` |
| Retry on failure | Custom retry wrapper |
| Deduplicate requests | In-flight Promise cache |
| Paginated data | Async generator + `for await` |
| Auth on every request | Fetch wrapper with interceptor |

---

## Further Reading

- [MDN: Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [MDN: fetch()](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)
- [MDN: Response](https://developer.mozilla.org/en-US/docs/Web/API/Response)
- [MDN: AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)
- [MDN: Streams API](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API)
- [WHATWG Fetch Specification](https://fetch.spec.whatwg.org/)
- [JavaScript.info: Promises](https://javascript.info/promise-basics)
- [JavaScript.info: Fetch](https://javascript.info/fetch)

---

*All examples use modern JavaScript (ES2021+) and run in Node.js 18+ or any modern browser.*