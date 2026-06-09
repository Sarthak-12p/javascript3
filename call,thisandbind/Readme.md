# `this`, `call`, `apply`, and `bind` in JavaScript — In Depth

> A complete reference covering how `this` is determined in every context, explicit binding with `call`/`apply`/`bind`, arrow functions, class fields, common pitfalls, and real-world patterns.

---

## Table of Contents

1. [What is `this`?](#1-what-is-this)
2. [The 6 Rules of `this` Binding](#2-the-6-rules-of-this-binding)
3. [Default Binding](#3-default-binding)
4. [Implicit Binding](#4-implicit-binding)
5. [Explicit Binding — `call` and `apply`](#5-explicit-binding--call-and-apply)
6. [Hard Binding — `bind`](#6-hard-binding--bind)
7. [New Binding](#7-new-binding)
8. [Arrow Functions & Lexical `this`](#8-arrow-functions--lexical-this)
9. [Binding in Classes](#9-binding-in-classes)
10. [Binding Priority (Who Wins?)](#10-binding-priority-who-wins)
11. [`call` vs `apply` vs `bind` — Deep Comparison](#11-call-vs-apply-vs-bind--deep-comparison)
12. [Losing `this` — The Classic Pitfalls](#12-losing-this--the-classic-pitfalls)
13. [Real-World Patterns](#13-real-world-patterns)
14. [Edge Cases & Gotchas](#14-edge-cases--gotchas)
15. [Quick Reference](#15-quick-reference)

---

## 1. What is `this`?

`this` is a special keyword in JavaScript that refers to an **execution context** — the object that is the "owner" or "caller" of the currently executing function.

Unlike most other languages, `this` in JavaScript is **not determined by where a function is defined**, but by **how and where it is called** at runtime.

```js
function greet() {
  console.log(this.name);
}

const alice = { name: "Alice", greet };
const bob   = { name: "Bob",   greet };

alice.greet(); // "Alice" — this = alice
bob.greet();   // "Bob"   — this = bob
greet();       // undefined (or global.name in non-strict)
```

Same function, three different `this` values — purely based on the **call site**.

### `this` is NOT

```js
// ❌ this is NOT the function itself
function counter() {
  this.count++; // 'this' here is NOT 'counter'
}

// ❌ this is NOT the function's lexical scope
function outer() {
  const x = 10;
  function inner() {
    console.log(this.x); // NOT 10 — 'this' doesn't inherit scope
  }
  inner();
}
```

---

## 2. The 6 Rules of `this` Binding

JavaScript determines `this` using exactly these rules, in priority order:

```
┌─────────────────────────────────────────────────────────────┐
│            How is the function called?                      │
├──────┬──────────────────────────────────────────────────────┤
│  1   │  Arrow function?        → Lexical (inherits outer)  │
│  2   │  new fn()               → New object created        │
│  3   │  fn.call/apply/bind()   → Explicit argument         │
│  4   │  obj.fn()               → The object (obj)          │
│  5   │  fn() in strict mode    → undefined                  │
│  6   │  fn() in sloppy mode    → globalThis                 │
└──────┴──────────────────────────────────────────────────────┘
```

---

## 3. Default Binding

When a function is called as a plain function (no object, no `new`, no explicit binding), `this` falls back to the **default**:

- **Strict mode** → `undefined`
- **Sloppy mode** → `globalThis` (browser: `window`, Node.js: `global`)

```js
function showThis() {
  console.log(this);
}

// Sloppy mode
showThis(); // window (browser) or global (Node)

// Strict mode
"use strict";
function showThisStrict() {
  console.log(this);
}
showThisStrict(); // undefined
```

### Why strict mode matters

```js
function multiply(a, b) {
  "use strict";
  // If called as a plain function, this is undefined
  // Avoids accidentally polluting globalThis
  return a * b;
}
```

### globalThis

`globalThis` is the universal way to reference the global object across environments:

```js
globalThis === window;  // true in browser
globalThis === global;  // true in Node.js
```

---

## 4. Implicit Binding

When a function is called as a **method on an object**, `this` is set to that object.

```js
const user = {
  name: "Alice",
  greet() {
    console.log(`Hi, I'm ${this.name}`);
  },
};

user.greet(); // "Hi, I'm Alice" — this = user
```

### Only the immediate calling object matters

```js
const inner = {
  name: "Inner",
  greet() {
    console.log(this.name);
  },
};

const outer = {
  name: "Outer",
  inner,
};

outer.inner.greet(); // "Inner" — this = inner (the last object in the chain)
```

### Implicit Binding Loss

The most common source of bugs — extracting a method loses its binding:

```js
const user = {
  name: "Alice",
  greet() {
    console.log(this.name);
  },
};

user.greet();           // "Alice" ✅ — called on user

const fn = user.greet;
fn();                   // undefined ❌ — called as plain function, lost binding
```

```js
// Same problem with callbacks
setTimeout(user.greet, 1000); // undefined ❌ — setTimeout calls it as plain fn
[1, 2, 3].forEach(user.greet); // undefined ❌
```

---

## 5. Explicit Binding — `call` and `apply`

Both `call` and `apply` let you **explicitly specify** what `this` should be when calling a function.

### Function.prototype.call()

```
fn.call(thisArg, arg1, arg2, arg3, ...)
```

Calls `fn` immediately with `this` set to `thisArg`, passing arguments **individually**.

```js
function greet(greeting, punctuation) {
  console.log(`${greeting}, ${this.name}${punctuation}`);
}

const user = { name: "Alice" };

greet.call(user, "Hello", "!");   // "Hello, Alice!"
greet.call(user, "Hi", ".");      // "Hi, Alice."
```

```js
// Calling a method from another object's context
const cat  = { sound: "meow", speak() { console.log(this.sound); } };
const dog  = { sound: "woof" };

cat.speak.call(dog); // "woof" — dog borrows cat's method
```

### Function.prototype.apply()

```
fn.apply(thisArg, [arg1, arg2, arg3, ...])
```

Identical to `call`, but arguments are passed as an **array** (or array-like object):

```js
function greet(greeting, punctuation) {
  console.log(`${greeting}, ${this.name}${punctuation}`);
}

const user = { name: "Alice" };

greet.apply(user, ["Hello", "!"]);  // "Hello, Alice!"
```

### call vs apply — When to Use Which

```js
// call: when you have arguments individually
fn.call(ctx, a, b, c);

// apply: when arguments are already in an array
fn.apply(ctx, argsArray);

// Modern alternative to apply: spread operator
fn.call(ctx, ...argsArray); // Same as apply
```

### Classic apply Tricks (Pre-Spread)

```js
const numbers = [5, 2, 8, 1, 9];

// Before spread operator, apply was used to spread arrays
const max = Math.max.apply(null, numbers); // 9
const min = Math.min.apply(null, numbers); // 1

// Modern equivalent (use this now)
const max2 = Math.max(...numbers); // 9
```

### Borrowing Methods

`call` and `apply` shine when borrowing methods between objects:

```js
// Array methods on array-like objects
function logArgs() {
  // arguments is array-like, not a real Array
  const args = Array.prototype.slice.call(arguments);
  // Modern: Array.from(arguments) or [...arguments]
  console.log(args);
}

// toString for type checking
const typeOf = (val) =>
  Object.prototype.toString.call(val).slice(8, -1).toLowerCase();

typeOf([]);          // "array"
typeOf({});          // "object"
typeOf(null);        // "null"
typeOf(new Date());  // "date"
typeOf(/regex/);     // "regexp"
```

### call with null / undefined

Passing `null` or `undefined` as `thisArg` uses the default binding:

```js
Math.max.call(null, 1, 2, 3); // 3 — null → globalThis (sloppy) or undefined (strict)

// Safe to use null for pure utility functions that don't use `this`
[1, [2, [3]]].reduce.call(null, ...); // Fine if fn doesn't use `this`
```

---

## 6. Hard Binding — `bind`

`bind` creates a **new function** with `this` permanently locked to a value. Unlike `call`/`apply`, it does **not** call the function immediately.

```
const boundFn = fn.bind(thisArg, arg1, arg2, ...)
```

```js
function greet(greeting) {
  console.log(`${greeting}, ${this.name}!`);
}

const user = { name: "Alice" };

const greetAlice = greet.bind(user);
greetAlice("Hello"); // "Hello, Alice!"
greetAlice("Hi");    // "Hi, Alice!"

// The binding is permanent — cannot be overridden
greetAlice.call({ name: "Bob" }, "Hello"); // "Hello, Alice!" — call can't override bind
```

### Partial Application with bind

`bind` can also pre-fill arguments (partial application / currying):

```js
function multiply(a, b) {
  return a * b;
}

const double = multiply.bind(null, 2); // Pre-fill a=2
const triple = multiply.bind(null, 3); // Pre-fill a=3

double(5);  // 10
double(10); // 20
triple(4);  // 12
triple(7);  // 21
```

```js
function log(level, message) {
  console.log(`[${level}] ${message}`);
}

const info  = log.bind(null, "INFO");
const error = log.bind(null, "ERROR");
const warn  = log.bind(null, "WARN");

info("Server started");   // [INFO] Server started
error("DB connection failed"); // [ERROR] DB connection failed
```

### Fixing Method Loss with bind

```js
class Timer {
  constructor() {
    this.seconds = 0;
    // Bind once in constructor so tick() always has correct `this`
    this.tick = this.tick.bind(this);
  }

  tick() {
    this.seconds++;
    console.log(this.seconds);
  }

  start() {
    setInterval(this.tick, 1000); // Safe — this.tick is already bound
  }
}
```

### bind Returns a New Function

```js
function fn() {}

const bound1 = fn.bind(null);
const bound2 = fn.bind(null);

bound1 === fn;     // false — new function
bound1 === bound2; // false — different new functions each time
```

This matters for **event listener removal** — you must keep a reference:

```js
class Component {
  constructor() {
    // ✅ Save reference so we can remove it later
    this._onClick = this.handleClick.bind(this);
    document.addEventListener("click", this._onClick);
  }

  handleClick(e) {
    console.log("Clicked:", this);
  }

  destroy() {
    document.removeEventListener("click", this._onClick); // ✅ Works
  }
}
```

---

## 7. New Binding

When a function is called with `new`, JavaScript:

1. Creates a brand new empty object `{}`
2. Sets `this` to that new object
3. Sets the object's `[[Prototype]]` to `fn.prototype`
4. Executes the function body
5. Returns the new object (unless the function explicitly returns another object)

```js
function Person(name, age) {
  // `this` is the newly created object
  this.name = name;
  this.age  = age;
  // Implicit: return this
}

const alice = new Person("Alice", 30);
console.log(alice.name); // "Alice"
console.log(alice.age);  // 30
```

### What new does step by step

```js
// What `new Person("Alice", 30)` does internally:
const obj = Object.create(Person.prototype); // Step 1 + 3
Person.call(obj, "Alice", 30);               // Step 2 + 4
// return obj;                               // Step 5 (implicit)
```

### Explicit return in a constructor

```js
function Person(name) {
  this.name = name;
  // Returning a primitive is ignored
  return 42; // Ignored — still returns `this`
}

function PersonObj(name) {
  this.name = name;
  // Returning an object overrides `this`
  return { name: "Override" };
}

new Person("Alice").name;    // "Alice" — primitive return ignored
new PersonObj("Alice").name; // "Override" — object return wins
```

---

## 8. Arrow Functions & Lexical `this`

Arrow functions do **not have their own `this`**. They capture `this` from the **enclosing lexical scope** at the time they are defined, not called.

```js
const user = {
  name: "Alice",

  // Regular function — this is dynamic
  greetRegular: function() {
    console.log(this.name); // "Alice" when called as user.greetRegular()
  },

  // Arrow function — this is lexical (from outer scope)
  greetArrow: () => {
    console.log(this.name); // undefined — outer scope is module/global, not user
  },
};

user.greetRegular(); // "Alice"
user.greetArrow();   // undefined
```

### Where Arrow Functions Shine

Inside callbacks within a method, you often want `this` to refer to the object:

```js
class Timer {
  constructor() {
    this.seconds = 0;
  }

  start() {
    // ❌ Regular function loses `this`
    setInterval(function() {
      this.seconds++; // `this` is undefined (strict) or globalThis
    }, 1000);

    // ✅ Arrow function captures `this` from start()
    setInterval(() => {
      this.seconds++; // `this` = Timer instance ✅
    }, 1000);
  }
}
```

```js
class DataFetcher {
  constructor(url) {
    this.url = url;
    this.data = null;
  }

  async load() {
    const response = await fetch(this.url);
    const json = await response.json();

    // Arrow function preserves outer `this`
    json.items.forEach(item => {
      this.data = item; // `this` = DataFetcher instance ✅
    });
  }
}
```

### Arrow Functions Cannot be Rebound

Since arrow functions don't have their own `this`, `call`, `apply`, and `bind` cannot change it:

```js
const arrow = () => console.log(this);
const obj = { name: "Alice" };

arrow.call(obj);   // globalThis — call doesn't work on arrows
arrow.apply(obj);  // globalThis — apply doesn't work on arrows
const bound = arrow.bind(obj);
bound();           // globalThis — bind doesn't work on arrows
```

### Arrow Functions Cannot be Used with `new`

```js
const Fn = () => {};
new Fn(); // TypeError: Fn is not a constructor
```

### Where NOT to use Arrow Functions

```js
// ❌ Object method — loses object as `this`
const obj = {
  value: 42,
  getValue: () => this.value, // `this` is outer scope, not obj
};
obj.getValue(); // undefined

// ❌ Prototype method — same issue
function MyClass() { this.value = 1; }
MyClass.prototype.getValue = () => this.value; // ❌

// ❌ Event handlers where you need `this` to be the element
button.addEventListener("click", () => {
  this.classList.toggle("active"); // `this` is NOT the button
});

// ✅ Use regular function for event handlers
button.addEventListener("click", function() {
  this.classList.toggle("active"); // `this` IS the button
});
```

---

## 9. Binding in Classes

### The Problem

Class methods are not auto-bound to instances:

```js
class Counter {
  constructor() {
    this.count = 0;
  }

  increment() {
    this.count++;
    console.log(this.count);
  }
}

const counter = new Counter();
counter.increment(); // 1 ✅

const fn = counter.increment;
fn(); // TypeError: Cannot set properties of undefined ❌
```

### Solution 1: bind in Constructor

```js
class Counter {
  constructor() {
    this.count = 0;
    this.increment = this.increment.bind(this); // ✅
  }

  increment() {
    this.count++;
  }
}
```

### Solution 2: Class Field Arrow Functions (Modern, Recommended)

```js
class Counter {
  count = 0;

  // Arrow function as class field — auto-bound to instance
  increment = () => {
    this.count++;
    console.log(this.count);
  };
}

const counter = new Counter();
const fn = counter.increment;
fn(); // Works ✅ — arrow captured `this` at definition time
```

**Tradeoff:** Class field arrows create a new function **per instance** (not shared on prototype), which uses more memory for large numbers of instances.

```js
// Regular method — shared on prototype
Counter.prototype.increment; // exists

// Class field arrow — per instance
const c = new Counter();
c.hasOwnProperty("increment"); // true (class field) vs false (prototype method)
```

### Solution 3: Proxy-based Auto-binding

```js
function autoBind(instance) {
  return new Proxy(instance, {
    get(target, prop) {
      const value = target[prop];
      if (typeof value === "function") {
        return value.bind(target);
      }
      return value;
    },
  });
}

class Counter {
  constructor() {
    this.count = 0;
    return autoBind(this);
  }

  increment() { this.count++; }
}
```

### Static Methods and `this`

In static methods, `this` refers to the **class itself**, not an instance:

```js
class MathUtils {
  static PI = 3.14159;

  static circleArea(r) {
    return this.PI * r * r; // `this` = MathUtils class
  }
}

MathUtils.circleArea(5); // Works ✅
const fn = MathUtils.circleArea;
fn(5); // TypeError — `this` is undefined in strict mode
```

---

## 10. Binding Priority (Who Wins?)

When multiple binding rules could apply, this is the priority:

```
  Highest ──────────────────────────────────── Lowest
  ┌──────┬──────────────┬──────────────┬──────────────┐
  │  new │  Explicit    │  Implicit    │   Default    │
  │      │  call/apply/ │  obj.fn()    │   fn()       │
  │      │  bind        │              │              │
  └──────┴──────────────┴──────────────┴──────────────┘
     1          2              3              4
```

Arrow functions sit outside this hierarchy — they **ignore all binding rules** and always use lexical `this`.

### Priority Demonstrations

```js
// new vs explicit (bind)
function fn() { console.log(this.x); }
const bound = fn.bind({ x: "explicit" });

bound();         // "explicit" — explicit wins over default
new bound();     // undefined — new wins over bind (new creates fresh object)
```

```js
// explicit vs implicit
function fn() { console.log(this.x); }
const obj = { x: "implicit", fn };

obj.fn();               // "implicit" — implicit binding
obj.fn.call({ x: "explicit" }); // "explicit" — explicit wins over implicit
```

```js
// implicit vs default
const obj = { x: "implicit", fn() { console.log(this.x); } };
const extracted = obj.fn;

obj.fn();    // "implicit" — implicit binding
extracted(); // undefined  — default binding (lost implicit)
```

---

## 11. `call` vs `apply` vs `bind` — Deep Comparison

| Feature | `call` | `apply` | `bind` |
|---------|--------|---------|--------|
| Calls function immediately | ✅ Yes | ✅ Yes | ❌ No |
| Returns | Function result | Function result | New bound function |
| How args are passed | Individually | As array | Individually (pre-filled) |
| Can change `this` on arrows | ❌ No | ❌ No | ❌ No |
| Can be overridden by `new` | N/A | N/A | ✅ Yes, `new` wins |
| Partial application | ❌ No | ❌ No | ✅ Yes |

```js
function describe(role, city) {
  return `${this.name} is a ${role} from ${city}`;
}

const person = { name: "Alice" };

// call — immediate, args spread
describe.call(person, "developer", "NYC");
// "Alice is a developer from NYC"

// apply — immediate, args as array
describe.apply(person, ["developer", "NYC"]);
// "Alice is a developer from NYC"

// bind — returns new function, doesn't call yet
const describePerson = describe.bind(person);
describePerson("developer", "NYC");
// "Alice is a developer from NYC"

// bind with partial args
const describeAliceDev = describe.bind(person, "developer");
describeAliceDev("NYC");   // "Alice is a developer from NYC"
describeAliceDev("London"); // "Alice is a developer from London"
```

### Memory Aid

```
call  → Comma separated args, Calls immediately
apply → Array of args, Also calls immediately
bind  → Binds and returns, doesn't call (yet)
```

---

## 12. Losing `this` — The Classic Pitfalls

### Pitfall 1: Method Extracted as Variable

```js
const obj = {
  name: "Alice",
  greet() { console.log(this.name); }
};

const fn = obj.greet;
fn(); // undefined ❌ — lost binding

// Fix: bind
const fn = obj.greet.bind(obj); // ✅
// Fix: arrow wrapper
const fn = () => obj.greet();   // ✅
```

### Pitfall 2: Passed as Callback

```js
class Button {
  constructor(label) {
    this.label = label;
  }

  handleClick() {
    console.log(`${this.label} clicked`);
  }
}

const btn = new Button("Submit");

// ❌ All of these lose `this`
setTimeout(btn.handleClick, 1000);
document.addEventListener("click", btn.handleClick);
[1, 2].forEach(btn.handleClick);

// ✅ Fix: bind
setTimeout(btn.handleClick.bind(btn), 1000);
document.addEventListener("click", btn.handleClick.bind(btn));

// ✅ Fix: arrow wrapper
setTimeout(() => btn.handleClick(), 1000);

// ✅ Fix: class field arrow (defined in class)
// handleClick = () => { ... }
```

### Pitfall 3: Nested Regular Functions

```js
const obj = {
  name: "Alice",
  outer() {
    console.log(this.name); // "Alice" ✅

    function inner() {
      console.log(this.name); // undefined ❌ — inner is a plain function call
    }
    inner();
  },
};

obj.outer();

// Fix 1: capture `this` in a variable
outer() {
  const self = this; // capture
  function inner() {
    console.log(self.name); // ✅
  }
  inner();
}

// Fix 2: use arrow function for inner
outer() {
  const inner = () => console.log(this.name); // ✅
  inner();
}
```

### Pitfall 4: Destructuring Methods

```js
const api = {
  baseUrl: "https://example.com",
  async get(path) {
    return fetch(this.baseUrl + path);
  }
};

// ❌ Destructuring loses `this`
const { get } = api;
await get("/users"); // TypeError: Cannot read 'baseUrl' of undefined

// ✅ Fix: keep reference or bind
const get = api.get.bind(api);
await get("/users"); // ✅
```

### Pitfall 5: Optional Chaining and `this`

```js
const obj = {
  name: "Alice",
  nested: {
    greet() {
      console.log(this.name); // `this` = obj.nested, NOT obj
    }
  }
};

obj.nested.greet(); // undefined — `this` is obj.nested, which has no `name`
```

---

## 13. Real-World Patterns

### Event Handler Pattern

```js
class Modal {
  constructor(element) {
    this.element = element;
    this.isOpen = false;

    // Bind all event handlers once
    this._onKeydown = this.handleKeydown.bind(this);
    this._onOverlayClick = this.handleOverlayClick.bind(this);
  }

  open() {
    this.isOpen = true;
    this.element.classList.add("open");
    document.addEventListener("keydown", this._onKeydown);
    this.element.addEventListener("click", this._onOverlayClick);
  }

  close() {
    this.isOpen = false;
    this.element.classList.remove("open");
    // Can properly remove because we saved the bound reference
    document.removeEventListener("keydown", this._onKeydown);
    this.element.removeEventListener("click", this._onOverlayClick);
  }

  handleKeydown(e) {
    if (e.key === "Escape") this.close(); // `this` is the Modal instance ✅
  }

  handleOverlayClick(e) {
    if (e.target === this.element) this.close(); // ✅
  }
}
```

### Mixin Pattern

```js
const Serializable = {
  serialize() {
    return JSON.stringify(this); // `this` = object that borrows the method
  },
  deserialize(json) {
    return Object.assign(this, JSON.parse(json));
  },
};

class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
}

// Mix in methods using call
User.prototype.serialize = Serializable.serialize;
User.prototype.deserialize = Serializable.deserialize;

const user = new User("Alice", "alice@example.com");
const json = user.serialize(); // {"name":"Alice","email":"alice@example.com"}
```

### Method Borrowing

```js
// Borrow Array methods for NodeList (which is array-like)
const divs = document.querySelectorAll("div");
const divsArray = Array.prototype.slice.call(divs);
// Or modern: Array.from(divs)

// Borrow hasOwnProperty safely
// (Safer than obj.hasOwnProperty — object may have overridden it)
const hasOwn = Object.prototype.hasOwnProperty;

function safeHasOwnProperty(obj, key) {
  return hasOwn.call(obj, key);
}

const obj = Object.create(null); // No prototype!
obj.name = "Alice";
safeHasOwnProperty(obj, "name"); // true ✅
// obj.hasOwnProperty("name") would throw — no prototype
```

### Partial Application Factory

```js
function partial(fn, ...presetArgs) {
  return function(...laterArgs) {
    return fn.call(this, ...presetArgs, ...laterArgs);
    // Passes through `this` so partial works on methods too
  };
}

const obj = {
  multiplier: 3,
  multiply(a, b) {
    return this.multiplier * a * b;
  },
};

obj.double = partial(obj.multiply, 2);
obj.double(5); // 3 * 2 * 5 = 30 — `this` = obj ✅
```

### React Class Component Pattern (Historical)

```js
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };

    // Option 1: bind in constructor
    this.increment = this.increment.bind(this);
  }

  // Option 2: class field arrow (preferred modern approach)
  decrement = () => {
    this.setState(s => ({ count: s.count - 1 }));
  };

  increment() {
    this.setState(s => ({ count: s.count + 1 }));
  }

  render() {
    return (
      <div>
        <button onClick={this.decrement}>-</button>
        <span>{this.state.count}</span>
        <button onClick={this.increment}>+</button>

        {/* Option 3: inline arrow (creates new fn each render — avoid for perf) */}
        <button onClick={() => this.increment()}>+</button>
      </div>
    );
  }
}
```

### Function Composition with `this`

```js
class QueryBuilder {
  constructor(table) {
    this.table = table;
    this.conditions = [];
    this.fields = ["*"];
  }

  select(...fields) {
    this.fields = fields;
    return this; // Return `this` to enable chaining
  }

  where(condition) {
    this.conditions.push(condition);
    return this;
  }

  build() {
    const fields = this.fields.join(", ");
    const where = this.conditions.length
      ? `WHERE ${this.conditions.join(" AND ")}`
      : "";
    return `SELECT ${fields} FROM ${this.table} ${where}`.trim();
  }
}

const query = new QueryBuilder("users")
  .select("id", "name", "email")
  .where("age > 18")
  .where("active = true")
  .build();
// "SELECT id, name, email FROM users WHERE age > 18 AND active = true"
```

---

## 14. Edge Cases & Gotchas

### `this` in Getters and Setters

```js
const obj = {
  _name: "Alice",
  get name() {
    return this._name; // `this` = obj ✅
  },
  set name(val) {
    this._name = val;  // `this` = obj ✅
  },
};
```

### `this` in Computed Property Callbacks

```js
const obj = {
  value: 10,
  double() {
    return [1, 2, 3].map(function(n) {
      return n * this.value; // ❌ `this` is undefined (strict) or global
    });
  },
  doubleFixed() {
    return [1, 2, 3].map(n => n * this.value); // ✅ arrow captures outer `this`
  },
};
```

### `this` in Promise Chains

```js
class DataService {
  constructor() {
    this.data = [];
  }

  load(url) {
    return fetch(url)
      .then(function(res) {
        return res.json();
      })
      .then(function(data) {
        this.data = data; // ❌ `this` is undefined — regular function in .then()
      });
  }

  loadFixed(url) {
    return fetch(url)
      .then(res => res.json())
      .then(data => {
        this.data = data; // ✅ arrow functions preserve `this`
      });
  }
}
```

### `this` in `eval`

```js
// eval inherits the surrounding `this`
function fn() {
  eval("console.log(this)"); // Same `this` as fn's context
}
fn.call({ x: 1 }); // { x: 1 }
```

### `this` in `with` Statement

```js
// The `with` statement (avoid in all cases — strict mode forbids it)
// can affect `this` in obscure ways. Never use `with`.
```

### Nullish thisArg in call/apply

```js
// In sloppy mode, null/undefined thisArg → globalThis
function fn() { return this; }
fn.call(null);      // globalThis (sloppy)
fn.call(undefined); // globalThis (sloppy)

// In strict mode, null/undefined is preserved as-is
"use strict";
function fnStrict() { return this; }
fnStrict.call(null);      // null
fnStrict.call(undefined); // undefined
```

### Double bind — Second bind is ignored

```js
function fn() { console.log(this.x); }

const bound1 = fn.bind({ x: "first" });
const bound2 = bound1.bind({ x: "second" }); // Trying to re-bind

bound2(); // "first" — first bind wins, second is ignored
```

### bind and `length` / `name`

```js
function greet(a, b, c) {}

greet.length;            // 3
greet.name;              // "greet"

const bound = greet.bind(null, 1);
bound.length;            // 2  — reduced by pre-filled args
bound.name;              // "bound greet" — prefixed with "bound "
```

---

## 15. Quick Reference

### Determining `this` — Decision Tree

```
Is it an arrow function?
   YES → `this` from enclosing lexical scope (defined at creation, never changes)
   NO  → continue...

Was it called with `new`?
   YES → `this` = newly created object
   NO  → continue...

Was it called with call / apply / bind?
   YES → `this` = first argument (thisArg)
   NO  → continue...

Was it called as a method? (obj.fn())
   YES → `this` = the object before the dot
   NO  → continue...

Default:
   Strict mode  → `this` = undefined
   Sloppy mode  → `this` = globalThis (window / global)
```

### Cheat Sheet

```js
// call — immediate, spread args
fn.call(thisArg, arg1, arg2)

// apply — immediate, array args
fn.apply(thisArg, [arg1, arg2])

// bind — returns new fn, doesn't call
const bound = fn.bind(thisArg, arg1) // arg1 pre-filled
bound(arg2)

// Arrow — inherits outer this, ignores call/apply/bind
const arrow = () => this

// new — this = new object
const obj = new Fn()

// Implicit — this = calling object
obj.fn()

// Default — undefined (strict) or globalThis (sloppy)
fn()
```

### Common Fixes for `this` Loss

| Situation | Fix |
|-----------|-----|
| Method passed as callback | `.bind(obj)` or `() => obj.method()` |
| Method in setTimeout | `.bind(this)` or arrow wrapper |
| Method in forEach/map | Arrow function or `.bind(this)` |
| React event handler | Class field arrow or bind in constructor |
| Nested function inside method | Arrow function or `const self = this` |
| Destructured method | `const { fn } = obj; fn.bind(obj)` |

---

## Further Reading

- [MDN: this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)
- [MDN: Function.prototype.call()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)
- [MDN: Function.prototype.apply()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)
- [MDN: Function.prototype.bind()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
- [MDN: Arrow Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
- [You Don't Know JS: this & Object Prototypes](https://github.com/getify/You-Dont-Know-JS/tree/1st-ed/this%20%26%20object%20prototypes)
- [JavaScript.info: Object methods, "this"](https://javascript.info/object-methods)

---

*All examples use modern JavaScript and are valid in strict mode unless otherwise noted.*