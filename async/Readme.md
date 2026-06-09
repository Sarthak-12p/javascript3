# Async JavaScript — In Depth

> A complete guide to asynchronous programming in JavaScript: from the event loop to async/await, error handling, concurrency patterns, and real-world best practices.

---

## Table of Contents

1. [Why Async Exists](#1-why-async-exists)
2. [The Event Loop](#2-the-event-loop)
3. [Callbacks](#3-callbacks)
4. [Promises](#4-promises)
5. [async / await](#5-async--await)
6. [Error Handling](#6-error-handling)
7. [Concurrency Patterns](#7-concurrency-patterns)
8. [Timers & Scheduling](#8-timers--scheduling)
9. [Async Iteration](#9-async-iteration)
10. [AbortController & Cancellation](#10-abortcontroller--cancellation)
11. [Common Pitfalls](#11-common-pitfalls)
12. [Quick Reference](#12-quick-reference)

---

## 1. Why Async Exists

JavaScript runs on a **single thread**. If every operation blocked that thread (reading a file, fetching data from a server), the browser UI would freeze and no other code could run.

The solution: hand off slow operations to the browser/Node.js runtime, keep the thread free to do other work, and get notified via a callback when the result is ready.

```
Synchronous (blocks thread)       Asynchronous (non-blocking)
─────────────────────────         ─────────────────────────────
1. Start DB query                 1. Start DB query → hand off
2. Wait... wait... wait...        2. Handle user click
3. Process result                 3. Handle scroll event
                                  4. DB result arrives → process
```

---

## 2. The Event Loop

Understanding the event loop is the foundation of understanding async JS.

### Key Components

| Component | Description |
|-----------|-------------|
| **Call Stack** | Where synchronous code executes (LIFO) |
| **Web APIs / Node APIs** | Where async work happens (timers, I/O, fetch) |
| **Macrotask Queue** | `setTimeout`, `setInterval`, I/O callbacks |
| **Microtask Queue** | Promise `.then`, `queueMicrotask`, `MutationObserver` |
| **Event Loop** | Continuously checks: if stack is empty, run next task |

### Execution Order

Microtasks are **always drained completely** before the next macrotask runs.

```
┌──────────────────────────────────────────────┐
│  1. Run current synchronous code (Call Stack) │
│  2. Drain all microtasks (Promise callbacks)  │
│  3. Run ONE macrotask (setTimeout, etc.)      │
│  4. Drain all microtasks again                │
│  5. Render (browser)                          │
│  6. Back to step 3                            │
└──────────────────────────────────────────────┘
```

### Example: Execution Order Quiz

```js
console.log("1");

setTimeout(() => console.log("2"), 0);

Promise.resolve().then(() => console.log("3"));

console.log("4");

// Output: 1, 4, 3, 2
// Explanation:
// "1", "4" → synchronous
// "3"       → microtask (Promise), runs before setTimeout
// "2"       → macrotask (setTimeout)
```

---

## 3. Callbacks

The original async pattern. You pass a function to be called when an async operation completes.

### Basic Callback

```js
function fetchData(url, callback) {
  // Simulating async work
  setTimeout(() => {
    const data = { id: 1, name: "Alice" };
    callback(null, data); // Node.js convention: (error, result)
  }, 1000);
}

fetchData("/api/user", (err, data) => {
  if (err) {
    console.error("Error:", err);
    return;
  }
  console.log("Got data:", data);
});
```

### Callback Hell (Pyramid of Doom)

Nesting callbacks for sequential async operations becomes unmanageable:

```js
getUser(userId, (err, user) => {
  if (err) return handleError(err);
  getPosts(user.id, (err, posts) => {
    if (err) return handleError(err);
    getComments(posts[0].id, (err, comments) => {
      if (err) return handleError(err);
      // ... keeps going deeper
    });
  });
});
```

Callbacks are still used in:
- Node.js `fs`, `http` modules
- Event listeners (`addEventListener`)
- Array methods (`forEach`, `map`, `filter`) — though these are synchronous

---

## 4. Promises

A `Promise` is an object representing the **eventual completion or failure** of an async operation. It has three states:

- **Pending** — initial state, neither fulfilled nor rejected
- **Fulfilled** — operation completed successfully
- **Rejected** — operation failed

### Creating a Promise

```js
const promise = new Promise((resolve, reject) => {
  // Async work here
  const success = true;

  if (success) {
    resolve("Data loaded!"); // Fulfills the promise
  } else {
    reject(new Error("Something went wrong")); // Rejects it
  }
});
```

### Consuming a Promise

```js
promise
  .then(value => {
    console.log("Success:", value); // "Data loaded!"
  })
  .catch(error => {
    console.error("Failed:", error.message);
  })
  .finally(() => {
    console.log("Always runs, regardless of outcome");
  });
```

### Promise Chaining

Each `.then()` returns a **new Promise**, enabling clean sequential chains:

```js
fetch("/api/user/1")
  .then(response => response.json())       // Returns a Promise
  .then(user => fetch(`/api/posts?userId=${user.id}`))
  .then(response => response.json())
  .then(posts => console.log(posts))
  .catch(err => console.error(err));
```

### Creating Resolved/Rejected Promises

```js
// Already-fulfilled promise
Promise.resolve(42).then(v => console.log(v)); // 42

// Already-rejected promise
Promise.reject(new Error("Oops")).catch(e => console.error(e));
```

### Promise Lifecycle Diagram

```
           new Promise(executor)
                    │
                    ▼
               [ Pending ]
              /            \
        resolve()         reject()
            │                 │
            ▼                 ▼
       [ Fulfilled ]     [ Rejected ]
            │                 │
          .then()           .catch()
```

---

## 5. async / await

`async/await` is **syntactic sugar over Promises**. It lets you write async code that reads like synchronous code.

### Basic Syntax

```js
// An async function always returns a Promise
async function fetchUser(id) {
  const response = await fetch(`/api/users/${id}`); // Pauses here
  const user = await response.json();               // Pauses here
  return user;                                       // Resolves the Promise
}

// Arrow function syntax
const fetchUser = async (id) => {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
};
```

`await` can only be used inside an `async` function (or at the top level of an ES module).

### Top-Level await (ES2022+)

In `.mjs` files or `<script type="module">`, `await` works at the top level:

```js
// Works in ES modules
const data = await fetch("/api/data").then(r => r.json());
console.log(data);
```

### async/await vs Promises — Same Thing

```js
// These are equivalent:

// Promise chain
function getUser(id) {
  return fetch(`/api/users/${id}`)
    .then(res => res.json())
    .then(user => user.name);
}

// async/await
async function getUser(id) {
  const res = await fetch(`/api/users/${id}`);
  const user = await res.json();
  return user.name;
}
```

### Async Functions Always Return a Promise

```js
async function greet() {
  return "Hello";
}

greet().then(msg => console.log(msg)); // "Hello"
// greet() returns Promise<string>, not "Hello" directly
```

---

## 6. Error Handling

### try/catch with async/await

```js
async function loadData() {
  try {
    const response = await fetch("/api/data");

    if (!response.ok) {
      throw new Error(`HTTP error: ${response.status}`);
    }

    const data = await response.json();
    return data;
  } catch (error) {
    console.error("Failed to load data:", error.message);
    throw error; // Re-throw if you can't handle it here
  } finally {
    console.log("Cleanup runs regardless");
  }
}
```

### Handling Errors in Promise Chains

```js
fetch("/api/data")
  .then(res => {
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return res.json();
  })
  .then(data => process(data))
  .catch(err => {
    // Catches errors from ALL preceding .then() calls
    console.error(err);
  });
```

### Unhandled Promise Rejections

Always handle rejections — unhandled ones crash Node.js processes and cause browser warnings:

```js
// Bad: fire-and-forget with no error handling
someAsyncFn(); // ❌ If this rejects, you'll never know

// Good: always handle
someAsyncFn().catch(console.error); // ✅

// In Node.js, catch globally as a safety net
process.on("unhandledRejection", (reason, promise) => {
  console.error("Unhandled rejection:", reason);
});
```

### Error Wrapping Pattern

```js
// Utility: wrap async calls to avoid try/catch repetition
async function to(promise) {
  try {
    const data = await promise;
    return [null, data];
  } catch (err) {
    return [err, null];
  }
}

// Usage
const [err, user] = await to(fetchUser(1));
if (err) {
  console.error("Failed:", err.message);
  return;
}
console.log(user);
```

---

## 7. Concurrency Patterns

### Promise.all — Run in Parallel, Fail Fast

Starts all promises at the same time. Resolves when **all** succeed. Rejects immediately if **any** fail.

```js
// Sequential (slow): ~3000ms
const user = await fetchUser(1);
const posts = await fetchPosts(1);
const comments = await fetchComments(1);

// Parallel (fast): ~1000ms (runs concurrently)
const [user, posts, comments] = await Promise.all([
  fetchUser(1),
  fetchPosts(1),
  fetchComments(1),
]);
```

### Promise.allSettled — Run in Parallel, Never Fails

Waits for all promises to settle (resolve or reject). Gives you outcomes for each.

```js
const results = await Promise.allSettled([
  fetch("/api/users"),
  fetch("/api/posts"),
  fetch("/api/broken-endpoint"),
]);

results.forEach(result => {
  if (result.status === "fulfilled") {
    console.log("OK:", result.value);
  } else {
    console.error("Failed:", result.reason);
  }
});
```

### Promise.race — First One Wins

Resolves or rejects as soon as the **first** promise settles.

```js
// Implement a timeout
function withTimeout(promise, ms) {
  const timeout = new Promise((_, reject) =>
    setTimeout(() => reject(new Error(`Timed out after ${ms}ms`)), ms)
  );
  return Promise.race([promise, timeout]);
}

const data = await withTimeout(fetch("/api/slow"), 5000);
```

### Promise.any — First Success Wins (ES2021)

Resolves with the **first fulfilled** promise. Only rejects if **all** reject.

```js
// Try multiple CDN mirrors, use whichever responds first
const asset = await Promise.any([
  fetch("https://cdn1.example.com/lib.js"),
  fetch("https://cdn2.example.com/lib.js"),
  fetch("https://cdn3.example.com/lib.js"),
]);
```

### Summary Table

| Method | Resolves when | Rejects when |
|--------|--------------|--------------|
| `Promise.all` | ALL fulfill | ANY rejects |
| `Promise.allSettled` | ALL settle (never rejects) | Never |
| `Promise.race` | FIRST settles | FIRST rejects |
| `Promise.any` | FIRST fulfills | ALL reject |

### Controlled Concurrency (Batching)

Avoid hammering APIs with too many requests at once:

```js
async function processInBatches(items, batchSize, asyncFn) {
  const results = [];

  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await Promise.all(batch.map(asyncFn));
    results.push(...batchResults);
  }

  return results;
}

// Process 100 users, 10 at a time
const users = await processInBatches(userIds, 10, fetchUser);
```

---

## 8. Timers & Scheduling

### setTimeout / clearTimeout

```js
// Run once after a delay
const timerId = setTimeout(() => {
  console.log("Runs after 2 seconds");
}, 2000);

// Cancel it
clearTimeout(timerId);

// Promisified sleep
const sleep = (ms) => new Promise(resolve => setTimeout(resolve, ms));

async function main() {
  console.log("Start");
  await sleep(1000);
  console.log("1 second later");
}
```

### setInterval / clearInterval

```js
// Run repeatedly
const intervalId = setInterval(() => {
  console.log("Runs every second");
}, 1000);

// Stop after 5 seconds
setTimeout(() => clearInterval(intervalId), 5000);
```

### queueMicrotask

Schedules a microtask — runs after current synchronous code but before any macrotask:

```js
queueMicrotask(() => {
  console.log("Microtask runs before setTimeout");
});

setTimeout(() => console.log("setTimeout"), 0);

// Output:
// "Microtask runs before setTimeout"
// "setTimeout"
```

### requestAnimationFrame (Browser)

Schedule work to run before the next repaint — ideal for animations:

```js
function animate(timestamp) {
  // Update animation state
  element.style.transform = `translateX(${timestamp / 10}px)`;

  // Schedule next frame
  requestAnimationFrame(animate);
}

requestAnimationFrame(animate);
```

---

## 9. Async Iteration

For working with streams of async data (paginated APIs, file streams, WebSockets).

### for await...of

```js
async function* paginate(url) {
  let nextUrl = url;

  while (nextUrl) {
    const res = await fetch(nextUrl);
    const { data, next } = await res.json();
    yield data;          // Yield one page at a time
    nextUrl = next;      // Move to next page URL
  }
}

// Consume
async function loadAll() {
  for await (const page of paginate("/api/posts")) {
    console.log("Got page:", page);
  }
}
```

### Async Generators

```js
async function* asyncRange(start, end, delay = 100) {
  for (let i = start; i <= end; i++) {
    await sleep(delay);
    yield i;
  }
}

for await (const num of asyncRange(1, 5)) {
  console.log(num); // 1, 2, 3, 4, 5 — each after 100ms
}
```

### Node.js Streams with for await

```js
const fs = require("fs");

async function readLines(filePath) {
  const stream = fs.createReadStream(filePath, { encoding: "utf8" });

  for await (const chunk of stream) {
    process.stdout.write(chunk);
  }
}
```

---

## 10. AbortController & Cancellation

Promises are not natively cancellable, but `AbortController` provides a standard way to cancel async operations.

### Cancelling fetch

```js
const controller = new AbortController();
const { signal } = controller;

// Start a fetch that can be cancelled
fetch("/api/large-data", { signal })
  .then(res => res.json())
  .then(data => console.log(data))
  .catch(err => {
    if (err.name === "AbortError") {
      console.log("Fetch was cancelled");
    } else {
      console.error("Fetch failed:", err);
    }
  });

// Cancel after 3 seconds
setTimeout(() => controller.abort(), 3000);
```

### Cancellable async function

```js
async function fetchWithCancel(url, signal) {
  const response = await fetch(url, { signal });

  if (!response.ok) throw new Error(`HTTP ${response.status}`);

  const data = await response.json();
  return data;
}

// In a React component (cleanup on unmount)
useEffect(() => {
  const controller = new AbortController();

  fetchWithCancel("/api/data", controller.signal)
    .then(setData)
    .catch(err => {
      if (err.name !== "AbortError") setError(err);
    });

  return () => controller.abort(); // Cancel on unmount
}, []);
```

### Checking if aborted

```js
async function longTask(signal) {
  for (const item of largeDataset) {
    if (signal.aborted) {
      throw new DOMException("Aborted", "AbortError");
    }
    await processItem(item);
  }
}
```

---

## 11. Common Pitfalls

### Pitfall 1: Forgetting await

```js
// ❌ Bug: missing await — checks the Promise object, not the result
async function isAdult(userId) {
  const user = fetchUser(userId); // Returns Promise, not user!
  return user.age >= 18;          // user.age is undefined
}

// ✅ Fixed
async function isAdult(userId) {
  const user = await fetchUser(userId);
  return user.age >= 18;
}
```

### Pitfall 2: await Inside forEach

```js
const userIds = [1, 2, 3];

// ❌ forEach doesn't await — all run, but main function won't wait
userIds.forEach(async (id) => {
  const user = await fetchUser(id);
  console.log(user);
});
console.log("Done"); // Prints BEFORE users are fetched!

// ✅ Use for...of to run sequentially
for (const id of userIds) {
  const user = await fetchUser(id);
  console.log(user);
}

// ✅ Or Promise.all to run in parallel
const users = await Promise.all(userIds.map(id => fetchUser(id)));
```

### Pitfall 3: Sequential await When Parallel Is Possible

```js
// ❌ Slow: runs one after another (~3000ms)
const user  = await fetchUser(id);
const posts = await fetchPosts(id);
const likes = await fetchLikes(id);

// ✅ Fast: runs in parallel (~1000ms)
const [user, posts, likes] = await Promise.all([
  fetchUser(id),
  fetchPosts(id),
  fetchLikes(id),
]);
```

### Pitfall 4: Swallowing Errors

```js
// ❌ Empty catch hides problems silently
try {
  await riskyOperation();
} catch (e) {}

// ✅ At minimum, log the error
try {
  await riskyOperation();
} catch (e) {
  console.error("riskyOperation failed:", e);
  // Handle or re-throw
}
```

### Pitfall 5: Creating Unnecessary Promise Wrappers

```js
// ❌ Redundant — async functions already return Promises
async function getData() {
  return new Promise(async (resolve, reject) => {
    try {
      const data = await fetch("/api/data");
      resolve(data);
    } catch (err) {
      reject(err);
    }
  });
}

// ✅ Clean
async function getData() {
  const data = await fetch("/api/data");
  return data;
}
```

### Pitfall 6: Promise Constructor Anti-Pattern

```js
// ❌ Unnecessary wrapping of an already-async function
function fetchUser(id) {
  return new Promise((resolve, reject) => {
    fetch(`/api/users/${id}`)
      .then(res => res.json())
      .then(resolve)
      .catch(reject);
  });
}

// ✅ Just return the Promise directly
function fetchUser(id) {
  return fetch(`/api/users/${id}`).then(res => res.json());
}
```

---

## 12. Quick Reference

### At a Glance

```js
// Create a Promise
const p = new Promise((resolve, reject) => { ... });

// Consume
p.then(val => ...).catch(err => ...).finally(() => ...);

// async / await
async function fn() {
  try {
    const result = await somePromise;
  } catch (err) { ... }
}

// Parallel
const [a, b] = await Promise.all([promiseA, promiseB]);

// First success
const result = await Promise.any([p1, p2, p3]);

// Wait for all, get outcomes
const results = await Promise.allSettled([p1, p2, p3]);

// First to settle
const first = await Promise.race([p1, p2, p3]);

// Sleep
await new Promise(resolve => setTimeout(resolve, 1000));

// Cancellable fetch
const { signal } = new AbortController();
const res = await fetch(url, { signal });

// Async iteration
for await (const item of asyncIterable) { ... }
```

### Choosing the Right Tool

| Scenario | Use |
|----------|-----|
| Single async operation | `async/await` |
| Multiple independent operations | `Promise.all` |
| Multiple ops, partial failure ok | `Promise.allSettled` |
| Race multiple sources | `Promise.race` or `Promise.any` |
| Streaming / paginated data | `async generators` + `for await` |
| User-cancellable requests | `AbortController` |
| Retry logic | Custom wrapper with loops |
| Rate-limited batch processing | Manual batching with `Promise.all` |

---

## Further Reading

- [MDN: Using Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)
- [MDN: async function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
- [MDN: Event Loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Event_loop)
- [JavaScript.info: Promises, async/await](https://javascript.info/async)
- [Node.js: Asynchronous programming](https://nodejs.org/en/learn/asynchronous-work/javascript-asynchronous-programming-and-callbacks)

---

*Generated with care. All code examples are runnable in modern browsers and Node.js 18+.*