# JavaScript Data Types

JavaScript data types define the kind of value a variable can store.

```javascript
let age = 18;
let name = "Bro";
```

Here:

* `age` stores a number.
* `name` stores a string.

---

# Types of Data Types in JavaScript

JavaScript has **8 data types**.

## Primitive Data Types (Stored by Value)

### 1. Number

Used for integers and decimal numbers.

```javascript
let age = 18;
let price = 99.99;
```

```javascript
console.log(typeof age); // number
```

---

### 2. String

Used for text.

```javascript
let name = "Bro";
let city = 'Pune';
```

```javascript
console.log(typeof name); // string
```

---

### 3. Boolean

Represents true or false.

```javascript
let isLoggedIn = true;
let isAdmin = false;
```

```javascript
console.log(typeof isLoggedIn); // boolean
```

---

### 4. Undefined

A variable declared but not assigned a value.

```javascript
let x;
console.log(x);
```

Output:

```javascript
undefined
```

---

### 5. Null

Represents an intentional empty value.

```javascript
let user = null;
```

Example:

```javascript
let selectedUser = null;
```

Meaning: No user is selected.

---

### 6. BigInt

Used for very large integers.

```javascript
let bigNumber = 123456789012345678901234567890n;
```

The `n` at the end makes it a BigInt.

---

### 7. Symbol

Creates unique identifiers.

```javascript
let id1 = Symbol("id");
let id2 = Symbol("id");
```

```javascript
console.log(id1 === id2); // false
```

Even though both have the same description, they are unique.

---

## Non-Primitive Data Type (Reference Type)

### 8. Object

Objects store collections of data.

```javascript
let person = {
    name: "Bro",
    age: 18
};
```

Access values:

```javascript
console.log(person.name);
console.log(person.age);
```

---

# Arrays

Arrays are actually objects in JavaScript.

```javascript
let fruits = ["Apple", "Banana", "Mango"];
```

```javascript
console.log(typeof fruits); // object
```

Access elements:

```javascript
console.log(fruits[0]); // Apple
```

---

# Functions

Functions are also objects.

```javascript
function greet() {
    console.log("Hello");
}
```

```javascript
console.log(typeof greet); // function
```

---

# Primitive vs Reference Types

## Primitive

When copied, the actual value is copied.

```javascript
let a = 10;
let b = a;

b = 20;

console.log(a); // 10
console.log(b); // 20
```

---

## Reference

When copied, the reference (address) is copied.

```javascript
let obj1 = {
    name: "Bro"
};

let obj2 = obj1;

obj2.name = "Alex";

console.log(obj1.name); // Alex
```

Both variables point to the same object.

---

# typeof Operator

Used to check data type.

```javascript
console.log(typeof 10);          // number
console.log(typeof "Hello");     // string
console.log(typeof true);        // boolean
console.log(typeof undefined);   // undefined
console.log(typeof null);        // object (historical bug)
console.log(typeof {});          // object
console.log(typeof []);          // object
console.log(typeof function(){});// function
```

---

# Quick Summary Table

| Data Type | Example         |
| --------- | --------------- |
| Number    | `10`, `3.14`    |
| String    | `"Hello"`       |
| Boolean   | `true`, `false` |
| Undefined | `let x;`        |
| Null      | `null`          |
| BigInt    | `123456789n`    |
| Symbol    | `Symbol("id")`  |
| Object    | `{name: "Bro"}` |

---

# Easy Memory Trick

**Primitive = NSSUBBS**

* **N** → Number
* **S** → String
* **S** → Symbol
* **U** → Undefined
* **B** → Boolean
* **B** → BigInt
* **N** → Null

Everything else (Objects, Arrays, Functions, Dates, Maps, Sets) is an **Object/Reference Type**.

> **Rule:** If it's a simple single value → Primitive.
> If it can contain multiple values or properties → Reference Type (Object).
