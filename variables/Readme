# JavaScript Variables: `var`, `let`, and `const`

JavaScript provides three ways to declare variables:

* `var` (old way)
* `let` (modern way)
* `const` (modern way for constants)

---

# 1. `var`

`var` is the traditional way of declaring variables in JavaScript.

### Definition

A variable declared with `var` can be **redeclared** and **updated**.

### Example

```javascript
var name = "John";
name = "Alex";      // Updated

var name = "David"; // Redeclared

console.log(name);
```

### Features

✅ Can be updated

✅ Can be redeclared

❌ Not block scoped

### Block Scope Example

```javascript
if (true) {
    var age = 18;
}

console.log(age); // 18
```

`age` is accessible outside the block because `var` is function-scoped.

---

# 2. `let`

Introduced in ES6 (2015).

### Definition

A variable declared with `let` can be **updated** but **cannot be redeclared in the same scope**.

### Example

```javascript
let score = 10;

score = 20; // Allowed

console.log(score);
```

### Invalid Example

```javascript
let score = 10;
let score = 20; // Error
```

### Features

✅ Can be updated

❌ Cannot be redeclared in the same scope

✅ Block scoped

### Block Scope Example

```javascript
if (true) {
    let marks = 95;
}

console.log(marks); // Error
```

`marks` only exists inside the block.

---

# 3. `const`

Introduced in ES6.

### Definition

A variable declared with `const` cannot be reassigned after initialization.

### Example

```javascript
const PI = 3.14159;

console.log(PI);
```

### Invalid Example

```javascript
const PI = 3.14159;

PI = 3.14; // Error
```

### Features

❌ Cannot be updated

❌ Cannot be redeclared

✅ Block scoped

### Important Note

For objects and arrays, the contents can still be modified.

```javascript
const numbers = [1, 2, 3];

numbers.push(4);

console.log(numbers);
```

Output:

```javascript
[1, 2, 3, 4]
```

The variable reference is constant, not the contents.

---

# Comparison Table

| Feature         | var | let | const |
| --------------- | --- | --- | ----- |
| Can Update      | ✅   | ✅   | ❌     |
| Can Redeclare   | ✅   | ❌   | ❌     |
| Block Scoped    | ❌   | ✅   | ✅     |
| Must Initialize | ❌   | ❌   | ✅     |

---

# Which One Should You Use?

### Use `const` by default

```javascript
const company = "OpenAI";
```

### Use `let` when the value will change

```javascript
let score = 0;
score++;
```

### Avoid `var` in modern JavaScript

```javascript
// Not recommended
var name = "John";
```

---

# Quick Rule

```text
Value never changes?  → const
Value changes?        → let
Writing modern JS?    → Avoid var
```

Most modern JavaScript code uses **`const` first**, and **`let` when reassignment is needed**.
