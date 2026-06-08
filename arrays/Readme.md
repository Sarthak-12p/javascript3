# 📦 JavaScript Arrays — In-Depth Guide

A comprehensive reference covering everything you need to know about arrays in JavaScript, from the basics to advanced patterns.

---

## Table of Contents

1. [What is an Array?](#1-what-is-an-array)
2. [Creating Arrays](#2-creating-arrays)
3. [Accessing & Modifying Elements](#3-accessing--modifying-elements)
4. [Array Properties](#4-array-properties)
5. [Adding & Removing Elements](#5-adding--removing-elements)
6. [Searching & Finding](#6-searching--finding)
7. [Iterating Over Arrays](#7-iterating-over-arrays)
8. [Transforming Arrays](#8-transforming-arrays)
9. [Sorting & Reversing](#9-sorting--reversing)
10. [Combining & Slicing](#10-combining--slicing)
11. [Flattening Arrays](#11-flattening-arrays)
12. [Array Destructuring](#12-array-destructuring)
13. [Spread & Rest with Arrays](#13-spread--rest-with-arrays)
14. [Multidimensional Arrays](#14-multidimensional-arrays)
15. [Array-Like Objects](#15-array-like-objects)
16. [Common Patterns & Recipes](#16-common-patterns--recipes)
17. [Performance Tips](#17-performance-tips)
18. [Quick Reference Cheat Sheet](#18-quick-reference-cheat-sheet)

---

## 1. What is an Array?

An **array** is an ordered, zero-indexed collection of values. In JavaScript, arrays are objects — they can hold any mix of types, are dynamically sized, and come with a rich set of built-in methods.

```js
const mixed = [42, "hello", true, null, { name: "Alice" }, [1, 2]];
```

> **Key characteristics:**
> - Zero-indexed (`arr[0]` is the first element)
> - Dynamic size (grows/shrinks at runtime)
> - Heterogeneous (can hold different types)
> - Reference type (assigned/passed by reference)

---

## 2. Creating Arrays

### Array Literal (preferred)
```js
const fruits = ["apple", "banana", "cherry"];
const empty  = [];
```

### Array Constructor
```js
const arr1 = new Array(3);          // [ <3 empty items> ] — sparse array of length 3
const arr2 = new Array(1, 2, 3);    // [1, 2, 3]
```

> ⚠️ `new Array(n)` with a single number creates a **sparse** array of that length, not an array containing that number.

### Array.of()
```js
Array.of(3);        // [3]  — avoids the single-number pitfall above
Array.of(1, 2, 3);  // [1, 2, 3]
```

### Array.from()
Creates arrays from iterables or array-like objects.

```js
Array.from("hello");             // ['h', 'e', 'l', 'l', 'o']
Array.from({ length: 5 }, (_, i) => i * 2); // [0, 2, 4, 6, 8]
Array.from(new Set([1, 2, 2, 3]));           // [1, 2, 3]
```

### Fill
```js
new Array(4).fill(0);       // [0, 0, 0, 0]
new Array(4).fill(0, 1, 3); // [empty, 0, 0, empty]  (start=1, end=3)
```

---

## 3. Accessing & Modifying Elements

```js
const colors = ["red", "green", "blue"];

// Read
colors[0];      // "red"
colors[2];      // "blue"
colors[-1];     // undefined  (negative indices don't work directly)

// Modern: Array.prototype.at() — supports negative indices
colors.at(0);   // "red"
colors.at(-1);  // "blue"  ✅

// Write
colors[1] = "yellow";  // ["red", "yellow", "blue"]

// Beyond length — creates sparse array
colors[5] = "pink";    // ["red", "yellow", "blue", <2 empty>, "pink"]
```

---

## 4. Array Properties

### length
```js
const arr = [10, 20, 30];
arr.length;      // 3

// Truncate by setting length
arr.length = 2;  // [10, 20]

// Clear an array
arr.length = 0;  // []
```

---

## 5. Adding & Removing Elements

| Method | Where | Returns | Mutates |
|--------|-------|---------|---------|
| `push(...items)` | End | New length | ✅ |
| `pop()` | End | Removed item | ✅ |
| `unshift(...items)` | Start | New length | ✅ |
| `shift()` | Start | Removed item | ✅ |
| `splice(start, deleteCount, ...items)` | Any index | Removed items array | ✅ |

```js
const arr = [1, 2, 3];

arr.push(4);          // arr → [1, 2, 3, 4]
arr.pop();            // returns 4, arr → [1, 2, 3]
arr.unshift(0);       // arr → [0, 1, 2, 3]
arr.shift();          // returns 0, arr → [1, 2, 3]

// splice: remove 1 element at index 1, insert 99
arr.splice(1, 1, 99); // returns [2], arr → [1, 99, 3]

// splice: insert without removing
arr.splice(2, 0, 50, 60); // arr → [1, 99, 50, 60, 3]

// splice: remove to end
arr.splice(2);        // returns [50, 60, 3], arr → [1, 99]
```

---

## 6. Searching & Finding

### indexOf / lastIndexOf
```js
const a = [1, 2, 3, 2, 1];
a.indexOf(2);         // 1 (first occurrence)
a.lastIndexOf(2);     // 3 (last occurrence)
a.indexOf(99);        // -1 (not found)
```

### includes
```js
a.includes(3);        // true
a.includes(99);       // false

// Works correctly with NaN (indexOf doesn't)
[NaN].includes(NaN);  // true
[NaN].indexOf(NaN);   // -1  ⚠️
```

### find & findIndex
```js
const users = [
  { id: 1, name: "Alice" },
  { id: 2, name: "Bob" }
];

users.find(u => u.id === 2);       // { id: 2, name: "Bob" }
users.findIndex(u => u.id === 2);  // 1
users.find(u => u.id === 99);      // undefined
```

### findLast & findLastIndex *(ES2023)*
```js
[1, 2, 3, 4].findLast(x => x % 2 === 0);      // 4
[1, 2, 3, 4].findLastIndex(x => x % 2 === 0); // 3
```

---

## 7. Iterating Over Arrays

### for loop (classic)
```js
for (let i = 0; i < arr.length; i++) {
  console.log(arr[i]);
}
```

### for...of (recommended for values)
```js
for (const fruit of fruits) {
  console.log(fruit);
}
```

### forEach
```js
fruits.forEach((fruit, index) => {
  console.log(`${index}: ${fruit}`);
});
```
> ⚠️ `forEach` always returns `undefined`. You cannot `break` or `return` out of it.

### entries / keys / values
```js
for (const [i, v] of fruits.entries()) {
  console.log(i, v);
}

[...fruits.keys()];   // [0, 1, 2]
[...fruits.values()]; // ["apple", "banana", "cherry"]
```

---

## 8. Transforming Arrays

These methods are **non-mutating** — they return a new array.

### map
Transforms each element.
```js
[1, 2, 3].map(x => x * x);          // [1, 4, 9]
["a", "b"].map((v, i) => `${i}:${v}`); // ["0:a", "1:b"]
```

### filter
Keeps elements that pass a test.
```js
[1, 2, 3, 4, 5].filter(x => x % 2 === 0); // [2, 4]
```

### reduce
Accumulates values into a single result.
```js
[1, 2, 3, 4].reduce((acc, cur) => acc + cur, 0); // 10

// Building an object from an array
["a", "b", "c"].reduce((obj, letter, i) => {
  obj[letter] = i;
  return obj;
}, {}); // { a: 0, b: 1, c: 2 }
```

### reduceRight
Same as `reduce` but iterates right-to-left.
```js
[[1], [2], [3]].reduceRight((acc, cur) => acc.concat(cur)); // [3, 2, 1]
```

### every & some
```js
[2, 4, 6].every(x => x % 2 === 0);  // true  — all pass?
[1, 2, 3].some(x => x > 2);         // true  — at least one passes?
[1, 3, 5].some(x => x % 2 === 0);   // false
```

### flatMap
Maps then flattens one level — very handy.
```js
["hello world", "foo bar"].flatMap(s => s.split(" "));
// ["hello", "world", "foo", "bar"]
```

---

## 9. Sorting & Reversing

### reverse
Mutates in place and returns the array.
```js
[1, 2, 3].reverse(); // [3, 2, 1]  ⚠️ mutates!
```

### toReversed *(ES2023)*
Non-mutating version.
```js
const original = [1, 2, 3];
original.toReversed(); // [3, 2, 1]
original;              // [1, 2, 3] — untouched ✅
```

### sort
Sorts **in place** using a comparator function.
```js
// ⚠️ Default sort converts to strings — wrong for numbers!
[10, 9, 2, 100].sort();           // [10, 100, 2, 9]  ❌

// Correct numeric sort
[10, 9, 2, 100].sort((a, b) => a - b); // [2, 9, 10, 100] ✅

// Descending
[10, 9, 2, 100].sort((a, b) => b - a); // [100, 10, 9, 2]

// Alphabetical strings
["banana", "apple", "cherry"].sort((a, b) => a.localeCompare(b));
// ["apple", "banana", "cherry"]

// Sort objects by property
users.sort((a, b) => a.name.localeCompare(b.name));
```

### toSorted *(ES2023)*
Non-mutating sort.
```js
const nums = [3, 1, 2];
nums.toSorted((a, b) => a - b); // [1, 2, 3]
nums; // [3, 1, 2] — untouched ✅
```

---

## 10. Combining & Slicing

### concat
Joins arrays (non-mutating).
```js
[1, 2].concat([3, 4], [5]); // [1, 2, 3, 4, 5]
```

### slice
Extracts a portion (non-mutating).
```js
const a = [0, 1, 2, 3, 4];
a.slice(1, 3);  // [1, 2]      (start inclusive, end exclusive)
a.slice(2);     // [2, 3, 4]
a.slice(-2);    // [3, 4]      (negative = from end)
a.slice();      // [0,1,2,3,4] (shallow copy)
```

### join
Converts array to a string.
```js
["a", "b", "c"].join("-");  // "a-b-c"
["a", "b", "c"].join("");   // "abc"
["a", "b", "c"].join();     // "a,b,c"  (default comma)
```

### copyWithin
Copies part of the array to another location within itself (mutating).
```js
[1, 2, 3, 4, 5].copyWithin(0, 3); // [4, 5, 3, 4, 5]
//  copies from index 3 to index 0
```

### with *(ES2023)*
Non-mutating index replacement.
```js
const arr = [1, 2, 3];
arr.with(1, 99); // [1, 99, 3]
arr;             // [1, 2, 3] — untouched ✅
```

---

## 11. Flattening Arrays

### flat
```js
[1, [2, [3, [4]]]].flat();     // [1, 2, [3, [4]]]  (1 level)
[1, [2, [3, [4]]]].flat(2);    // [1, 2, 3, [4]]
[1, [2, [3, [4]]]].flat(Infinity); // [1, 2, 3, 4]
```

### flatMap
Equivalent to `.map(...).flat(1)` but more efficient.
```js
[1, 2, 3].flatMap(x => [x, x * 2]); // [1, 2, 2, 4, 3, 6]
```

---

## 12. Array Destructuring

```js
const [a, b, c] = [10, 20, 30];
// a=10, b=20, c=30

// Skip elements
const [, second, , fourth] = [1, 2, 3, 4];
// second=2, fourth=4

// Default values
const [x = 0, y = 0] = [5];
// x=5, y=0

// Rest element
const [first, ...rest] = [1, 2, 3, 4];
// first=1, rest=[2, 3, 4]

// Swap variables
let p = 1, q = 2;
[p, q] = [q, p];
// p=2, q=1

// Nested
const [[a1, a2], [b1, b2]] = [[1, 2], [3, 4]];
```

---

## 13. Spread & Rest with Arrays

### Spread operator (`...`)
```js
// Shallow copy
const copy = [...original];

// Merge arrays
const merged = [...arr1, ...arr2, ...arr3];

// Pass array as function arguments
Math.max(...[3, 1, 4, 1, 5]); // 5

// Insert into the middle
const inserted = [...arr.slice(0, 2), 99, ...arr.slice(2)];
```

### Rest parameters
```js
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}
sum(1, 2, 3, 4); // 10
```

---

## 14. Multidimensional Arrays

```js
// Create a 3x3 matrix
const matrix = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
];

// Access
matrix[1][2]; // 6

// Iterate
for (const row of matrix) {
  for (const cell of row) {
    process(cell);
  }
}

// Create programmatically (avoid Array(n).fill([]) — shared reference!)
const grid = Array.from({ length: 3 }, () => new Array(3).fill(0));
```

---

## 15. Array-Like Objects

Some objects look like arrays (have `length` and indexed properties) but aren't arrays.

```js
// Examples: arguments, NodeList, string, TypedArrays
function example() {
  console.log(arguments);        // array-like, not a real array
  arguments.forEach(/* ❌ */);   // TypeError
}

// Convert to array
const args = Array.from(arguments);
const args2 = [...arguments];
const args3 = Array.prototype.slice.call(arguments);

// Check if something is a real array
Array.isArray([1, 2, 3]);        // true
Array.isArray("hello");          // false
Array.isArray({ length: 3 });   // false
```

---

## 16. Common Patterns & Recipes

### Remove duplicates
```js
[...new Set([1, 2, 2, 3, 3, 4])]; // [1, 2, 3, 4]
```

### Flatten and deduplicate
```js
[...new Set([1, [2, 2], 3].flat())]; // [1, 2, 3]
```

### Group array items *(Array.prototype.group — check support)*
```js
// Manual grouping with reduce
const people = [
  { name: "Alice", dept: "Eng" },
  { name: "Bob",   dept: "HR" },
  { name: "Carol", dept: "Eng" }
];

people.reduce((groups, person) => {
  const key = person.dept;
  (groups[key] ||= []).push(person);
  return groups;
}, {});
// { Eng: [...], HR: [...] }
```

### Chunk an array
```js
function chunk(arr, size) {
  return Array.from({ length: Math.ceil(arr.length / size) },
    (_, i) => arr.slice(i * size, i * size + size)
  );
}
chunk([1, 2, 3, 4, 5], 2); // [[1,2], [3,4], [5]]
```

### Zip two arrays
```js
const zip = (a, b) => a.map((v, i) => [v, b[i]]);
zip([1, 2, 3], ["a", "b", "c"]); // [[1,"a"], [2,"b"], [3,"c"]]
```

### Unique by property
```js
const uniqueById = arr =>
  [...new Map(arr.map(x => [x.id, x])).values()];
```

### Intersection / Union / Difference
```js
const A = [1, 2, 3, 4];
const B = [3, 4, 5, 6];

const union        = [...new Set([...A, ...B])];          // [1,2,3,4,5,6]
const intersection = A.filter(x => B.includes(x));        // [3,4]
const difference   = A.filter(x => !B.includes(x));       // [1,2]
```

### Shuffle (Fisher-Yates)
```js
function shuffle(arr) {
  const a = [...arr];
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}
```

---

## 17. Performance Tips

| Scenario | Recommendation |
|----------|---------------|
| Adding to end | `push` — O(1) amortized |
| Adding to start | Avoid `unshift` on large arrays (O(n)); use a deque structure |
| Removing from middle | `splice` is O(n); consider linked list for frequent mid-removes |
| Searching unsorted | `indexOf`/`includes` is O(n); sort + binary search for large datasets |
| Large iterations | `for` loop is often faster than `forEach`/`map` in hot paths |
| Avoid sparse arrays | They can cause deoptimization in JS engines |
| Immutable patterns | `map`/`filter`/`slice` create new arrays — be mindful in loops |
| TypedArrays | Use `Int32Array`, `Float64Array` etc. for numeric-heavy computation |

---

## 18. Quick Reference Cheat Sheet

```
Creation       : []  Array.of()  Array.from()  new Array()
Copy           : [...arr]  arr.slice()  Array.from(arr)
Add end        : push()           Remove end   : pop()
Add start      : unshift()        Remove start : shift()
Any position   : splice()
Non-mut add    : toSpliced() [ES2023]

Search         : indexOf  lastIndexOf  includes  find  findIndex
                 findLast  findLastIndex  [ES2023]

Transform      : map  filter  reduce  reduceRight  flatMap
Test           : every  some
Iterate        : forEach  for...of  entries  keys  values

Sort           : sort (mutating)   toSorted (non-mut, ES2023)
Reverse        : reverse (mutating) toReversed (non-mut, ES2023)

Combine        : concat  [...a, ...b]
Slice          : slice(start, end)
Join → string  : join(separator)

Flatten        : flat(depth)  flatMap()

Utility        : Array.isArray()  fill()  copyWithin()
                 arr.at(index)  arr.with(index, value) [ES2023]
```

---

## Further Reading

- [MDN — Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)
- [ECMAScript Specification](https://tc39.es/ecma262/#sec-array-objects)
- [You Don't Know JS — Types & Grammar](https://github.com/getify/You-Dont-Know-JS)

---

*Generated with ❤️ — covers ES5 through ES2023*