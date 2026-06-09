# Loops in JavaScript

Loops let you run a block of code repeatedly — iterating over data, counting, waiting for a condition, or walking through collections. JavaScript gives you several loop types, each suited for different situations.

---

## Table of Contents

1. [for Loop](#1-for-loop)
2. [while Loop](#2-while-loop)
3. [do...while Loop](#3-dowhile-loop)
4. [for...of Loop](#4-forof-loop)
5. [for...in Loop](#5-forin-loop)
6. [forEach Method](#6-foreach-method)
7. [Loop Control — break & continue](#7-loop-control--break--continue)
8. [Nested Loops](#8-nested-loops)
9. [Infinite Loops (and how to avoid them)](#9-infinite-loops-and-how-to-avoid-them)
10. [Loop Performance Tips](#10-loop-performance-tips)
11. [Quick Comparison Table](#11-quick-comparison-table)

---

## 1. `for` Loop

The classic, most-used loop. Best when you **know how many times** you want to iterate.

### Syntax

```js
for (initialization; condition; update) {
  // body
}
```

| Part             | Purpose                              |
|------------------|--------------------------------------|
| `initialization` | Runs once before the loop starts     |
| `condition`      | Checked before every iteration       |
| `update`         | Runs after every iteration           |

### Example

```js
for (let i = 0; i < 5; i++) {
  console.log("Count:", i);
}
// Count: 0
// Count: 1
// Count: 2
// Count: 3
// Count: 4
```

### Looping Over an Array

```js
const colors = ["red", "green", "blue"];

for (let i = 0; i < colors.length; i++) {
  console.log(i, colors[i]);
}
// 0 red
// 1 green
// 2 blue
```

### Counting Backwards

```js
for (let i = 5; i >= 1; i--) {
  console.log(i);
}
// 5 4 3 2 1
```

### Looping with a Step

```js
for (let i = 0; i <= 10; i += 2) {
  console.log(i); // 0 2 4 6 8 10
}
```

---

## 2. `while` Loop

Repeats a block **as long as a condition is true**. The condition is evaluated **before** each iteration.

### Syntax

```js
while (condition) {
  // body
}
```

### Example

```js
let count = 1;

while (count <= 5) {
  console.log("Count:", count);
  count++;
}
// Count: 1 through Count: 5
```

### Real-World Use — Keep asking until valid input

```js
let input = "";

while (input !== "yes" && input !== "no") {
  input = prompt("Enter yes or no:");
}

console.log("You entered:", input);
```

### Important

> If the condition is **never false**, the loop runs forever. Always make sure the condition can become false.

---

## 3. `do...while` Loop

Like `while`, but the body always executes **at least once** — the condition is checked **after** the body runs.

### Syntax

```js
do {
  // body
} while (condition);
```

### Example

```js
let attempts = 0;

do {
  console.log("Attempt:", attempts);
  attempts++;
} while (attempts < 3);
// Attempt: 0
// Attempt: 1
// Attempt: 2
```

### Why use `do...while`?

```js
// Show menu at least once, then repeat if user wants to continue
let choice;

do {
  choice = prompt("Choose: (1) Play  (2) Quit");
  console.log("You chose:", choice);
} while (choice !== "2");
```

### `while` vs `do...while`

```js
let x = 10;

// while — never runs (10 < 5 is false from the start)
while (x < 5) {
  console.log("while:", x);
}

// do...while — runs once even though 10 < 5 is false
do {
  console.log("do...while:", x); // "do...while: 10"
} while (x < 5);
```

---

## 4. `for...of` Loop

Iterates over the **values** of any iterable — arrays, strings, Sets, Maps, NodeLists.

### Syntax

```js
for (const value of iterable) {
  // body
}
```

### Iterating Over an Array

```js
const fruits = ["apple", "banana", "cherry"];

for (const fruit of fruits) {
  console.log(fruit);
}
// apple
// banana
// cherry
```

### Iterating Over a String

```js
for (const char of "hello") {
  console.log(char);
}
// h e l l o
```

### Iterating Over a Set

```js
const unique = new Set([1, 2, 2, 3, 3, 4]);

for (const num of unique) {
  console.log(num);
}
// 1 2 3 4
```

### Iterating Over a Map

```js
const scores = new Map([
  ["Alice", 95],
  ["Bob", 82],
  ["Carol", 88],
]);

for (const [name, score] of scores) {
  console.log(`${name}: ${score}`);
}
// Alice: 95
// Bob: 82
// Carol: 88
```

### With `entries()` to get index + value

```js
const animals = ["cat", "dog", "bird"];

for (const [index, animal] of animals.entries()) {
  console.log(`${index}: ${animal}`);
}
// 0: cat
// 1: dog
// 2: bird
```

---

## 5. `for...in` Loop

Iterates over the **enumerable keys** of an object.

### Syntax

```js
for (const key in object) {
  // body
}
```

### Example

```js
const person = {
  name: "Alice",
  age: 30,
  city: "Mumbai",
};

for (const key in person) {
  console.log(`${key}: ${person[key]}`);
}
// name: Alice
// age: 30
// city: Mumbai
```

### Dynamic Key Access

```js
const config = { theme: "dark", language: "en", debug: true };

for (const key in config) {
  if (config[key] === true) {
    console.log(`${key} is enabled`);
  }
}
// debug is enabled
```

### ⚠️ Gotcha — `for...in` on Arrays

```js
const arr = ["a", "b", "c"];

for (const key in arr) {
  console.log(key); // "0", "1", "2" — these are STRING keys, not numbers!
}
```

> Use `for...of` or `forEach` for arrays. Use `for...in` for plain objects.

### Checking Own Properties

```js
function Animal(name) {
  this.name = name;
}
Animal.prototype.type = "mammal"; // inherited

const dog = new Animal("Rex");

for (const key in dog) {
  if (dog.hasOwnProperty(key)) {
    console.log(key); // only "name", skips inherited "type"
  }
}
```

---

## 6. `forEach` Method

An array method that runs a function for each element. Clean and readable for arrays.

### Syntax

```js
array.forEach((element, index, array) => {
  // body
});
```

### Basic Example

```js
const numbers = [10, 20, 30, 40];

numbers.forEach((num) => {
  console.log(num * 2);
});
// 20 40 60 80
```

### With Index

```js
const items = ["pen", "book", "bag"];

items.forEach((item, index) => {
  console.log(`${index + 1}. ${item}`);
});
// 1. pen
// 2. book
// 3. bag
```

### ⚠️ `forEach` cannot be stopped early

```js
// This does NOT stop at 3
[1, 2, 3, 4, 5].forEach((n) => {
  if (n === 3) return; // just skips this iteration, doesn't break
  console.log(n); // 1 2 4 5
});

// Use a for...of + break if you need to stop early
```

---

## 7. Loop Control — `break` & `continue`

### `break` — Exit the Loop Immediately

```js
for (let i = 0; i < 10; i++) {
  if (i === 5) break; // stop when i hits 5
  console.log(i);
}
// 0 1 2 3 4
```

### `continue` — Skip the Current Iteration

```js
for (let i = 0; i < 6; i++) {
  if (i % 2 === 0) continue; // skip even numbers
  console.log(i);
}
// 1 3 5
```

### Labeled `break` — Escape Nested Loops

```js
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (i === 1 && j === 1) {
      break outer; // breaks BOTH loops
    }
    console.log(`i=${i}, j=${j}`);
  }
}
// i=0, j=0
// i=0, j=1
// i=0, j=2
// i=1, j=0
```

### Labeled `continue` — Skip in Outer Loop

```js
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (j === 1) continue outer; // skip rest of inner, go to next outer iteration
    console.log(`i=${i}, j=${j}`);
  }
}
// i=0, j=0
// i=1, j=0
// i=2, j=0
```

---

## 8. Nested Loops

A loop inside another loop. Used for 2D data, grids, or pairing elements.

### Multiplication Table

```js
for (let i = 1; i <= 3; i++) {
  for (let j = 1; j <= 3; j++) {
    process.stdout.write(`${i * j}\t`);
  }
  console.log(); // new line
}
//  1  2  3
//  2  4  6
//  3  6  9
```

### Iterating a 2D Array (Matrix)

```js
const matrix = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9],
];

for (const row of matrix) {
  for (const cell of row) {
    process.stdout.write(cell + " ");
  }
  console.log();
}
// 1 2 3
// 4 5 6
// 7 8 9
```

### Finding Pairs

```js
const nums = [1, 2, 3, 4];
const target = 5;

for (let i = 0; i < nums.length; i++) {
  for (let j = i + 1; j < nums.length; j++) {
    if (nums[i] + nums[j] === target) {
      console.log(`Pair: (${nums[i]}, ${nums[j]})`);
    }
  }
}
// Pair: (1, 4)
// Pair: (2, 3)
```

---

## 9. Infinite Loops (and how to avoid them)

An infinite loop runs forever and crashes your program. Always ensure the condition can become false.

### ❌ Infinite Loop Examples

```js
// Missing update — i never changes
for (let i = 0; i < 5; ) {
  console.log(i); // runs forever
}

// Condition never becomes false
while (true) {
  console.log("forever"); // runs forever
}
```

### ✅ Intentional `while(true)` with `break`

Sometimes you genuinely want an infinite loop that you exit manually:

```js
while (true) {
  const input = prompt("Type 'quit' to exit:");
  if (input === "quit") break;
  console.log("You typed:", input);
}
```

### Common Causes of Accidental Infinite Loops

| Mistake                     | Fix                                      |
|-----------------------------|------------------------------------------|
| Forgot to increment `i`     | Add `i++` in the update expression       |
| Decrementing when should increment | Double-check loop direction        |
| Modifying the array inside the loop | Use a copy or iterate backwards  |
| Wrong comparison operator   | `<` vs `<=` — know your boundary         |

---

## 10. Loop Performance Tips

### Cache array length in classic `for`

```js
const arr = new Array(1_000_000).fill(0);

// Slightly slower — arr.length evaluated every iteration
for (let i = 0; i < arr.length; i++) { }

// Faster — length cached once
for (let i = 0, len = arr.length; i < len; i++) { }
```

### Prefer `for...of` over `forEach` for early exit

```js
// forEach cannot break — you waste iterations
arr.forEach(item => { if (found) return; });

// for...of can break as soon as you find what you need
for (const item of arr) {
  if (condition) break;
}
```

### Use array methods for transformations (not loops)

```js
const numbers = [1, 2, 3, 4, 5];

// Loop approach
const doubled = [];
for (const n of numbers) {
  doubled.push(n * 2);
}

// Cleaner with map
const doubled2 = numbers.map(n => n * 2);
```

---

## 11. Quick Comparison Table

| Loop          | Iterates Over          | Index Available | Can `break`  | Best Used For                          |
|---------------|------------------------|-----------------|--------------|----------------------------------------|
| `for`         | Count / index          | ✅ Yes           | ✅ Yes        | Known iterations, index-based logic    |
| `while`       | Condition              | ✅ Manual        | ✅ Yes        | Unknown iterations, condition-driven   |
| `do...while`  | Condition (after body) | ✅ Manual        | ✅ Yes        | Must run at least once                 |
| `for...of`    | Iterable values        | ✅ With entries() | ✅ Yes       | Arrays, strings, Sets, Maps            |
| `for...in`    | Object keys            | ✅ (is the key)  | ✅ Yes        | Plain objects                          |
| `forEach`     | Array values           | ✅ 2nd param     | ❌ No         | Side effects on every array element    |

---

## Key Takeaways

- Use **`for`** when you need precise index control or a known count.
- Use **`while`** when the number of iterations is unpredictable.
- Use **`do...while`** when the body must run at least once.
- Use **`for...of`** for clean iteration over arrays and other iterables.
- Use **`for...in`** only for plain objects, never arrays.
- Use **`forEach`** for readable side effects on arrays when you don't need `break`.
- Use **`break`** to exit early, **`continue`** to skip an iteration.
- Always verify your loop will eventually terminate to avoid infinite loops.