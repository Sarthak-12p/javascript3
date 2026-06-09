# 🚀 JavaScript Functions & Parameters — The Complete Guide

> Everything you'll ever need to know, in one place.

---

## Table of Contents

1. [What is a Function?](#1-what-is-a-function)
2. [Ways to Define a Function](#2-ways-to-define-a-function)
3. [Parameters vs Arguments](#3-parameters-vs-arguments)
4. [Default Parameters](#4-default-parameters)
5. [Rest Parameters (`...args`)](#5-rest-parameters-args)
6. [Destructured Parameters](#6-destructured-parameters)
7. [The `arguments` Object (Legacy)](#7-the-arguments-object-legacy)
8. [Return Values](#8-return-values)
9. [First-Class Functions](#9-first-class-functions)
10. [Higher-Order Functions](#10-higher-order-functions)
11. [Closures](#11-closures)
12. [`this` in Functions](#12-this-in-functions)
13. [Pure Functions](#13-pure-functions)
14. [Common Gotchas](#14-common-gotchas)
15. [Quick Reference Cheatsheet](#15-quick-reference-cheatsheet)

---

## 1. What is a Function?

A **function** is a reusable block of code that performs a specific task. You define it once and call it as many times as you want.

```js
// Define
function greet(name) {
  return `Hello, ${name}!`;
}

// Call
greet("Alice"); // → "Hello, Alice!"
greet("Bob");   // → "Hello, Bob!"
```

Think of a function as a machine:
- **Parameters** = the input slots on the machine
- **Return value** = the output that comes out

---

## 2. Ways to Define a Function

JavaScript gives you several syntaxes — each with slightly different behavior.

### Function Declaration

```js
function add(a, b) {
  return a + b;
}
```

✅ **Hoisted** — you can call it before it's defined in the file.

---

### Function Expression

```js
const add = function(a, b) {
  return a + b;
};
```

❌ **Not hoisted** — must be defined before calling.

---

### Arrow Function

```js
const add = (a, b) => a + b;
```

Shorter syntax. Great for callbacks.

- Single parameter → parentheses optional: `x => x * 2`
- Single expression body → `return` is implicit
- Multi-line body → needs `{}` and explicit `return`

```js
const square = x => x * x;                  // implicit return
const cube   = x => { return x * x * x; };  // explicit return
```

⚠️ Arrow functions do **not** have their own `this` — important for methods (see §12).

---

### Named Function Expression

```js
const factorial = function fact(n) {
  return n <= 1 ? 1 : n * fact(n - 1); // can call itself by name
};
```

Useful for **recursion** inside expressions.

---

### Immediately Invoked Function Expression (IIFE)

```js
(function() {
  console.log("Runs immediately!");
})();
```

Creates a private scope — nothing leaks out.

---

### Generator Function

```js
function* counter() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = counter();
gen.next().value; // 1
gen.next().value; // 2
```

Pauses execution at each `yield`. Great for lazy sequences.

---

### Async Function

```js
async function fetchUser(id) {
  const res = await fetch(`/api/users/${id}`);
  return res.json();
}
```

Always returns a **Promise**. `await` pauses until the promise resolves.

---

## 3. Parameters vs Arguments

These two words are often confused — here's the clear distinction:

| Term | When | Example |
|------|------|---------|
| **Parameter** | When *defining* the function | `function greet(name)` — `name` is the parameter |
| **Argument** | When *calling* the function | `greet("Alice")` — `"Alice"` is the argument |

```js
//              ↓ parameter
function greet(name) {
  return `Hi, ${name}`;
}

greet("Alice"); // ← argument
```

### What happens with wrong number of arguments?

```js
function add(a, b) {
  return a + b;
}

add(1, 2, 3); // → 3   (extra argument 3 is ignored)
add(1);       // → NaN (b is undefined; 1 + undefined = NaN)
```

JavaScript never throws an error for argument count mismatch — it's up to you to handle it.

---

## 4. Default Parameters

Provide a fallback value when an argument is missing or `undefined`.

```js
function greet(name = "stranger") {
  return `Hello, ${name}!`;
}

greet("Alice"); // → "Hello, Alice!"
greet();        // → "Hello, stranger!"
greet(undefined); // → "Hello, stranger!"  (undefined triggers default)
greet(null);      // → "Hello, null!"      (null does NOT trigger default)
```

### Defaults can reference earlier parameters

```js
function createBox(width = 10, height = width) {
  return { width, height };
}

createBox();      // → { width: 10, height: 10 }
createBox(5);     // → { width: 5,  height: 5  }
createBox(5, 20); // → { width: 5,  height: 20 }
```

### Defaults can be expressions or function calls

```js
function getTimestamp(date = new Date()) {
  return date.toISOString();
}
```

The expression is evaluated **at call time**, not at definition time.

---

## 5. Rest Parameters (`...args`)

Collect any number of remaining arguments into a **real array**.

```js
function sum(...numbers) {
  return numbers.reduce((total, n) => total + n, 0);
}

sum(1, 2, 3);       // → 6
sum(1, 2, 3, 4, 5); // → 15
sum();              // → 0
```

### Combine with regular parameters

```js
function log(level, ...messages) {
  console.log(`[${level}]`, ...messages);
}

log("INFO", "Server started", "Port: 3000");
// [INFO] Server started Port: 3000
```

⚠️ Rules:
- Only **one** rest parameter allowed per function
- Must be the **last** parameter

```js
// ✅ Valid
function f(a, b, ...rest) {}

// ❌ Invalid — rest must be last
function f(...rest, a) {} // SyntaxError
```

---

## 6. Destructured Parameters

Unpack objects or arrays directly in the parameter list.

### Object Destructuring

```js
function displayUser({ name, age, role = "user" }) {
  console.log(`${name} (${age}) — ${role}`);
}

displayUser({ name: "Alice", age: 30, role: "admin" });
// Alice (30) — admin

displayUser({ name: "Bob", age: 25 });
// Bob (25) — user   (role gets default)
```

### Array Destructuring

```js
function getFirst([first, second, ...rest]) {
  return { first, second, rest };
}

getFirst([10, 20, 30, 40]);
// → { first: 10, second: 20, rest: [30, 40] }
```

### Rename while destructuring

```js
function connect({ host: serverHost, port: serverPort = 3000 }) {
  console.log(`Connecting to ${serverHost}:${serverPort}`);
}

connect({ host: "localhost" });
// Connecting to localhost:3000
```

### Nested destructuring

```js
function getCity({ address: { city } }) {
  return city;
}

getCity({ address: { city: "Mumbai", zip: "400001" } });
// → "Mumbai"
```

---

## 7. The `arguments` Object (Legacy)

Before rest parameters, `arguments` was the way to handle variable args.

```js
function oldSum() {
  let total = 0;
  for (let i = 0; i < arguments.length; i++) {
    total += arguments[i];
  }
  return total;
}
```

⚠️ `arguments` is **array-like but not a real array** — it has no `.map()`, `.filter()` etc.

**Prefer `...rest` in modern code.** The only reason to know `arguments` is when reading legacy codebases.

Arrow functions do **not** have their own `arguments` object.

---

## 8. Return Values

Every function returns a value. If you don't specify one, it returns `undefined`.

```js
function nothing() {}
nothing(); // → undefined
```

### Return early

```js
function divide(a, b) {
  if (b === 0) return null; // early exit
  return a / b;
}
```

### Return multiple values (via array or object)

```js
function minMax(arr) {
  return {
    min: Math.min(...arr),
    max: Math.max(...arr),
  };
}

const { min, max } = minMax([3, 1, 4, 1, 5, 9]);
// min → 1, max → 9
```

---

## 9. First-Class Functions

In JavaScript, functions are **values** — they can be:

- Stored in variables
- Passed as arguments
- Returned from other functions

```js
// Store in a variable
const sayHi = function() { return "Hi!"; };

// Store in an object
const obj = {
  greet: function() { return "Hello!"; }
};

// Store in an array
const ops = [
  (a, b) => a + b,
  (a, b) => a - b,
  (a, b) => a * b,
];

ops[0](5, 3); // → 8
ops[1](5, 3); // → 2
ops[2](5, 3); // → 15
```

---

## 10. Higher-Order Functions

A function that **takes a function as argument** or **returns a function** is called a higher-order function.

### Takes a function (callback pattern)

```js
function repeat(times, action) {
  for (let i = 0; i < times; i++) {
    action(i);
  }
}

repeat(3, i => console.log(`Step ${i + 1}`));
// Step 1
// Step 2
// Step 3
```

### Returns a function (factory pattern)

```js
function multiplier(factor) {
  return (number) => number * factor; // returns a new function
}

const double = multiplier(2);
const triple = multiplier(3);

double(5); // → 10
triple(5); // → 15
```

### Built-in higher-order functions

```js
const nums = [1, 2, 3, 4, 5];

nums.map(n => n * 2);          // [2, 4, 6, 8, 10]
nums.filter(n => n % 2 === 0); // [2, 4]
nums.reduce((sum, n) => sum + n, 0); // 15
```

---

## 11. Closures

A closure is a function that **remembers the variables from its outer scope** even after the outer function has finished.

```js
function makeCounter(start = 0) {
  let count = start; // this variable is "closed over"

  return {
    increment: () => ++count,
    decrement: () => --count,
    value:     () => count,
  };
}

const counter = makeCounter(10);
counter.increment(); // 11
counter.increment(); // 12
counter.decrement(); // 11
counter.value();     // 11
```

`count` is private — nothing outside can touch it directly. This is the JS equivalent of private state.

### Classic closure gotcha

```js
// ❌ Bug — all callbacks share the same `i`
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Logs: 3, 3, 3

// ✅ Fix — use `let` (block-scoped, new binding per iteration)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Logs: 0, 1, 2
```

---

## 12. `this` in Functions

`this` refers to the **execution context** — it changes based on how a function is called.

| How called | `this` value |
|---|---|
| Regular function call | `undefined` (strict) or `globalThis` |
| Method on an object | The object |
| `new` constructor | The new instance |
| `.call(obj)` / `.apply(obj)` | `obj` |
| `.bind(obj)` | `obj` (permanently) |
| Arrow function | Inherited from enclosing scope — never changes |

```js
const user = {
  name: "Alice",
  greet() {
    return `Hi, I'm ${this.name}`; // `this` = user
  },
  greetArrow: () => {
    return `Hi, I'm ${this.name}`; // `this` = outer scope (NOT user!)
  }
};

user.greet();       // → "Hi, I'm Alice"  ✅
user.greetArrow();  // → "Hi, I'm undefined" ⚠️
```

### Explicit binding

```js
function introduce(greeting) {
  return `${greeting}, I'm ${this.name}`;
}

const alice = { name: "Alice" };

introduce.call(alice, "Hello");       // → "Hello, I'm Alice"
introduce.apply(alice, ["Hello"]);    // → "Hello, I'm Alice"

const boundFn = introduce.bind(alice);
boundFn("Hey");                       // → "Hey, I'm Alice"
```

---

## 13. Pure Functions

A **pure function**:
1. Always returns the same output for the same input
2. Has **no side effects** (doesn't modify external state)

```js
// ✅ Pure
function add(a, b) {
  return a + b;
}

// ❌ Impure — depends on external state
let tax = 0.1;
function price(amount) {
  return amount + amount * tax; // depends on `tax`
}

// ❌ Impure — side effect (mutates input)
function appendItem(arr, item) {
  arr.push(item); // mutates the original array
  return arr;
}

// ✅ Pure version — returns new array
function appendItem(arr, item) {
  return [...arr, item];
}
```

Pure functions are:
- Easy to test
- Easy to reason about
- Safe to run in parallel

---

## 14. Common Gotchas

### Forgetting to `return`

```js
// ❌ Returns undefined
const double = n => { n * 2; };

// ✅ Returns the value
const double = n => n * 2;
// or
const double = n => { return n * 2; };
```

### Accidentally calling vs referencing

```js
setTimeout(greet, 1000);   // ✅ pass the function — called after 1 second
setTimeout(greet(), 1000); // ❌ calls greet immediately, passes its return value
```

### `null` doesn't use default parameters

```js
function greet(name = "stranger") { return name; }

greet(undefined); // → "stranger"  ✅
greet(null);      // → null        ⚠️ — null is a real value, not missing
```

### Arrow functions and object literals

```js
// ❌ Ambiguous — JS thinks {} is a function body, not an object
const makeObj = () => { x: 1 };

// ✅ Wrap object in parentheses
const makeObj = () => ({ x: 1 });
```

---

## 15. Quick Reference Cheatsheet

```js
// === SYNTAX ===

function named(a, b) { return a + b; }          // declaration (hoisted)
const expr = function(a, b) { return a + b; };  // expression
const arrow = (a, b) => a + b;                  // arrow (implicit return)
const arrow2 = (a, b) => { return a + b; };     // arrow (explicit return)
async function load(id) { return await fetch(id); } // async


// === PARAMETERS ===

function defaults(a, b = 10) {}                 // default value
function rest(a, ...others) {}                  // rest (must be last)
function destructObj({ x, y = 0 }) {}           // destructured object
function destructArr([first, ...tail]) {}        // destructured array


// === CALLING ===

fn(1, 2, 3);                                    // normal call
fn.call(ctx, 1, 2, 3);                          // explicit this
fn.apply(ctx, [1, 2, 3]);                       // this + array args
const bound = fn.bind(ctx);  bound(1, 2, 3);    // permanent this
new Fn(args);                                    // constructor call


// === PATTERNS ===

// Factory
const make = (n) => () => n * 2;

// Callback
arr.forEach(item => console.log(item));

// IIFE
(function() { /* private scope */ })();

// Composition
const compose = (f, g) => x => f(g(x));
```

---

## 16. Functions with Objects

This section covers every way functions and objects interact in JavaScript.

---

### 16.1 Functions as Object Methods

A function stored as a property of an object is called a **method**.

```js
const calculator = {
  value: 0,

  add(n) {
    this.value += n;
    return this; // return `this` to enable chaining
  },

  subtract(n) {
    this.value -= n;
    return this;
  },

  result() {
    return this.value;
  }
};

calculator.add(10).add(5).subtract(3).result(); // → 12
```

---

### 16.2 Shorthand Method Syntax

ES6 introduced a cleaner way to write methods inside objects.

```js
// ❌ Old verbose way
const obj = {
  greet: function() { return "Hello"; }
};

// ✅ Modern shorthand
const obj = {
  greet() { return "Hello"; }
};
```

Both are equivalent — shorthand is preferred in modern code.

---

### 16.3 `this` Inside Object Methods

`this` inside a method refers to the object the method belongs to.

```js
const user = {
  name: "Alice",
  age: 30,

  describe() {
    return `${this.name} is ${this.age} years old`;
  },

  birthday() {
    this.age++;         // modifies the object's own property
    return this.age;
  }
};

user.describe();  // → "Alice is 30 years old"
user.birthday();  // → 31
user.describe();  // → "Alice is 31 years old"
```

⚠️ Losing `this` — a common bug:

```js
const user = {
  name: "Alice",
  greet() { return `Hi, I'm ${this.name}`; }
};

const fn = user.greet;   // detached from object
fn();                    // → "Hi, I'm undefined" ❌

const fn2 = user.greet.bind(user); // bind fixes it
fn2();                   // → "Hi, I'm Alice" ✅
```

---

### 16.4 Constructor Functions

Before `class` syntax, constructor functions were the standard way to create multiple objects of the same "shape".

```js
function Person(name, age) {
  this.name = name;
  this.age  = age;

  this.greet = function() {
    return `Hi, I'm ${this.name}`;
  };
}

const alice = new Person("Alice", 30);
const bob   = new Person("Bob", 25);

alice.greet(); // → "Hi, I'm Alice"
bob.greet();   // → "Hi, I'm Bob"

alice instanceof Person; // → true
```

What `new` does step by step:
1. Creates a blank object `{}`
2. Sets its prototype to `Person.prototype`
3. Runs the function with `this` = that new object
4. Returns the object (unless you explicitly return something else)

---

### 16.5 Prototype Methods (Efficient Constructor Pattern)

Defining methods inside the constructor creates a **new copy** for every instance — wasteful. Put shared methods on the prototype instead.

```js
function Person(name, age) {
  this.name = name; // instance data — each object gets its own copy
  this.age  = age;
}

// Defined once, shared by ALL instances
Person.prototype.greet = function() {
  return `Hi, I'm ${this.name}`;
};

Person.prototype.birthday = function() {
  this.age++;
  return `Happy birthday ${this.name}! Now ${this.age}.`;
};

const alice = new Person("Alice", 30);
const bob   = new Person("Bob", 25);

alice.greet();    // → "Hi, I'm Alice"
bob.greet();      // → "Hi, I'm Bob"

// Both share ONE copy of greet in memory
alice.greet === bob.greet; // → true ✅
```

---

### 16.6 Factory Functions (No `new` Needed)

A factory function is a regular function that **builds and returns an object**. Cleaner and safer than constructors.

```js
function createUser(name, role = "viewer") {
  // private variable — not accessible outside
  let loginCount = 0;

  return {
    name,
    role,

    login() {
      loginCount++;
      return `${name} logged in (${loginCount} times)`;
    },

    getLoginCount() {
      return loginCount;
    }
  };
}

const alice = createUser("Alice", "admin");
const bob   = createUser("Bob");

alice.login();         // → "Alice logged in (1 times)"
alice.login();         // → "Alice logged in (2 times)"
alice.getLoginCount(); // → 2

alice.loginCount;      // → undefined (truly private!) ✅
```

Factory vs Constructor:

| | Factory Function | Constructor Function |
|---|---|---|
| Call with `new`? | No | Yes |
| Private state | Easy (closure) | Hard |
| Prototype chain | No | Yes |
| `instanceof` works | No | Yes |

---

### 16.7 Functions that Accept Objects as Parameters

Passing a single options object instead of many arguments makes functions far more readable and flexible.

```js
// ❌ Hard to read — what do these booleans mean?
createServer("localhost", 3000, true, false, 5000);

// ✅ Self-documenting options object
function createServer({
  host    = "localhost",
  port    = 3000,
  ssl     = false,
  logging = true,
  timeout = 5000
} = {}) {
  console.log(`Starting ${ssl ? "https" : "http"}://${host}:${port}`);
  console.log(`Logging: ${logging}, Timeout: ${timeout}ms`);
}

createServer({ port: 8080, ssl: true });
createServer(); // all defaults — works because of `= {}` at the end
```

The `= {}` at the end means the whole options argument is optional.

---

### 16.8 Functions that Return Objects

```js
// Simple object return
function makePoint(x, y) {
  return { x, y }; // shorthand for { x: x, y: y }
}

const p = makePoint(3, 4);
// → { x: 3, y: 4 }


// Return object with methods (mini-class pattern)
function createStack() {
  const items = [];

  return {
    push(item)  { items.push(item); },
    pop()       { return items.pop(); },
    peek()      { return items[items.length - 1]; },
    isEmpty()   { return items.length === 0; },
    size()      { return items.length; },
  };
}

const stack = createStack();
stack.push(1);
stack.push(2);
stack.peek();   // → 2
stack.pop();    // → 2
stack.size();   // → 1
```

---

### 16.9 Getters and Setters

Define computed properties that look like plain values but run a function behind the scenes.

```js
const circle = {
  radius: 5,

  get area() {
    return Math.PI * this.radius ** 2; // computed on access
  },

  get diameter() {
    return this.radius * 2;
  },

  set diameter(d) {
    this.radius = d / 2; // updating diameter updates radius
  }
};

circle.area;       // → 78.539... (no parentheses — looks like a property)
circle.diameter;   // → 10
circle.diameter = 20;
circle.radius;     // → 10 (automatically updated via setter)
```

---

### 16.10 Dynamic Function Properties on Objects

You can add, remove, and call functions on objects at runtime.

```js
const actions = {};

// Add functions dynamically
actions["greet"]  = (name) => `Hello, ${name}!`;
actions["shout"]  = (name) => `HEY, ${name.toUpperCase()}!`;
actions["whisper"]= (name) => `psst... ${name.toLowerCase()}...`;

// Call by name
function runAction(type, name) {
  if (actions[type]) {
    return actions[type](name);
  }
  return "Unknown action";
}

runAction("greet",   "Alice"); // → "Hello, Alice!"
runAction("shout",   "Alice"); // → "HEY, ALICE!"
runAction("whisper", "Alice"); // → "psst... alice..."
runAction("dance",   "Alice"); // → "Unknown action"
```

This pattern (a **dispatch table**) is a clean alternative to long `if/else` or `switch` chains.

---

### 16.11 Merging Functions into Objects (`Object.assign` / Spread)

```js
const baseAnimal = {
  breathe() { return "breathing..."; },
  sleep()   { return "sleeping..."; }
};

const dogMethods = {
  speak() { return `${this.name} says: Woof!`; },
  fetch() { return `${this.name} fetches the ball!`; }
};

function createDog(name, breed) {
  return Object.assign(
    { name, breed },   // instance data
    baseAnimal,        // shared base methods
    dogMethods         // species-specific methods
  );
}

// Same with spread (cleaner):
function createDog(name, breed) {
  return {
    name, breed,
    ...baseAnimal,
    ...dogMethods
  };
}

const rex = createDog("Rex", "Labrador");
rex.speak();   // → "Rex says: Woof!"
rex.breathe(); // → "breathing..."
```

---

### 16.12 Full Real-World Example

Putting it all together — a mini shopping cart:

```js
function createCart(currency = "USD") {
  const items = [];

  const formatPrice = (n) =>
    new Intl.NumberFormat("en-US", { style: "currency", currency }).format(n);

  return {
    addItem({ name, price, qty = 1 }) {
      const existing = items.find(i => i.name === name);
      if (existing) {
        existing.qty += qty;
      } else {
        items.push({ name, price, qty });
      }
      return this; // chainable
    },

    removeItem(name) {
      const idx = items.findIndex(i => i.name === name);
      if (idx !== -1) items.splice(idx, 1);
      return this;
    },

    get total() {
      return items.reduce((sum, i) => sum + i.price * i.qty, 0);
    },

    get count() {
      return items.reduce((sum, i) => sum + i.qty, 0);
    },

    summary() {
      const lines = items.map(
        i => `  ${i.name} x${i.qty}  ${formatPrice(i.price * i.qty)}`
      );
      return [
        "--- Cart ---",
        ...lines,
        `Items: ${this.count}`,
        `Total: ${formatPrice(this.total)}`
      ].join("\n");
    }
  };
}

const cart = createCart("USD");

cart
  .addItem({ name: "Coffee", price: 4.5, qty: 2 })
  .addItem({ name: "Muffin", price: 3.0 })
  .addItem({ name: "Coffee", price: 4.5 }); // adds to existing

console.log(cart.summary());
// --- Cart ---
//   Coffee x3  $13.50
//   Muffin x1  $3.00
// Items: 4
// Total: $16.50
```

---

*Last updated: June 2026 | JavaScript ES2022+*