# JavaScript Numbers, Date & Time (In-Depth Guide)

---

# Numbers in JavaScript

A **Number** represents both integer and floating-point values.

Unlike languages like C++ or Java, JavaScript has only **one numeric type** for most calculations.

```javascript
let age = 18;
let price = 99.99;
let temperature = -5;

console.log(typeof age); // number
```

---

## Number Types

### Integers

```javascript
let count = 100;
```

### Floating Point Numbers

```javascript
let pi = 3.14159;
```

### Negative Numbers

```javascript
let debt = -500;
```

### Scientific Notation

```javascript
let num = 1e6;

console.log(num); // 1000000
```

---

## Special Number Values

### Infinity

Occurs when a number becomes too large.

```javascript
console.log(1 / 0);
```

Output:

```javascript
Infinity
```

---

### -Infinity

```javascript
console.log(-1 / 0);
```

Output:

```javascript
-Infinity
```

---

### NaN (Not a Number)

Represents an invalid mathematical operation.

```javascript
console.log("hello" * 5);
```

Output:

```javascript
NaN
```

---

## Number Methods

### toString()

Converts number to string.

```javascript
let num = 100;

console.log(num.toString());
```

Output:

```javascript
"100"
```

---

### toFixed()

Controls decimal places.

```javascript
let pi = 3.14159265;

console.log(pi.toFixed(2));
```

Output:

```javascript
3.14
```

---

### toPrecision()

Controls total digits.

```javascript
let num = 123.456;

console.log(num.toPrecision(4));
```

Output:

```javascript
123.5
```

---

## Converting to Numbers

### Number()

```javascript
Number("100");
```

Output:

```javascript
100
```

---

### parseInt()

Extracts integer.

```javascript
parseInt("100px");
```

Output:

```javascript
100
```

---

### parseFloat()

Extracts decimal number.

```javascript
parseFloat("99.99kg");
```

Output:

```javascript
99.99
```

---

## Math Object

JavaScript provides the built-in `Math` object.

### Math.round()

```javascript
Math.round(4.7);
```

Output:

```javascript
5
```

---

### Math.floor()

Rounds downward.

```javascript
Math.floor(4.9);
```

Output:

```javascript
4
```

---

### Math.ceil()

Rounds upward.

```javascript
Math.ceil(4.1);
```

Output:

```javascript
5
```

---

### Math.random()

Generates random number between 0 and 1.

```javascript
Math.random();
```

Example:

```javascript
0.67342
```

---

### Random Number Between 1 and 100

```javascript
let random = Math.floor(Math.random() * 100) + 1;
```

---

# Date & Time in JavaScript

JavaScript stores date and time using the built-in `Date` object.

---

## Creating a Date

### Current Date and Time

```javascript
let now = new Date();

console.log(now);
```

Example Output:

```javascript
2026-06-08T15:45:20.000Z
```

---

### Specific Date

```javascript
let birthday = new Date("2007-10-15");
```

---

### Using Individual Values

```javascript
let date = new Date(2026, 5, 8);
```

⚠ Months start from 0

| Month    | Value |
| -------- | ----- |
| January  | 0     |
| February | 1     |
| March    | 2     |
| April    | 3     |
| May      | 4     |
| June     | 5     |

```javascript
new Date(2026, 5, 8);
```

means

```javascript
8 June 2026
```

---

# Getting Date Components

Assume:

```javascript
let now = new Date();
```

---

## getFullYear()

```javascript
console.log(now.getFullYear());
```

Output:

```javascript
2026
```

---

## getMonth()

```javascript
console.log(now.getMonth());
```

Output:

```javascript
5
```

(June)

---

## getDate()

Returns day of month.

```javascript
console.log(now.getDate());
```

Output:

```javascript
8
```

---

## getDay()

Returns weekday.

| Day       | Value |
| --------- | ----- |
| Sunday    | 0     |
| Monday    | 1     |
| Tuesday   | 2     |
| Wednesday | 3     |
| Thursday  | 4     |
| Friday    | 5     |
| Saturday  | 6     |

```javascript
console.log(now.getDay());
```

Output:

```javascript
1
```

(Monday)

---

## getHours()

```javascript
console.log(now.getHours());
```

Example:

```javascript
18
```

---

## getMinutes()

```javascript
console.log(now.getMinutes());
```

Example:

```javascript
30
```

---

## getSeconds()

```javascript
console.log(now.getSeconds());
```

Example:

```javascript
45
```

---

# Setting Date Values

### setFullYear()

```javascript
let date = new Date();

date.setFullYear(2030);
```

---

### setMonth()

```javascript
date.setMonth(11);
```

December

---

### setDate()

```javascript
date.setDate(25);
```

---

# Timestamp

Every Date internally stores milliseconds since:

```text
1 January 1970 UTC
```

called the **Unix Epoch**.

---

## Date.now()

```javascript
console.log(Date.now());
```

Output:

```javascript
1780843000000
```

(milliseconds)

---

## Measuring Execution Time

```javascript
let start = Date.now();

// Some code

let end = Date.now();

console.log(end - start);
```

Output:

```javascript
25
```

(milliseconds)

---

# Formatting Dates

### toDateString()

```javascript
let now = new Date();

console.log(now.toDateString());
```

Output:

```javascript
Mon Jun 08 2026
```

---

### toTimeString()

```javascript
console.log(now.toTimeString());
```

Output:

```javascript
18:45:20 GMT+0530
```

---

### toLocaleDateString()

```javascript
console.log(
    now.toLocaleDateString()
);
```

Output:

```javascript
08/06/2026
```

---

### toLocaleTimeString()

```javascript
console.log(
    now.toLocaleTimeString()
);
```

Output:

```javascript
6:45:20 PM
```

---

# Useful Real-World Examples

## Digital Clock

```javascript
setInterval(() => {
    let now = new Date();

    console.log(
        now.toLocaleTimeString()
    );
}, 1000);
```

Runs every second.

---

## Calculate Age

```javascript
let birthYear = 2007;

let currentYear =
    new Date().getFullYear();

let age =
    currentYear - birthYear;

console.log(age);
```

---

## Countdown Example

```javascript
let examDate =
    new Date("2027-01-15");

let today =
    new Date();

let difference =
    examDate - today;

let days =
    Math.floor(
        difference /
        (1000 * 60 * 60 * 24)
    );

console.log(days);
```

---

# Interview-Level Concepts

### Number is a Primitive

```javascript
let a = 10;
```

Stored directly as a value.

---

### Date is an Object

```javascript
let d = new Date();
```

Stored as an object reference.

---

### Comparison

```javascript
let d1 = new Date();
let d2 = new Date();

console.log(d1 === d2);
```

Output:

```javascript
false
```

Different object references.

---

# Quick Revision

```text
Number
├── Integer
├── Decimal
├── Infinity
├── -Infinity
└── NaN

Useful Number Functions
├── Number()
├── parseInt()
├── parseFloat()
├── toFixed()
├── toPrecision()
└── Math Object

Date
├── new Date()
├── getFullYear()
├── getMonth()
├── getDate()
├── getDay()
├── getHours()
├── getMinutes()
├── getSeconds()
├── setDate()
├── setMonth()
└── Date.now()

Time
├── Timestamp
├── Unix Epoch
├── toLocaleTimeString()
├── toLocaleDateString()
└── setInterval()
```

If you're learning backend JavaScript (Node.js), mastering **Numbers, Math, Date, Time, Timestamps, Time Zones, and Date Formatting** is essential because they're used constantly in APIs, databases, authentication systems, logging, scheduling, and analytics.
