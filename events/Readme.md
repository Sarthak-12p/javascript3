# Events in JavaScript — A Complete In-Depth Guide

Events are the heartbeat of interactive JavaScript. Every click, keypress, scroll, resize, form submit, network response, and timer fires an event. Mastering events means mastering how your application responds to the world.

---

## Table of Contents

1. [What is an Event?](#1-what-is-an-event)
2. [Ways to Attach Events](#2-ways-to-attach-events)
3. [The Event Object](#3-the-event-object)
4. [Mouse Events](#4-mouse-events)
5. [Keyboard Events](#5-keyboard-events)
6. [Form Events](#6-form-events)
7. [Window & Document Events](#7-window--document-events)
8. [Clipboard Events](#8-clipboard-events)
9. [Drag & Drop Events](#9-drag--drop-events)
10. [Touch Events](#10-touch-events)
11. [Media Events](#11-media-events)
12. [Custom Events](#12-custom-events)
13. [Event Propagation — Bubbling & Capturing](#13-event-propagation--bubbling--capturing)
14. [Event Delegation](#14-event-delegation)
15. [Removing Event Listeners](#15-removing-event-listeners)
16. [Event Listener Options](#16-event-listener-options)
17. [preventDefault vs stopPropagation](#17-preventdefault-vs-stoppropagation)
18. [Debounce & Throttle](#18-debounce--throttle)
19. [Async Events & Promises](#19-async-events--promises)
20. [Common Patterns & Real-World Examples](#20-common-patterns--real-world-examples)
21. [Quick Reference Cheat Sheet](#21-quick-reference-cheat-sheet)

---

## 1. What is an Event?

An **event** is a signal fired by the browser (or your code) when something happens — a user action, a browser lifecycle moment, or a programmatic trigger.

```
User clicks a button
       ↓
Browser creates an Event object
       ↓
Browser dispatches the event on the target element
       ↓
Event travels through the DOM (capture → target → bubble)
       ↓
Your listener function runs with the Event object
```

### Event-Driven Programming

JavaScript runs on a **single thread** with an **event loop**. Your code doesn't run continuously — it sleeps and wakes up when an event fires.

```js
// This code does nothing until the user clicks
document.querySelector("button").addEventListener("click", () => {
  console.log("Woken up by a click!");
});
```

---

## 2. Ways to Attach Events

There are three ways to listen to events in JavaScript. Only one is recommended.

### ❌ Method 1 — Inline HTML Attribute (avoid)

```html
<!-- Mixes HTML and JS, hard to maintain, no separation of concerns -->
<button onclick="handleClick()">Click Me</button>

<script>
  function handleClick() {
    console.log("clicked");
  }
</script>
```

**Problems:** Only one handler allowed, pollutes global scope, can't easily remove, mixes concerns.

---

### ❌ Method 2 — DOM Property (avoid)

```js
const btn = document.querySelector("button");

btn.onclick = function () {
  console.log("clicked");
};

// Attaching another overwrites the first — only one at a time!
btn.onclick = function () {
  console.log("second handler — first is gone forever");
};
```

**Problems:** Only one listener per event per element.

---

### ✅ Method 3 — `addEventListener` (always use this)

```js
const btn = document.querySelector("button");

btn.addEventListener("click", () => {
  console.log("First handler");
});

btn.addEventListener("click", () => {
  console.log("Second handler — both run!");
});
```

**Advantages:**
- Multiple listeners on the same event
- Can be removed cleanly
- Supports options (`once`, `passive`, `capture`)
- Works on any EventTarget (elements, window, document, XMLHttpRequest, etc.)

---

## 3. The Event Object

Every event listener receives an **Event object** automatically. It contains everything about what happened.

```js
document.addEventListener("click", function (event) {
  console.log(event);
});
```

### Core Properties (all events)

```js
element.addEventListener("click", (e) => {
  console.log(e.type);           // "click" — the event type
  console.log(e.target);         // element that triggered the event
  console.log(e.currentTarget);  // element the listener is attached to
  console.log(e.timeStamp);      // milliseconds since page load
  console.log(e.isTrusted);      // true = real user action, false = scripted
  console.log(e.bubbles);        // whether this event bubbles
  console.log(e.cancelable);     // whether preventDefault() works
  console.log(e.defaultPrevented); // true if preventDefault() was called
});
```

### `target` vs `currentTarget`

This is one of the most important distinctions in event handling.

```html
<div id="outer">
  <button id="btn">Click</button>
</div>
```

```js
document.getElementById("outer").addEventListener("click", (e) => {
  console.log(e.target);        // <button id="btn"> — what was actually clicked
  console.log(e.currentTarget); // <div id="outer">  — where the listener lives
});
```

> `target` — origin of the event (could be a deep child)
> `currentTarget` — the element whose listener is currently running

---

## 4. Mouse Events

### All Mouse Events

```js
const box = document.querySelector(".box");

box.addEventListener("click",      (e) => console.log("Single click"));
box.addEventListener("dblclick",   (e) => console.log("Double click"));
box.addEventListener("mousedown",  (e) => console.log("Mouse button pressed"));
box.addEventListener("mouseup",    (e) => console.log("Mouse button released"));
box.addEventListener("mousemove",  (e) => console.log("Mouse moving"));
box.addEventListener("mouseenter", (e) => console.log("Mouse entered element"));  // no bubble
box.addEventListener("mouseleave", (e) => console.log("Mouse left element"));     // no bubble
box.addEventListener("mouseover",  (e) => console.log("Mouse over (bubbles)"));
box.addEventListener("mouseout",   (e) => console.log("Mouse out (bubbles)"));
box.addEventListener("contextmenu",(e) => {
  e.preventDefault();
  console.log("Right click at", e.clientX, e.clientY);
});
```

### Mouse Position Properties

```js
document.addEventListener("mousemove", (e) => {
  console.log(e.clientX, e.clientY);  // relative to viewport
  console.log(e.pageX,   e.pageY);    // relative to full page (includes scroll)
  console.log(e.screenX, e.screenY);  // relative to the monitor screen
  console.log(e.offsetX, e.offsetY);  // relative to the target element
  console.log(e.movementX, e.movementY); // delta since last mousemove
});
```

### Which Button Was Pressed

```js
document.addEventListener("mousedown", (e) => {
  switch (e.button) {
    case 0: console.log("Left button");   break;
    case 1: console.log("Middle button"); break;
    case 2: console.log("Right button");  break;
    case 3: console.log("Back button");   break;
    case 4: console.log("Forward button");break;
  }

  // Check multiple buttons held at once via e.buttons (bitmask)
  if (e.buttons === 3) console.log("Left + Right held simultaneously");
});
```

### Modifier Keys During Mouse Events

```js
document.addEventListener("click", (e) => {
  if (e.shiftKey) console.log("Shift + Click");
  if (e.ctrlKey)  console.log("Ctrl + Click");
  if (e.altKey)   console.log("Alt + Click");
  if (e.metaKey)  console.log("Cmd + Click (Mac)");
});
```

### Real-World — Live Cursor Tracker

```js
const cursor = document.querySelector(".cursor");

document.addEventListener("mousemove", (e) => {
  cursor.style.transform = `translate(${e.clientX}px, ${e.clientY}px)`;
});
```

### Real-World — Drag to Move an Element

```js
const box = document.querySelector(".draggable");
let dragging = false;
let offsetX, offsetY;

box.addEventListener("mousedown", (e) => {
  dragging = true;
  offsetX  = e.clientX - box.offsetLeft;
  offsetY  = e.clientY - box.offsetTop;
});

document.addEventListener("mousemove", (e) => {
  if (!dragging) return;
  box.style.left = `${e.clientX - offsetX}px`;
  box.style.top  = `${e.clientY - offsetY}px`;
});

document.addEventListener("mouseup", () => {
  dragging = false;
});
```

---

## 5. Keyboard Events

### Event Types

| Event      | Fires When                       | Repeats on Hold |
|------------|----------------------------------|-----------------|
| `keydown`  | Key is pressed down              | ✅ Yes           |
| `keyup`    | Key is released                  | ❌ No            |
| `keypress` | Character key pressed (deprecated)| ✅ Yes          |

> Always use `keydown` or `keyup`. `keypress` is deprecated and doesn't fire for non-character keys.

### Key Properties

```js
document.addEventListener("keydown", (e) => {
  console.log(e.key);      // "a", "Enter", "ArrowUp", " " (space)
  console.log(e.code);     // "KeyA", "Enter", "ArrowUp", "Space" — physical key
  console.log(e.keyCode);  // 65, 13, 38, 32 — deprecated number code
  console.log(e.repeat);   // true if key is being held down

  // Modifier keys
  console.log(e.shiftKey); // true if Shift held
  console.log(e.ctrlKey);  // true if Ctrl held
  console.log(e.altKey);   // true if Alt held
  console.log(e.metaKey);  // true if Cmd (Mac) / Win (Windows)
});
```

### `e.key` vs `e.code`

```js
// e.key — what the key MEANS (layout-dependent)
// e.code — the PHYSICAL key on the keyboard (layout-independent)

// On a US keyboard, pressing the "A" key:
// e.key  = "a" (or "A" if Shift held)
// e.code = "KeyA" (always)

// Use e.key for character input
// Use e.code for shortcuts and game controls (WASD, arrow keys)
```

### Common Key Values

```js
document.addEventListener("keydown", (e) => {
  if (e.key === "Enter")      console.log("Enter");
  if (e.key === "Escape")     console.log("Escape");
  if (e.key === "Tab")        console.log("Tab");
  if (e.key === "Backspace")  console.log("Backspace");
  if (e.key === "Delete")     console.log("Delete");
  if (e.key === "ArrowUp")    console.log("Up arrow");
  if (e.key === "ArrowDown")  console.log("Down arrow");
  if (e.key === "ArrowLeft")  console.log("Left arrow");
  if (e.key === "ArrowRight") console.log("Right arrow");
  if (e.key === " ")          console.log("Spacebar");
});
```

### Keyboard Shortcuts

```js
document.addEventListener("keydown", (e) => {
  // Ctrl+S — save
  if (e.ctrlKey && e.key === "s") {
    e.preventDefault(); // stop browser save dialog
    saveDocument();
  }

  // Ctrl+Z — undo
  if (e.ctrlKey && e.key === "z") {
    e.preventDefault();
    undoAction();
  }

  // Ctrl+Shift+Z — redo
  if (e.ctrlKey && e.shiftKey && e.key === "z") {
    e.preventDefault();
    redoAction();
  }

  // Escape — close modal
  if (e.key === "Escape") closeModal();
});
```

### Real-World — WASD Game Controls

```js
const keys = {};

document.addEventListener("keydown", (e) => { keys[e.code] = true; });
document.addEventListener("keyup",   (e) => { keys[e.code] = false; });

function gameLoop() {
  if (keys["KeyW"] || keys["ArrowUp"])    movePlayer("up");
  if (keys["KeyS"] || keys["ArrowDown"])  movePlayer("down");
  if (keys["KeyA"] || keys["ArrowLeft"])  movePlayer("left");
  if (keys["KeyD"] || keys["ArrowRight"]) movePlayer("right");

  requestAnimationFrame(gameLoop);
}
gameLoop();
```

---

## 6. Form Events

### All Form Events

```js
const form  = document.querySelector("form");
const input = document.querySelector("input");

// Form-level events
form.addEventListener("submit", (e) => {
  e.preventDefault(); // stops page reload
  console.log("Form submitted");
});

form.addEventListener("reset", (e) => {
  console.log("Form reset — all fields cleared");
});

// Input-level events
input.addEventListener("focus",  (e) => console.log("Input gained focus"));
input.addEventListener("blur",   (e) => console.log("Input lost focus"));
input.addEventListener("input",  (e) => console.log("Live value:", e.target.value));
input.addEventListener("change", (e) => console.log("Committed value:", e.target.value));

// Select dropdown
document.querySelector("select").addEventListener("change", (e) => {
  console.log("Selected:", e.target.value);
});

// Checkbox
document.querySelector("input[type=checkbox]").addEventListener("change", (e) => {
  console.log("Checked:", e.target.checked);
});
```

### `input` vs `change`

| Event    | Fires                                    | Use For                      |
|----------|------------------------------------------|------------------------------|
| `input`  | Every keystroke / every value change     | Live search, char count       |
| `change` | When focus leaves and value has changed  | Final validation, select menus|

```js
const searchInput = document.querySelector("#search");

// input — fires on every character
searchInput.addEventListener("input", (e) => {
  liveSearch(e.target.value);
});

// change — fires only when user leaves the field
searchInput.addEventListener("change", (e) => {
  saveLastSearch(e.target.value);
});
```

### Reading All Form Data at Once

```js
form.addEventListener("submit", (e) => {
  e.preventDefault();

  const data = new FormData(form);

  // Read individual fields
  console.log(data.get("username"));
  console.log(data.get("email"));
  console.log(data.getAll("interests")); // for multi-select checkboxes

  // Convert to plain object
  const obj = Object.fromEntries(data);
  console.log(obj); // { username: "Alice", email: "alice@...", ... }

  // Convert to JSON
  const json = JSON.stringify(obj);
});
```

### Live Password Strength Checker

```js
const passwordInput = document.querySelector("#password");
const strengthBar   = document.querySelector("#strength-bar");

passwordInput.addEventListener("input", (e) => {
  const val = e.target.value;
  let strength = 0;

  if (val.length >= 8)        strength++;
  if (/[A-Z]/.test(val))      strength++;
  if (/[0-9]/.test(val))      strength++;
  if (/[^A-Za-z0-9]/.test(val)) strength++;

  const levels = ["", "weak", "fair", "good", "strong"];
  strengthBar.className = `strength-bar ${levels[strength]}`;
});
```

---

## 7. Window & Document Events

### Page Lifecycle Events

```js
// Fires when the HTML is parsed and DOM is ready (before images/CSS load)
document.addEventListener("DOMContentLoaded", () => {
  console.log("DOM ready — safe to query elements");
  initApp();
});

// Fires when EVERYTHING is loaded (images, CSS, fonts, iframes)
window.addEventListener("load", () => {
  console.log("Full page loaded");
});

// Fires just before the user leaves (tab close, navigation)
window.addEventListener("beforeunload", (e) => {
  e.preventDefault();
  e.returnValue = ""; // shows browser's "Leave site?" dialog
});

// Fires when the page is being unloaded
window.addEventListener("unload", () => {
  // Best effort cleanup — no async allowed here
  navigator.sendBeacon("/analytics", JSON.stringify({ event: "page_exit" }));
});
```

### `DOMContentLoaded` vs `load`

```
HTML parsed       → DOMContentLoaded fires ← use this for DOM manipulation
Images loaded     ↓
CSS loaded        ↓
Fonts loaded      ↓
Everything ready  → load fires ← use this for anything that needs full assets
```

### Scroll Events

```js
window.addEventListener("scroll", () => {
  console.log("Scroll Y:", window.scrollY);
  console.log("Scroll X:", window.scrollX);

  // Percentage scrolled
  const scrollPct = (window.scrollY / (document.body.scrollHeight - window.innerHeight)) * 100;
  document.querySelector("#progress-bar").style.width = scrollPct + "%";
});
```

### Resize Events

```js
window.addEventListener("resize", () => {
  console.log("Viewport:", window.innerWidth, "x", window.innerHeight);
});
```

### Visibility Events

```js
// Fires when the user switches tabs or minimizes window
document.addEventListener("visibilitychange", () => {
  if (document.hidden) {
    console.log("Tab is hidden — pause video/animations");
    pauseVideo();
  } else {
    console.log("Tab is visible again — resume");
    resumeVideo();
  }
});
```

### Network Status Events

```js
window.addEventListener("online",  () => console.log("You are back online"));
window.addEventListener("offline", () => console.log("No internet connection"));

// Check current status
console.log(navigator.onLine); // true / false
```

### Storage Events (Cross-Tab Communication)

```js
// Fires when localStorage changes in ANOTHER tab
window.addEventListener("storage", (e) => {
  console.log("Key changed:", e.key);
  console.log("Old value:", e.oldValue);
  console.log("New value:", e.newValue);
  console.log("Origin:", e.url);
});
```

### History Events (SPA Routing)

```js
window.addEventListener("popstate", (e) => {
  console.log("Browser back/forward clicked");
  console.log("State:", e.state);
  renderPage(location.pathname);
});

// Navigate programmatically
history.pushState({ page: "about" }, "", "/about");
history.replaceState({ page: "home" }, "", "/");
```

### Hash Change

```js
window.addEventListener("hashchange", () => {
  console.log("Hash changed to:", location.hash);
});
```

---

## 8. Clipboard Events

```js
document.addEventListener("copy", (e) => {
  e.preventDefault();
  const selected = window.getSelection().toString().toUpperCase();
  e.clipboardData.setData("text/plain", selected); // modify what gets copied
  console.log("Copied (uppercased):", selected);
});

document.addEventListener("cut", (e) => {
  console.log("User cut:", window.getSelection().toString());
});

document.addEventListener("paste", (e) => {
  e.preventDefault();
  const pasted = e.clipboardData.getData("text/plain");
  console.log("User pasted:", pasted);
  // Insert sanitized version instead
  document.execCommand("insertText", false, pasted.trim());
});
```

---

## 9. Drag & Drop Events

### On the Draggable Element

```html
<div class="card" draggable="true">Drag Me</div>
```

```js
const card = document.querySelector(".card");

card.addEventListener("dragstart", (e) => {
  e.dataTransfer.setData("text/plain", card.id);
  e.dataTransfer.effectAllowed = "move";
  card.classList.add("dragging");
});

card.addEventListener("dragend", (e) => {
  card.classList.remove("dragging");
  console.log("Drop effect:", e.dataTransfer.dropEffect);
});
```

### On the Drop Target

```js
const dropZone = document.querySelector(".drop-zone");

dropZone.addEventListener("dragover", (e) => {
  e.preventDefault(); // REQUIRED — allows drop
  e.dataTransfer.dropEffect = "move";
  dropZone.classList.add("drag-over");
});

dropZone.addEventListener("dragleave", (e) => {
  dropZone.classList.remove("drag-over");
});

dropZone.addEventListener("drop", (e) => {
  e.preventDefault();
  const id   = e.dataTransfer.getData("text/plain");
  const item = document.getElementById(id);
  dropZone.appendChild(item);
  dropZone.classList.remove("drag-over");
});
```

### File Drop (Upload by Dragging)

```js
dropZone.addEventListener("drop", (e) => {
  e.preventDefault();
  const files = [...e.dataTransfer.files];

  files.forEach((file) => {
    console.log("File:", file.name, file.size, file.type);
    const reader = new FileReader();
    reader.onload = (event) => {
      console.log("Content:", event.target.result);
    };
    reader.readAsText(file);
  });
});
```

---

## 10. Touch Events

Touch events fire on mobile/tablet devices.

```js
const area = document.querySelector(".touch-area");

area.addEventListener("touchstart", (e) => {
  const touch = e.touches[0]; // first finger
  console.log("Touch started at:", touch.clientX, touch.clientY);
  e.preventDefault(); // prevent scrolling
});

area.addEventListener("touchmove", (e) => {
  const touch = e.touches[0];
  console.log("Moving:", touch.clientX, touch.clientY);
});

area.addEventListener("touchend", (e) => {
  const touch = e.changedTouches[0]; // use changedTouches on touchend
  console.log("Touch ended at:", touch.clientX, touch.clientY);
});

area.addEventListener("touchcancel", (e) => {
  console.log("Touch cancelled (e.g., incoming call)");
});
```

### Touch Object Properties

```js
area.addEventListener("touchmove", (e) => {
  const touch = e.touches[0];

  console.log(touch.clientX,  touch.clientY);   // viewport position
  console.log(touch.pageX,    touch.pageY);      // page position
  console.log(touch.screenX,  touch.screenY);    // screen position
  console.log(touch.identifier);                 // unique ID for this finger
  console.log(e.touches.length);                 // number of fingers on screen
});
```

### Detect Swipe Direction

```js
let startX, startY;

area.addEventListener("touchstart", (e) => {
  startX = e.touches[0].clientX;
  startY = e.touches[0].clientY;
});

area.addEventListener("touchend", (e) => {
  const dx = e.changedTouches[0].clientX - startX;
  const dy = e.changedTouches[0].clientY - startY;

  if (Math.abs(dx) > Math.abs(dy)) {
    console.log(dx > 0 ? "Swipe Right" : "Swipe Left");
  } else {
    console.log(dy > 0 ? "Swipe Down" : "Swipe Up");
  }
});
```

---

## 11. Media Events

Events on `<video>` and `<audio>` elements.

```js
const video = document.querySelector("video");

video.addEventListener("play",           () => console.log("Playing"));
video.addEventListener("pause",          () => console.log("Paused"));
video.addEventListener("ended",          () => console.log("Video ended"));
video.addEventListener("timeupdate",     () => console.log("Current time:", video.currentTime));
video.addEventListener("volumechange",   () => console.log("Volume:", video.volume));
video.addEventListener("seeked",         () => console.log("Seeked to:", video.currentTime));
video.addEventListener("loadeddata",     () => console.log("Video data loaded"));
video.addEventListener("canplay",        () => console.log("Can start playing"));
video.addEventListener("canplaythrough", () => console.log("Can play without buffering"));
video.addEventListener("waiting",        () => console.log("Buffering..."));
video.addEventListener("error",          () => console.log("Media error:", video.error));

// Build a custom progress bar
video.addEventListener("timeupdate", () => {
  const pct = (video.currentTime / video.duration) * 100;
  document.querySelector(".progress-fill").style.width = pct + "%";
});
```

---

## 12. Custom Events

Create and dispatch your own events to build loosely coupled components.

### Create & Dispatch a Custom Event

```js
// Create
const myEvent = new Event("my-event");
document.dispatchEvent(myEvent);

// Listen
document.addEventListener("my-event", () => {
  console.log("Custom event fired!");
});
```

### `CustomEvent` — Attach Data with `detail`

```js
// Create with data
const loginEvent = new CustomEvent("user:login", {
  detail: {
    userId:   42,
    username: "Alice",
    role:     "admin",
  },
  bubbles:    true,  // allow it to bubble up
  cancelable: true,  // allow preventDefault
});

// Dispatch on a specific element
document.querySelector(".app").dispatchEvent(loginEvent);

// Listen anywhere in the tree
document.addEventListener("user:login", (e) => {
  console.log("User logged in:", e.detail.username);
  console.log("Role:", e.detail.role);
});
```

### Component Communication Pattern

```js
// Component A — fires an event when something happens
class SearchBox {
  constructor(el) {
    this.el = el;
    this.el.querySelector("input").addEventListener("input", (e) => {
      this.el.dispatchEvent(new CustomEvent("search:query", {
        detail: { query: e.target.value },
        bubbles: true,
      }));
    });
  }
}

// Component B — listens and reacts
class ResultsList {
  constructor(el) {
    document.addEventListener("search:query", (e) => {
      this.render(e.detail.query);
    });
  }

  render(query) {
    console.log("Rendering results for:", query);
  }
}
```

### EventEmitter Pattern

```js
class EventEmitter {
  constructor() {
    this._listeners = {};
  }

  on(event, callback) {
    if (!this._listeners[event]) this._listeners[event] = [];
    this._listeners[event].push(callback);
    return this; // allow chaining
  }

  off(event, callback) {
    if (!this._listeners[event]) return;
    this._listeners[event] = this._listeners[event].filter((cb) => cb !== callback);
    return this;
  }

  emit(event, ...args) {
    (this._listeners[event] || []).forEach((cb) => cb(...args));
    return this;
  }

  once(event, callback) {
    const wrapper = (...args) => {
      callback(...args);
      this.off(event, wrapper);
    };
    return this.on(event, wrapper);
  }
}

// Usage
const emitter = new EventEmitter();

emitter.on("data", (payload) => console.log("Received:", payload));
emitter.once("connect", () => console.log("Connected once"));

emitter.emit("data", { id: 1, name: "Alice" });
emitter.emit("connect"); // runs
emitter.emit("connect"); // does NOT run again
```

---

## 13. Event Propagation — Bubbling & Capturing

When an event fires it travels through **three phases**:

```
  Phase 1 — CAPTURE (top → down)
  ┌──────────────────────────────────┐
  │  document                        │
  │    └── html                      │
  │          └── body                │
  │                └── div#outer     │
  │                      └── button  │  ← Phase 2: TARGET
  └──────────────────────────────────┘
  Phase 3 — BUBBLE  (bottom → up)
```

### Default — Bubbling

```html
<div id="grandparent">
  <div id="parent">
    <button id="child">Click</button>
  </div>
</div>
```

```js
document.getElementById("child").addEventListener("click",       () => console.log("1. child"));
document.getElementById("parent").addEventListener("click",      () => console.log("2. parent"));
document.getElementById("grandparent").addEventListener("click", () => console.log("3. grandparent"));
document.addEventListener("click",                               () => console.log("4. document"));

// Clicking the button prints:
// 1. child
// 2. parent
// 3. grandparent
// 4. document
```

### Capture Phase

```js
document.getElementById("grandparent").addEventListener("click", () => {
  console.log("grandparent — CAPTURE");
}, { capture: true }); // or just: true as third argument

document.getElementById("child").addEventListener("click", () => {
  console.log("child — BUBBLE");
});

// Clicking button prints:
// grandparent — CAPTURE  (capture fires first, top-down)
// child — BUBBLE
```

### `stopPropagation`

Stops the event from traveling further — neither up (bubble) nor to siblings on same element.

```js
document.getElementById("child").addEventListener("click", (e) => {
  e.stopPropagation();
  console.log("child — propagation stopped here");
});

document.getElementById("parent").addEventListener("click", () => {
  console.log("parent — this will NOT fire");
});
```

### `stopImmediatePropagation`

Stops ALL further listeners — including other listeners on the SAME element.

```js
const btn = document.querySelector("button");

btn.addEventListener("click", (e) => {
  e.stopImmediatePropagation();
  console.log("First listener — blocks everything else");
});

btn.addEventListener("click", () => {
  console.log("Second listener — NEVER runs");
});
```

### Events That Don't Bubble

Some events only fire on the target and never bubble:

| Event       | Alternative That Bubbles |
|-------------|--------------------------|
| `focus`     | `focusin`                |
| `blur`      | `focusout`               |
| `mouseenter`| `mouseover`              |
| `mouseleave`| `mouseout`               |
| `load`      | —                        |
| `scroll`    | —                        |

---

## 14. Event Delegation

Attach **one listener on a parent** instead of many listeners on children. Works for dynamic content and is much more memory-efficient.

### The Problem Without Delegation

```js
// If you add new <li> elements later, they won't have listeners
document.querySelectorAll("li").forEach((li) => {
  li.addEventListener("click", handler); // 100 listeners for 100 items
});
```

### With Delegation

```js
const list = document.querySelector("ul");

list.addEventListener("click", (e) => {
  // Check what was clicked
  if (e.target.tagName === "LI") {
    console.log("Clicked item:", e.target.textContent);
    e.target.classList.toggle("done");
  }
});

// Dynamically added items AUTOMATICALLY work
const newItem = document.createElement("li");
newItem.textContent = "New Task";
list.appendChild(newItem); // click on this also works!
```

### Using `closest()` for Nested Structures

```html
<ul>
  <li class="task" data-id="1">
    <span class="task-name">Buy groceries</span>
    <button class="complete-btn">✓</button>
    <button class="delete-btn">✗</button>
  </li>
</ul>
```

```js
document.querySelector("ul").addEventListener("click", (e) => {
  const task = e.target.closest(".task");
  if (!task) return; // click was outside a task

  const id = task.dataset.id;

  if (e.target.matches(".complete-btn")) {
    task.classList.toggle("completed");
    completeTask(id);
  }

  if (e.target.matches(".delete-btn")) {
    task.remove();
    deleteTask(id);
  }
});
```

### Delegation on `document` for Global Shortcuts

```js
document.addEventListener("click", (e) => {
  // Close any open dropdown when clicking outside
  if (!e.target.closest(".dropdown")) {
    document.querySelectorAll(".dropdown.open")
      .forEach((d) => d.classList.remove("open"));
  }

  // Handle modal open from any button with data-modal attribute
  const trigger = e.target.closest("[data-modal]");
  if (trigger) openModal(trigger.dataset.modal);
});
```

---

## 15. Removing Event Listeners

### Named Function Reference (required for removal)

```js
function handleClick(e) {
  console.log("clicked");
}

btn.addEventListener("click", handleClick);

// Later — remove it
btn.removeEventListener("click", handleClick); // same function reference required
```

### ❌ Arrow Functions Can't Be Removed This Way

```js
btn.addEventListener("click", (e) => console.log("clicked"));

// This does NOT work — different function reference
btn.removeEventListener("click", (e) => console.log("clicked"));
```

### `{ once: true }` — Auto-Remove After First Fire

```js
btn.addEventListener("click", () => {
  console.log("Fires once, then automatically removed");
}, { once: true });
```

### `AbortController` — Remove Multiple Listeners at Once

The cleanest way to tear down many listeners at once.

```js
const controller = new AbortController();
const { signal } = controller;

document.addEventListener("keydown", handleKeydown, { signal });
document.addEventListener("click",   handleClick,   { signal });
document.addEventListener("scroll",  handleScroll,  { signal });

// Remove ALL at once
controller.abort();
```

### Cleanup in Components

```js
class Modal {
  constructor(el) {
    this.el = el;
    this.controller = new AbortController();

    document.addEventListener("keydown", (e) => {
      if (e.key === "Escape") this.close();
    }, { signal: this.controller.signal });
  }

  close() {
    this.el.classList.remove("open");
    this.controller.abort(); // clean up all listeners
  }
}
```

---

## 16. Event Listener Options

The third argument to `addEventListener` accepts an options object:

```js
element.addEventListener("click", handler, {
  capture: false,  // use bubbling phase (default: false)
  once:    false,  // auto-remove after first fire (default: false)
  passive: false,  // never calls preventDefault (default: false)
  signal:  null,   // AbortController signal for bulk removal
});
```

### `capture: true`

```js
// Listen during capture phase (fires before bubbling listeners)
parent.addEventListener("click", handler, { capture: true });

// Shorthand (old style)
parent.addEventListener("click", handler, true);
```

### `once: true`

```js
// Fires once, then removes itself automatically
btn.addEventListener("click", () => {
  console.log("Welcome! This only shows once.");
}, { once: true });
```

### `passive: true`

Tells the browser the handler will never call `preventDefault()` — enables scroll performance optimisations.

```js
// Scroll and touch listeners should almost always be passive
window.addEventListener("scroll", onScroll, { passive: true });
window.addEventListener("touchstart", onTouch, { passive: true });

// ⚠️ If you set passive: true and then call preventDefault(), it's ignored
```

---

## 17. `preventDefault` vs `stopPropagation`

These are two completely different things — they are NOT the same.

### `preventDefault()`

Stops the **browser's default behaviour** for the event. The event still propagates.

```js
// Prevent link navigation
document.querySelector("a").addEventListener("click", (e) => {
  e.preventDefault();
  console.log("Link blocked, event still bubbles up");
});

// Prevent form page reload
form.addEventListener("submit", (e) => {
  e.preventDefault();
  handleSubmit(); // custom submission
});

// Prevent right-click menu
document.addEventListener("contextmenu", (e) => {
  e.preventDefault();
  showCustomMenu(e.clientX, e.clientY);
});

// Prevent spacebar scrolling
document.addEventListener("keydown", (e) => {
  if (e.key === " ") e.preventDefault();
});
```

### `stopPropagation()`

Stops the **event from traveling** through the DOM. The browser's default behaviour still happens.

```js
// Inner button click won't trigger the outer div's listener
innerBtn.addEventListener("click", (e) => {
  e.stopPropagation(); // bubble stops here
  console.log("inner handled");
});

outerDiv.addEventListener("click", () => {
  console.log("outer — not called");
});
```

### Both Together

```js
link.addEventListener("click", (e) => {
  e.preventDefault();    // stop browser navigation
  e.stopPropagation();   // stop event bubbling
  handleLinkClick(link.href);
});
```

### Comparison Table

| Method                    | Stops Default | Stops Bubbling |
|---------------------------|---------------|----------------|
| `preventDefault()`        | ✅ Yes         | ❌ No           |
| `stopPropagation()`       | ❌ No          | ✅ Yes          |
| Both together             | ✅ Yes         | ✅ Yes          |

---

## 18. Debounce & Throttle

High-frequency events like `scroll`, `resize`, `mousemove`, and `input` can fire hundreds of times per second. Use debounce or throttle to control how often your handler runs.

### Debounce — Wait Until Activity Stops

Fires the function only after the user **stops** triggering the event for a given delay.

```js
function debounce(fn, delay) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

// Usage — search only fires 300ms after typing stops
const searchInput = document.querySelector("#search");

searchInput.addEventListener("input", debounce((e) => {
  console.log("Searching for:", e.target.value);
  fetchResults(e.target.value);
}, 300));

// Also useful for resize
window.addEventListener("resize", debounce(() => {
  console.log("Resize done:", window.innerWidth);
}, 200));
```

### Throttle — Limit to Once Per Interval

Fires the function **at most once** in every given time interval, no matter how often the event fires.

```js
function throttle(fn, limit) {
  let lastRun = 0;
  return function (...args) {
    const now = Date.now();
    if (now - lastRun >= limit) {
      lastRun = now;
      fn.apply(this, args);
    }
  };
}

// Usage — scroll handler fires at most once every 100ms
window.addEventListener("scroll", throttle(() => {
  console.log("Scroll Y:", window.scrollY);
}, 100));

// Mousemove for canvas drawing — once every 16ms (~60fps)
canvas.addEventListener("mousemove", throttle((e) => {
  draw(e.offsetX, e.offsetY);
}, 16));
```

### Debounce vs Throttle

| Scenario                        | Use         |
|---------------------------------|-------------|
| Search input / autocomplete     | Debounce    |
| Window resize layout recalc     | Debounce    |
| Scroll position tracking        | Throttle    |
| Mouse move / canvas drawing     | Throttle    |
| Button that fires API calls     | Debounce    |
| Rate-limit game loop input      | Throttle    |

---

## 19. Async Events & Promises

Event listeners can be `async` — useful for API calls in response to events.

### Async Event Listener

```js
btn.addEventListener("click", async (e) => {
  btn.disabled = true;
  btn.textContent = "Loading...";

  try {
    const response = await fetch("/api/data");
    const data     = await response.json();
    renderData(data);
  } catch (err) {
    console.error("Failed:", err);
    showError("Something went wrong");
  } finally {
    btn.disabled = false;
    btn.textContent = "Load Data";
  }
});
```

### Promisify an Event

Convert a one-time event into a Promise.

```js
function waitForEvent(element, eventName) {
  return new Promise((resolve) => {
    element.addEventListener(eventName, resolve, { once: true });
  });
}

// Usage
const btn = document.querySelector("button");
console.log("Waiting for click...");
const event = await waitForEvent(btn, "click");
console.log("Button clicked!", event);
```

### Wait for Image Load

```js
function loadImage(src) {
  return new Promise((resolve, reject) => {
    const img  = new Image();
    img.onload  = () => resolve(img);
    img.onerror = () => reject(new Error(`Failed to load: ${src}`));
    img.src     = src;
  });
}

const img = await loadImage("photo.jpg");
document.body.appendChild(img);
```

### Async Animation with Events

```js
function animateElement(el, className) {
  return new Promise((resolve) => {
    el.classList.add(className);
    el.addEventListener("animationend", resolve, { once: true });
  });
}

await animateElement(modal, "fade-in");
console.log("Animation done — now do something");
```

---

## 20. Common Patterns & Real-World Examples

### Pattern 1 — Modal Open / Close

```js
const modal   = document.querySelector(".modal");
const overlay = document.querySelector(".overlay");
const closeBtn = modal.querySelector(".close-btn");

function openModal() {
  modal.classList.add("open");
  overlay.classList.add("visible");
  document.body.style.overflow = "hidden"; // prevent scroll behind modal
}

function closeModal() {
  modal.classList.remove("open");
  overlay.classList.remove("visible");
  document.body.style.overflow = "";
}

overlay.addEventListener("click", closeModal);
closeBtn.addEventListener("click", closeModal);
document.addEventListener("keydown", (e) => {
  if (e.key === "Escape") closeModal();
});
```

---

### Pattern 2 — Infinite Scroll

```js
window.addEventListener("scroll", throttle(() => {
  const nearBottom =
    window.scrollY + window.innerHeight >= document.body.scrollHeight - 300;

  if (nearBottom && !isLoading) {
    isLoading = true;
    loadMoreItems().then(() => { isLoading = false; });
  }
}, 200));
```

---

### Pattern 3 — Click Outside to Close

```js
const dropdown = document.querySelector(".dropdown");

document.addEventListener("click", (e) => {
  if (!dropdown.contains(e.target)) {
    dropdown.classList.remove("open");
  }
});

dropdown.querySelector(".toggle").addEventListener("click", (e) => {
  e.stopPropagation(); // don't close immediately
  dropdown.classList.toggle("open");
});
```

---

### Pattern 4 — Double-Click to Edit

```js
const label = document.querySelector(".label");

label.addEventListener("dblclick", () => {
  const input = document.createElement("input");
  input.type  = "text";
  input.value = label.textContent;

  label.replaceWith(input);
  input.focus();
  input.select();

  function save() {
    const newLabel   = document.createElement("span");
    newLabel.className = "label";
    newLabel.textContent = input.value || label.textContent;
    input.replaceWith(newLabel);

    // Re-attach the event
    newLabel.addEventListener("dblclick", arguments.callee);
  }

  input.addEventListener("blur",    save);
  input.addEventListener("keydown", (e) => {
    if (e.key === "Enter")  save();
    if (e.key === "Escape") input.replaceWith(label); // cancel
  });
});
```

---

### Pattern 5 — Keyboard Navigation for Custom Dropdown

```js
const dropdown = document.querySelector(".custom-select");
const options  = [...dropdown.querySelectorAll(".option")];
let   focused  = -1;

dropdown.addEventListener("keydown", (e) => {
  if (e.key === "ArrowDown") {
    e.preventDefault();
    focused = Math.min(focused + 1, options.length - 1);
    options[focused].focus();
  }
  if (e.key === "ArrowUp") {
    e.preventDefault();
    focused = Math.max(focused - 1, 0);
    options[focused].focus();
  }
  if (e.key === "Enter" && focused >= 0) {
    options[focused].click();
  }
  if (e.key === "Escape") {
    dropdown.classList.remove("open");
    dropdown.focus();
  }
});
```

---

## 21. Quick Reference Cheat Sheet

### Attaching & Removing

```js
el.addEventListener("click", handler)
el.removeEventListener("click", handler)
el.addEventListener("click", handler, { once: true, passive: true, capture: true })

const ctrl = new AbortController();
el.addEventListener("click", handler, { signal: ctrl.signal });
ctrl.abort(); // removes all listeners using this signal
```

### Event Object

```js
e.type            // "click", "keydown"
e.target          // element that triggered the event
e.currentTarget   // element the listener is attached to
e.isTrusted       // true = real user action
e.timeStamp       // ms since page load
e.preventDefault()
e.stopPropagation()
e.stopImmediatePropagation()
```

### Mouse

```js
e.clientX / e.clientY   // viewport position
e.pageX   / e.pageY     // page position
e.offsetX / e.offsetY   // element-relative
e.button  / e.buttons   // which button
e.shiftKey / e.ctrlKey / e.altKey / e.metaKey
```

### Keyboard

```js
e.key     // "Enter", "a", "ArrowUp"
e.code    // "KeyA", "Enter", "Space"
e.repeat  // true if key held
e.shiftKey / e.ctrlKey / e.altKey / e.metaKey
```

### Custom Events

```js
new CustomEvent("name", { detail: {}, bubbles: true, cancelable: true })
element.dispatchEvent(event)
```

### Common Event Names

| Category  | Events                                                      |
|-----------|-------------------------------------------------------------|
| Mouse     | `click` `dblclick` `mousedown` `mouseup` `mousemove` `mouseenter` `mouseleave` `contextmenu` |
| Keyboard  | `keydown` `keyup`                                           |
| Form      | `submit` `reset` `input` `change` `focus` `blur`           |
| Window    | `load` `DOMContentLoaded` `resize` `scroll` `beforeunload` `visibilitychange` |
| Touch     | `touchstart` `touchmove` `touchend` `touchcancel`          |
| Drag      | `dragstart` `drag` `dragover` `drop` `dragend`             |
| Media     | `play` `pause` `ended` `timeupdate` `volumechange`         |
| Network   | `online` `offline`                                          |
| Clipboard | `copy` `cut` `paste`                                        |

---

## Key Takeaways

- Always use **`addEventListener`** — never inline handlers or DOM properties.
- Every event listener receives an **Event object** full of context about what happened.
- Understand **`target` vs `currentTarget`** — they differ in delegated listeners.
- Events **bubble** by default — use this to your advantage with **event delegation**.
- Use **`preventDefault()`** to block browser defaults; use **`stopPropagation()`** to stop DOM traversal — they are NOT the same.
- Use **`AbortController`** to cleanly tear down multiple listeners at once.
- Use **`{ once: true }`** for one-time listeners instead of manually removing them.
- Use **debounce** for input/resize; use **throttle** for scroll/mousemove.
- Use **`{ passive: true }`** on scroll and touch listeners for better performance.
- **Custom events** are the cleanest way to communicate between decoupled components.
- Event listeners can be **`async`** — await API calls directly inside handlers.