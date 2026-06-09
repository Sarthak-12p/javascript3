# Control Flow in JavaScript

Control flow is the order in which the JavaScript engine executes statements in a script. By default, code runs top to bottom — but control flow structures let you branch, repeat, skip, or jump based on conditions and logic.

---

## 1. Conditional Statements

### `if / else if / else`

The most fundamental control flow tool. Runs a block only when a condition is `true`.

```js
const score = 72;

if (score >= 90) {
  console.log("Grade: A");
} else if (score >= 75) {
  console.log("Grade: B");
} else if (score >= 60) {
  console.log("Grade: C");
} else {
  console.log("Grade: F");
}
// Output: "Grade: C"
```

> **Key point:** JavaScript evaluates each condition top to bottom and stops at the first `true` match.

---

### `switch`

Best when you're comparing **one value** against many fixed cases.

```js
const day = "Monday";

switch (day) {
  case "Monday":
  case "Tuesday":
  case "Wednesday":
  case "Thursday":
  case "Friday":
    console.log("Weekday");
    break;
  case "Saturday":
  case "Sunday":
    console.log("Weekend");
    break;
  default:
    console.log("Unknown day");
}
// Output: "Weekday"
```

> ⚠️ Always use `break` — without it, execution "falls through" into the next case.

---

### Ternary Operator `? :`

A compact one-liner for simple if/else logic.

```js
const age = 20;
const status = age >= 18 ? "Adult" : "Minor";
console.log(status); // "Adult"
```

---

### Nullish Coalescing `??`

Returns the right-hand value only when the left side is `null` or `undefined`.

```js
const username = null;
console.log(username ?? "Guest"); // "Guest"

const count = 0;
console.log(count ?? 10); // 0  (0 is not null/undefined)
```

---

### Optional Chaining `?.`

Safely access deeply nested properties without crashing on `null`/`undefined`.

```js
const user = { profile: { name: "Alice" } };

console.log(user?.profile?.name);   // "Alice"
console.log(user?.address?.city);   // undefined (no crash)
```

---

## 2. Loops

Loops repeat a block of code as long as a condition holds.

---

### `for` Loop

The classic loop — best when you know the iteration count.

```js
for (let i = 0; i < 5; i++) {
  console.log(i);
}
// Output: 0 1 2 3 4
```

Anatomy:
```
for ( initializer ; condition ; update ) { body }
      let i = 0  ;  i < 5   ;  i++
```

---

### `while` Loop

Repeats while the condition is `true`. Check happens **before** each iteration.

```js
let count = 3;

while (count > 0) {
  console.log(count);
  count--;
}
// Output: 3 2 1
```

---

### `do...while` Loop

Like `while`, but the body runs **at least once** because the check happens **after**.

```js
let attempts = 0;

do {
  console.log("Attempt:", attempts);
  attempts++;
} while (attempts < 3);
// Output: Attempt: 0, Attempt: 1, Attempt: 2
```

---

### `for...of` Loop

Iterates over **iterable values** — arrays, strings, Sets, Maps.

```js
const fruits = ["apple", "banana", "cherry"];

for (const fruit of fruits) {
  console.log(fruit);
}
// Output: apple, banana, cherry
```

Works on strings too:
```js
for (const char of "hello") {
  console.log(char); // h e l l o
}
```

---

### `for...in` Loop

Iterates over the **keys** of an object.

```js
const car = { brand: "Toyota", model: "Camry", year: 2023 };

for (const key in car) {
  console.log(`${key}: ${car[key]}`);
}
// brand: Toyota
// model: Camry
// year: 2023
```

> ⚠️ Use `for...in` for plain objects. For arrays, prefer `for...of` or `forEach`.

---

## 3. Loop Control Statements

### `break`

Exits the loop immediately.

```js
for (let i = 0; i < 10; i++) {
  if (i === 5) break;
  console.log(i);
}
// Output: 0 1 2 3 4
```

---

### `continue`

Skips the current iteration and moves to the next.

```js
for (let i = 0; i < 6; i++) {
  if (i % 2 === 0) continue; // skip even numbers
  console.log(i);
}
// Output: 1 3 5
```

---

### Labeled Statements

Used with `break` or `continue` to target an **outer loop**.

```js
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (j === 1) break outer; // breaks out of both loops
    console.log(i, j);
  }
}
// Output: 0 0
```

---

## 4. Exception Handling

Control flow also includes handling runtime errors gracefully.

### `try / catch / finally`

```js
function divide(a, b) {
  if (b === 0) throw new Error("Division by zero!");
  return a / b;
}

try {
  console.log(divide(10, 2));  // 5
  console.log(divide(10, 0));  // throws
} catch (error) {
  console.log("Error:", error.message); // "Error: Division by zero!"
} finally {
  console.log("Always runs"); // cleanup code
}
```

| Block     | When it runs                          |
|-----------|---------------------------------------|
| `try`     | Always attempted                      |
| `catch`   | Only if an error is thrown in `try`   |
| `finally` | Always — error or not                 |

---

## 5. Short-Circuit Evaluation

JavaScript's logical operators `&&` and `||` use short-circuiting — they stop evaluating as soon as the result is determined.

```js
// && — stops at first falsy
const user = null;
const name = user && user.name; // null (stops early)

// || — stops at first truthy
const setting = null;
const theme = setting || "dark"; // "dark"

// Practical use: conditionally run a function
isLoggedIn && showDashboard();
```

---

## 6. Asynchronous Control Flow

Modern JavaScript handles async operations with Promises and `async/await`.

### Callbacks (old style)

```js
setTimeout(() => {
  console.log("Runs after 1 second");
}, 1000);
```

### Promises

```js
fetch("https://api.example.com/data")
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

### `async / await` (modern, preferred)

Makes async code read like synchronous code.

```js
async function loadUser(id) {
  try {
    const response = await fetch(`/api/users/${id}`);
    const user = await response.json();
    console.log(user.name);
  } catch (error) {
    console.error("Failed to load user:", error);
  }
}

loadUser(42);
```

> `await` can only be used inside an `async` function. It pauses execution until the Promise resolves.

---

## 7. Quick Reference

| Structure          | Use When                                          |
|--------------------|---------------------------------------------------|
| `if/else`          | Branching on a boolean condition                  |
| `switch`           | One value, many possible cases                    |
| `? :`              | Simple true/false inline assignment               |
| `??`               | Fallback for `null` / `undefined` only            |
| `?.`               | Safe access on possibly null objects              |
| `for`              | Known number of iterations                        |
| `while`            | Repeat while a condition holds                    |
| `do...while`       | Must run at least once                            |
| `for...of`         | Iterate over array/string values                  |
| `for...in`         | Iterate over object keys                          |
| `break`            | Exit a loop early                                 |
| `continue`         | Skip current iteration                            |
| `try/catch`        | Handle errors gracefully                          |
| `async/await`      | Manage async operations cleanly                   |

---

## Key Takeaways

- **Conditions** branch your code — `if/else`, `switch`, and ternaries handle most cases.
- **Loops** repeat work — choose the right one for the data structure you're working with.
- **`break` and `continue`** give you fine control inside loops.
- **`try/catch`** keeps your app alive when things go wrong.
- **`async/await`** is the modern, readable way to handle asynchronous control flow.

Master these building blocks and you can control exactly how and when JavaScript executes any piece of code.