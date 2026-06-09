# 🔭 JavaScript Scope — The Complete Guide

> Where variables live, who can see them, and why it matters.

---

## Table of Contents

1. [What is Scope?](#1-what-is-scope)
2. [Global Scope](#2-global-scope)
3. [Local / Function Scope](#3-local--function-scope)
4. [Block Scope (`let` & `const`)](#4-block-scope-let--const)
5. [var vs let vs const — Scope Comparison](#5-var-vs-let-vs-const--scope-comparison)
6. [Nested Scope & the Scope Chain](#6-nested-scope--the-scope-chain)
7. [Lexical Scope](#7-lexical-scope)
8. [Hoisting](#8-hoisting)
9. [Closures & Scope](#9-closures--scope)
10. [Module Scope](#10-module-scope)
11. [Scope in Loops](#11-scope-in-loops)
12. [Scope in Callbacks & Async Code](#12-scope-in-callbacks--async-code)
13. [Common Scope Bugs & Fixes](#13-common-scope-bugs--fixes)
14. [Real-World Patterns Using Scope](#14-real-world-patterns-using-scope)
15. [Quick Reference Cheatsheet](#15-quick-reference-cheatsheet)

---

## 1. What is Scope?

**Scope** is the set of rules that determines **where a variable can be accessed** in your code.

Think of scope like **visibility zones**:

```
┌──────────────────────────────────────┐
│           GLOBAL SCOPE               │
│   var globalVar = "I'm everywhere"   │
│                                      │
│  ┌───────────────────────────────┐   │
│  │       FUNCTION SCOPE          │   │
│  │  var localVar = "local only"  │   │
│  │                               │   │
│  │  ┌─────────────────────────┐  │   │
│  │  │      BLOCK SCOPE        │  │   │
│  │  │  let blockVar = "here"  │  │   │
│  │  └─────────────────────────┘  │   │
│  └───────────────────────────────┘   │
└──────────────────────────────────────┘
```

- Inner scopes can **see** outer scopes
- Outer scopes **cannot see** inner scopes
- Same-level scopes **cannot see** each other

---

## 2. Global Scope

A variable is **global** when it is declared **outside of any function or block**.

```js
// These are in global scope
const appName = "MyApp";
let userCount = 0;
var legacyVar = "old style";

function showApp() {
  console.log(appName);   // ✅ accessible — inner sees outer
}

showApp(); // → "MyApp"
console.log(appName); // → "MyApp"
```

### The Global Object

In browsers, the global object is `window`. In Node.js it's `global`. In both, `globalThis` works everywhere.

```js
var score = 100; // var in global scope attaches to window

console.log(window.score);     // → 100  (browser only)
console.log(globalThis.score); // → 100  (works everywhere)

// let and const do NOT attach to the global object
let username = "Alice";
console.log(window.username); // → undefined ⚠️
```

### Dangers of Global Variables

```js
// file1.js
var count = 0;

// file2.js (loaded after file1)
var count = 99; // ❌ silently overwrites file1's count!

// This is why global variables are dangerous in large apps
```

**Best practice:** Keep globals to an absolute minimum. Use modules instead (see §10).

---

## 3. Local / Function Scope

Variables declared **inside a function** are local to that function — invisible to the outside world.

```js
function makeGreeting() {
  const message = "Hello, World!"; // local variable
  console.log(message);            // ✅ works fine inside
}

makeGreeting(); // → "Hello, World!"
console.log(message); // ❌ ReferenceError: message is not defined
```

### Each function call gets its own scope

Every time a function is called, a **brand new, independent scope** is created.

```js
function createId() {
  let id = Math.random(); // new `id` created on every call
  return id;
}

const a = createId(); // has its own `id`
const b = createId(); // has its own completely different `id`

// `a` and `b` are different — their scopes never overlapped
```

### Functions can read outer variables

```js
const TAX_RATE = 0.18; // global/outer

function calculatePrice(amount) {
  return amount + amount * TAX_RATE; // reads outer variable ✅
}

calculatePrice(100); // → 118
```

### But outer code cannot read inner variables

```js
function processOrder() {
  const discount = 0.1; // local
  return discount;
}

processOrder();
console.log(discount); // ❌ ReferenceError — discount doesn't exist out here
```

---

## 4. Block Scope (`let` & `const`)

A **block** is any code wrapped in `{ }` — including `if`, `for`, `while`, and standalone braces.

`let` and `const` are **block-scoped** — they only exist inside the `{ }` they were declared in.

```js
{
  let blockLet   = "I'm block-scoped";
  const blockConst = "Me too";
  var blockVar   = "I escape blocks!";
}

console.log(blockLet);   // ❌ ReferenceError
console.log(blockConst); // ❌ ReferenceError
console.log(blockVar);   // ✅ "I escape blocks!" — var ignores blocks
```

### `if` block scope

```js
const age = 20;

if (age >= 18) {
  let status = "adult";   // block-scoped to the if
  const pass  = true;
  var  legacy = "escapes"; // var leaks out!
}

console.log(status);  // ❌ ReferenceError
console.log(pass);    // ❌ ReferenceError
console.log(legacy);  // ✅ "escapes" — bad practice
```

### `for` loop block scope

```js
for (let i = 0; i < 3; i++) {
  // `i` is scoped to this loop block
}
console.log(i); // ❌ ReferenceError — i is gone after loop

// vs var:
for (var j = 0; j < 3; j++) {}
console.log(j); // ✅ 3 — leaks out (usually not what you want)
```

---

## 5. `var` vs `let` vs `const` — Scope Comparison

| Feature | `var` | `let` | `const` |
|---|---|---|---|
| Scope | Function | Block | Block |
| Hoisted? | ✅ (as `undefined`) | ✅ (but TDZ — unusable) | ✅ (but TDZ — unusable) |
| Re-declarable? | ✅ | ❌ | ❌ |
| Re-assignable? | ✅ | ✅ | ❌ |
| Attaches to `window`? | ✅ | ❌ | ❌ |
| Recommended? | ❌ Legacy | ✅ Yes | ✅ Prefer this |

```js
// var — function scoped, ignores blocks
function example() {
  if (true) {
    var x = 10;
  }
  console.log(x); // ✅ 10 — var leaked out of the if block
}

// let — block scoped
function example2() {
  if (true) {
    let y = 20;
  }
  console.log(y); // ❌ ReferenceError
}

// const — block scoped + can't be reassigned
function example3() {
  const Z = 30;
  Z = 99; // ❌ TypeError: Assignment to constant variable
}
```

### `const` with objects — a common misconception

```js
const user = { name: "Alice" };

user = { name: "Bob" };   // ❌ TypeError — can't reassign the binding

user.name = "Bob";        // ✅ fine — the object's contents can change
user.age  = 30;           // ✅ fine — adding new properties is allowed
```

`const` prevents reassigning the **variable**, not mutating the **value**.

---

## 6. Nested Scope & the Scope Chain

Functions can be nested inside other functions. Each level can access variables from all outer levels.

```js
const planet = "Earth"; // level 1 — global

function country() {
  const nation = "India"; // level 2

  function city() {
    const town = "Mumbai"; // level 3

    function street() {
      const road = "MG Road"; // level 4

      // Can see ALL outer levels ✅
      console.log(road);   // "MG Road"   — own scope
      console.log(town);   // "Mumbai"    — level 3
      console.log(nation); // "India"     — level 2
      console.log(planet); // "Earth"     — level 1
    }

    street();
    console.log(road); // ❌ ReferenceError — can't see inner scope
  }

  city();
}

country();
```

### How the Scope Chain works

When JavaScript looks up a variable, it searches:
1. Current scope
2. Parent scope
3. Parent's parent scope
4. ... all the way up to global
5. If not found anywhere → `ReferenceError`

```
street() looks for `nation`:
  street scope      → not found
  city scope        → not found
  country scope     → FOUND ✅ "India"
  (stops searching)
```

### Shadowing — inner variable hides outer

```js
const color = "blue"; // outer

function paint() {
  const color = "red"; // shadows the outer `color`
  console.log(color);  // → "red" (finds local first, stops searching)
}

paint();
console.log(color); // → "blue" (outer is unaffected)
```

---

## 7. Lexical Scope

**Lexical scope** means a function's scope is determined by **where it is written** in the code, not where it is called from.

```js
const language = "JavaScript"; // written in global scope

function getLanguage() {
  return language; // captured at definition time
}

function runInDifferentContext() {
  const language = "Python"; // this does NOT affect getLanguage
  return getLanguage();
}

runInDifferentContext(); // → "JavaScript" (not "Python"!)
```

`getLanguage` was **written** in global scope, so it always uses the global `language`, no matter where it's **called** from.

This is fundamentally different from dynamic scoping (which JS does NOT use).

```js
// Another clear example
function outer() {
  const secret = "I belong to outer";

  function inner() {
    return secret; // inner was WRITTEN inside outer, so it sees `secret`
  }

  return inner;
}

const fn = outer();
fn(); // → "I belong to outer"
// Even though outer() already finished, inner still remembers `secret`
// This is lexical scope + closure working together
```

---

## 8. Hoisting

**Hoisting** is JavaScript's behavior of moving declarations to the top of their scope before code runs.

### Function declarations are fully hoisted

```js
// Call BEFORE definition — works fine!
greet("Alice"); // → "Hello, Alice!"

function greet(name) {
  return `Hello, ${name}!`;
}
```

The entire function is available from the top of its scope.

### `var` is hoisted (as `undefined`)

```js
console.log(score); // → undefined (not an error — hoisted but not initialized)
var score = 100;
console.log(score); // → 100

// JavaScript actually runs it like this:
var score;          // declaration hoisted to top
console.log(score); // undefined
score = 100;        // assignment stays in place
console.log(score); // 100
```

### `let` and `const` — Temporal Dead Zone (TDZ)

`let` and `const` are hoisted too, but they are **NOT initialized** — accessing them before declaration throws a `ReferenceError`.

```js
console.log(name); // ❌ ReferenceError: Cannot access 'name' before initialization
let name = "Alice";
```

The period between the start of the block and the declaration is called the **Temporal Dead Zone (TDZ)**.

```js
{
  // TDZ for `name` starts here ↓
  console.log(name); // ❌ ReferenceError — in TDZ
  let name = "Alice"; // TDZ ends here ↑
  console.log(name);  // ✅ "Alice"
}
```

### Function expressions are NOT hoisted

```js
greet("Alice"); // ❌ TypeError: greet is not a function

const greet = function(name) {
  return `Hello, ${name}!`;
};
```

Only the variable declaration is hoisted (as `undefined` for `var`, TDZ for `let`/`const`), not the function value.

---

## 9. Closures & Scope

A **closure** is formed when an inner function retains access to its outer function's scope even after the outer function has returned.

```js
function makeMultiplier(factor) {
  // `factor` lives in makeMultiplier's scope

  return function(number) {
    return number * factor; // inner function closes over `factor`
  };
}

const double = makeMultiplier(2);
const triple = makeMultiplier(3);

// makeMultiplier has returned, but `factor` is still alive in the closure
double(5); // → 10
triple(5); // → 15
```

### Closures create private state

```js
function createBankAccount(initialBalance) {
  let balance = initialBalance; // private — no one outside can touch this directly

  return {
    deposit(amount) {
      if (amount > 0) balance += amount;
      return balance;
    },
    withdraw(amount) {
      if (amount > balance) return "Insufficient funds";
      balance -= amount;
      return balance;
    },
    getBalance() {
      return balance;
    }
  };
}

const account = createBankAccount(1000);
account.deposit(500);    // → 1500
account.withdraw(200);   // → 1300
account.getBalance();    // → 1300
account.balance;         // → undefined (truly private!) ✅
```

### Each closure has its own independent scope

```js
function makeCounter() {
  let count = 0;
  return () => ++count;
}

const counterA = makeCounter();
const counterB = makeCounter();

counterA(); // → 1
counterA(); // → 2
counterA(); // → 3
counterB(); // → 1  (completely independent!)
counterB(); // → 2
counterA(); // → 4  (continues from where it left off)
```

---

## 10. Module Scope

ES6 Modules give every file its **own scope** — nothing leaks to global by default.

```js
// math.js — module scope
const PI = 3.14159; // not global, lives only in this module

export function circleArea(r) {
  return PI * r * r; // PI is accessible within the module
}

export function circumference(r) {
  return 2 * PI * r;
}

// PI is NOT accessible outside unless explicitly exported
```

```js
// main.js
import { circleArea, circumference } from "./math.js";

circleArea(5);      // → 78.53...
circumference(5);   // → 31.41...
console.log(PI);    // ❌ ReferenceError — PI stays in math.js
```

### Why modules solve the global scope problem

```js
// Without modules (old way — pollutes global)
var utils = {};          // had to namespace everything
utils.format = function() { ... };
utils.parse  = function() { ... };

// With modules (modern way — each file is its own scope)
// utils.js
export const format = () => { ... };
export const parse  = () => { ... };
// Nothing pollutes global. Clean. ✅
```

---

## 11. Scope in Loops

Loops are one of the most common sources of scope bugs.

### The Classic `var` Loop Bug

```js
// ❌ Bug — all callbacks share the SAME `i`
for (var i = 0; i < 3; i++) {
  setTimeout(function() {
    console.log(i); // by the time this runs, loop is done
  }, 1000);
}
// Logs: 3, 3, 3  (not 0, 1, 2 !)
```

Why? `var` is function-scoped — there's only **one `i`** for the whole loop.

### Fix 1: Use `let` (block-scoped)

```js
// ✅ Each iteration gets its own `i`
for (let i = 0; i < 3; i++) {
  setTimeout(function() {
    console.log(i);
  }, 1000);
}
// Logs: 0, 1, 2  ✅
```

`let` creates a **new binding** for each iteration.

### Fix 2: IIFE (old approach, before `let`)

```js
// ✅ Captures `i` in its own scope per iteration
for (var i = 0; i < 3; i++) {
  (function(captured_i) {
    setTimeout(function() {
      console.log(captured_i);
    }, 1000);
  })(i);
}
// Logs: 0, 1, 2  ✅
```

### `const` in `for...of` loops

```js
// ✅ const works perfectly in for...of — new binding each iteration
const fruits = ["apple", "banana", "cherry"];

for (const fruit of fruits) {
  console.log(fruit);
}
// apple
// banana
// cherry

// ❌ const in regular for loop — fails because i++ reassigns
for (const i = 0; i < 3; i++) { // TypeError on second iteration
}
```

---

## 12. Scope in Callbacks & Async Code

Async code runs later, but closures preserve the scope from when the function was **defined**.

### Callbacks capture scope correctly

```js
function loadUser(id) {
  const userId = id; // captured in closure

  fetch(`/api/users/${id}`)
    .then(function(response) {
      console.log(`Got user ${userId}`); // ✅ userId is remembered
      return response.json();
    })
    .then(function(data) {
      console.log(`Name: ${data.name}, requested id: ${userId}`); // ✅ still there
    });
}

loadUser(42);
```

### `async/await` scope works naturally

```js
async function processOrder(orderId) {
  const startTime = Date.now(); // captured in this function's scope

  const order = await fetchOrder(orderId);
  const user  = await fetchUser(order.userId);

  // startTime, order, user all live in the same scope
  const elapsed = Date.now() - startTime;
  console.log(`Processed order ${orderId} for ${user.name} in ${elapsed}ms`);
}
```

### Watch out: scope doesn't wait for async

```js
// ❌ This does NOT work as expected
const results = [];

for (var i = 0; i < 3; i++) {
  setTimeout(() => results.push(i), 100);
}
// results → [3, 3, 3]  (var's single `i` = 3 by the time callbacks run)

// ✅ Fix with let
for (let i = 0; i < 3; i++) {
  setTimeout(() => results.push(i), 100);
}
// results → [0, 1, 2]
```

---

## 13. Common Scope Bugs & Fixes

### Bug 1: Accidental Global Variable

```js
// ❌ Forgetting `let`/`const`/`var` creates a global
function calculate() {
  result = 42; // no declaration keyword — becomes global!
}

calculate();
console.log(result); // ✅ works, but result is now polluting global scope

// ✅ Fix: always declare variables
function calculate() {
  const result = 42; // local, safe
}
```

Use `"use strict"` to catch this — it turns accidental globals into errors:

```js
"use strict";

function calculate() {
  result = 42; // ❌ ReferenceError in strict mode — caught!
}
```

### Bug 2: Variable Shadowing Confusion

```js
let status = "active";

function checkStatus() {
  let status = "inactive"; // shadows outer status — is this intentional?
  return status;
}

checkStatus();    // → "inactive"
console.log(status); // → "active" (outer untouched, but this can be confusing)

// ✅ Fix: use distinct names if they mean different things
let globalStatus = "active";

function checkStatus() {
  let localStatus = "inactive";
  return localStatus;
}
```

### Bug 3: Thinking `var` is Block-Scoped

```js
// ❌ Common mistake — expecting `discount` to be local to if block
function getPrice(amount, isMember) {
  if (isMember) {
    var discount = 0.2; // var leaks out of if block!
  }
  return amount - amount * discount; // `discount` is undefined if !isMember → NaN
}

getPrice(100, false); // → NaN

// ✅ Fix: use let
function getPrice(amount, isMember) {
  let discount = 0; // declare outside with a safe default
  if (isMember) {
    discount = 0.2;
  }
  return amount - amount * discount;
}

getPrice(100, false); // → 100
getPrice(100, true);  // → 80
```

### Bug 4: Closure Capturing a Mutable Variable

```js
// ❌ All functions close over the SAME `i` reference
const fns = [];
for (var i = 0; i < 3; i++) {
  fns.push(() => i);
}

fns[0](); // → 3
fns[1](); // → 3
fns[2](); // → 3

// ✅ Fix: use let
const fns = [];
for (let i = 0; i < 3; i++) {
  fns.push(() => i);
}

fns[0](); // → 0
fns[1](); // → 1
fns[2](); // → 2
```

### Bug 5: Using a Variable Before Declaration (TDZ)

```js
// ❌ Accessing let before declaration
console.log(name); // ReferenceError: Cannot access 'name' before initialization
let name = "Alice";

// ✅ Always declare at the top of the block
let name = "Alice";
console.log(name); // → "Alice"
```

---

## 14. Real-World Patterns Using Scope

### Pattern 1: Module Pattern (IIFE)

Simulate a module before ES6 modules existed — keeps everything private.

```js
const ShoppingCart = (function() {
  // Private — completely hidden from outside
  let items = [];
  let discount = 0;

  function calculateTotal() {
    const subtotal = items.reduce((sum, i) => sum + i.price * i.qty, 0);
    return subtotal - subtotal * discount;
  }

  // Public API — only these are exposed
  return {
    addItem(item)       { items.push(item); },
    setDiscount(d)      { discount = d; },
    getTotal()          { return calculateTotal(); },
    getItemCount()      { return items.length; }
  };
})();

ShoppingCart.addItem({ name: "Book", price: 15, qty: 2 });
ShoppingCart.setDiscount(0.1);
ShoppingCart.getTotal(); // → 27

ShoppingCart.items;    // → undefined (private!) ✅
ShoppingCart.discount; // → undefined (private!) ✅
```

### Pattern 2: Configuration with Closure

```js
function createLogger(prefix, level = "INFO") {
  // These are captured by the returned function
  const timestamp = () => new Date().toISOString();

  return {
    log(message) {
      console.log(`[${timestamp()}] [${level}] [${prefix}] ${message}`);
    },
    warn(message) {
      console.warn(`[${timestamp()}] [WARN] [${prefix}] ${message}`);
    },
    error(message) {
      console.error(`[${timestamp()}] [ERROR] [${prefix}] ${message}`);
    }
  };
}

const authLogger = createLogger("AUTH");
const dbLogger   = createLogger("DATABASE", "DEBUG");

authLogger.log("User logged in");
// [2026-06-09T10:00:00.000Z] [INFO] [AUTH] User logged in

dbLogger.log("Query executed");
// [2026-06-09T10:00:00.001Z] [DEBUG] [DATABASE] Query executed
```

### Pattern 3: Memoization (Caching with Closure)

```js
function memoize(fn) {
  const cache = {}; // private cache, lives in closure

  return function(...args) {
    const key = JSON.stringify(args);

    if (key in cache) {
      console.log("(from cache)");
      return cache[key];
    }

    cache[key] = fn(...args);
    return cache[key];
  };
}

function expensiveCalc(n) {
  // simulate heavy computation
  let result = 0;
  for (let i = 0; i < n * 1000000; i++) result += i;
  return result;
}

const fastCalc = memoize(expensiveCalc);

fastCalc(10); // computes (slow)
fastCalc(10); // → (from cache) — instant!
fastCalc(20); // computes (slow)
fastCalc(20); // → (from cache) — instant!
```

### Pattern 4: Event Listeners with Scope

```js
function setupButton(buttonId, label) {
  let clickCount = 0; // private to each button setup

  const button = document.getElementById(buttonId);

  button.addEventListener("click", function() {
    clickCount++; // closes over clickCount
    button.textContent = `${label} (clicked ${clickCount} times)`;
  });
}

// Each button has its own independent `clickCount`
setupButton("btn1", "Submit");
setupButton("btn2", "Cancel");
```

### Pattern 5: Once Function (Run Only Once)

```js
function once(fn) {
  let hasRun = false; // private state in closure
  let result;

  return function(...args) {
    if (!hasRun) {
      result  = fn(...args);
      hasRun  = true;
    }
    return result;
  };
}

const initApp = once(function() {
  console.log("App initialized!");
  return { status: "ready" };
});

initApp(); // → "App initialized!" → { status: "ready" }
initApp(); // → { status: "ready" }  (silent — won't run again)
initApp(); // → { status: "ready" }  (same)
```

---

## 15. Quick Reference Cheatsheet

```
SCOPE TYPES
───────────────────────────────────────────────────
Global Scope    → Outside everything. Accessible anywhere.
Function Scope  → Inside a function. var lives here.
Block Scope     → Inside { }. let and const live here.
Module Scope    → Inside an ES6 module file. Private by default.
Lexical Scope   → Scope determined by where code is WRITTEN, not called.


VARIABLE SCOPE RULES
───────────────────────────────────────────────────
var   → function-scoped, hoisted as undefined, re-declarable
let   → block-scoped, TDZ (hoisted but not usable), no re-declare
const → block-scoped, TDZ, no re-declare, no re-assign


SCOPE CHAIN (lookup order)
───────────────────────────────────────────────────
Current scope → Parent scope → ... → Global scope → ReferenceError


VISIBILITY RULES
───────────────────────────────────────────────────
Inner scope   CAN    see outer scope      ✅
Outer scope   CANNOT see inner scope      ❌
Same level    CANNOT see each other       ❌


HOISTING SUMMARY
───────────────────────────────────────────────────
function declaration  → fully hoisted (callable before declaration)
var                   → hoisted as undefined
let / const           → hoisted but in TDZ (unusable before declaration)
function expression   → NOT hoisted (const/let/var rules apply)


COMMON GOTCHAS
───────────────────────────────────────────────────
var in for loop         → use let instead
Accidental global       → always use let/const/var
Shadowing confusion     → use distinct names
Closure over loop var   → use let in loop
TDZ access              → declare before use
```

```js
// REAL EXAMPLES AT A GLANCE

// Global
const API_URL = "https://api.example.com";

// Function scope
function fetchData() {
  const endpoint = "/users"; // local
  return API_URL + endpoint; // reads outer
}

// Block scope
if (true) {
  let x = 10;   // block-only
  const y = 20; // block-only
}

// Closure
function counter() {
  let n = 0;
  return () => ++n;
}
const tick = counter();
tick(); // 1
tick(); // 2

// Module
export const util = () => {};   // scoped to file
const private = "hidden";       // never leaves file
```

---

*Last updated: June 2026 | JavaScript ES2022+*