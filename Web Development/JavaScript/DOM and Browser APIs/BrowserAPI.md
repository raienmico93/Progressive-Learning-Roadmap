# Browser APIs — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Browser APIs are the built-in interfaces and objects provided by web browsers that allow JavaScript to interact with the browser environment, the operating system, and web platform features beyond the core language.

**Technical Definition:** Per MDN, the Web APIs are typically used with JavaScript, although this doesn't always have to be the case. Web APIs are the interfaces provided by the browser that extend the capabilities of JavaScript. They include the Window interface (global object), Navigator interface (browser and device information), Location interface (current URL), History interface (session history), Screen interface (display information), Console interface (debugging), and many more. These APIs are defined by standards bodies like WHATWG and W3C, and are implemented by browser vendors.

**Beginner-Friendly Explanation:** Browser APIs are the "toolbox" the browser gives you to do things beyond basic JavaScript. Want to know the screen size? Use `window.innerWidth`. Want to redirect the user? Use `window.location`. Want to ask for their location? Use `navigator.geolocation`. Want to log something for debugging? Use `console.log()`. These APIs are what make JavaScript truly powerful in the browser.

### Key Characteristics

- **Globally available:** Most browser APIs are available on the `window` object (the global object in browsers).
- **Standardised:** Defined by WHATWG and W3C, implemented consistently across modern browsers.
- **Asynchronous for sensitive operations:** APIs like geolocation, clipboard, and media devices use Promises or callbacks.
- **Permission-gated:** Many APIs require user permission before they can be used (geolocation, camera, microphone, notifications).
- **Environment-specific:** Some APIs are only available in secure contexts (HTTPS) or specific browsers.
- **Extensive:** The browser API surface is vast, covering everything from network requests to device sensors.

### Prerequisites

- Basic understanding of JavaScript (variables, functions, objects, Promises).
- Familiarity with the DOM and DOM events.
- Awareness of the same-origin policy and secure contexts.

### Related Programming Areas

- **Web application development:** Building interactive web apps.
- **Progressive Web Apps (PWAs):** Using Service Workers, Cache API, and storage APIs.
- **Geolocation services:** Location-based applications.
- **Media processing:** Camera, microphone, and audio/video APIs.
- **Clipboard operations:** Copy/paste functionality.
- **Debugging and monitoring:** Console API for development.

---

## `window`

### Definitions

**Core Definition:** The `window` object is the global object in browser JavaScript, representing the browser window or tab. It contains all global variables, functions, and browser APIs.

**Technical Definition:** Per MDN, the `Window` interface represents a window containing a DOM document; the `document` property points to the DOM document loaded in that window. A global variable `window`, representing the window in which the script is running, is exposed to JavaScript code. In tabbed browsers, each tab is represented by its own `Window` object.

### Key Properties

| Property | Description |
|----------|-------------|
| `window.innerWidth` | The width of the window's content area in pixels |
| `window.innerHeight` | The height of the window's content area in pixels |
| `window.location` | The `Location` object for the current document |
| `window.history` | The `History` object for the current session |
| `window.navigator` | The `Navigator` object with browser/device information |
| `window.screen` | The `Screen` object with display information |
| `window.scrollX` | The horizontal scroll position |
| `window.scrollY` | The vertical scroll position |
| `window.document` | The DOM document |

### `window.innerWidth` / `window.innerHeight`

```js
console.log(window.innerWidth);  // e.g., 1024
console.log(window.innerHeight); // e.g., 768
```

### `window.location`

```js
console.log(window.location.href);     // "https://example.com/page"
console.log(window.location.hostname); // "example.com"
console.log(window.location.pathname); // "/page"
console.log(window.location.protocol); // "https:"
```

### `window.history`

```js
window.history.back();    // Go back
window.history.forward(); // Go forward
window.history.go(-2);    // Go back two steps
```

### `window.navigator`

```js
console.log(window.navigator.userAgent); // Browser user agent string
console.log(window.navigator.language);  // e.g., "en-US"
console.log(window.navigator.onLine);    // true/false
```

### `window.screen`

```js
console.log(window.screen.width);      // Screen width
console.log(window.screen.height);     // Screen height
console.log(window.screen.availWidth); // Available width
```

### `window.scrollX` / `window.scrollY`

```js
console.log(window.scrollX); // Horizontal scroll position
console.log(window.scrollY); // Vertical scroll position
```

### `window.scrollTo()`

```js
// Scroll to a specific position:
window.scrollTo(0, 500);

// Smooth scroll:
window.scrollTo({ top: 500, behavior: "smooth" });
```

### `window.open()`

```js
// Open a new window:
const newWindow = window.open("https://example.com", "_blank", "width=600,height=400");

// Close it:
// newWindow.close();
```

### `window.close()`

```js
// Close the current window (only works for windows opened by script):
window.close();
```

### `window.alert()`, `window.confirm()`, `window.prompt()`

```js
// Alert:
window.alert("Hello!");

// Confirm (returns true/false):
const confirmed = window.confirm("Are you sure?");
console.log(confirmed);

// Prompt (returns string or null):
const name = window.prompt("What's your name?");
console.log(name);
```

### Annotated Code Example

```js
// Step 1: Get window dimensions.
console.log(`Viewport: ${window.innerWidth}x${window.innerHeight}`);

// Step 2: Check scroll position.
console.log(`Scroll: ${window.scrollX}, ${window.scrollY}`);

// Step 3: Navigate programmatically.
// window.location.href = "https://example.com";

// Step 4: Open a popup.
// window.open("https://example.com", "_blank");
```

**Expected Output:**
```
Viewport: 1024x768
Scroll: 0, 0
```

### Real-World Cases

- **Responsive design:** Using `innerWidth`/`innerHeight` for layout decisions.
- **Navigation:** Using `location` for redirects and URL parsing.
- **Popups:** Using `window.open` for OAuth flows and popup windows.
- **Scroll restoration:** Using `scrollY` and `scrollTo` to restore scroll positions.

---

## `navigator`

### Definitions

**Core Definition:** The `navigator` object provides information about the browser and the operating system, and exposes APIs for device capabilities.

**Technical Definition:** Per MDN, the `Navigator` interface represents the state and the identity of the user agent. It allows scripts to query it and to register themselves to carry on some activities. The `navigator` object is accessed via `window.navigator` (or just `navigator`).

### Key Properties

| Property | Description |
|----------|-------------|
| `navigator.userAgent` | The user agent string |
| `navigator.language` | The preferred language (e.g., "en-US") |
| `navigator.languages` | An array of preferred languages |
| `navigator.clipboard` | The Clipboard API |
| `navigator.geolocation` | The Geolocation API |
| `navigator.mediaDevices` | Media devices (camera, microphone) |
| `navigator.serviceWorker` | The Service Worker API |
| `navigator.storage` | The Storage Manager API |
| `navigator.onLine` | Whether the browser is online |

### `navigator.userAgent`

```js
console.log(navigator.userAgent);
// "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 ..."
```

### `navigator.language` / `navigator.languages`

```js
console.log(navigator.language);  // "en-US"
console.log(navigator.languages); // ["en-US", "en", "fr"]
```

### `navigator.clipboard`

```js
// Write to clipboard:
await navigator.clipboard.writeText("Hello, clipboard!");

// Read from clipboard:
const text = await navigator.clipboard.readText();
console.log(text);
```

### `navigator.geolocation`

```js
navigator.geolocation.getCurrentPosition(
  (position) => {
    console.log(position.coords.latitude);
    console.log(position.coords.longitude);
  },
  (error) => {
    console.error("Geolocation error:", error.message);
  }
);
```

### `navigator.mediaDevices`

```js
const stream = await navigator.mediaDevices.getUserMedia({ video: true, audio: true });
// Use stream with a <video> element
```

### `navigator.serviceWorker`

```js
const registration = await navigator.serviceWorker.register("/sw.js");
console.log("Service Worker registered:", registration.scope);
```

### `navigator.storage`

```js
const estimate = await navigator.storage.estimate();
console.log(`Using ${estimate.usage} of ${estimate.quota} bytes`);
```

### `navigator.onLine`

```js
console.log(navigator.onLine); // true or false

window.addEventListener("online", () => console.log("Online"));
window.addEventListener("offline", () => console.log("Offline"));
```

### Annotated Code Example

```js
// Step 1: Check online status.
if (navigator.onLine) {
  console.log("You are online");
} else {
  console.log("You are offline");
}

// Step 2: Get preferred language.
console.log("Language:", navigator.language);

// Step 3: Copy to clipboard.
await navigator.clipboard.writeText("Copied text");
console.log("Text copied");
```

**Expected Output:**
```
You are online
Language: en-US
Text copied
```

### Real-World Cases

- **Localisation:** Using `navigator.language` to set the app language.
- **Clipboard:** Implementing copy-to-clipboard buttons.
- **Geolocation:** Location-based services.
- **Offline detection:** Showing offline banners.
- **Service Workers:** Enabling PWA features.

---

## `location`

### Definitions

**Core Definition:** The `location` object represents the current URL of the document and provides methods for navigating to new URLs.

**Technical Definition:** Per MDN, the `Location` interface represents the location (URL) of the object it is linked to. Changes done on it are reflected on the object it relates to. Both the `Document` and `Window` interfaces have a linked `Location`.

### Key Properties

| Property | Description |
|----------|-------------|
| `href` | The full URL |
| `protocol` | The protocol (e.g., "https:") |
| `host` | The hostname and port |
| `hostname` | The hostname |
| `port` | The port |
| `pathname` | The path |
| `search` | The query string |
| `hash` | The fragment identifier |
| `origin` | The origin (protocol + host + port) |

### Key Methods

| Method | Description |
|--------|-------------|
| `assign(url)` | Navigates to the URL |
| `replace(url)` | Navigates to the URL without adding a history entry |
| `reload()` | Reloads the current page |
| `toString()` | Returns the full URL |

### Annotated Code Example

```js
// Step 1: Read URL components.
console.log(location.href);     // "https://example.com:8080/page?q=hello#section"
console.log(location.protocol); // "https:"
console.log(location.hostname); // "example.com"
console.log(location.port);     // "8080"
console.log(location.pathname); // "/page"
console.log(location.search);   // "?q=hello"
console.log(location.hash);     // "#section"
console.log(location.origin);   // "https://example.com:8080"

// Step 2: Navigate.
// location.assign("https://example.com/other");
// location.replace("https://example.com/other"); // No history entry
// location.reload();
```

### Real-World Cases

- **Redirects:** `location.href = "/login"`.
- **Query string parsing:** Using `location.search` with `URLSearchParams`.
- **Hash navigation:** Using `location.hash` for in-page anchors.
- **SPA routing:** Using the History API with `location`.

---

## `history`

### Definitions

**Core Definition:** The `history` object provides access to the browser's session history (the pages visited in the current tab) and allows navigation forward and backward.

**Technical Definition:** Per MDN, the `History` interface allows manipulation of the browser session history, that is the pages visited in the tab or frame that the current page is loaded in.

### Key Properties

| Property | Description |
|----------|-------------|
| `length` | The number of entries in the session history |
| `state` | The current state object |
| `scrollRestoration` | Whether scroll restoration is automatic or manual |

### Key Methods

| Method | Description |
|--------|-------------|
| `back()` | Goes back one page |
| `forward()` | Goes forward one page |
| `go(delta)` | Goes to a specific position in history |
| `pushState(state, title, url)` | Adds a new history entry |
| `replaceState(state, title, url)` | Replaces the current history entry |

### Annotated Code Example

```js
// Step 1: Navigate back and forward.
history.back();    // Equivalent to clicking the back button
history.forward(); // Equivalent to clicking the forward button
history.go(-2);    // Go back two pages

// Step 2: Add a new history entry (SPA routing).
history.pushState({ page: "settings" }, "", "/settings");
console.log(location.pathname); // "/settings"

// Step 3: Listen for popstate events.
window.addEventListener("popstate", (event) => {
  console.log("State:", event.state);
});
```

**Expected Output:**
```
/settings
State: { page: "settings" } (when navigating back)
```

### Real-World Cases

- **Single-page applications:** Using `pushState` for client-side routing.
- **Back/forward navigation:** Handling browser navigation buttons.
- **State management:** Storing state in history entries.

---

## `screen`

### Definitions

**Core Definition:** The `screen` object provides information about the user's display screen.

**Technical Definition:** Per MDN, the `Screen` interface represents a screen, usually the one on which the current window is being rendered, and is obtained using `window.screen`.

### Key Properties

| Property | Description |
|----------|-------------|
| `width` | The screen width in pixels |
| `height` | The screen height in pixels |
| `availWidth` | The available screen width (excluding taskbars) |
| `availHeight` | The available screen height |
| `colorDepth` | The colour depth in bits |
| `pixelDepth` | The pixel depth in bits |
| `orientation` | The screen orientation |

### Annotated Code Example

```js
console.log(`Screen: ${screen.width}x${screen.height}`);
console.log(`Available: ${screen.availWidth}x${screen.availHeight}`);
console.log(`Colour depth: ${screen.colorDepth}-bit`);
console.log(`Orientation: ${screen.orientation.type}`);
```

**Expected Output:**
```
Screen: 1920x1080
Available: 1920x1040
Colour depth: 24-bit
Orientation: landscape-primary
```

### Real-World Cases

- **Responsive design:** Adapting layouts to screen size.
- **Fullscreen detection:** Using `availWidth`/`availHeight`.
- **Orientation handling:** Responding to device rotation.

---

## `console`

### Definitions

**Core Definition:** The `console` object provides methods for outputting information to the browser's developer console, used for debugging, logging, and performance measurement.

**Technical Definition:** Per MDN, the `console` object provides access to the browser's debugging console (e.g., the Web Console in Firefox). The specifics of how it works vary from browser to browser, but there is a de facto set of features that are typically provided.

### Logging Methods

| Method | Description |
|--------|-------------|
| `console.log()` | Outputs a message to the console |
| `console.error()` | Outputs an error message |
| `console.warn()` | Outputs a warning message |
| `console.info()` | Outputs an informational message |
| `console.debug()` | Outputs a debug message |
| `console.table()` | Displays tabular data as a table |
| `console.group()` | Creates an inline group |
| `console.groupEnd()` | Exits the current inline group |
| `console.time(label)` | Starts a timer |
| `console.timeEnd(label)` | Stops a timer and logs the elapsed time |
| `console.trace()` | Outputs a stack trace |
| `console.assert()` | Logs a message if the assertion is false |
| `console.count()` | Logs the number of times it has been called |
| `console.dir()` | Displays an interactive list of object properties |
| `console.clear()` | Clears the console |

### Annotated Code Examples

#### Example 1: Basic Logging

```js
console.log("Simple message");
console.error("Error message");
console.warn("Warning message");
console.info("Info message");
console.debug("Debug message");
```

**Expected Output (in console):**
```
Simple message
Error message
Warning message
Info message
Debug message
```

#### Example 2: Tabular Data

```js
const users = [
  { name: "Alice", age: 30 },
  { name: "Bob", age: 25 },
];
console.table(users);
```

**Expected Output (in console):** A formatted table with columns `name` and `age`.

#### Example 3: Grouping

```js
console.group("User Details");
console.log("Name: Alice");
console.log("Age: 30");
console.groupEnd();
```

**Expected Output (in console):** A collapsible group labelled "User Details".

#### Example 4: Timing

```js
console.time("loop");
for (let i = 0; i < 1000000; i++) {}
console.timeEnd("loop"); // "loop: 5.23ms"
```

**Expected Output (in console):**
```
loop: 5.23ms
```

#### Example 5: Assertions

```js
console.assert(1 === 1, "This won't show");
console.assert(1 === 2, "This will show"); // Logs an error
```

**Expected Output (in console):**
```
Assertion failed: This will show
```

#### Example 6: Counting

```js
console.count("click");
console.count("click");
console.count("click");
```

**Expected Output (in console):**
```
click: 1
click: 2
click: 3
```

#### Example 7: Stack Trace

```js
function foo() {
  console.trace("Trace from foo");
}
foo();
```

**Expected Output (in console):** A stack trace showing the call path.

#### Example 8: Directory

```js
const obj = { a: 1, b: { c: 2 } };
console.dir(obj);
```

**Expected Output (in console):** An interactive list of the object's properties.

### Real-World Cases

- **Debugging:** Using `console.log` and `console.table` to inspect data.
- **Performance:** Using `console.time`/`console.timeEnd` to measure code execution.
- **Error tracking:** Using `console.error` to log errors.
- **Code validation:** Using `console.assert` to check assumptions.

---

## References

- MDN Web Docs – Web APIs — https://developer.mozilla.org/en-US/docs/Web/API
- MDN Web Docs – Window — https://developer.mozilla.org/en-US/docs/Web/API/Window
- MDN Web Docs – Navigator — https://developer.mozilla.org/en-US/docs/Web/API/Navigator
- MDN Web Docs – Location — https://developer.mozilla.org/en-US/docs/Web/API/Location
- MDN Web Docs – History — https://developer.mozilla.org/en-US/docs/Web/API/History
- MDN Web Docs – Screen — https://developer.mozilla.org/en-US/docs/Web/API/Screen
- MDN Web Docs – Console — https://developer.mozilla.org/en-US/docs/Web/API/Console
- MDN Web Docs – Clipboard API — https://developer.mozilla.org/en-US/docs/Web/API/Clipboard_API
- MDN Web Docs – Geolocation API — https://developer.mozilla.org/en-US/docs/Web/API/Geolocation_API
- MDN Web Docs – MediaDevices — https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices
- MDN Web Docs – Service Worker API — https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API
- MDN Web Docs – Storage Manager — https://developer.mozilla.org/en-US/docs/Web/API/StorageManager
- MDN Web Docs – Window: open() — https://developer.mozilla.org/en-US/docs/Web/API/Window/open
- MDN Web Docs – Window: scrollTo() — https://developer.mozilla.org/en-US/docs/Web/API/Window/scrollTo
- MDN Web Docs – Window: alert() — https://developer.mozilla.org/en-US/docs/Web/API/Window/alert
- MDN Web Docs – Window: confirm() — https://developer.mozilla.org/en-US/docs/Web/API/Window/confirm
- MDN Web Docs – Window: prompt() — https://developer.mozilla.org/en-US/docs/Web/API/Window/prompt
- WHATWG – HTML Living Standard — https://html.spec.whatwg.org/
- W3C – Web APIs — https://www.w3.org/TR/
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf