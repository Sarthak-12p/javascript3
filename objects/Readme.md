# 🗂️ JavaScript Objects — In-Depth Guide

A comprehensive reference covering everything you need to know about objects in JavaScript, from the basics to advanced patterns.

---

## Table of Contents

1. [What is an Object?](#1-what-is-an-object)
2. [Creating Objects](#2-creating-objects)
3. [Accessing & Modifying Properties](#3-accessing--modifying-properties)
4. [Property Shorthand & Computed Keys](#4-property-shorthand--computed-keys)
5. [Methods in Objects](#5-methods-in-objects)
6. [Checking & Deleting Properties](#6-checking--deleting-properties)
7. [Iterating Over Objects](#7-iterating-over-objects)
8. [Copying & Merging Objects](#8-copying--merging-objects)
9. [Object Destructuring](#9-object-destructuring)
10. [Spread & Rest with Objects](#10-spread--rest-with-objects)
11. [Object Methods (Static)](#11-object-methods-static)
12. [Property Descriptors & defineProperty](#12-property-descriptors--defineproperty)
13. [Getters & Setters](#13-getters--setters)
14. [Prototypes & Inheritance](#14-prototypes--inheritance)
15. [Classes (Syntactic Sugar)](#15-classes-syntactic-sugar)
16. [Sealing, Freezing & Preventing Extensions](#16-sealing-freezing--preventing-extensions)
17. [Symbol Keys](#17-symbol-keys)
18. [WeakMap & WeakRef (Object-related)](#18-weakmap--weakref-object-related)
19. [Common Patterns & Recipes](#19-common-patterns--recipes)
20. [Performance Tips](#20-performance-tips)
21. [Quick Reference Cheat Sheet](#21-quick-reference-cheat-sheet)

---

## 1. What is an Object?

An **object** is an unordered collection of key-value pairs called **properties**. Keys are strings (or Symbols), and values can be anything.

```js
const person = {
  name: "Alice",
  age: 30,
  isAdmin: false,
  address: { city: "Pune" },
  greet() { return `Hi, I'm ${this.name}`; }
};
```

> **Key characteristics:**
> - Properties are key-value pairs
> - Keys are strings or Symbols (numbers are auto-coerced to strings)
> - Values can be any type, including functions and other objects
> - Objects are **reference types** — assigned/passed by reference
> - Almost everything in JS is an object (arrays, functions, dates, etc.)

---

## 2. Creating Objects

### Object Literal (preferred)
```js
const obj = { a: 1, b: 2 };
const empty = {};
```

### Object Constructor
```js
const obj = new Object();
obj.a = 1;
```

### Object.create()
Creates an object with a specified prototype.
```js
const proto = {
  greet() { return `Hello, ${this.name}`; }
};

const user = Object.create(proto);
user.name = "Bob";
user.greet(); // "Hello, Bob"

// No prototype at all (pure dictionary)
const dict = Object.create(null);
// dict has NO inherited properties — great for hash maps
```

### Factory Function
```js
function createUser(name, age) {
  return { name, age, greet() { return `Hi, I'm ${name}`; } };
}
const user = createUser("Alice", 30);
```

### Constructor Function
```js
function Person(name, age) {
  this.name = name;
  this.age  = age;
}
Person.prototype.greet = function() {
  return `Hi, I'm ${this.name}`;
};
const p = new Person("Alice", 30);
```

### Class (ES6+)
```js
class Person {
  constructor(name, age) {
    this.name = name;
    this.age  = age;
  }
  greet() { return `Hi, I'm ${this.name}`; }
}
const p = new Person("Alice", 30);
```

---

## 3. Accessing & Modifying Properties

### Dot notation
```js
const obj = { name: "Alice", age: 30 };

obj.name;          // "Alice"
obj.name = "Bob";  // set
obj.city = "Pune"; // add new property
```

### Bracket notation
Use when the key is dynamic, a variable, or has special characters.
```js
obj["name"];             // "Alice"

const key = "age";
obj[key];                // 30

obj["full name"] = "Alice Smith"; // keys with spaces
obj["123"];                       // numeric-string key
```

### Optional chaining `?.` *(ES2020)*
Safely access deeply nested properties without throwing.
```js
const user = { address: { city: "Pune" } };

user?.address?.city;         // "Pune"
user?.phone?.number;         // undefined (no error)
user?.getAge?.();            // undefined (method may not exist)
```

### Nullish coalescing `??`
Provide a default when a value is `null` or `undefined`.
```js
user.nickname ?? "Anonymous"; // "Anonymous" if nickname is null/undefined
user.age ?? 0;
```

---

## 4. Property Shorthand & Computed Keys

### Shorthand properties
```js
const name = "Alice";
const age  = 30;

// Old
const user = { name: name, age: age };

// Shorthand ✅
const user = { name, age };
```

### Computed property names
```js
const prefix = "get";
const field  = "Name";

const obj = {
  [`${prefix}${field}`]() { return this.name; }
};
obj.getName(); // works!

// Dynamic keys
const key = "score";
const result = { [key]: 100 }; // { score: 100 }
```

---

## 5. Methods in Objects

```js
const calc = {
  value: 0,

  // Method shorthand (preferred)
  add(n) { this.value += n; return this; },

  // Arrow function — ⚠️ does NOT bind its own `this`
  reset: () => { /* this is NOT calc here */ },

  // Old style
  multiply: function(n) { this.value *= n; return this; }
};

// Method chaining
calc.add(5).add(3).multiply(2); // calc.value === 16
```

> ⚠️ **Arrow functions as methods** inherit `this` from the enclosing scope (usually `window`/`undefined`). Use regular functions or shorthand for methods that need `this`.

---

## 6. Checking & Deleting Properties

### in operator
Checks own **and** inherited properties.
```js
"name" in person;   // true
"toString" in {};   // true (inherited from Object.prototype)
```

### hasOwnProperty / Object.hasOwn *(ES2022)*
Checks **own** properties only.
```js
person.hasOwnProperty("name");   // true
person.hasOwnProperty("toString"); // false

// Preferred modern way (safe even on null-prototype objects)
Object.hasOwn(person, "name");   // true ✅
```

### delete
```js
const obj = { a: 1, b: 2 };
delete obj.a;   // obj → { b: 2 }
delete obj.x;   // silent no-op (returns true)
```

> ⚠️ `delete` only removes **own** properties. It cannot delete variables or non-configurable properties.

---

## 7. Iterating Over Objects

### for...in
Iterates over **all enumerable** keys, including inherited ones.
```js
for (const key in obj) {
  if (Object.hasOwn(obj, key)) { // guard against inherited props
    console.log(key, obj[key]);
  }
}
```

### Object.keys / values / entries *(ES2017)*
Returns arrays of own, enumerable properties only.
```js
const scores = { alice: 90, bob: 85, carol: 92 };

Object.keys(scores);    // ["alice", "bob", "carol"]
Object.values(scores);  // [90, 85, 92]
Object.entries(scores); // [["alice", 90], ["bob", 85], ["carol", 92]]

// Iterate with destructuring
for (const [name, score] of Object.entries(scores)) {
  console.log(`${name}: ${score}`);
}
```

### Object.fromEntries *(ES2019)*
Converts key-value pairs back into an object.
```js
const doubled = Object.fromEntries(
  Object.entries(scores).map(([k, v]) => [k, v * 2])
);
// { alice: 180, bob: 170, carol: 184 }
```

---

## 8. Copying & Merging Objects

### Shallow copy
```js
// Spread (preferred)
const copy = { ...original };

// Object.assign
const copy2 = Object.assign({}, original);

// ⚠️ Both are SHALLOW — nested objects are still shared references!
const a = { x: { y: 1 } };
const b = { ...a };
b.x.y = 99;
console.log(a.x.y); // 99 — both point to same nested object
```

### Deep copy
```js
// structuredClone (ES2022, built-in) — preferred
const deep = structuredClone(original);

// JSON round-trip (loses functions, undefined, Dates become strings)
const deep2 = JSON.parse(JSON.stringify(original));
```

### Merging
```js
const defaults = { color: "red", size: "M", qty: 1 };
const custom   = { color: "blue", qty: 5 };

const merged = { ...defaults, ...custom };
// { color: "blue", size: "M", qty: 5 }
// Later keys WIN

// Object.assign (mutates first arg)
Object.assign(defaults, custom); // mutates defaults ⚠️
```

---

## 9. Object Destructuring

```js
const user = { name: "Alice", age: 30, city: "Pune" };

// Basic
const { name, age } = user;

// Rename while destructuring
const { name: userName, age: userAge } = user;
// userName="Alice", userAge=30

// Default values
const { role = "guest" } = user;
// role="guest" (since user.role is undefined)

// Rest
const { name: n, ...rest } = user;
// n="Alice", rest={ age: 30, city: "Pune" }

// Nested
const { address: { city } } = { address: { city: "Pune" } };

// In function parameters
function display({ name, age = 0 }) {
  console.log(`${name} is ${age}`);
}
display(user);

// Mixed with rename + default
const { a: x = 10, b: y = 20 } = { a: 5 };
// x=5, y=20
```

---

## 10. Spread & Rest with Objects

### Spread
```js
// Shallow clone
const clone = { ...obj };

// Merge with override
const config = { ...defaults, ...userConfig, version: "2.0" };

// Conditional properties
const extra = condition ? { debug: true } : {};
const merged = { ...base, ...extra };

// or inline
const merged2 = { ...base, ...(condition && { debug: true }) };
```

### Rest in destructuring
```js
const { id, ...data } = record;
// id = record.id
// data = everything else

function update({ id, ...fields }) {
  db.update(id, fields);
}
```

---

## 11. Object Methods (Static)

### Object.assign(target, ...sources)
Copies own enumerable properties into target (mutates target).
```js
Object.assign({}, src1, src2); // safe merge into new object
```

### Object.keys / values / entries / fromEntries
*(covered in section 7)*

### Object.create(proto, propertiesObject)
```js
const obj = Object.create(proto, {
  name: { value: "Alice", writable: true, enumerable: true, configurable: true }
});
```

### Object.getPrototypeOf / setPrototypeOf
```js
Object.getPrototypeOf(obj);        // returns prototype
Object.setPrototypeOf(obj, proto); // ⚠️ avoid — slow, use Object.create instead
```

### Object.getOwnPropertyNames
Returns ALL own property names (including non-enumerable).
```js
Object.getOwnPropertyNames(obj); // includes hidden properties
```

### Object.getOwnPropertySymbols
```js
Object.getOwnPropertySymbols(obj); // returns Symbol keys
```

### Object.is(a, b)
Strict equality without type coercion, handles edge cases.
```js
Object.is(NaN, NaN);   // true  (=== returns false)
Object.is(0, -0);      // false (=== returns true)
Object.is(1, 1);       // true
```

---

## 12. Property Descriptors & defineProperty

Every property has a descriptor with these flags:

| Flag | Default | Meaning |
|------|---------|---------|
| `value` | `undefined` | The property's value |
| `writable` | `true` | Can be reassigned |
| `enumerable` | `true` | Shows in `for...in`, `Object.keys` |
| `configurable` | `true` | Can be deleted or redefined |

```js
const obj = {};

Object.defineProperty(obj, "id", {
  value: 42,
  writable: false,      // read-only
  enumerable: false,    // hidden from iteration
  configurable: false   // cannot be deleted or redefined
});

obj.id = 99;   // silently fails (or throws in strict mode)
obj.id;        // 42

Object.keys(obj); // [] — id is non-enumerable
```

### defineProperties
```js
Object.defineProperties(obj, {
  firstName: { value: "Alice", writable: true, enumerable: true, configurable: true },
  lastName:  { value: "Smith", writable: true, enumerable: true, configurable: true }
});
```

### getOwnPropertyDescriptor
```js
Object.getOwnPropertyDescriptor(obj, "id");
// { value: 42, writable: false, enumerable: false, configurable: false }
```

---

## 13. Getters & Setters

Define computed or validated properties.

```js
const person = {
  firstName: "Alice",
  lastName: "Smith",

  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  },

  set fullName(value) {
    [this.firstName, this.lastName] = value.split(" ");
  }
};

person.fullName;           // "Alice Smith"
person.fullName = "Bob Jones";
person.firstName;          // "Bob"
```

### Via defineProperty
```js
Object.defineProperty(obj, "fullName", {
  get() { return `${this.first} ${this.last}`; },
  enumerable: true,
  configurable: true
});
```

### Use cases
```js
// Validation
set age(val) {
  if (val < 0) throw new Error("Age cannot be negative");
  this._age = val;
}
get age() { return this._age; }

// Lazy / cached computation
get expensiveValue() {
  const result = heavyCalc();
  Object.defineProperty(this, "expensiveValue", { value: result });
  return result;
}
```

---

## 14. Prototypes & Inheritance

Every object has an internal `[[Prototype]]` link. Property lookup walks this chain.

```js
const animal = {
  breathe() { return "breathing"; }
};

const dog = Object.create(animal);
dog.bark = function() { return "woof"; };

dog.bark();    // "woof"      — own property
dog.breathe(); // "breathing" — inherited from animal

Object.getPrototypeOf(dog) === animal; // true
```

### Prototype chain
```
dog → animal → Object.prototype → null
```

### instanceof
```js
dog instanceof Object; // true — Object.prototype is in the chain
```

### Shadowing
```js
dog.breathe = function() { return "dog breathing"; }; // shadows animal.breathe
dog.breathe(); // "dog breathing"
```

---

## 15. Classes (Syntactic Sugar)

Classes are syntactic sugar over prototype-based inheritance.

```js
class Animal {
  #sound; // private field (ES2022)

  constructor(name, sound) {
    this.name  = name;
    this.#sound = sound;
  }

  speak() {
    return `${this.name} says ${this.#sound}`;
  }

  static create(name, sound) {
    return new Animal(name, sound);
  }
}

class Dog extends Animal {
  constructor(name) {
    super(name, "woof");
    this.tricks = [];
  }

  learn(trick) {
    this.tricks.push(trick);
    return this; // for chaining
  }

  speak() {
    return super.speak() + "!"; // call parent method
  }
}

const d = new Dog("Rex");
d.learn("sit").learn("shake");
d.speak(); // "Rex says woof!"
```

### Private fields & methods *(ES2022)*
```js
class BankAccount {
  #balance = 0;             // private field

  deposit(amount) {
    this.#validate(amount); // private method
    this.#balance += amount;
  }

  get balance() { return this.#balance; }

  #validate(amount) {
    if (amount <= 0) throw new Error("Invalid amount");
  }
}
```

### Static fields & methods
```js
class Config {
  static version = "1.0";
  static getInstance() { /* singleton */ }
}

Config.version;      // "1.0"
Config.getInstance();
```

---

## 16. Sealing, Freezing & Preventing Extensions

### Object.preventExtensions
No new properties can be added. Existing ones can still be modified/deleted.
```js
Object.preventExtensions(obj);
obj.newProp = 1; // silently fails (throws in strict mode)
Object.isExtensible(obj); // false
```

### Object.seal
No adding or deleting properties. Existing values can still be changed.
```js
Object.seal(obj);
obj.newProp = 1;   // fails
delete obj.name;   // fails
obj.name = "Bob";  // ✅ still works
Object.isSealed(obj); // true
```

### Object.freeze
No adding, deleting, or modifying properties (shallow freeze).
```js
Object.freeze(obj);
obj.name = "Bob"; // silently fails (throws in strict mode)
Object.isFrozen(obj); // true

// ⚠️ SHALLOW — nested objects are not frozen!
const obj = Object.freeze({ nested: { x: 1 } });
obj.nested.x = 99; // works! nested is not frozen
```

### Deep freeze (custom)
```js
function deepFreeze(obj) {
  Object.getOwnPropertyNames(obj).forEach(name => {
    const val = obj[name];
    if (typeof val === "object" && val !== null) deepFreeze(val);
  });
  return Object.freeze(obj);
}
```

---

## 17. Symbol Keys

Symbols are unique, non-enumerable keys — great for metadata or avoiding name collisions.

```js
const id = Symbol("id");
const obj = { [id]: 123, name: "Alice" };

obj[id];                  // 123
Object.keys(obj);         // ["name"]  — Symbol not included
JSON.stringify(obj);      // '{"name":"Alice"}'  — Symbol not included

// Access symbol keys
Object.getOwnPropertySymbols(obj); // [Symbol(id)]
```

### Well-known Symbols
JS uses built-in symbols to customize behaviour.
```js
class Range {
  constructor(from, to) {
    this.from = from;
    this.to   = to;
  }

  [Symbol.iterator]() {
    let current = this.from;
    const last  = this.to;
    return {
      next() {
        return current <= last
          ? { value: current++, done: false }
          : { done: true };
      }
    };
  }
}

[...new Range(1, 5)]; // [1, 2, 3, 4, 5]
```

---

## 18. WeakMap & WeakRef (Object-related)

### WeakMap
Stores key-value pairs where **keys must be objects** and are held **weakly** (won't prevent garbage collection).

```js
const cache = new WeakMap();

function process(obj) {
  if (cache.has(obj)) return cache.get(obj);
  const result = heavyCompute(obj);
  cache.set(obj, result);
  return result;
}
// When obj is GC'd, its cache entry disappears automatically
```

> Use WeakMap to attach private data to objects without memory leaks.

### WeakRef *(ES2021)*
Holds a weak reference to an object.
```js
const ref = new WeakRef(someObject);
const obj = ref.deref(); // may be undefined if GC'd
if (obj) { /* use it */ }
```

---

## 19. Common Patterns & Recipes

### Pick (select specific keys)
```js
const pick = (obj, keys) =>
  Object.fromEntries(keys.map(k => [k, obj[k]]));

pick({ a: 1, b: 2, c: 3 }, ["a", "c"]); // { a: 1, c: 3 }
```

### Omit (exclude specific keys)
```js
const omit = (obj, keys) =>
  Object.fromEntries(
    Object.entries(obj).filter(([k]) => !keys.includes(k))
  );

omit({ a: 1, b: 2, c: 3 }, ["b"]); // { a: 1, c: 3 }
```

### Deep merge
```js
function deepMerge(target, source) {
  for (const key of Object.keys(source)) {
    if (source[key] instanceof Object && !Array.isArray(source[key])) {
      target[key] = deepMerge(target[key] ?? {}, source[key]);
    } else {
      target[key] = source[key];
    }
  }
  return target;
}
```

### Map over object values
```js
const mapValues = (obj, fn) =>
  Object.fromEntries(Object.entries(obj).map(([k, v]) => [k, fn(v)]));

mapValues({ a: 1, b: 2 }, x => x * 10); // { a: 10, b: 20 }
```

### Filter object keys
```js
const filterKeys = (obj, fn) =>
  Object.fromEntries(Object.entries(obj).filter(([k, v]) => fn(k, v)));

filterKeys({ a: 1, b: 2, c: 3 }, (k, v) => v > 1); // { b: 2, c: 3 }
```

### Invert keys and values
```js
const invert = obj =>
  Object.fromEntries(Object.entries(obj).map(([k, v]) => [v, k]));

invert({ a: "1", b: "2" }); // { "1": "a", "2": "b" }
```

### Flatten nested object
```js
function flatten(obj, prefix = "", res = {}) {
  for (const [k, v] of Object.entries(obj)) {
    const key = prefix ? `${prefix}.${k}` : k;
    if (v && typeof v === "object" && !Array.isArray(v)) {
      flatten(v, key, res);
    } else {
      res[key] = v;
    }
  }
  return res;
}

flatten({ a: { b: { c: 1 } }, d: 2 });
// { "a.b.c": 1, "d": 2 }
```

### Memoization with objects
```js
function memoize(fn) {
  const cache = {};
  return function(...args) {
    const key = JSON.stringify(args);
    return cache[key] ?? (cache[key] = fn(...args));
  };
}
```

### Observable / Proxy
```js
const handler = {
  set(target, key, value) {
    console.log(`Setting ${key} = ${value}`);
    target[key] = value;
    return true;
  },
  get(target, key) {
    console.log(`Getting ${key}`);
    return target[key];
  }
};

const observed = new Proxy({}, handler);
observed.name = "Alice"; // logs: Setting name = Alice
observed.name;           // logs: Getting name
```

---

## 20. Performance Tips

| Scenario | Recommendation |
|----------|---------------|
| Property access | Dot notation is slightly faster than bracket for known keys |
| Many properties | JS engines optimize "shape" — keep property order consistent |
| Delete vs undefined | Setting `obj.key = undefined` is faster than `delete obj.key` |
| Cloning | `structuredClone` > `JSON.parse/stringify` for correctness; spread for speed |
| Null-prototype objects | `Object.create(null)` for pure hash maps — no prototype overhead |
| Frozen objects | `Object.freeze` can enable engine optimizations (immutable shape) |
| Avoid dynamic keys | Adding unexpected properties post-construction can deoptimize V8 |
| WeakMap for metadata | Prevents memory leaks when attaching data to external objects |
| Computed keys in hot paths | Pre-compute dynamic keys outside tight loops |

---

## 21. Quick Reference Cheat Sheet

```
Create         : {}  Object.create()  new ClassName()  factory fn
Access         : obj.key  obj["key"]  obj?.key  obj[key]
Modify         : obj.key = val  delete obj.key
Default        : obj.key ?? fallback

Keys/Values    : Object.keys()  Object.values()  Object.entries()
From pairs     : Object.fromEntries(pairs)

Clone          : { ...obj }  Object.assign({}, obj)        [shallow]
Deep clone     : structuredClone(obj)                      [deep]
Merge          : { ...a, ...b }  Object.assign({}, a, b)

Check key      : "key" in obj      (own + inherited)
               : Object.hasOwn(obj, "key")  (own only)

Iterate        : for...in (all enumerable)
               : Object.entries + for...of (own enumerable)

Descriptors    : Object.defineProperty()  getOwnPropertyDescriptor()
Getters/Setters: get prop() {}  set prop(v) {}

Freeze/Seal    : Object.freeze()  Object.seal()  Object.preventExtensions()
Test           : Object.isFrozen()  Object.isSealed()  Object.isExtensible()

Prototype      : Object.getPrototypeOf()  Object.create(proto)
Class          : class Foo extends Bar { #private; constructor(); method() }

Equality       : Object.is(a, b)  (handles NaN and -0)
Symbol keys    : obj[Symbol("key")]  Object.getOwnPropertySymbols()
Weak storage   : WeakMap  WeakRef
```

---

## Further Reading

- [MDN — Working with Objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_objects)
- [MDN — Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)
- [You Don't Know JS — this & Object Prototypes](https://github.com/getify/You-Dont-Know-JS/tree/2nd-ed/objects-classes)
- [ECMAScript Specification — Object](https://tc39.es/ecma262/#sec-ordinary-and-exotic-objects-behaviours)

---

*Generated with ❤️ — covers ES5 through ES2022*