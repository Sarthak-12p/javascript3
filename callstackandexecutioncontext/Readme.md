# ⚙️ JavaScript Call Stack & Execution Context — The Complete Guide

> How JavaScript actually runs your code, step by step.

---

## Table of Contents

1. [The Big Picture](#1-the-big-picture)
2. [Execution Context — What It Is](#2-execution-context--what-it-is)
3. [Types of Execution Context](#3-types-of-execution-context)
4. [Two Phases of Every Execution Context](#4-two-phases-of-every-execution-context)
5. [The Call Stack — What It Is](#5-the-call-stack--what-it-is)
6. [Call Stack — Step-by-Step Walkthrough](#6-call-stack--step-by-step-walkthrough)
7. [The Scope Chain (Inside Execution Contexts)](#7-the-scope-chain-inside-execution-contexts)
8. [Hoisting — Explained by the Creation Phase](#8-hoisting--explained-by-the-creation-phase)
9. [`this` Binding in Execution Contexts](#9-this-binding-in-execution-contexts)
10. [Closures — When Execution Contexts Live On](#10-closures--when-execution-contexts-live-on)
11. [The Event Loop & Call Stack Together](#11-the-event-loop--call-stack-together)
12. [Stack Overflow — What Causes It](#12-stack-overflow--what-causes-it)
13. [Async/Await & the Call Stack](#13-asyncawait--the-call-stack)
14. [Debugging the Call Stack](#14-debugging-the-call-stack)
15. [Quick Reference Cheatsheet](#15-quick-reference-cheatsheet)

---

## 1. The Big Picture

Before a single line of your code runs, JavaScript does a lot of invisible setup work. Understanding this setup is the key to understanding hoisting, closures, `this`, and scope — all at once.

Here's the core mental model:

```
When JS runs your code:

1. It creates an EXECUTION CONTEXT  ← "What do I need to run this code?"
2. It pushes it onto the CALL STACK ← "I'm working on this now"
3. It runs the code inside           ← "Executing..."
4. It pops it off the stack          ← "Done, cleaning up"
```

Everything else — hoisting, closures, scope, `this` — falls out of these four steps.

---

## 2. Execution Context — What It Is

An **Execution Context (EC)** is an internal data structure that JavaScript creates every time it needs to run some code. It holds:

- All the variables and functions accessible in that scope
- The value of `this`
- A reference to the outer (parent) scope

Think of it as a **"bubble" of everything a piece of code needs to execute**.

```
┌─────────────────────────────────────────┐
│         EXECUTION CONTEXT               │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │  Variable Environment           │    │
│  │  (var, function declarations)   │    │
│  └─────────────────────────────────┘    │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │  Lexical Environment            │    │
│  │  (let, const + outer env ref)   │    │
│  └─────────────────────────────────┘    │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │  this Binding                   │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
```

---

## 3. Types of Execution Context

There are three types — you'll mostly deal with the first two.

### Global Execution Context (GEC)

Created once when your script loads. There is only ever **one** GEC.

```js
// This code lives in the Global Execution Context
const appName = "MyApp"; // stored in GEC's variable environment
let version   = "1.0";

function init() { ... } // stored in GEC as a function declaration
```

In the GEC:
- `this` = `window` (browser) or `global` (Node.js)
- Outer environment reference = `null` (it has no parent)
- All `var` declarations and function declarations are stored here

### Function Execution Context (FEC)

Created **every time a function is called**. Each call creates a brand new, independent EC.

```js
function greet(name) {
  // A new FEC is created each time greet() is called
  const message = `Hello, ${name}!`; // lives in THIS call's FEC
  return message;
}

greet("Alice"); // FEC #1 created → runs → destroyed
greet("Bob");   // FEC #2 created → runs → destroyed (independent of #1)
```

In each FEC:
- `this` depends on how the function was called
- Parameters are stored as local variables
- Outer environment reference = the EC where the function was **written** (lexical)

### Eval Execution Context

Created when code is run via `eval()`. Avoid in production — security risk.

```js
eval("var x = 5"); // creates its own EC — don't use this
```

---

## 4. Two Phases of Every Execution Context

Every EC goes through exactly two phases before and during execution.

### Phase 1 — Creation Phase

Before any code runs, JavaScript scans the scope and sets up memory.

```
CREATION PHASE — what happens:

1. Variable Environment is created
   - var declarations → hoisted, set to undefined
   - function declarations → hoisted with full function body
   - let / const → hoisted but put in TDZ (not accessible yet)

2. Lexical Environment is created
   - let / const stored here (in TDZ)
   - Reference to outer scope set up

3. this binding is determined
   - Global EC → window / globalThis
   - Function EC → depends on how it's called
```

### Phase 2 — Execution Phase

JavaScript executes your code line by line. Variables get their actual values.

```js
// What the JS engine "sees" during each phase:

// ── CREATION PHASE ──────────────────────────
// greet    → stored as full function ✅
// score    → undefined (var hoisted) ⚠️
// username → TDZ — unusable ❌

// ── EXECUTION PHASE (line by line) ──────────
greet("Alice");     // ✅ works — greet is already in memory from creation phase
console.log(score); // ⚠️ undefined — score exists but hasn't been assigned yet
console.log(username); // ❌ ReferenceError — still in TDZ

var score = 100;         // NOW score gets assigned 100
let username = "Alice";  // NOW username exits TDZ

function greet(name) {
  return `Hello, ${name}`;
}
```

---

## 5. The Call Stack — What It Is

The **Call Stack** is a data structure that keeps track of which execution context is currently running.

It works like a stack of plates:
- When a function is called → a new EC is **pushed** (added to the top)
- When a function returns → its EC is **popped** (removed from the top)
- The function on **top** is always the one currently executing

```
Call Stack — visual model:

       ┌──────────────┐  ← currently executing (top)
       │   c()  EC    │
       ├──────────────┤
       │   b()  EC    │  ← paused, waiting for c() to return
       ├──────────────┤
       │   a()  EC    │  ← paused, waiting for b() to return
       ├──────────────┤
       │  Global  EC  │  ← always at the bottom
       └──────────────┘
```

Key properties:
- **LIFO** — Last In, First Out
- **Single-threaded** — only one function runs at a time
- **Synchronous by default** — functions complete before the next one starts

---

## 6. Call Stack — Step-by-Step Walkthrough

Let's trace exactly what happens with this code:

```js
function c() {
  return 1 + 1;
}

function b() {
  return c();
}

function a() {
  return b();
}

a(); // ← start here
```

### Step 0 — Script loads

```
Stack:   [ Global EC ]
Action:  GEC created. a, b, c are stored in memory (creation phase).
         a() is called.
```

### Step 1 — a() is called

```
Stack:   [ Global EC → a() EC ]
Action:  New FEC for a() pushed onto stack.
         a() starts executing.
         a() calls b() — so a() is PAUSED.
```

### Step 2 — b() is called

```
Stack:   [ Global EC → a() EC → b() EC ]
Action:  New FEC for b() pushed.
         b() starts executing.
         b() calls c() — so b() is PAUSED.
```

### Step 3 — c() is called

```
Stack:   [ Global EC → a() EC → b() EC → c() EC ]
Action:  New FEC for c() pushed. Now 4 frames deep.
         c() evaluates 1 + 1 = 2.
         c() has nothing left to do — it RETURNS 2.
```

### Step 4 — c() returns

```
Stack:   [ Global EC → a() EC → b() EC ]
Action:  c()'s EC is DESTROYED and popped.
         b() RESUMES — receives the value 2 from c().
         b() returns 2.
```

### Step 5 — b() returns

```
Stack:   [ Global EC → a() EC ]
Action:  b()'s EC is DESTROYED and popped.
         a() RESUMES — receives 2 from b().
         a() returns 2.
```

### Step 6 — a() returns

```
Stack:   [ Global EC ]
Action:  a()'s EC is DESTROYED and popped.
         Control returns to the global level.
         The call is complete — result is 2.
```

---

## 7. The Scope Chain (Inside Execution Contexts)

Every Execution Context has a reference to its **outer environment**. When a variable isn't found in the current EC, JavaScript walks up this chain.

```
┌─────────────────────────────┐
│     city() EC               │
│     town = "Mumbai"         │
│     outer → country() EC   ──────────┐
└─────────────────────────────┘         │
                                        ↓
                              ┌─────────────────────────────┐
                              │     country() EC            │
                              │     nation = "India"        │
                              │     outer → Global EC      ──────────┐
                              └─────────────────────────────┘         │
                                                                       ↓
                                                             ┌─────────────────────────────┐
                                                             │     Global EC               │
                                                             │     planet = "Earth"        │
                                                             │     outer → null            │
                                                             └─────────────────────────────┘
```

```js
const planet = "Earth"; // Global EC

function country() {
  const nation = "India"; // country EC

  function city() {
    const town = "Mumbai"; // city EC

    // Looking up `nation`:
    // 1. Check city EC     → not found
    // 2. Check country EC  → FOUND ✅ "India"
    console.log(nation); // → "India"

    // Looking up `planet`:
    // 1. Check city EC     → not found
    // 2. Check country EC  → not found
    // 3. Check Global EC   → FOUND ✅ "Earth"
    console.log(planet); // → "Earth"
  }

  city();
}

country();
```

The chain is built at **definition time** (lexical scoping), not at call time. This is why a function always sees the scope where it was written, regardless of where it's called from.

---

## 8. Hoisting — Explained by the Creation Phase

Hoisting isn't magic — it's just the **creation phase** happening before the **execution phase**.

```js
// What you write:
console.log(score);  // → undefined
greet("Alice");      // → "Hello, Alice!"

var score = 100;
function greet(name) { return `Hello, ${name}`; }


// What the creation phase builds in memory BEFORE line 1 runs:
//
// Variable Environment:
//   score = undefined   ← var gets placeholder value
//   greet = function    ← full function stored
//
// Execution phase then runs your code top to bottom.
// That's why `greet` works but `score` is undefined.
```

### `let` and `const` — Temporal Dead Zone (TDZ)

`let` and `const` ARE hoisted during creation phase — but they're placed in a "dead zone" where you can't access them.

```js
// Creation phase: `name` exists in memory, but in TDZ
// Execution phase reaches this line before `let name`:

console.log(name); // ❌ ReferenceError: Cannot access 'name' before initialization

let name = "Alice"; // TDZ ends here — now accessible
console.log(name);  // ✅ "Alice"
```

```
Timeline for `let name = "Alice"`:

 Script start
      │
      │  ← TDZ starts (name is hoisted but unusable)
      │
      │  console.log(name) ← ❌ ReferenceError here
      │
      │  let name = "Alice" ← TDZ ends, name = "Alice"
      │
      │  console.log(name) ← ✅ "Alice"
      ↓
```

---

## 9. `this` Binding in Execution Contexts

The value of `this` is determined during the **creation phase** — and it depends entirely on how the function is called.

```js
// 1. Global EC — this = window (browser) or globalThis
console.log(this); // → Window {...}


// 2. Regular function call — this = undefined (strict) or globalThis
function showThis() {
  console.log(this);
}
showThis(); // → undefined (strict mode) or Window


// 3. Method call — this = the object before the dot
const user = {
  name: "Alice",
  greet() {
    console.log(this.name); // this = user
  }
};
user.greet(); // → "Alice"


// 4. Constructor call — this = the new instance
function Person(name) {
  this.name = name; // this = brand new object
}
const alice = new Person("Alice");
alice.name; // → "Alice"


// 5. Arrow function — this = inherited from outer scope (NEVER its own)
const obj = {
  name: "Bob",
  greet: () => {
    console.log(this.name); // this = outer scope (NOT obj)
  }
};
obj.greet(); // → undefined ⚠️


// 6. Explicit binding — this = whatever you pass
function introduce() {
  return `I'm ${this.name}`;
}
introduce.call({ name: "Charlie" }); // → "I'm Charlie"
introduce.bind({ name: "Dana" })();  // → "I'm Dana"
```

---

## 10. Closures — When Execution Contexts Live On

Normally, when a function returns, its EC is destroyed. But if an inner function **references variables** from the outer EC, those variables are **kept alive** — this is a closure.

```js
function createCounter() {
  let count = 0; // Lives in createCounter's EC

  return function increment() {
    // increment closes over `count` from the outer EC
    count++;
    return count;
  };
}

const counter = createCounter();
// createCounter's EC is "done" — but `count` is NOT garbage collected
// because `increment` still holds a reference to it

counter(); // → 1
counter(); // → 2
counter(); // → 3
```

What's happening under the hood:

```
createCounter() returns and its EC would normally be destroyed.
But the returned `increment` function holds a reference to
createCounter's Lexical Environment.

The garbage collector sees:
  counter (variable) → increment (function) → createCounter's env → count
  
Since count is reachable via this chain, it stays alive in memory.
```

Each call to `createCounter` creates a brand new, independent closure:

```js
const counterA = createCounter(); // its own `count` starting at 0
const counterB = createCounter(); // completely separate `count` at 0

counterA(); // → 1
counterA(); // → 2
counterB(); // → 1  (independent!)
counterA(); // → 3
```

---

## 11. The Event Loop & Call Stack Together

JavaScript is single-threaded — one call stack — but it handles async code via the **Event Loop**.

```
┌─────────────────┐      ┌──────────────────┐      ┌───────────────┐
│   Call Stack    │      │  Web APIs         │      │ Callback Queue│
│                 │      │ (setTimeout,      │      │               │
│  [Global EC]   ─┼─────▶│  fetch, events)  ─┼─────▶│  [callback]  │
│                 │      │                  │      │               │
└────────┬────────┘      └──────────────────┘      └───────┬───────┘
         │                                                  │
         │         Event Loop watches:                      │
         │         "Is the call stack empty?"               │
         │◀─────── If yes → push next callback ────────────┘
```

```js
console.log("1 — sync");           // → runs immediately (stack)

setTimeout(() => {
  console.log("3 — async");        // → goes to Web API, then callback queue
}, 0);

console.log("2 — sync");           // → runs immediately (stack)

// Output:
// 1 — sync
// 2 — sync
// 3 — async    ← even with 0ms delay, runs AFTER the stack clears
```

The callback for `setTimeout` never interrupts the running stack — it waits in the queue until the stack is completely empty.

---

## 12. Stack Overflow — What Causes It

The call stack has a size limit. **Infinite recursion** fills the stack faster than frames can be popped.

```js
// ❌ Stack overflow — function calls itself forever
function infinite() {
  return infinite(); // pushes a new frame every call, never pops
}

infinite(); // → RangeError: Maximum call stack size exceeded
```

Each call pushes a new frame without ever returning, until the stack runs out of space.

```
Stack grows:
  [Global → infinite → infinite → infinite → ... → BOOM]
                                                  ↑
                                       RangeError thrown here
```

### Fix: ensure a base case in recursion

```js
// ✅ Every recursion must have a base case that stops the calls
function countdown(n) {
  if (n <= 0) return "done!"; // base case — stack starts popping
  return countdown(n - 1);    // recursive call
}

countdown(5); // → "done!" — stack never overflows
```

### Fix: use iteration for very deep recursion

```js
// ✅ Iterative version — uses O(1) stack space (no new frames)
function countdownIterative(n) {
  while (n > 0) n--;
  return "done!";
}
```

---

## 13. Async/Await & the Call Stack

`async/await` doesn't block the call stack — it suspends the function and frees the stack.

```js
async function loadUserData(id) {
  console.log("A — before await"); // runs synchronously, on the stack

  const user = await fetchUser(id);
  // ↑ function is SUSPENDED here
  // Its EC is saved, the stack frame is REMOVED
  // Other code can run while waiting
  // When fetchUser resolves, the function RESUMES (re-pushed to stack)

  console.log("C — after await");  // runs after resume
  return user;
}

console.log("B — after calling loadUserData");

loadUserData(1);
// Output order:
// A — before await
// B — after calling loadUserData  ← stack free while awaiting
// C — after await                 ← resumes when fetch completes
```

The "saved state" of the suspended async function is managed by JavaScript's microtask queue — not the main callback queue.

---

## 14. Debugging the Call Stack

The call stack is visible in browser DevTools — invaluable for debugging.

### Reading a stack trace

```
Error: user is not defined
    at formatUser (app.js:12)      ← where error occurred (top)
    at processData (app.js:28)     ← called formatUser
    at handleResponse (app.js:45)  ← called processData
    at (anonymous) (app.js:60)     ← original caller (bottom)
```

Read from top to bottom:
- Top = where the error happened
- Bottom = what started the chain

### Using DevTools breakpoints

```js
function calculateTax(amount) {
  debugger; // ← execution pauses here in DevTools
            // You can inspect the call stack panel on the right
            // See every active EC and its variables
  return amount * 0.18;
}
```

In Chrome DevTools → Sources tab → you'll see the full call stack in the right panel whenever paused on a breakpoint.

### `console.trace()` — print the stack

```js
function c() {
  console.trace("Tracing from c"); // prints the full call stack
}
function b() { c(); }
function a() { b(); }
a();

// Output:
// Tracing from c
//   at c (script.js:2)
//   at b (script.js:5)
//   at a (script.js:8)
//   at script.js:10
```

---

## 15. Quick Reference Cheatsheet

```
EXECUTION CONTEXT (EC)
───────────────────────────────────────────────────────────
What it is:    A container holding everything needed to run code
Created when:  Script loads (Global EC) or function is called (Function EC)
Destroyed:     When the function returns (Global EC lives until tab closes)
Contains:      Variable Environment + Lexical Environment + this binding


TYPES OF EC
───────────────────────────────────────────────────────────
Global EC     → Created once. this = window. No outer scope.
Function EC   → Created per call. this = depends on call type.
Eval EC       → Avoid. Created by eval().


TWO PHASES
───────────────────────────────────────────────────────────
Creation Phase   → Memory allocated. var = undefined. Functions stored.
                   let/const in TDZ. this binding set. Outer env linked.
Execution Phase  → Code runs line by line. Variables assigned real values.


CALL STACK
───────────────────────────────────────────────────────────
What it is:  LIFO stack tracking which EC is currently running
Push:        When a function is called → new EC added to top
Pop:         When a function returns → EC removed from top
Single-thread: Only one EC runs at a time (the top one)
Overflow:    Infinite recursion fills stack → RangeError


SCOPE CHAIN
───────────────────────────────────────────────────────────
Each EC has a reference to its outer EC (where it was WRITTEN)
Lookup order: current EC → outer EC → ... → Global EC → ReferenceError
Direction: inner sees outer ✅, outer cannot see inner ❌


THIS BINDING (set in creation phase)
───────────────────────────────────────────────────────────
Global call       → undefined (strict) / globalThis
Method call       → the object (left of the dot)
Constructor (new) → the new instance
Arrow function    → inherited from outer EC (never its own)
.call/.apply      → explicitly passed object
.bind             → permanently bound object


HOISTING (explained by creation phase)
───────────────────────────────────────────────────────────
var                → hoisted as undefined  ⚠️
function decl      → fully hoisted ✅
let / const        → hoisted in TDZ (unusable before declaration) ❌
function expr      → same as its variable keyword


CLOSURES (EC lives on)
───────────────────────────────────────────────────────────
When an inner function references outer variables,
the outer EC's variables are kept alive even after it returns.
The inner function carries a reference to its birth scope.
```

```js
// EVERYTHING IN ONE EXAMPLE

// ─ Global EC created ─────────────────────────────────────
var globalVar = "I'm hoisted as undefined first";
let blockVar  = "I was in TDZ until this line";

function outer() {
  // ─ outer() EC created ──────────────────────────────────
  const x = 10;

  function inner() {
    // ─ inner() EC created ────────────────────────────────
    const y = 20;
    return x + y; // scope chain: y(own) → x(outer's EC) ✅
    // ─ inner() EC destroyed on return ───────────────────
  }

  return inner; // inner closes over x — outer's env lives on
  // ─ outer() EC would be destroyed BUT x is kept for inner
}

const fn = outer();
fn(); // → 30 — closure still has x = 10 from outer's env


// ─ Call stack trace for fn() ────────────────────────────
//   [Global EC → fn/inner() EC]
//   inner runs, returns 30
//   [Global EC]  ← back to just global
```

---

*Last updated: June 2026 | JavaScript ES2022+*