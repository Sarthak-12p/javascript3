# The Magic of Prototypes & Implementing `new` in JavaScript — In Depth

> A complete reference covering the prototype chain, `[[Prototype]]`, `Object.create`, constructor functions, `class` sugar, implementing `new` from scratch, inheritance patterns, and the full object model.

---

## Table of Contents

1. [What is a Prototype?](#1-what-is-a-prototype)
2. [The Prototype Chain](#2-the-prototype-chain)
3. [`[[Prototype]]` — The Internal Slot](#3-prototype--the-internal-slot)
4. [`prototype` vs `__proto__` vs `[[Prototype]]`](#4-prototype-vs-__proto__-vs-prototype)
5. [Constructor Functions](#5-constructor-functions)
6. [How `new` Works — Step by Step](#6-how-new-works--step-by-step)
7. [Implementing `new` from Scratch](#7-implementing-new-from-scratch)
8. [`Object.create` — Pure Prototypal Inheritance](#8-objectcreate--pure-prototypal-inheritance)
9. [Property Lookup — The Full Algorithm](#9-property-lookup--the-full-algorithm)
10. [Property Descriptors & the Prototype Chain](#10-property-descriptors--the-prototype-chain)
11. [ES6 Classes — Syntactic Sugar](#11-es6-classes--syntactic-sugar)
12. [Inheritance with Prototypes](#12-inheritance-with-prototypes)
13. [Implementing Classical Inheritance from Scratch](#13-implementing-classical-inheritance-from-scratch)
14. [Introspection & Prototype Utilities](#14-introspection--prototype-utilities)
15. [Built-in Prototype Chain](#15-built-in-prototype-chain)
16. [Extending Built-ins — Safely](#16-extending-built-ins--safely)
17. [Common Pitfalls](#17-common-pitfalls)
18. [Real-World Patterns](#18-real-world-patterns)
19. [Quick Reference](#19-quick-reference)

---

## 1. What is a Prototype?

In JavaScript, **every object can have a parent object** — called its **prototype**. When you try to access a property on an object, and it doesn't exist on that object directly, JavaScript automatically looks at its prototype, then the prototype's prototype, and so on — until it either finds the property or reaches the end of the chain (`null`).

This mechanism is called **prototypal inheritance** — and it is the foundational object model of JavaScript. Everything else (`class`, `extends`, `new`) is built on top of it.

```
┌─────────────────────────────────────────────────────────────────┐
│                    Prototypal Inheritance                       │
│                                                                 │
│  object ──[[Prototype]]──▶ parent ──[[Prototype]]──▶ null      │
│                                                                 │
│  "If I don't have it, ask my parent.                           │
│   If my parent doesn't have it, ask their parent.             │
│   Keep going until null — then return undefined."             │
└─────────────────────────────────────────────────────────────────┘
```

### JavaScript vs Classical Inheritance

| Classical (Java, C++) | Prototypal (JavaScript) |
|-----------------------|-------------------------|
| Classes are blueprints | Objects delegate to other objects |
| Instances are copies of a class | Instances share a live reference |
| Inheritance is static | Prototype chain is dynamic |
| `extends` copies methods | `extends` links prototypes |
| Objects created from classes | Objects created from objects |

JavaScript's `class` keyword is purely **syntactic sugar** — under the hood, it's still prototype delegation.

---

## 2. The Prototype Chain

```js
const animal = {
  breathe() {
    return `${this.name} breathes air`;
  },
};

const dog = {
  bark() {
    return `${this.name} says Woof!`;
  },
};

const myDog = {
  name: "Rex",
};

// Set up chain: myDog → dog → animal → Object.prototype → null
Object.setPrototypeOf(dog,   animal);
Object.setPrototypeOf(myDog, dog);

myDog.name;     // "Rex"        — found on myDog itself
myDog.bark();   // "Rex says Woof!" — found on dog (1 level up)
myDog.breathe();// "Rex breathes air" — found on animal (2 levels up)
myDog.toString();// "[object Object]" — found on Object.prototype (3 levels up)
myDog.xyz;      // undefined    — not found anywhere in chain
```

### Visualizing the Chain

```
myDog  { name: "Rex" }
  │
  │ [[Prototype]]
  ▼
dog    { bark() }
  │
  │ [[Prototype]]
  ▼
animal { breathe() }
  │
  │ [[Prototype]]
  ▼
Object.prototype { toString(), hasOwnProperty(), valueOf(), ... }
  │
  │ [[Prototype]]
  ▼
null  ← End of chain
```

---

## 3. `[[Prototype]]` — The Internal Slot

Every JavaScript object has an internal (hidden) slot called `[[Prototype]]`. It either points to another object or is `null`.

You cannot access `[[Prototype]]` directly — it's a spec concept. JavaScript exposes it through:

| Method | Description |
|--------|-------------|
| `Object.getPrototypeOf(obj)` | **Read** the prototype (preferred) |
| `Object.setPrototypeOf(obj, proto)` | **Write** the prototype (avoid in hot paths) |
| `obj.__proto__` | Legacy accessor (deprecated, avoid) |

```js
const parent = { x: 1 };
const child  = Object.create(parent); // child.[[Prototype]] = parent

Object.getPrototypeOf(child) === parent; // true
child.__proto__ === parent;              // true (legacy — avoid)
```

### Why `__proto__` is Deprecated

```js
// __proto__ is a getter/setter on Object.prototype
// It can be shadowed or break on null-prototype objects

const obj = Object.create(null); // No prototype, no __proto__!
obj.__proto__; // undefined — property doesn't exist (not the internal slot)

// Use the official API always:
Object.getPrototypeOf(obj); // null ✅
Object.setPrototypeOf(obj, someProto); // ✅
```

---

## 4. `prototype` vs `__proto__` vs `[[Prototype]]`

This is one of the most confusing parts of JavaScript. Three distinct things share similar names:

```
┌───────────────────┬────────────────────────────────────────────────────┐
│ Term              │ What it is                                         │
├───────────────────┼────────────────────────────────────────────────────┤
│ [[Prototype]]     │ Internal spec slot. The actual parent object.      │
│                   │ Every object has one. Not directly accessible.     │
├───────────────────┼────────────────────────────────────────────────────┤
│ __proto__         │ Legacy getter/setter on Object.prototype that      │
│                   │ reads/writes [[Prototype]]. Deprecated. Avoid.     │
├───────────────────┼────────────────────────────────────────────────────┤
│ .prototype        │ A regular property on FUNCTION objects.            │
│                   │ Becomes [[Prototype]] of objects created with new. │
│                   │ NOT the function's own [[Prototype]].              │
└───────────────────┴────────────────────────────────────────────────────┘
```

```js
function Dog(name) {
  this.name = name;
}

Dog.prototype.bark = function() {
  return `${this.name} barks!`;
};

const rex = new Dog("Rex");

// Dog.prototype  — the object that rex delegates to
// rex.[[Prototype]] === Dog.prototype  — the link on the instance
// Dog.[[Prototype]] === Function.prototype  — Dog is itself a function object

Dog.prototype;                          // { bark: fn, constructor: Dog }
Object.getPrototypeOf(rex);             // { bark: fn, constructor: Dog }
Dog.prototype === Object.getPrototypeOf(rex); // true

// Dog's OWN prototype (as a function) is Function.prototype
Object.getPrototypeOf(Dog);             // Function.prototype
```

### The Diagram

```
        ┌─────────────┐          .prototype          ┌───────────────────────┐
        │  Dog (fn)   │ ─────────────────────────▶  │    Dog.prototype       │
        │             │                              │  { bark, constructor } │
        └─────────────┘                              └───────────────────────┘
               │                                              ▲
               │ [[Prototype]]                                │ [[Prototype]] of rex
               ▼                                              │
        Function.prototype                         ┌──────────────────┐
                                                   │   rex (instance) │
                                                   │   { name: "Rex" }│
                                                   └──────────────────┘
```

---

## 5. Constructor Functions

Before `class`, constructor functions were the standard way to create objects with shared methods.

```js
function Person(name, age) {
  // `this` is the newly created object (when called with `new`)
  this.name = name;
  this.age  = age;
}

// Methods go on the prototype — shared across all instances
Person.prototype.greet = function() {
  return `Hi, I'm ${this.name}, ${this.age} years old.`;
};

Person.prototype.birthday = function() {
  this.age++;
  return this.age;
};

const alice = new Person("Alice", 30);
const bob   = new Person("Bob",   25);

alice.greet();   // "Hi, I'm Alice, 30 years old."
bob.greet();     // "Hi, I'm Bob, 25 years old."

// Methods are SHARED — not duplicated per instance
alice.greet === bob.greet; // true ✅ — same function on prototype
```

### Why Methods go on `.prototype`, Not Inside the Constructor

```js
// ❌ Bad: method recreated for every instance — wastes memory
function PersonBad(name) {
  this.name = name;
  this.greet = function() {  // New function object created for EACH instance
    return `Hi, I'm ${this.name}`;
  };
}

const a = new PersonBad("Alice");
const b = new PersonBad("Bob");
a.greet === b.greet; // false — two separate function objects!

// ✅ Good: method on prototype — shared by all instances
function PersonGood(name) {
  this.name = name;
}
PersonGood.prototype.greet = function() {
  return `Hi, I'm ${this.name}`;
};

const c = new PersonGood("Alice");
const d = new PersonGood("Bob");
c.greet === d.greet; // true — same function object ✅
```

### The `constructor` Property

Every function's `.prototype` automatically gets a `constructor` property pointing back to the function:

```js
function Dog(name) { this.name = name; }

Dog.prototype.constructor === Dog; // true

const rex = new Dog("Rex");
rex.constructor === Dog; // true (found via prototype chain)

// Use for dynamic construction
const anotherDog = new rex.constructor("Max");
anotherDog instanceof Dog; // true
```

**Warning:** Replacing `prototype` entirely breaks `constructor`:

```js
// ❌ Bad — loses constructor reference
Dog.prototype = {
  bark() { return "Woof"; },
};
Dog.prototype.constructor === Dog; // false — now points to Object

// ✅ Fix — restore it
Dog.prototype = {
  constructor: Dog,   // Manually restore
  bark() { return "Woof"; },
};
```

---

## 6. How `new` Works — Step by Step

When you write `new Fn(arg1, arg2)`, JavaScript does exactly 5 things:

```
Step 1: Create a new empty object
        obj = {}

Step 2: Set its [[Prototype]] to Fn.prototype
        obj.[[Prototype]] = Fn.prototype

Step 3: Execute Fn with `this` = obj
        Fn.call(obj, arg1, arg2)

Step 4: If Fn explicitly returns an object, use that.
        Otherwise, return obj.

Step 5: (The result from step 4 is the value of `new Fn(...)`)
```

### Visualized

```
new Person("Alice", 30)
        │
        ▼
┌────────────────────────────────────────────────────┐
│  1. obj = Object.create(Person.prototype)          │
│           ↳ creates {}  with [[Prototype]] linked  │
│                                                    │
│  2. result = Person.call(obj, "Alice", 30)         │
│           ↳ runs constructor, sets this.name etc.  │
│                                                    │
│  3. return (result is object?) result : obj        │
│           ↳ since Person returns undefined,        │
│             return obj                             │
└────────────────────────────────────────────────────┘
        │
        ▼
  { name: "Alice", age: 30 }
  with [[Prototype]] → Person.prototype
```

### The Return Value Rule

```js
// Returning a primitive → ignored, `this` is returned
function A() {
  this.x = 1;
  return 42;        // primitive — ignored
}
new A(); // { x: 1 }

// Returning an object → that object is returned instead
function B() {
  this.x = 1;
  return { y: 2 }; // object — returned instead of `this`
}
new B(); // { y: 2 }

// Returning null → ignored (null is object type but treated like primitive)
function C() {
  this.x = 1;
  return null;     // ignored
}
new C(); // { x: 1 }
```

---

## 7. Implementing `new` from Scratch

Now that we understand the steps, let's implement `new` ourselves:

### Basic Implementation

```js
function myNew(Constructor, ...args) {
  // Step 1 & 2: Create object with [[Prototype]] set to Constructor.prototype
  const obj = Object.create(Constructor.prototype);

  // Step 3: Call constructor with `this` = our new object
  const result = Constructor.call(obj, ...args);

  // Step 4: If constructor returned an object, use it; otherwise use obj
  return (result !== null && typeof result === "object") ? result : obj;
}
```

### Testing It

```js
function Person(name, age) {
  this.name = name;
  this.age  = age;
}

Person.prototype.greet = function() {
  return `Hi, I'm ${this.name}`;
};

// Using built-in new
const alice = new Person("Alice", 30);
alice.greet();              // "Hi, I'm Alice"
alice instanceof Person;    // true

// Using our myNew
const bob = myNew(Person, "Bob", 25);
bob.greet();                // "Hi, I'm Bob"
bob instanceof Person;      // true
Object.getPrototypeOf(bob) === Person.prototype; // true
```

### Handling the Return Object Case

```js
function myNew(Constructor, ...args) {
  const obj = Object.create(Constructor.prototype);
  const result = Constructor.call(obj, ...args);

  // Only return result if it's a non-null object
  if (result !== null && typeof result === "object") {
    return result;
  }
  return obj;
}

// Test with a constructor that returns an object
function Overrider() {
  this.from = "constructor";
  return { from: "return value" }; // Returns object
}

const o = myNew(Overrider);
o.from; // "return value" — returned object wins
```

### Extended Implementation (handles edge cases)

```js
function myNew(Constructor, ...args) {
  // Guard: ensure Constructor is actually a function
  if (typeof Constructor !== "function") {
    throw new TypeError(`${Constructor} is not a constructor`);
  }

  // Create the new object, linked to Constructor.prototype
  // If Constructor.prototype is not an object (e.g., it was set to null),
  // fall back to Object.prototype
  const proto = Constructor.prototype;
  const obj   = Object.create(
    proto !== null && typeof proto === "object" ? proto : Object.prototype
  );

  // Execute the constructor
  const result = Constructor.call(obj, ...args);

  // Return object result or the newly created object
  return (result !== null && typeof result === "object") ? result : obj;
}
```

### Simulating `instanceof` as well

```js
function myInstanceOf(obj, Constructor) {
  // instanceof walks the prototype chain looking for Constructor.prototype
  let proto = Object.getPrototypeOf(obj);

  while (proto !== null) {
    if (proto === Constructor.prototype) return true;
    proto = Object.getPrototypeOf(proto);
  }

  return false;
}

const alice = myNew(Person, "Alice", 30);
myInstanceOf(alice, Person); // true
myInstanceOf(alice, Object); // true — Person.prototype's chain includes Object.prototype
myInstanceOf(alice, Array);  // false
```

---

## 8. `Object.create` — Pure Prototypal Inheritance

`Object.create(proto)` creates a new object whose `[[Prototype]]` is exactly `proto`. It's the most direct way to set up prototypal delegation.

```js
const vehicleProto = {
  describe() {
    return `${this.make} ${this.model} (${this.year})`;
  },
  start() {
    return `${this.make} engine started`;
  },
};

// Create an object that delegates to vehicleProto
const car = Object.create(vehicleProto);
car.make  = "Toyota";
car.model = "Camry";
car.year  = 2024;

car.describe(); // "Toyota Camry (2024)"
car.start();    // "Toyota engine started"

Object.getPrototypeOf(car) === vehicleProto; // true
```

### `Object.create(null)` — Prototype-free Objects

```js
// Creates a truly empty object — no toString, no hasOwnProperty, etc.
const dict = Object.create(null);
dict.name  = "Alice";
dict.age   = 30;

dict.toString;         // undefined — no Object.prototype!
dict.hasOwnProperty;   // undefined

// Useful for:
// 1. Safe dictionaries (no prototype pollution)
// 2. Key-value stores without inherited property conflicts
// 3. Performance in tight loops (no prototype chain lookup)

// Check for own properties safely
Object.prototype.hasOwnProperty.call(dict, "name"); // true
// Or use Object.hasOwn (ES2022)
Object.hasOwn(dict, "name"); // true
```

### `Object.create` with Property Descriptors

```js
const obj = Object.create(
  someProto,         // First arg: prototype
  {                  // Second arg: property descriptors
    name: {
      value: "Alice",
      writable: true,
      enumerable: true,
      configurable: true,
    },
    id: {
      value: 1,
      writable: false,   // Read-only
      enumerable: false, // Hidden from for...in, Object.keys
      configurable: false,
    },
  }
);
```

### Implementing Object.create from Scratch

```js
function myObjectCreate(proto) {
  if (proto !== null && typeof proto !== "object" && typeof proto !== "function") {
    throw new TypeError("proto must be an object or null");
  }

  function F() {} // Temporary empty constructor
  F.prototype = proto;
  return new F();
}

// ES5 polyfill used exactly this technique
const child = myObjectCreate(parent);
Object.getPrototypeOf(child) === parent; // true
```

---

## 9. Property Lookup — The Full Algorithm

When you access `obj.prop`, JavaScript runs this exact algorithm:

```
1. Is `prop` an OWN property of obj?
   ─ Found (data descriptor): return its value
   ─ Found (accessor descriptor): call getter, return result
   ─ Not found: go to step 2

2. Does obj have a [[Prototype]]?
   ─ No (proto is null): return undefined
   ─ Yes: set obj = obj.[[Prototype]], go to step 1

(Repeat until found or null)
```

```js
const A = { x: 1 };
const B = Object.create(A); // B.[[Prototype]] = A
const C = Object.create(B); // C.[[Prototype]] = B

C.x;   // Lookup: C? No → B? No → A? YES → 1
C.y;   // Lookup: C? No → B? No → A? No → null → undefined
```

### Shadowing

When an object has a property with the same name as one in its prototype chain, the own property **shadows** the prototype's:

```js
const parent = { x: 1, greet() { return "parent"; } };
const child  = Object.create(parent);

child.x;       // 1 — found on parent
child.x = 99;  // Adds x directly to child (shadows parent.x)
child.x;       // 99 — found on child (shadow)
parent.x;      // 1 — parent unchanged

// Delete the shadow to reveal parent's property
delete child.x;
child.x;       // 1 — back to parent's value
```

### Shadowing with Prototype Setters

```js
const proto = {
  get value() { return this._value; },
  set value(v) { this._value = v * 2; }, // setter doubles it
};

const obj = Object.create(proto);
obj.value = 10;  // Calls proto's setter — obj._value = 20
obj.value;       // Calls proto's getter — returns obj._value = 20
```

---

## 10. Property Descriptors & the Prototype Chain

Every property has a **descriptor** controlling its behavior:

```js
// Data descriptor
Object.defineProperty(obj, "name", {
  value: "Alice",
  writable: true,      // Can be reassigned?
  enumerable: true,    // Shows in for...in, Object.keys?
  configurable: true,  // Can be deleted/redefined?
});

// Accessor descriptor
Object.defineProperty(obj, "fullName", {
  get() { return `${this.firstName} ${this.lastName}`; },
  set(v) { [this.firstName, this.lastName] = v.split(" "); },
  enumerable: true,
  configurable: true,
});
```

### Non-writable Properties and Prototype Shadowing

If a property in the chain is **non-writable**, you cannot create a shadow on the child in non-strict mode (silently fails); strict mode throws:

```js
const proto = {};
Object.defineProperty(proto, "x", { value: 1, writable: false });

const child = Object.create(proto);

// Sloppy mode: silently fails
child.x = 99;
child.x; // 1 — shadow was NOT created!

// Strict mode: throws TypeError
"use strict";
child.x = 99; // TypeError: Cannot assign to read only property 'x'

// Only way to shadow: Object.defineProperty directly
Object.defineProperty(child, "x", { value: 99, writable: true });
child.x; // 99
```

### Inspecting Descriptors

```js
function Person(name) { this.name = name; }
Person.prototype.greet = function() {};

const alice = new Person("Alice");

// Own property
Object.getOwnPropertyDescriptor(alice, "name");
// { value: "Alice", writable: true, enumerable: true, configurable: true }

// Prototype property
Object.getOwnPropertyDescriptor(Person.prototype, "greet");
// { value: [Function], writable: true, enumerable: true, configurable: true }

// Listing all own property names (including non-enumerable)
Object.getOwnPropertyNames(alice);          // ["name"]
Object.getOwnPropertyNames(Person.prototype); // ["constructor", "greet"]
```

---

## 11. ES6 Classes — Syntactic Sugar

`class` is 100% built on prototypes. It provides cleaner syntax but changes nothing fundamentally.

```js
class Person {
  constructor(name, age) {
    this.name = name;
    this.age  = age;
  }

  greet() {                   // → Person.prototype.greet
    return `Hi, I'm ${this.name}`;
  }

  static create(name, age) {  // → Person.create (on the constructor itself)
    return new Person(name, age);
  }
}

// These are IDENTICAL:
typeof Person;                // "function" — class IS a function
Person.prototype.greet;       // the greet method
Object.getPrototypeOf(new Person("Alice", 30)) === Person.prototype; // true
```

### Class vs Constructor Function — The Differences

```js
// 1. Classes are NOT hoisted like function declarations
// new MyClass(); // ReferenceError — in temporal dead zone
class MyClass {}

// 2. Class bodies are always in strict mode
class Strict {
  method() { /* always strict */ }
}

// 3. Class methods are non-enumerable
class C { method() {} }
for (const k in new C()) { }          // method NOT iterated ✅
// vs constructor function:
function D() {}
D.prototype.method = function() {};
for (const k in new D()) { console.log(k); } // "method" IS iterated

// 4. Classes cannot be called without `new`
class E {}
E(); // TypeError: Class constructor E cannot be invoked without 'new'
// Constructor functions can be called without new (unsafe)
```

### Private Fields (ES2022)

```js
class BankAccount {
  #balance = 0;      // Private field — truly private, not on prototype
  #owner;

  constructor(owner, initial = 0) {
    this.#owner   = owner;
    this.#balance = initial;
  }

  deposit(amount) {
    if (amount <= 0) throw new Error("Amount must be positive");
    this.#balance += amount;
    return this;
  }

  get balance() { return this.#balance; }

  toString() {
    return `${this.#owner}: $${this.#balance}`;
  }
}

const acc = new BankAccount("Alice", 1000);
acc.deposit(500).deposit(200);
acc.balance;    // 1700
acc.#balance;   // SyntaxError — private, truly inaccessible outside class
```

---

## 12. Inheritance with Prototypes

### The Prototype Chain for Inheritance

```js
function Animal(name) {
  this.name = name;
}
Animal.prototype.breathe = function() {
  return `${this.name} breathes`;
};

function Dog(name, breed) {
  Animal.call(this, name); // Step 1: Call parent constructor
  this.breed = breed;
}

// Step 2: Set up prototype chain
Dog.prototype = Object.create(Animal.prototype);

// Step 3: Restore constructor (Object.create doesn't set it)
Dog.prototype.constructor = Dog;

// Step 4: Add Dog-specific methods
Dog.prototype.bark = function() {
  return `${this.name} barks!`;
};

const rex = new Dog("Rex", "Labrador");

rex.breathe(); // "Rex breathes" — from Animal.prototype
rex.bark();    // "Rex barks!"   — from Dog.prototype
rex instanceof Dog;    // true
rex instanceof Animal; // true
```

### Prototype Chain Diagram for Inheritance

```
rex (instance)
  { name: "Rex", breed: "Labrador" }
        │
        │ [[Prototype]]
        ▼
Dog.prototype
  { bark, constructor: Dog }
        │
        │ [[Prototype]]
        ▼
Animal.prototype
  { breathe, constructor: Animal }
        │
        │ [[Prototype]]
        ▼
Object.prototype
  { toString, hasOwnProperty, ... }
        │
        │ [[Prototype]]
        ▼
      null
```

### ES6 Class Inheritance

```js
class Animal {
  constructor(name) {
    this.name = name;
  }

  breathe() {
    return `${this.name} breathes`;
  }

  toString() {
    return `Animal(${this.name})`;
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);        // MUST call super before using `this`
    this.breed = breed;
  }

  bark() {
    return `${this.name} barks!`;
  }

  toString() {
    return `Dog(${this.name}, ${this.breed})`; // Override
  }
}

class GuideDog extends Dog {
  constructor(name, breed, owner) {
    super(name, breed);
    this.owner = owner;
  }

  guide() {
    return `${this.name} guides ${this.owner}`;
  }
}

const buddy = new GuideDog("Buddy", "Golden Retriever", "Alice");
buddy.breathe(); // "Buddy breathes"   — Animal.prototype
buddy.bark();    // "Buddy barks!"     — Dog.prototype
buddy.guide();   // "Buddy guides Alice" — GuideDog.prototype
```

### Calling Super Methods

```js
class Shape {
  area() { return 0; }
  describe() { return `Area: ${this.area()}`; }
}

class Circle extends Shape {
  constructor(r) {
    super();
    this.r = r;
  }

  area() {
    return Math.PI * this.r ** 2;
  }

  describe() {
    return `Circle. ${super.describe()}`; // Call parent's describe
  }
}

new Circle(5).describe();
// "Circle. Area: 78.53981633974483"
```

---

## 13. Implementing Classical Inheritance from Scratch

Let's build the full inheritance machinery that `class`/`extends` uses internally:

```js
// Step 1: Implement `new`
function myNew(Constructor, ...args) {
  const obj    = Object.create(Constructor.prototype);
  const result = Constructor.call(obj, ...args);
  return (result !== null && typeof result === "object") ? result : obj;
}

// Step 2: Implement `extends` (prototype chain linkage)
function myExtends(Child, Parent) {
  // Link Child.prototype → Parent.prototype
  Child.prototype = Object.create(Parent.prototype);
  // Restore constructor
  Child.prototype.constructor = Child;
  // Link Child → Parent (for static inheritance: Child.staticMethod)
  Object.setPrototypeOf(Child, Parent);
}

// Step 3: Implement `super()` (parent constructor call)
function mySuper(Child, instance, ...args) {
  const Parent = Object.getPrototypeOf(Child);
  Parent.call(instance, ...args);
}

// Step 4: Use it all together
function Animal(name) {
  this.name = name;
}
Animal.prototype.speak = function() {
  return `${this.name} makes a sound`;
};

function Cat(name, indoor) {
  mySuper(Cat, this, name);   // super(name)
  this.indoor = indoor;
}
myExtends(Cat, Animal);       // extends Animal

Cat.prototype.speak = function() {
  // Access "super.speak()" manually
  const parentSpeak = Animal.prototype.speak.call(this);
  return `${parentSpeak} (meow)`;
};

const kitty = myNew(Cat, "Whiskers", true);
kitty.speak();            // "Whiskers makes a sound (meow)"
kitty instanceof Cat;     // via myInstanceOf
kitty.name;               // "Whiskers"
kitty.indoor;             // true
```

### Full Class Desugaring

Here's exactly what `class Dog extends Animal { ... }` compiles to:

```js
// ES6 class:
class Dog extends Animal {
  constructor(name, breed) {
    super(name);
    this.breed = breed;
  }
  bark() { return `${this.name} barks`; }
  static fromObj(obj) { return new Dog(obj.name, obj.breed); }
}

// ─────────── Desugared to: ───────────

// 1. Constructor function
function Dog(name, breed) {
  Animal.call(this, name); // super(name)
  this.breed = breed;
}

// 2. Instance method on prototype (non-enumerable)
Object.defineProperty(Dog.prototype, "bark", {
  value: function bark() { return `${this.name} barks`; },
  writable: true,
  configurable: true,
  enumerable: false,  // ← key difference from manual assignment
});

// 3. Static method on the constructor
Object.defineProperty(Dog, "fromObj", {
  value: function fromObj(obj) { return new Dog(obj.name, obj.breed); },
  writable: true,
  configurable: true,
  enumerable: false,
});

// 4. Prototype chain for instances: Dog.prototype → Animal.prototype
Object.setPrototypeOf(Dog.prototype, Animal.prototype);

// 5. Prototype chain for statics: Dog → Animal (static inheritance)
Object.setPrototypeOf(Dog, Animal);
```

---

## 14. Introspection & Prototype Utilities

### Checking the Chain

```js
const alice = new Person("Alice");

// instanceof — walks the prototype chain
alice instanceof Person; // true
alice instanceof Object; // true

// isPrototypeOf — more flexible than instanceof
Person.prototype.isPrototypeOf(alice); // true
Object.prototype.isPrototypeOf(alice); // true

// Get the exact prototype
Object.getPrototypeOf(alice); // Person.prototype

// Check own vs inherited property
alice.hasOwnProperty("name");  // true  — own
alice.hasOwnProperty("greet"); // false — inherited from prototype
Object.hasOwn(alice, "name");  // true  (ES2022, preferred)
```

### Enumerating Properties

```js
const obj = Object.create({ inherited: true });
obj.own = true;
Object.defineProperty(obj, "hidden", { value: true, enumerable: false });

// Own enumerable keys
Object.keys(obj);               // ["own"]

// Own + inherited enumerable keys
for (const key in obj) console.log(key); // "own", "inherited"

// All own keys (including non-enumerable)
Object.getOwnPropertyNames(obj); // ["own", "hidden"]

// Own enumerable + non-enumerable + symbols
Object.getOwnPropertyDescriptors(obj);

// Check own property (any)
Object.hasOwn(obj, "hidden");   // true
Object.hasOwn(obj, "inherited"); // false
```

### Walking the Chain

```js
function getFullChain(obj) {
  const chain = [];
  let current = obj;

  while (current !== null) {
    chain.push({
      object: current,
      ownKeys: Object.getOwnPropertyNames(current),
    });
    current = Object.getPrototypeOf(current);
  }

  return chain;
}

// All properties accessible on obj (own + inherited)
function getAllProperties(obj) {
  const props = new Set();
  let current = obj;

  while (current !== null) {
    Object.getOwnPropertyNames(current).forEach(k => props.add(k));
    current = Object.getPrototypeOf(current);
  }

  return [...props];
}
```

---

## 15. Built-in Prototype Chain

Every JavaScript built-in follows the same prototype pattern:

```
┌──────────────────────────────────────────────────────────────┐
│  [1, 2, 3]                                                   │
│      │ [[Prototype]]                                         │
│      ▼                                                       │
│  Array.prototype  { push, pop, map, filter, ... }           │
│      │ [[Prototype]]                                         │
│      ▼                                                       │
│  Object.prototype { toString, hasOwnProperty, ... }         │
│      │ [[Prototype]]                                         │
│      ▼                                                       │
│    null                                                      │
└──────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│  function fn() {}                                            │
│      │ [[Prototype]]                                         │
│      ▼                                                       │
│  Function.prototype  { call, apply, bind, ... }             │
│      │ [[Prototype]]                                         │
│      ▼                                                       │
│  Object.prototype                                            │
│      │ [[Prototype]]                                         │
│      ▼                                                       │
│    null                                                      │
└──────────────────────────────────────────────────────────────┘
```

```js
// Proof
Object.getPrototypeOf([])           === Array.prototype;    // true
Object.getPrototypeOf(Array.prototype) === Object.prototype; // true
Object.getPrototypeOf(Object.prototype) === null;            // true

Object.getPrototypeOf(function(){})  === Function.prototype; // true
Object.getPrototypeOf(/regex/)       === RegExp.prototype;   // true
Object.getPrototypeOf(new Map())     === Map.prototype;       // true
```

### Function.prototype is a Function

```js
// Fascinating: Function.prototype is itself callable (it's a no-op function)
typeof Function.prototype; // "function"
Function.prototype();      // undefined (no-op, doesn't throw)

// And Function itself:
Object.getPrototypeOf(Function) === Function.prototype; // true
// Function is its own "instance"!
```

---

## 16. Extending Built-ins — Safely

### Do NOT Modify Built-in Prototypes

```js
// ❌ NEVER do this — breaks third-party code and future JS features
Array.prototype.last = function() { return this[this.length - 1]; };
String.prototype.truncate = function(n) { return this.slice(0, n) + "..."; };
```

### Safe Alternatives

```js
// ✅ 1. Standalone utility functions (preferred)
function last(arr) { return arr[arr.length - 1]; }
function truncate(str, n) { return str.slice(0, n) + "..."; }

// ✅ 2. Wrap in a utility module/class
class Arr {
  constructor(array) { this._arr = array; }
  last() { return this._arr[this._arr.length - 1]; }
  static from(arr) { return new Arr(arr); }
}

Arr.from([1, 2, 3]).last(); // 3

// ✅ 3. Extend with a subclass
class SuperArray extends Array {
  last() { return this[this.length - 1]; }
  first() { return this[0]; }
  sum() { return this.reduce((a, b) => a + b, 0); }
}

const arr = new SuperArray(1, 2, 3, 4);
arr.last();  // 4
arr.first(); // 1
arr.sum();   // 10
arr.map(x => x * 2); // SuperArray [2, 4, 6, 8] — map returns SuperArray!
```

### Symbol.species — Control What Subclass Methods Return

```js
class MyArray extends Array {
  // Make map/filter return plain Array, not MyArray
  static get [Symbol.species]() { return Array; }

  sum() { return this.reduce((a, b) => a + b, 0); }
}

const a = new MyArray(1, 2, 3);
a instanceof MyArray;          // true
a.map(x => x).constructor;    // Array (due to Symbol.species)
```

---

## 17. Common Pitfalls

### Pitfall 1: Shared Mutable State on Prototype

```js
// ❌ Dangerous — array is shared across ALL instances
function Team(name) {
  this.name = name;
}
Team.prototype.members = []; // Shared reference!

const a = new Team("Alpha");
const b = new Team("Beta");

a.members.push("Alice");
b.members; // ["Alice"] ❌ — shared!

// ✅ Fix: initialize in constructor
function Team(name) {
  this.name    = name;
  this.members = []; // Own property per instance
}
```

### Pitfall 2: Replacing `prototype` After Creating Instances

```js
function Dog(name) { this.name = name; }
const rex = new Dog("Rex");

// ❌ Replacing prototype severs connection to existing instances
Dog.prototype = { bark() { return "Woof!"; } };

rex.bark(); // TypeError — rex still points to OLD Dog.prototype

// ✅ Extend the existing prototype, don't replace it
Dog.prototype.bark = function() { return "Woof!"; };
rex.bark(); // "Woof!" ✅
```

### Pitfall 3: Forgetting `new`

```js
function Person(name) {
  this.name = name; // Without new, `this` = globalThis (sloppy) or error (strict)
}

const alice = Person("Alice"); // No new!
alice;       // undefined
window.name; // "Alice" — polluted global! ❌

// Guard against accidental non-new calls
function PersonSafe(name) {
  if (!(this instanceof PersonSafe)) {
    return new PersonSafe(name); // Auto-correct
  }
  this.name = name;
}

// Or use classes — they enforce `new`
class PersonClass {
  constructor(name) { this.name = name; }
}
PersonClass(); // TypeError ✅ enforced
```

### Pitfall 4: `instanceof` Across Realms

```js
// Each iframe/worker has its own Array constructor
// So instanceof fails across realms
const iframe  = document.createElement("iframe");
document.body.appendChild(iframe);
const iframeArray = iframe.contentWindow.Array;

const arr = new iframeArray();
arr instanceof Array;       // false ❌ — different Array constructor
Array.isArray(arr);         // true  ✅ — works across realms
```

### Pitfall 5: `Object.create` vs `new` Confusion

```js
function Dog(name) { this.name = name; }

// new — runs constructor, sets properties
const a = new Dog("Rex");
a.name; // "Rex" ✅

// Object.create — only sets prototype, does NOT run constructor
const b = Object.create(Dog.prototype);
b.name; // undefined ❌ — constructor was never called

// If you want Object.create + constructor behavior, do both:
const c = Object.create(Dog.prototype);
Dog.call(c, "Max"); // Manually call constructor
c.name; // "Max" ✅ — this is what `new` does under the hood
```

---

## 18. Real-World Patterns

### Object Factory with Prototypal Delegation (OLOO — Objects Linked to Other Objects)

```js
// Kyle Simpson's OLOO pattern — explicit, no `new`, no constructor confusion
const AnimalProto = {
  init(name, sound) {
    this.name  = name;
    this.sound = sound;
    return this;
  },
  speak() {
    return `${this.name} says ${this.sound}`;
  },
};

const DogProto = Object.create(AnimalProto);
DogProto.initDog = function(name) {
  AnimalProto.init.call(this, name, "Woof");
  this.tricks = [];
  return this;
};
DogProto.learn = function(trick) {
  this.tricks.push(trick);
  return this;
};

// Create instances
const dog = Object.create(DogProto).initDog("Rex");
dog.learn("sit").learn("shake");
dog.speak();   // "Rex says Woof"
dog.tricks;    // ["sit", "shake"]
```

### Mixin Pattern

```js
// Mixins — compose behaviors without deep inheritance
const Serializable = {
  serialize()              { return JSON.stringify(this); },
  deserialize(json)        { return Object.assign(Object.create(Object.getPrototypeOf(this)), JSON.parse(json)); },
};

const Validatable = {
  validate() {
    return Object.keys(this.rules || {}).every(key => {
      const rule = this.rules[key];
      return rule(this[key]);
    });
  },
};

const Timestamped = {
  touch() { this.updatedAt = new Date(); return this; },
};

// Mix into a class
class User {
  constructor(name, email) {
    this.name  = name;
    this.email = email;
    this.rules = {
      name:  v => typeof v === "string" && v.length > 0,
      email: v => /\S+@\S+\.\S+/.test(v),
    };
  }
}

Object.assign(User.prototype, Serializable, Validatable, Timestamped);

const u = new User("Alice", "alice@example.com");
u.validate();   // true
u.serialize();  // '{"name":"Alice","email":"alice@example.com","rules":{}}'
u.touch();
u.updatedAt;    // current Date
```

### Prototype-based Plugin System

```js
class EventEmitter {
  constructor() { this._events = {}; }

  on(event, listener) {
    (this._events[event] ??= []).push(listener);
    return this;
  }

  off(event, listener) {
    this._events[event] = (this._events[event] || []).filter(l => l !== listener);
    return this;
  }

  emit(event, ...args) {
    (this._events[event] || []).forEach(l => l(...args));
    return this;
  }
}

// Extend — prototype chain does the work
class Store extends EventEmitter {
  constructor(initial = {}) {
    super();
    this._state = initial;
  }

  get(key)        { return this._state[key]; }

  set(key, value) {
    const old = this._state[key];
    this._state[key] = value;
    this.emit("change", { key, value, old }); // From EventEmitter
    return this;
  }
}

const store = new Store({ count: 0 });
store.on("change", ({ key, value }) => console.log(`${key} → ${value}`));
store.set("count", 1); // logs: "count → 1"
store.set("name", "Alice"); // logs: "name → Alice"
```

---

## 19. Quick Reference

### The Prototype Toolbox

```js
// Create object with specific prototype
const obj = Object.create(proto);
const obj = Object.create(null);           // No prototype

// Read / write prototype
Object.getPrototypeOf(obj);                // Get [[Prototype]]
Object.setPrototypeOf(obj, newProto);      // Set [[Prototype]]

// Implement new manually
function myNew(Ctor, ...args) {
  const obj = Object.create(Ctor.prototype);
  const ret = Ctor.call(obj, ...args);
  return (ret !== null && typeof ret === "object") ? ret : obj;
}

// Check prototype relationships
obj instanceof Constructor;                // Walk chain for Constructor.prototype
Proto.isPrototypeOf(obj);                  // Is Proto anywhere in obj's chain?
Object.getPrototypeOf(obj) === Proto;      // Direct parent check

// Own vs inherited
Object.hasOwn(obj, "key");                 // ES2022 preferred
obj.hasOwnProperty("key");                 // Legacy

// Enumerate
Object.keys(obj);                          // Own enumerable string keys
Object.getOwnPropertyNames(obj);           // Own string keys (+ non-enumerable)
for (const k in obj) {}                    // Own + inherited enumerable keys
```

### Mental Model Summary

```
┌────────────────────────────────────────────────────────────┐
│               JavaScript Object Model                      │
│                                                            │
│  Every object has a [[Prototype]] (another object | null)  │
│                                                            │
│  Property lookup walks the chain until found or null       │
│                                                            │
│  `new Fn()`:                                               │
│    1. Object.create(Fn.prototype)                         │
│    2. Fn.call(obj, ...args)                               │
│    3. return obj (unless Fn returned an object)           │
│                                                            │
│  `class` is sugar over constructor functions + prototypes  │
│                                                            │
│  `extends` sets Child.prototype = Object.create(Parent.prototype)
│  and Object.setPrototypeOf(Child, Parent) for statics     │
│                                                            │
│  Arrow functions, `bind`, getters/setters are all         │
│  just objects with the right prototypes                   │
└────────────────────────────────────────────────────────────┘
```

---

## Further Reading

- [MDN: Inheritance and the prototype chain](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)
- [MDN: Object.create()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create)
- [MDN: new operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new)
- [ECMA-262: OrdinaryObjectCreate](https://tc39.es/ecma262/#sec-ordinaryobjectcreate)
- [You Don't Know JS: this & Object Prototypes](https://github.com/getify/You-Dont-Know-JS/tree/1st-ed/this%20%26%20object%20prototypes)
- [JavaScript.info: Prototypes, inheritance](https://javascript.info/prototype-inheritance)
- [MDN: Classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)

---

*All examples are valid ES2022+ JavaScript. Constructor function patterns are included for understanding the machinery — prefer `class` syntax in production code.*