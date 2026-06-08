# JavaScript Operators - Complete Guide

Operators are special symbols that perform operations on values and variables.

```javascript
let a = 10;
let b = 5;

console.log(a + b);
```

Here `+` is an operator.

---

# Types of Operators in JavaScript

```text
Operators
├── Arithmetic Operators
├── Assignment Operators
├── Comparison Operators
├── Logical Operators
├── Bitwise Operators
├── String Operators
├── Conditional (Ternary) Operator
├── Nullish Coalescing Operator
├── Optional Chaining Operator
├── Type Operators
├── Spread Operator
└── Comma Operator
```

---

# 1. Arithmetic Operators

Used for mathematical calculations.

| Operator | Meaning        |
| -------- | -------------- |
| +        | Addition       |
| -        | Subtraction    |
| *        | Multiplication |
| /        | Division       |
| %        | Modulus        |
| **       | Exponentiation |
| ++       | Increment      |
| --       | Decrement      |

---

## Addition (+)

```javascript
let a = 10;
let b = 20;

console.log(a + b);
```

Output

```javascript
30
```

---

## Subtraction (-)

```javascript
console.log(20 - 5);
```

Output

```javascript
15
```

---

## Multiplication (*)

```javascript
console.log(5 * 4);
```

Output

```javascript
20
```

---

## Division (/)

```javascript
console.log(20 / 4);
```

Output

```javascript
5
```

---

## Modulus (%)

Returns remainder.

```javascript
console.log(10 % 3);
```

Output

```javascript
1
```

### Why Useful?

Check even/odd.

```javascript
let num = 12;

if (num % 2 === 0) {
    console.log("Even");
}
```

---

## Exponentiation (**)

```javascript
console.log(2 ** 3);
```

Output

```javascript
8
```

Equivalent to:

```javascript
Math.pow(2, 3);
```

---

# Increment Operator (++)

Increases value by 1.

```javascript
let score = 5;

score++;

console.log(score);
```

Output

```javascript
6
```

---

## Post Increment (x++)

```javascript
let x = 5;

console.log(x++);
console.log(x);
```

Output

```javascript
5
6
```

### What Happens?

Step 1:

```javascript
console.log(x);
```

Prints current value.

Step 2:

```javascript
x = x + 1;
```

Updates value.

---

## Pre Increment (++x)

```javascript
let x = 5;

console.log(++x);
```

Output

```javascript
6
```

### What Happens?

```javascript
x = x + 1;
console.log(x);
```

---

## Difference

```javascript
let a = 5;

console.log(a++);
```

Output

```javascript
5
```

---

```javascript
let a = 5;

console.log(++a);
```

Output

```javascript
6
```

---

# Decrement (--)

Reduces value by 1.

```javascript
let count = 10;

count--;

console.log(count);
```

Output

```javascript
9
```

---

# Arithmetic Assignment Operators

| Operator | Example | Meaning    |
| -------- | ------- | ---------- |
| +=       | a += 5  | a = a + 5  |
| -=       | a -= 5  | a = a - 5  |
| *=       | a *= 5  | a = a * 5  |
| /=       | a /= 5  | a = a / 5  |
| %=       | a %= 5  | a = a % 5  |
| **=      | a **= 2 | a = a ** 2 |

---

Example

```javascript
let x = 10;

x += 5;

console.log(x);
```

Output

```javascript
15
```

---

# 2. Comparison Operators

Used to compare values.

Result is always:

```javascript
true
```

or

```javascript
false
```

---

| Operator | Meaning            |
| -------- | ------------------ |
| ==       | Equal Value        |
| ===      | Strict Equal       |
| !=       | Not Equal          |
| !==      | Strict Not Equal   |
| >        | Greater Than       |
| <        | Less Than          |
| >=       | Greater Than Equal |
| <=       | Less Than Equal    |

---

# Loose Equality (==)

Checks value only.

```javascript
console.log(5 == "5");
```

Output

```javascript
true
```

### Why?

JavaScript converts types automatically.

```javascript
Number("5")
```

becomes

```javascript
5
```

Then:

```javascript
5 == 5
```

Result:

```javascript
true
```

---

# Strict Equality (===)

Checks:

1. Value
2. Data Type

```javascript
console.log(5 === "5");
```

Output

```javascript
false
```

Because:

```javascript
number !== string
```

---

## Most Important Rule

Always prefer:

```javascript
===
```

instead of

```javascript
==
```

in modern JavaScript.

---

# Not Equal (!=)

```javascript
console.log(10 != 5);
```

Output

```javascript
true
```

---

# Strict Not Equal (!==)

```javascript
console.log(10 !== "10");
```

Output

```javascript
true
```

Types differ.

---

# Greater Than (>)

```javascript
console.log(20 > 10);
```

Output

```javascript
true
```

---

# Less Than (<)

```javascript
console.log(5 < 10);
```

Output

```javascript
true
```

---

# Greater Than or Equal (>=)

```javascript
console.log(10 >= 10);
```

Output

```javascript
true
```

---

# Less Than or Equal (<=)

```javascript
console.log(8 <= 10);
```

Output

```javascript
true
```

---

# String Comparison

JavaScript compares Unicode values.

```javascript
console.log("apple" < "banana");
```

Output

```javascript
true
```

---

```javascript
console.log("A" < "a");
```

Output

```javascript
true
```

Because:

```text
Unicode(A) = 65
Unicode(a) = 97
```

---

# 3. Logical Operators

Used to combine conditions.

| Operator | Meaning |
| -------- | ------- |
| &&       | AND     |
| ||       | OR      |
| !        | NOT     |

---

# AND (&&)

Both conditions must be true.

```javascript
let age = 20;

console.log(age > 18 && age < 30);
```

Output

```javascript
true
```

---

# OR (||)

Only one condition needs to be true.

```javascript
console.log(false || true);
```

Output

```javascript
true
```

---

# NOT (!)

Reverses result.

```javascript
console.log(!true);
```

Output

```javascript
false
```

---

# Short Circuiting

### AND

```javascript
false && anything
```

Result:

```javascript
false
```

Second part never runs.

---

### OR

```javascript
true || anything
```

Result:

```javascript
true
```

Second part never runs.

---

# 4. String Operator

The `+` operator joins strings.

```javascript
let first = "Hello";
let second = "World";

console.log(first + " " + second);
```

Output

```javascript
Hello World
```

---

# Template Literals (Preferred)

```javascript
let name = "Bro";

console.log(`Hello ${name}`);
```

Output

```javascript
Hello Bro
```

---

# 5. Ternary Operator

Short form of if-else.

Syntax

```javascript
condition ? value1 : value2
```

Example

```javascript
let age = 20;

let result =
    age >= 18
    ? "Adult"
    : "Minor";

console.log(result);
```

Output

```javascript
Adult
```

---

# 6. Nullish Coalescing Operator (??)

Returns right value only if left value is:

```javascript
null
```

or

```javascript
undefined
```

---

Example

```javascript
let username = null;

console.log(username ?? "Guest");
```

Output

```javascript
Guest
```

---

Difference from OR

```javascript
console.log(0 || 100);
```

Output

```javascript
100
```

---

```javascript
console.log(0 ?? 100);
```

Output

```javascript
0
```

Because 0 is not null or undefined.

---

# 7. Optional Chaining (?.)

Prevents errors when property doesn't exist.

```javascript
let user = {};

console.log(user.address?.city);
```

Output

```javascript
undefined
```

No crash.

---

Without Optional Chaining

```javascript
user.address.city
```

Error:

```javascript
Cannot read properties of undefined
```

---

# 8. Type Operators

## typeof

Returns data type.

```javascript
console.log(typeof 42);
```

Output

```javascript
number
```

---

```javascript
console.log(typeof "hello");
```

Output

```javascript
string
```

---

## instanceof

Checks object type.

```javascript
let date = new Date();

console.log(date instanceof Date);
```

Output

```javascript
true
```

---

# 9. Spread Operator (...)

Used to copy or expand arrays and objects.

### Arrays

```javascript
let nums = [1, 2, 3];

let copy = [...nums];
```

---

### Objects

```javascript
let user = {
    name: "Bro"
};

let newUser = {
    ...user,
    age: 18
};
```

---

# 10. Bitwise Operators

Used at binary level.

| Operator | Meaning     |
| -------- | ----------- |
| &        | AND         |
| |        | OR          |
| ^        | XOR         |
| ~        | NOT         |
| <<       | Left Shift  |
| >>       | Right Shift |

Example

```javascript
console.log(5 & 3);
```

Binary

```text
5 = 101
3 = 011
------------
    001
```

Output

```javascript
1
```

---

# Operator Precedence

```javascript
console.log(2 + 3 * 4);
```

Output

```javascript
14
```

Because:

```text
3 * 4 = 12
2 + 12 = 14
```

---

Use parentheses:

```javascript
console.log((2 + 3) * 4);
```

Output

```javascript
20
```

---

# Quick Revision

```text
Arithmetic
+ - * / % ** ++ --

Assignment
= += -= *= /= %= **=

Comparison
== === != !== > < >= <=

Logical
&& || !

String
+

Conditional
? :

Nullish
??

Optional Chaining
?.

Type
typeof
instanceof

Spread
...

Bitwise
& | ^ ~ << >>

Best Practice:
✔ Use === instead of ==
✔ Use const by default
✔ Use let when value changes
✔ Use ?? for defaults
✔ Use ?. for safe property access
✔ Use template literals instead of string concatenation
```

For modern JavaScript interviews, the most commonly tested operator topics are **`=== vs ==`**, **pre/post increment (`++x` vs `x++`)**, **short-circuiting with `&&` and `||`**, **nullish coalescing (`??`)**, **optional chaining (`?.`)**, and **operator precedence**.
