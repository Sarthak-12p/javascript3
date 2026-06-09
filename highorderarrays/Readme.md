# Higher-Order Array Methods & Iterators in JavaScript

Higher-order array methods are functions that take another function as an argument and apply it to each element of an array. Combined with iteration patterns like `for...of` and `for...in`, these tools form the backbone of modern JavaScript data handling.

---

## Table of Contents

1. [forEach](#1-foreach)
2. [map](#2-map)
3. [filter](#3-filter)
4. [reduce](#4-reduce)
5. [find & findIndex](#5-find--findindex)
6. [some & every](#6-some--every)
7. [flat & flatMap](#7-flat--flatmap)
8. [sort](#8-sort)
9. [for...of](#9-forof)
10. [for...in](#10-forin)
11. [entries, keys, values](#11-entries-keys-values)
12. [Chaining Methods](#12-chaining-methods)
13. [Method Comparison Table](#13-method-comparison-table)

---

## 1. `forEach`

Executes a function **once for every element**. Used purely for side effects — it always returns `undefined`.

### Syntax

```js
array.forEach((element, index, array) => {
  // body
});
```

### Basic Example

```js
const fruits = ["apple", "banana", "cherry"];

fruits.forEach((fruit) => {
  console.log(fruit);
});
// apple
// banana
// cherry
```

### With Index

```js
const players = ["Alice", "Bob", "Carol"];

players.forEach((player, index) => {
  console.log(`${index + 1}. ${player}`);
});
// 1. Alice
// 2. Bob
// 3. Carol
```

### With the Full Array Reference

```js
[10, 20, 30].forEach((num, i, arr) => {
  console.log(`${num} is at index ${i} in [${arr}]`);
});
// 10 is at index 0 in [10,20,30]
```

### ⚠️ `forEach` Cannot Break Early

```js
// return inside forEach only skips the current iteration
[1, 2, 3, 4, 5].forEach((n) => {
  if (n === 3) return; // does NOT stop the loop
  console.log(n);      // 1 2 4 5
});

// Use for...of with break if you need early exit
```

### Real-World Use

```js
const cart = [
  { name: "Shirt", price: 499 },
  { name: "Shoes", price: 1299 },
  { name: "Cap", price: 299 },
];

cart.forEach(({ name, price }) => {
  console.log(`${name} — ₹${price}`);
});
// Shirt — ₹499
// Shoes — ₹1299
// Cap — ₹299
```

---

## 2. `map`

Creates a **new array** by transforming every element using a function. The original array is never changed.

### Syntax

```js
const newArray = array.map((element, index, array) => {
  return transformedValue;
});
```

### Basic Example

```js
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map((n) => n * 2);

console.log(doubled);  // [2, 4, 6, 8, 10]
console.log(numbers);  // [1, 2, 3, 4, 5] — unchanged
```

### Transform Objects

```js
const users = [
  { firstName: "Alice", lastName: "Smith" },
  { firstName: "Bob",   lastName: "Jones" },
];

const fullNames = users.map((user) => `${user.firstName} ${user.lastName}`);
console.log(fullNames); // ["Alice Smith", "Bob Jones"]
```

### Extract a Single Property

```js
const products = [
  { id: 1, name: "Laptop", price: 75000 },
  { id: 2, name: "Phone",  price: 45000 },
  { id: 3, name: "Tablet", price: 30000 },
];

const names  = products.map((p) => p.name);
const prices = products.map((p) => p.price);

console.log(names);  // ["Laptop", "Phone", "Tablet"]
console.log(prices); // [75000, 45000, 30000]
```

### Using Index

```js
const items = ["pen", "book", "bag"];
const numbered = items.map((item, i) => `${i + 1}. ${item}`);

console.log(numbered); // ["1. pen", "2. book", "3. bag"]
```

### `map` vs `forEach`

| Feature            | `map`                     | `forEach`              |
|--------------------|---------------------------|------------------------|
| Returns            | New transformed array     | `undefined`            |
| Original array     | Unchanged                 | Unchanged              |
| Use for            | Transforming data         | Side effects (logging) |

---

## 3. `filter`

Returns a **new array** containing only the elements that pass a test (callback returns `true`).

### Syntax

```js
const filtered = array.filter((element, index, array) => {
  return true_or_false;
});
```

### Basic Example

```js
const numbers = [1, 2, 3, 4, 5, 6, 7, 8];
const evens = numbers.filter((n) => n % 2 === 0);

console.log(evens); // [2, 4, 6, 8]
```

### Filter Objects

```js
const students = [
  { name: "Alice", grade: 85 },
  { name: "Bob",   grade: 52 },
  { name: "Carol", grade: 91 },
  { name: "Dave",  grade: 47 },
];

const passed = students.filter((s) => s.grade >= 60);
console.log(passed);
// [{ name: "Alice", grade: 85 }, { name: "Carol", grade: 91 }]
```

### Filter Strings

```js
const words = ["apple", "ant", "banana", "avocado", "cherry"];
const startsWithA = words.filter((w) => w.startsWith("a"));

console.log(startsWithA); // ["apple", "ant", "avocado"]
```

### Remove Falsy Values

```js
const mixed = [0, 1, "", "hello", null, undefined, false, true, NaN];
const truthy = mixed.filter(Boolean);

console.log(truthy); // [1, "hello", true]
```

### Remove Duplicates

```js
const nums = [1, 2, 2, 3, 3, 3, 4];
const unique = nums.filter((n, index, arr) => arr.indexOf(n) === index);

console.log(unique); // [1, 2, 3, 4]
```

---

## 4. `reduce`

Reduces an entire array down to **a single value** — a number, string, object, or even another array.

### Syntax

```js
const result = array.reduce((accumulator, currentValue, index, array) => {
  return updatedAccumulator;
}, initialValue);
```

| Parameter        | Description                                  |
|------------------|----------------------------------------------|
| `accumulator`    | The running result (starts as `initialValue`)|
| `currentValue`   | The current element being processed          |
| `initialValue`   | Starting value of the accumulator (recommended always) |

### Sum All Numbers

```js
const nums = [10, 20, 30, 40, 50];
const total = nums.reduce((acc, n) => acc + n, 0);

console.log(total); // 150
```

### Find Maximum Value

```js
const scores = [45, 92, 67, 88, 23];
const max = scores.reduce((acc, n) => (n > acc ? n : acc), scores[0]);

console.log(max); // 92
```

### Count Occurrences

```js
const votes = ["Alice", "Bob", "Alice", "Carol", "Bob", "Alice"];

const tally = votes.reduce((acc, name) => {
  acc[name] = (acc[name] || 0) + 1;
  return acc;
}, {});

console.log(tally); // { Alice: 3, Bob: 2, Carol: 1 }
```

### Group Items by Property

```js
const people = [
  { name: "Alice", dept: "Engineering" },
  { name: "Bob",   dept: "Marketing"   },
  { name: "Carol", dept: "Engineering" },
  { name: "Dave",  dept: "Marketing"   },
];

const grouped = people.reduce((acc, person) => {
  const key = person.dept;
  if (!acc[key]) acc[key] = [];
  acc[key].push(person.name);
  return acc;
}, {});

console.log(grouped);
// { Engineering: ["Alice", "Carol"], Marketing: ["Bob", "Dave"] }
```

### Flatten an Array

```js
const nested = [[1, 2], [3, 4], [5, 6]];
const flat = nested.reduce((acc, arr) => acc.concat(arr), []);

console.log(flat); // [1, 2, 3, 4, 5, 6]
```

---

## 5. `find` & `findIndex`

### `find`

Returns the **first element** that satisfies the condition. Returns `undefined` if not found.

```js
const users = [
  { id: 1, name: "Alice" },
  { id: 2, name: "Bob"   },
  { id: 3, name: "Carol" },
];

const user = users.find((u) => u.id === 2);
console.log(user); // { id: 2, name: "Bob" }

const missing = users.find((u) => u.id === 99);
console.log(missing); // undefined
```

### `findIndex`

Returns the **index** of the first match. Returns `-1` if not found.

```js
const nums = [10, 25, 30, 45, 50];
const idx = nums.findIndex((n) => n > 30);

console.log(idx); // 3  (index of 45)
```

### `findLast` & `findLastIndex` (ES2023)

Search from the **end** of the array.

```js
const arr = [1, 2, 3, 4, 5];
const last = arr.findLast((n) => n % 2 === 0);  // 4
const lastIdx = arr.findLastIndex((n) => n % 2 === 0); // 3
```

---

## 6. `some` & `every`

Both return a **boolean**.

### `some` — At Least One Matches

```js
const ages = [15, 22, 17, 30];

const hasAdult = ages.some((age) => age >= 18);
console.log(hasAdult); // true
```

### `every` — All Must Match

```js
const scores = [80, 90, 75, 95];

const allPassed = scores.every((s) => s >= 60);
console.log(allPassed); // true

const allExcellent = scores.every((s) => s >= 90);
console.log(allExcellent); // false
```

### Practical Use

```js
const cart = [
  { name: "Book",  inStock: true  },
  { name: "Pen",   inStock: true  },
  { name: "Ruler", inStock: false },
];

const canCheckout = cart.every((item) => item.inStock);
console.log(canCheckout); // false — Ruler is out of stock

const anyAvailable = cart.some((item) => item.inStock);
console.log(anyAvailable); // true
```

---

## 7. `flat` & `flatMap`

### `flat`

Flattens nested arrays by the given depth (default: 1).

```js
const nested = [1, [2, 3], [4, [5, 6]]];

console.log(nested.flat());    // [1, 2, 3, 4, [5, 6]]  — depth 1
console.log(nested.flat(2));   // [1, 2, 3, 4, 5, 6]    — depth 2
console.log(nested.flat(Infinity)); // flatten any depth
```

### `flatMap`

Maps each element, then flattens one level. Equivalent to `.map(...).flat(1)` but more efficient.

```js
const sentences = ["Hello World", "foo bar", "JS is fun"];

const words = sentences.flatMap((s) => s.split(" "));
console.log(words);
// ["Hello", "World", "foo", "bar", "JS", "is", "fun"]
```

### Adding/Removing Elements While Mapping

```js
const nums = [1, 2, 3, 4];

// Skip negatives, double positives
const result = nums.flatMap((n) => (n % 2 === 0 ? [n, n * 2] : []));
console.log(result); // [2, 4, 4, 8]
```

---

## 8. `sort`

Sorts elements **in place** and returns the sorted array.

### Default Sort (Alphabetical — often wrong for numbers!)

```js
const fruits = ["banana", "apple", "cherry"];
fruits.sort();
console.log(fruits); // ["apple", "banana", "cherry"]

// ⚠️ Default sort converts to strings!
const nums = [10, 9, 2, 100];
nums.sort();
console.log(nums); // [10, 100, 2, 9] — WRONG for numbers
```

### Correct Numeric Sort

```js
const nums = [10, 9, 2, 100];

nums.sort((a, b) => a - b); // ascending
console.log(nums); // [2, 9, 10, 100]

nums.sort((a, b) => b - a); // descending
console.log(nums); // [100, 10, 9, 2]
```

### Sort Objects by Property

```js
const products = [
  { name: "Phone",  price: 45000 },
  { name: "Laptop", price: 75000 },
  { name: "Tablet", price: 30000 },
];

// Sort by price ascending
products.sort((a, b) => a.price - b.price);
console.log(products.map((p) => p.name));
// ["Tablet", "Phone", "Laptop"]
```

### Sort Strings Alphabetically (with `localeCompare`)

```js
const names = ["Zara", "alice", "Bob", "charlie"];
names.sort((a, b) => a.localeCompare(b, undefined, { sensitivity: "base" }));
console.log(names); // ["alice", "Bob", "charlie", "Zara"]
```

---

## 9. `for...of`

Iterates over **values** of any iterable (arrays, strings, Sets, Maps, generators).

### Syntax

```js
for (const value of iterable) {
  // body
}
```

### Array Values

```js
const colors = ["red", "green", "blue"];

for (const color of colors) {
  console.log(color);
}
// red green blue
```

### String Characters

```js
for (const char of "JavaScript") {
  process.stdout.write(char + "-");
}
// J-a-v-a-S-c-r-i-p-t-
```

### With `break` and `continue` (unlike `forEach`!)

```js
const numbers = [1, 2, 3, 4, 5, 6];

for (const n of numbers) {
  if (n === 4) break;       // stops at 4
  if (n % 2 === 0) continue; // skips even numbers
  console.log(n);
}
// 1 3
```

### Destructuring Inside `for...of`

```js
const entries = [
  ["name", "Alice"],
  ["age", 30],
  ["city", "Mumbai"],
];

for (const [key, value] of entries) {
  console.log(`${key}: ${value}`);
}
// name: Alice
// age: 30
// city: Mumbai
```

### Iterating a Map

```js
const map = new Map([["a", 1], ["b", 2], ["c", 3]]);

for (const [key, value] of map) {
  console.log(`${key} => ${value}`);
}
// a => 1   b => 2   c => 3
```

---

## 10. `for...in`

Iterates over the **enumerable keys** of an object (and inherited properties too).

### Syntax

```js
for (const key in object) {
  // body
}
```

### Basic Object

```js
const car = { brand: "Toyota", model: "Camry", year: 2023 };

for (const key in car) {
  console.log(`${key}: ${car[key]}`);
}
// brand: Toyota
// model: Camry
// year: 2023
```

### Dynamic Key Access

```js
const settings = { darkMode: true, notifications: false, autoSave: true };

for (const key in settings) {
  if (settings[key]) {
    console.log(`${key} is ON`);
  }
}
// darkMode is ON
// autoSave is ON
```

### `hasOwnProperty` — Skip Inherited Keys

```js
function Vehicle(type) {
  this.type = type;
}
Vehicle.prototype.wheels = 4; // inherited

const bike = new Vehicle("Bike");

for (const key in bike) {
  if (bike.hasOwnProperty(key)) {
    console.log(key); // only "type"
  }
}
```

### ⚠️ Don't Use `for...in` on Arrays

```js
const arr = ["a", "b", "c"];

for (const key in arr) {
  console.log(key); // "0", "1", "2" — STRING keys!
  // Also iterates over any added prototype methods
}

// Always use for...of or forEach for arrays
```

---

## 11. `entries()`, `keys()`, `values()`

These iterator methods let you loop over different views of an array or object.

### Array `entries()` — `[index, value]` pairs

```js
const fruits = ["apple", "banana", "cherry"];

for (const [index, fruit] of fruits.entries()) {
  console.log(`${index}: ${fruit}`);
}
// 0: apple
// 1: banana
// 2: cherry
```

### Array `keys()` — indices only

```js
const arr = ["x", "y", "z"];

for (const key of arr.keys()) {
  console.log(key); // 0 1 2
}
```

### Array `values()` — values only

```js
const arr = ["x", "y", "z"];

for (const val of arr.values()) {
  console.log(val); // x y z
}
```

### Object `Object.keys()`, `Object.values()`, `Object.entries()`

```js
const person = { name: "Alice", age: 30, city: "Mumbai" };

console.log(Object.keys(person));
// ["name", "age", "city"]

console.log(Object.values(person));
// ["Alice", 30, "Mumbai"]

console.log(Object.entries(person));
// [["name", "Alice"], ["age", 30], ["city", "Mumbai"]]

// Iterate an object's entries like an array
for (const [key, value] of Object.entries(person)) {
  console.log(`${key}: ${value}`);
}
// name: Alice
// age: 30
// city: Mumbai
```

---

## 12. Chaining Methods

Array methods can be chained together to build powerful, readable data pipelines.

### Filter → Map

```js
const students = [
  { name: "Alice", score: 85 },
  { name: "Bob",   score: 42 },
  { name: "Carol", score: 91 },
  { name: "Dave",  score: 55 },
];

const topStudents = students
  .filter((s) => s.score >= 60)         // keep passing students
  .map((s) => s.name)                   // extract names
  .sort();                              // sort alphabetically

console.log(topStudents); // ["Alice", "Carol"]
```

### Map → Filter → Reduce

```js
const orders = [
  { product: "Book",  qty: 2, price: 299 },
  { product: "Pen",   qty: 5, price: 49  },
  { product: "Bag",   qty: 1, price: 999 },
];

const totalRevenue = orders
  .map((o) => o.qty * o.price)          // calculate line totals
  .filter((total) => total > 200)       // only large orders
  .reduce((acc, total) => acc + total, 0); // sum them up

console.log(`₹${totalRevenue}`); // ₹1843
```

### Real Pipeline — Process Raw Data

```js
const rawData = "  Alice,30  \n  Bob,  \n  Carol,25  ";

const users = rawData
  .split("\n")
  .map((line) => line.trim())
  .filter((line) => line.length > 0)
  .map((line) => {
    const [name, age] = line.split(",").map((s) => s.trim());
    return { name, age: Number(age) || null };
  })
  .filter((user) => user.age !== null);

console.log(users);
// [{ name: "Alice", age: 30 }, { name: "Carol", age: 25 }]
```

---

## 13. Method Comparison Table

| Method        | Returns          | Mutates Original | Use For                                |
|---------------|------------------|------------------|----------------------------------------|
| `forEach`     | `undefined`      | ❌ No             | Side effects (log, DOM update)         |
| `map`         | New array        | ❌ No             | Transform every element                |
| `filter`      | New array        | ❌ No             | Keep elements that pass a test         |
| `reduce`      | Single value     | ❌ No             | Aggregate / accumulate                 |
| `find`        | Element / `undefined` | ❌ No       | First element matching condition       |
| `findIndex`   | Index / `-1`     | ❌ No             | Index of first matching element        |
| `some`        | `boolean`        | ❌ No             | At least one element passes            |
| `every`       | `boolean`        | ❌ No             | All elements pass                      |
| `flat`        | New array        | ❌ No             | Flatten nested arrays                  |
| `flatMap`     | New array        | ❌ No             | Map + flatten in one step              |
| `sort`        | Same array       | ✅ Yes            | Sort in place                          |
| `for...of`    | —                | Depends          | Iterate values, supports `break`       |
| `for...in`    | —                | Depends          | Iterate object keys                    |
| `entries()`   | Iterator         | ❌ No             | Get `[index, value]` pairs from array  |

---

## Key Takeaways

- **`forEach`** → run side effects on every element; cannot break early.
- **`map`** → transform each element into something new; always returns a new array.
- **`filter`** → keep only elements that pass a condition.
- **`reduce`** → collapse an array into one value (sum, object, grouped data).
- **`find`** → grab the first match; `findIndex` gives you its position.
- **`some` / `every`** → quick boolean checks on collections.
- **`flat` / `flatMap`** → tame nested arrays.
- **`sort`** → always provide a comparator function for numbers.
- **`for...of`** → cleanest way to iterate any iterable; supports `break`.
- **`for...in`** → iterates object keys; avoid on arrays.
- **`entries / keys / values`** → flexible views into arrays and objects.
- **Chain methods** to write clean, expressive data transformation pipelines.