# The DOM in JavaScript — A Complete In-Depth Guide

The **Document Object Model (DOM)** is the live, structured representation of an HTML page in memory. JavaScript uses the DOM to read, create, modify, and delete every part of a webpage — dynamically, without reloading.

---

## Table of Contents

1. [What is the DOM?](#1-what-is-the-dom)
2. [Selecting Elements](#2-selecting-elements)
3. [Traversing the DOM](#3-traversing-the-dom)
4. [Reading & Changing Content](#4-reading--changing-content)
5. [Reading & Changing Attributes](#5-reading--changing-attributes)
6. [Styling Elements](#6-styling-elements)
7. [Working with Classes](#7-working-with-classes)
8. [Creating & Inserting Elements](#8-creating--inserting-elements)
9. [Removing & Replacing Elements](#9-removing--replacing-elements)
10. [Cloning Elements](#10-cloning-elements)
11. [Event Handling](#11-event-handling)
12. [Event Object](#12-event-object)
13. [Event Propagation — Bubbling & Capturing](#13-event-propagation--bubbling--capturing)
14. [Event Delegation](#14-event-delegation)
15. [Forms & Input Handling](#15-forms--input-handling)
16. [DOM Dimensions & Scroll](#16-dom-dimensions--scroll)
17. [Mutation Observer](#17-mutation-observer)
18. [Performance Best Practices](#18-performance-best-practices)
19. [Quick Reference Cheat Sheet](#19-quick-reference-cheat-sheet)

---

## 1. What is the DOM?

When a browser loads an HTML page, it parses the markup and builds a **tree of objects** in memory. This tree is called the DOM.

```
document
└── html
    ├── head
    │   └── title
    └── body
        ├── h1
        ├── p
        └── div
            ├── span
            └── button
```

Every HTML tag becomes a **Node**. JavaScript can access, modify, add, or remove any node at any time.

### The `window` and `document` Objects

```js
// window — the global object representing the browser tab
console.log(window.innerWidth);  // viewport width

// document — the entry point to the DOM
console.log(document.title);     // page title
console.log(document.URL);       // current URL
console.log(document.body);      // <body> element
console.log(document.head);      // <head> element
```

### Node Types

| Node Type          | Description                        | Example              |
|--------------------|------------------------------------|----------------------|
| Element Node       | An HTML element                    | `<div>`, `<p>`       |
| Text Node          | Text inside an element             | `"Hello World"`      |
| Attribute Node     | An attribute of an element         | `class="box"`        |
| Comment Node       | HTML comments                      | `<!-- comment -->`   |
| Document Node      | The entire document                | `document`           |

---

## 2. Selecting Elements

This is how you grab references to HTML elements so you can work with them.

### `getElementById`

Returns a **single element** by its `id`.

```html
<h1 id="title">Hello World</h1>
```

```js
const title = document.getElementById("title");
console.log(title); // <h1 id="title">Hello World</h1>
```

### `getElementsByClassName`

Returns an **HTMLCollection** (live, array-like) of all elements with that class.

```html
<p class="note">Note 1</p>
<p class="note">Note 2</p>
<p class="note">Note 3</p>
```

```js
const notes = document.getElementsByClassName("note");
console.log(notes.length); // 3
console.log(notes[0]);     // <p class="note">Note 1</p>

// Convert to real array to use array methods
const notesArray = Array.from(notes);
notesArray.forEach((note) => console.log(note.textContent));
```

### `getElementsByTagName`

Returns all elements with a given tag name.

```js
const paragraphs = document.getElementsByTagName("p");
const allDivs     = document.getElementsByTagName("div");
const everything  = document.getElementsByTagName("*"); // every element
```

### `querySelector` ⭐

Returns the **first element** matching a CSS selector. The most flexible selector.

```js
const title     = document.querySelector("#title");         // by id
const note      = document.querySelector(".note");          // first .note
const btn       = document.querySelector("button");         // first button
const input     = document.querySelector("input[type=email]");
const firstItem = document.querySelector("ul > li:first-child");
const nested    = document.querySelector(".card .card-title");
```

### `querySelectorAll` ⭐

Returns a **NodeList** (static, array-like) of ALL elements matching a CSS selector.

```js
const allNotes  = document.querySelectorAll(".note");
const allInputs = document.querySelectorAll("input");
const checked   = document.querySelectorAll("input[type=checkbox]:checked");

// NodeList supports forEach directly
allNotes.forEach((note) => console.log(note.textContent));

// Convert to array for map/filter/reduce
const texts = [...allNotes].map((n) => n.textContent);
```

### Selector Comparison

| Method                    | Returns           | Live? | CSS Selector? |
|---------------------------|-------------------|-------|---------------|
| `getElementById`          | Single element    | Yes   | ❌             |
| `getElementsByClassName`  | HTMLCollection    | Yes   | ❌             |
| `getElementsByTagName`    | HTMLCollection    | Yes   | ❌             |
| `querySelector`           | Single element    | No    | ✅             |
| `querySelectorAll`        | NodeList          | No    | ✅             |

> **Live** means the collection automatically updates if the DOM changes.

---

## 3. Traversing the DOM

Navigate between elements using parent/child/sibling relationships.

### Parent

```html
<div class="container">
  <p id="para">Hello</p>
</div>
```

```js
const para = document.getElementById("para");

console.log(para.parentElement);      // <div class="container">
console.log(para.parentNode);         // same (parentNode includes non-element nodes)
```

### Children

```html
<ul id="list">
  <li>Item 1</li>
  <li>Item 2</li>
  <li>Item 3</li>
</ul>
```

```js
const list = document.getElementById("list");

console.log(list.children);           // HTMLCollection of <li> elements
console.log(list.childNodes);         // NodeList including text/comment nodes
console.log(list.firstElementChild);  // <li>Item 1</li>
console.log(list.lastElementChild);   // <li>Item 3</li>
console.log(list.childElementCount);  // 3
```

### Siblings

```js
const list = document.getElementById("list");
const first = list.firstElementChild;

console.log(first.nextElementSibling);     // <li>Item 2</li>
console.log(first.previousElementSibling); // null (no previous)

const last = list.lastElementChild;
console.log(last.previousElementSibling);  // <li>Item 2</li>
```

### Closest (Walk Up the Tree)

Find the nearest ancestor matching a selector — very useful in event handlers.

```js
// <div class="card">
//   <div class="card-body">
//     <button class="delete-btn">Delete</button>
//   </div>
// </div>

document.querySelector(".delete-btn").addEventListener("click", (e) => {
  const card = e.target.closest(".card");
  card.remove();
});
```

### `contains` and `matches`

```js
const list = document.querySelector("ul");
const item = document.querySelector("li");

console.log(list.contains(item));       // true — item is inside list
console.log(item.matches(".active"));   // true if item has class "active"
console.log(item.matches("li:first-child")); // true/false
```

---

## 4. Reading & Changing Content

### `textContent`

Gets or sets the raw text inside an element. Ignores HTML tags.

```js
const heading = document.querySelector("h1");

// Read
console.log(heading.textContent); // "Hello World"

// Write — safely sets text (no HTML parsing)
heading.textContent = "New Heading";
```

### `innerHTML`

Gets or sets the **HTML markup** inside an element.

```js
const box = document.querySelector(".box");

// Read
console.log(box.innerHTML); // "<p>Hello <strong>World</strong></p>"

// Write — parses and renders HTML
box.innerHTML = "<ul><li>One</li><li>Two</li></ul>";
```

> ⚠️ Never set `innerHTML` with untrusted user input — it creates **XSS vulnerabilities**.

### `outerHTML`

Gets or sets the element itself including its own tag.

```js
const p = document.querySelector("p");
console.log(p.outerHTML); // "<p class="note">Hello</p>"

// Replaces the element entirely
p.outerHTML = "<h2>Replaced!</h2>";
```

### `innerText` vs `textContent`

```js
// <p>Hello <span style="display:none">hidden</span> World</p>

const p = document.querySelector("p");
console.log(p.textContent); // "Hello hidden World" — reads ALL text nodes
console.log(p.innerText);   // "Hello  World"       — respects CSS visibility
```

### `value` — For Form Inputs

```js
const input = document.querySelector("input");
console.log(input.value);       // current value
input.value = "New Value";      // set value
```

---

## 5. Reading & Changing Attributes

### `getAttribute` / `setAttribute` / `removeAttribute`

```html
<a id="link" href="https://example.com" target="_blank">Visit</a>
```

```js
const link = document.getElementById("link");

// Read
console.log(link.getAttribute("href"));    // "https://example.com"
console.log(link.getAttribute("target"));  // "_blank"

// Write
link.setAttribute("href", "https://google.com");
link.setAttribute("title", "Go to Google");

// Remove
link.removeAttribute("target");

// Check
console.log(link.hasAttribute("href")); // true
```

### Direct Property Access

Most standard attributes are also accessible as properties:

```js
const img = document.querySelector("img");

img.src   = "photo.jpg";
img.alt   = "A beautiful photo";
img.width = 300;

const input = document.querySelector("input");
input.disabled = true;
input.required = true;
input.placeholder = "Enter name...";
```

### `data-*` Custom Attributes (Dataset)

```html
<div class="user-card" data-id="42" data-role="admin" data-active="true">
  Alice
</div>
```

```js
const card = document.querySelector(".user-card");

// Read
console.log(card.dataset.id);     // "42"
console.log(card.dataset.role);   // "admin"
console.log(card.dataset.active); // "true"

// Write
card.dataset.id = "99";
card.dataset.verified = "yes"; // creates data-verified="yes"

// Delete
delete card.dataset.active;
```

---

## 6. Styling Elements

### Inline Styles with `.style`

```js
const box = document.querySelector(".box");

box.style.backgroundColor = "royalblue";
box.style.color            = "white";
box.style.padding          = "20px";
box.style.borderRadius     = "8px";
box.style.fontSize         = "18px";
box.style.display          = "none";    // hide
box.style.display          = "";        // remove inline style (revert to CSS)
```

> CSS property names become camelCase in JS: `background-color` → `backgroundColor`

### Reading Computed Styles

`.style` only reads inline styles. To get the actual applied value use `getComputedStyle`:

```js
const box = document.querySelector(".box");
const styles = getComputedStyle(box);

console.log(styles.color);           // "rgb(0, 0, 0)"
console.log(styles.fontSize);        // "16px"
console.log(styles.backgroundColor); // "rgba(0, 0, 0, 0)"
console.log(styles.display);         // "block"
```

### CSS Custom Properties (Variables)

```js
// Read a CSS variable
const root = document.documentElement;
const primary = getComputedStyle(root).getPropertyValue("--primary-color").trim();

// Set a CSS variable globally
root.style.setProperty("--primary-color", "#6200ea");

// Set on a specific element
const card = document.querySelector(".card");
card.style.setProperty("--card-padding", "24px");
```

---

## 7. Working with Classes

### `classList` API ⭐

The cleanest way to add, remove, and toggle CSS classes.

```js
const box = document.querySelector(".box");

// Add a class
box.classList.add("active");
box.classList.add("highlighted", "visible"); // multiple at once

// Remove a class
box.classList.remove("active");

// Toggle — adds if absent, removes if present
box.classList.toggle("dark-mode");

// Check
console.log(box.classList.contains("active")); // true / false

// Replace one class with another
box.classList.replace("old-class", "new-class");

// Get all classes
console.log(box.classList);         // DOMTokenList ["box", "highlighted"]
console.log([...box.classList]);     // ["box", "highlighted"]
```

### Toggle with Force (Second Argument)

```js
// Force add (never removes)
box.classList.toggle("active", true);

// Force remove (never adds)
box.classList.toggle("active", false);
```

### `className` (Old Way)

```js
box.className = "card active highlighted"; // replaces ALL classes at once
console.log(box.className);               // "card active highlighted"
```

---

## 8. Creating & Inserting Elements

### `createElement`

```js
// Create a new element
const div = document.createElement("div");
div.className   = "card";
div.textContent = "New Card";
div.style.color = "blue";

// Must be inserted into the document to appear
document.body.appendChild(div);
```

### `createTextNode`

```js
const text = document.createTextNode("Hello World");
const p    = document.createElement("p");
p.appendChild(text);
document.body.appendChild(p);
```

### Insertion Methods

```html
<ul id="list">
  <li>Existing Item</li>
</ul>
```

```js
const list = document.getElementById("list");
const newItem = document.createElement("li");
newItem.textContent = "New Item";

// At the end of a parent
list.appendChild(newItem);

// At a specific position — insertBefore(newNode, referenceNode)
list.insertBefore(newItem, list.firstElementChild); // insert at top

// Modern insertAdjacentElement
// 'beforebegin' — before the element itself
// 'afterbegin'  — inside, before first child
// 'beforeend'   — inside, after last child
// 'afterend'    — after the element itself

list.insertAdjacentElement("beforeend", newItem);
list.insertAdjacentHTML("beforeend", "<li>Quick HTML Item</li>");
list.insertAdjacentText("beforeend", "Some text");
```

### `append` and `prepend` (Modern)

```js
const container = document.querySelector(".container");
const p1 = document.createElement("p");
const p2 = document.createElement("p");

p1.textContent = "Paragraph 1";
p2.textContent = "Paragraph 2";

container.append(p1, p2);             // insert multiple, accepts strings too
container.prepend("First text node"); // insert at beginning
container.append("Trailing text");    // append a string directly
```

### `before` and `after`

Insert elements relative to a sibling:

```js
const existing = document.querySelector(".existing");
const newDiv   = document.createElement("div");
newDiv.textContent = "I am inserted before";

existing.before(newDiv);  // inserts before .existing
existing.after(newDiv);   // inserts after  .existing
```

### Building a Full Component

```js
function createUserCard({ name, role, avatar }) {
  const card     = document.createElement("div");
  const img      = document.createElement("img");
  const nameEl   = document.createElement("h3");
  const roleEl   = document.createElement("p");

  card.className   = "user-card";
  img.src          = avatar;
  img.alt          = name;
  nameEl.textContent = name;
  roleEl.textContent = role;

  card.append(img, nameEl, roleEl);
  return card;
}

const card = createUserCard({
  name:   "Alice",
  role:   "Frontend Developer",
  avatar: "alice.jpg",
});

document.body.appendChild(card);
```

---

## 9. Removing & Replacing Elements

### `remove()`

```js
const element = document.querySelector(".toast");
element.remove(); // removes itself from the DOM
```

### `removeChild()`

```js
const parent = document.querySelector("ul");
const child  = document.querySelector("li.done");

parent.removeChild(child);
```

### `replaceWith()`

```js
const oldEl = document.querySelector(".old-header");
const newEl = document.createElement("h2");
newEl.textContent = "New Header";

oldEl.replaceWith(newEl); // replaces oldEl with newEl
oldEl.replaceWith("plain text"); // can replace with text too
```

### `replaceChild()`

```js
const parent = document.querySelector(".container");
const oldEl  = document.querySelector(".old");
const newEl  = document.createElement("div");
newEl.textContent = "Replacement";

parent.replaceChild(newEl, oldEl);
```

### Clear All Children

```js
const list = document.querySelector("ul");

// Method 1 — fastest
list.innerHTML = "";

// Method 2 — while loop
while (list.firstChild) {
  list.removeChild(list.firstChild);
}

// Method 3 — replaceChildren (modern)
list.replaceChildren(); // no arguments = clear all
```

---

## 10. Cloning Elements

### `cloneNode`

```js
const original = document.querySelector(".card");

// Shallow clone — copies element only, not children
const shallowCopy = original.cloneNode(false);

// Deep clone — copies element AND all children
const deepCopy = original.cloneNode(true);

document.body.appendChild(deepCopy);
```

> ⚠️ `cloneNode(true)` copies `id` attributes too — update them to keep IDs unique.

---

## 11. Event Handling

Events let you respond to user interactions and browser actions.

### `addEventListener` ⭐

```js
const btn = document.querySelector("button");

btn.addEventListener("click", function (event) {
  console.log("Button clicked!", event);
});

// Arrow function
btn.addEventListener("click", (e) => {
  console.log("Clicked with arrow function");
});
```

### Removing an Event Listener

```js
function handleClick(e) {
  console.log("Clicked");
}

btn.addEventListener("click", handleClick);
btn.removeEventListener("click", handleClick); // must pass the same function reference
```

### Common Event Types

#### Mouse Events

```js
btn.addEventListener("click",       (e) => console.log("click"));
btn.addEventListener("dblclick",    (e) => console.log("double click"));
btn.addEventListener("mousedown",   (e) => console.log("mouse pressed"));
btn.addEventListener("mouseup",     (e) => console.log("mouse released"));
btn.addEventListener("mouseenter",  (e) => console.log("mouse entered"));  // no bubbling
btn.addEventListener("mouseleave",  (e) => console.log("mouse left"));     // no bubbling
btn.addEventListener("mouseover",   (e) => console.log("mouse over"));     // bubbles
btn.addEventListener("mousemove",   (e) => console.log(e.clientX, e.clientY));
btn.addEventListener("contextmenu", (e) => { e.preventDefault(); console.log("right click"); });
```

#### Keyboard Events

```js
document.addEventListener("keydown",  (e) => console.log("key down:", e.key));
document.addEventListener("keyup",    (e) => console.log("key up:",   e.key));
document.addEventListener("keypress", (e) => console.log("key press:", e.key)); // deprecated

// Specific key check
document.addEventListener("keydown", (e) => {
  if (e.key === "Enter")   console.log("Enter pressed");
  if (e.key === "Escape")  console.log("Escape pressed");
  if (e.ctrlKey && e.key === "s") {
    e.preventDefault();
    console.log("Ctrl+S pressed");
  }
});
```

#### Form Events

```js
const form  = document.querySelector("form");
const input = document.querySelector("input");
const select = document.querySelector("select");

form.addEventListener("submit",  (e) => { e.preventDefault(); console.log("form submitted"); });
form.addEventListener("reset",   (e) => console.log("form reset"));
input.addEventListener("focus",  (e) => console.log("input focused"));
input.addEventListener("blur",   (e) => console.log("input blurred"));
input.addEventListener("input",  (e) => console.log("live value:", e.target.value));
input.addEventListener("change", (e) => console.log("value committed:", e.target.value));
select.addEventListener("change",(e) => console.log("selected:", e.target.value));
```

#### Window / Document Events

```js
window.addEventListener("load",   ()  => console.log("all resources loaded"));
window.addEventListener("resize", ()  => console.log(window.innerWidth));
window.addEventListener("scroll", ()  => console.log(window.scrollY));
window.addEventListener("online",  () => console.log("Back online"));
window.addEventListener("offline", () => console.log("Gone offline"));

document.addEventListener("DOMContentLoaded", () => {
  console.log("DOM ready — before images/CSS");
});
```

### Event Listener Options

```js
btn.addEventListener("click", handler, {
  once:    true,   // fire only once, then auto-remove
  capture: false,  // use bubbling phase (default)
  passive: true,   // never calls preventDefault (for scroll perf)
});
```

---

## 12. Event Object

Every event handler receives an **event object** packed with information about what happened.

```js
document.addEventListener("click", (event) => {
  // What was clicked
  console.log(event.target);          // element that was clicked
  console.log(event.currentTarget);   // element the listener is attached to

  // Mouse position
  console.log(event.clientX, event.clientY); // relative to viewport
  console.log(event.pageX,   event.pageY);   // relative to full page
  console.log(event.offsetX, event.offsetY); // relative to the element

  // Keyboard
  console.log(event.key);       // "Enter", "a", "ArrowUp"
  console.log(event.code);      // "KeyA", "Enter", "Space"
  console.log(event.shiftKey);  // true/false
  console.log(event.ctrlKey);   // true/false
  console.log(event.altKey);    // true/false

  // Type
  console.log(event.type);      // "click", "keydown", etc.

  // Prevent default behavior (e.g. stop a link from navigating)
  event.preventDefault();

  // Stop propagation
  event.stopPropagation();
});
```

---

## 13. Event Propagation — Bubbling & Capturing

When an event fires on an element, it travels through the DOM in three phases:

```
1. CAPTURE  (top → down)  document → html → body → div → button
2. TARGET                 the element itself
3. BUBBLE   (bottom → up) button → div → body → html → document
```

### Bubbling Example

```html
<div id="outer">
  <div id="inner">
    <button id="btn">Click Me</button>
  </div>
</div>
```

```js
document.getElementById("btn").addEventListener("click",   () => console.log("button"));
document.getElementById("inner").addEventListener("click", () => console.log("inner div"));
document.getElementById("outer").addEventListener("click", () => console.log("outer div"));

// Clicking the button prints:
// "button"
// "inner div"
// "outer div"
```

### `stopPropagation`

Stops the event from bubbling further up.

```js
document.getElementById("btn").addEventListener("click", (e) => {
  e.stopPropagation(); // inner and outer won't fire
  console.log("button only");
});
```

### `stopImmediatePropagation`

Stops both bubbling AND any other listeners on the same element.

```js
btn.addEventListener("click", (e) => {
  e.stopImmediatePropagation();
  console.log("First handler — stops everything");
});

btn.addEventListener("click", () => {
  console.log("This never runs");
});
```

### Capture Phase

Pass `{ capture: true }` to listen during the capture phase (top-down).

```js
document.getElementById("outer").addEventListener("click", () => {
  console.log("outer — CAPTURE phase");
}, { capture: true });

document.getElementById("btn").addEventListener("click", () => {
  console.log("button — BUBBLE phase");
});

// Clicking button prints:
// "outer — CAPTURE phase"
// "button — BUBBLE phase"
```

---

## 14. Event Delegation

Instead of attaching listeners to every child, attach **one listener to a parent** and use `event.target` to identify what was clicked. This is efficient and works for dynamically added elements.

### Without Delegation (bad for many items)

```js
// Creates 100 event listeners
document.querySelectorAll("li").forEach((li) => {
  li.addEventListener("click", () => console.log(li.textContent));
});
```

### With Delegation (one listener for all)

```js
const list = document.querySelector("ul");

list.addEventListener("click", (e) => {
  if (e.target.tagName === "LI") {
    console.log("Clicked:", e.target.textContent);
    e.target.classList.toggle("done");
  }
});

// Works for dynamically added <li> elements too!
```

### Real-World Delegation — Action Buttons

```html
<div id="user-list">
  <div class="user" data-id="1">
    Alice
    <button class="edit-btn">Edit</button>
    <button class="delete-btn">Delete</button>
  </div>
</div>
```

```js
document.getElementById("user-list").addEventListener("click", (e) => {
  const card = e.target.closest(".user");
  if (!card) return;

  const id = card.dataset.id;

  if (e.target.matches(".edit-btn"))   editUser(id);
  if (e.target.matches(".delete-btn")) deleteUser(id);
});
```

---

## 15. Forms & Input Handling

### Reading Form Data

```html
<form id="signup-form">
  <input type="text"     name="username"  placeholder="Username">
  <input type="email"    name="email"     placeholder="Email">
  <input type="password" name="password"  placeholder="Password">
  <input type="checkbox" name="terms"     id="terms">
  <select name="country">
    <option value="in">India</option>
    <option value="us">USA</option>
  </select>
  <button type="submit">Sign Up</button>
</form>
```

```js
const form = document.getElementById("signup-form");

form.addEventListener("submit", (e) => {
  e.preventDefault();

  // Method 1 — direct access
  const username = form.username.value;
  const email    = form.email.value;
  const terms    = form.terms.checked;   // boolean for checkboxes
  const country  = form.country.value;

  // Method 2 — FormData API
  const data = new FormData(form);
  console.log(data.get("username"));     // "Alice"
  console.log(Object.fromEntries(data)); // { username: "Alice", email: "...", ... }
});
```

### Live Input Validation

```js
const emailInput = document.querySelector("input[type=email]");

emailInput.addEventListener("input", (e) => {
  const value = e.target.value;
  const isValid = value.includes("@") && value.includes(".");

  emailInput.style.borderColor = isValid ? "green" : "red";
});
```

### Focus and Blur

```js
const input = document.querySelector("input");

input.addEventListener("focus", () => {
  input.parentElement.classList.add("focused");
});

input.addEventListener("blur", () => {
  input.parentElement.classList.remove("focused");
  if (!input.value) {
    input.parentElement.classList.add("error");
  }
});
```

### Controlling the Form Programmatically

```js
const form = document.getElementById("my-form");

form.reset();               // clear all fields
form.checkValidity();       // true if all fields are valid
form.reportValidity();      // show built-in validation messages
document.querySelector("input").focus(); // auto-focus a field
```

---

## 16. DOM Dimensions & Scroll

### Element Size

```js
const box = document.querySelector(".box");

// Includes padding, excludes border/margin
console.log(box.clientWidth,  box.clientHeight);

// Includes padding + border (most useful)
console.log(box.offsetWidth,  box.offsetHeight);

// Includes overflow (scrollable content size)
console.log(box.scrollWidth,  box.scrollHeight);

// Precise decimal values — use for animations
const rect = box.getBoundingClientRect();
console.log(rect.top, rect.left, rect.width, rect.height);
console.log(rect.x,   rect.y);
```

### Position

```js
const box = document.querySelector(".box");

// Distance from positioned parent
console.log(box.offsetLeft, box.offsetTop);

// Distance from page top/left
function getAbsolutePosition(el) {
  const rect = el.getBoundingClientRect();
  return {
    x: rect.left + window.scrollX,
    y: rect.top  + window.scrollY,
  };
}
```

### Scrolling

```js
// Check scroll position
console.log(window.scrollX); // horizontal scroll
console.log(window.scrollY); // vertical scroll

// Scroll to position
window.scrollTo(0, 500);     // x, y
window.scrollTo({ top: 500, behavior: "smooth" });

// Scroll by a delta
window.scrollBy(0, 100);
window.scrollBy({ top: 100, behavior: "smooth" });

// Scroll element into view
document.querySelector("#section3").scrollIntoView({ behavior: "smooth" });

// Check if element is in viewport
function isInViewport(el) {
  const rect = el.getBoundingClientRect();
  return rect.top >= 0 && rect.bottom <= window.innerHeight;
}
```

---

## 17. Mutation Observer

Watch for changes to the DOM — added/removed elements, attribute changes, text changes.

```js
const target = document.querySelector("#container");

const observer = new MutationObserver((mutations) => {
  for (const mutation of mutations) {
    if (mutation.type === "childList") {
      console.log("Children changed:");
      mutation.addedNodes.forEach((n) => console.log("Added:", n));
      mutation.removedNodes.forEach((n) => console.log("Removed:", n));
    }
    if (mutation.type === "attributes") {
      console.log(`Attribute "${mutation.attributeName}" changed`);
    }
    if (mutation.type === "characterData") {
      console.log("Text content changed");
    }
  }
});

observer.observe(target, {
  childList:     true,   // watch for added/removed children
  attributes:    true,   // watch for attribute changes
  subtree:       true,   // watch all descendants (not just direct children)
  characterData: true,   // watch for text content changes
});

// Stop observing
observer.disconnect();
```

---

## 18. Performance Best Practices

### Batch DOM Reads and Writes

Interleaving reads and writes causes **layout thrashing** (the browser recalculates layout repeatedly).

```js
// ❌ Bad — reads and writes alternate, causing multiple reflows
elements.forEach((el) => {
  const height = el.offsetHeight;  // read
  el.style.height = height + "px"; // write
});

// ✅ Good — batch reads first, then all writes
const heights = elements.map((el) => el.offsetHeight); // all reads
elements.forEach((el, i) => {
  el.style.height = heights[i] + "px";                 // all writes
});
```

### Use `DocumentFragment` for Bulk Inserts

A `DocumentFragment` exists off-DOM. Build everything in it, then do one single insert.

```js
const list = document.querySelector("ul");
const fragment = document.createDocumentFragment();

for (let i = 0; i < 1000; i++) {
  const li = document.createElement("li");
  li.textContent = `Item ${i + 1}`;
  fragment.appendChild(li);
}

list.appendChild(fragment); // one single DOM update
```

### Debounce Expensive Scroll/Resize Handlers

```js
function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}

window.addEventListener("resize", debounce(() => {
  console.log("Resized to:", window.innerWidth);
}, 200));
```

### Use CSS Classes Instead of Inline Styles

```js
// ❌ Multiple style mutations = multiple reflows
el.style.width      = "100px";
el.style.height     = "100px";
el.style.background = "red";

// ✅ One class swap = one reflow
el.classList.add("active-state");
```

### Prefer `textContent` Over `innerHTML` for Plain Text

```js
// ❌ innerHTML parses HTML — slower, XSS risk
el.innerHTML = userInput;

// ✅ textContent is faster and safe
el.textContent = userInput;
```

---

## 19. Quick Reference Cheat Sheet

### Selecting

```js
document.getElementById("id")
document.querySelector(".class")          // first match
document.querySelectorAll("div > p")      // all matches
element.closest(".ancestor")             // nearest ancestor
element.matches(".selector")             // boolean test
```

### Traversal

```js
el.parentElement
el.children                  // element children only
el.firstElementChild
el.lastElementChild
el.nextElementSibling
el.previousElementSibling
```

### Content

```js
el.textContent = "text"      // safe plain text
el.innerHTML   = "<b>html</b>"  // renders HTML
el.value                     // for inputs
```

### Attributes

```js
el.getAttribute("href")
el.setAttribute("href", "url")
el.removeAttribute("href")
el.hasAttribute("href")
el.dataset.key               // data-key attribute
```

### Classes

```js
el.classList.add("cls")
el.classList.remove("cls")
el.classList.toggle("cls")
el.classList.contains("cls") // boolean
el.classList.replace("old", "new")
```

### Styles

```js
el.style.color = "red"
getComputedStyle(el).color
document.documentElement.style.setProperty("--var", "value")
```

### Creating & Inserting

```js
document.createElement("div")
parent.appendChild(child)
parent.append(el1, el2, "text")
parent.prepend(el)
el.before(newEl)
el.after(newEl)
el.insertAdjacentHTML("beforeend", "<p>html</p>")
```

### Removing & Replacing

```js
el.remove()
parent.removeChild(child)
el.replaceWith(newEl)
parent.replaceChildren()     // clear all children
```

### Events

```js
el.addEventListener("click", handler)
el.removeEventListener("click", handler)
e.preventDefault()
e.stopPropagation()
e.target
e.currentTarget
```

### Size & Scroll

```js
el.getBoundingClientRect()
el.offsetWidth / el.offsetHeight
window.scrollY
window.scrollTo({ top: 0, behavior: "smooth" })
el.scrollIntoView({ behavior: "smooth" })
```

---

## Key Takeaways

- The **DOM** is a live tree of objects representing your HTML — JavaScript can read and change every node.
- Use **`querySelector` / `querySelectorAll`** for flexible CSS-based selection.
- Use **`textContent`** for plain text and **`innerHTML`** only when you trust the source.
- **`classList`** is the safest and cleanest API for toggling visual states.
- Attach events with **`addEventListener`** — never use inline `onclick` HTML attributes.
- Understand **event bubbling** — most events travel up the DOM and can be caught at any ancestor.
- Use **event delegation** to handle dynamic content and reduce memory usage.
- Use **`DocumentFragment`** and batched writes to keep DOM operations fast.
- **`MutationObserver`** lets you reactively watch for DOM changes without polling.
- Always sanitize user content before inserting into the DOM to prevent **XSS attacks**.