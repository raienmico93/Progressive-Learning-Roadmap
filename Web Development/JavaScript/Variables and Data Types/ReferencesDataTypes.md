# JavaScript Reference Data Types — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Reference data types (also called reference types or objects) are JavaScript values that are stored and accessed by reference — a variable holding a reference type does not contain the value itself, but rather a pointer (reference) to a location in memory where the object is stored.

**Technical Definition:** In ECMAScript, all values that are not primitives are objects. Objects are mutable collections of properties, where a property is a key-value pair. Reference types include `Object`, `Array`, `Function`, `Date`, `RegExp`, `Map`, `Set`, `WeakMap`, `WeakSet`, `ArrayBuffer`, and Typed Arrays. When a reference-type value is assigned to a variable or passed to a function, the reference (memory address) is copied, not the underlying object. This means two variables can point to the same object, and mutations through one variable are visible through the other.

**Beginner-Friendly Explanation:** Imagine a reference type is like a remote control to a television. The variable is the remote, not the TV itself. If you give someone a copy of the remote (assign the variable), you both control the same TV — if one person changes the channel, the other sees the change. Primitives are more like a written note: if you copy the note, each person has their own independent copy.

### Key Characteristics

- **Stored by reference:** Variables hold a reference (pointer) to the object's location in memory, not the object itself.
- **Mutable:** Object contents can be changed without reassigning the variable.
- **Shared state:** Multiple variables can reference the same object; mutations are visible through all references.
- **`typeof` returns `"object"` for most:** Functions return `"function"`; all other reference types return `"object"`.
- **Not directly comparable:** `===` compares references, not contents (except for the same object reference).

### Prerequisites

- Understanding of primitive data types (String, Number, Boolean, `undefined`, `null`, Symbol, BigInt).
- Familiarity with variables and the `typeof` operator.
- Basic knowledge of memory concepts (stack vs. heap).

### Related Programming Areas

- **Immutability patterns:** Functional programming and state management (React, Redux) rely on understanding references.
- **Deep vs. shallow copying:** `structuredClone()`, spread syntax, `Object.assign()`.
- **Garbage collection:** WeakMap and WeakSet enable memory-efficient object associations.
- **Binary data processing:** ArrayBuffer and Typed Arrays for audio, video, and network protocols.

### Core Concepts / Features

---

## Object

### Definitions

**Core Definition:** The `Object` type is the fundamental reference type in JavaScript, used to store collections of key-value pairs and more complex entities.

**Technical Definition:** An object is an unordered collection of properties, where each property is a key-value pair. Keys are strings or symbols; values can be any JavaScript value. Objects inherit properties and methods from `Object.prototype` (unless created with `Object.create(null)`).

**Beginner-Friendly Explanation:** An object is like a labelled filing cabinet: each drawer has a name (key) and contains something (value). You can add, remove, or change drawers at any time.

### Purposes

- To group related data and behaviour into a single entity.
- To model real-world entities (users, products, orders).
- To serve as the foundation for all other reference types (arrays, functions, dates, etc.).
- To enable dynamic property addition and deletion.

### Syntax Rules and Structure

**Object Literal:**
```js
const obj = { key: "value", "quoted-key": 42, [computedKey]: true };
```

**Constructor:**
```js
const obj = new Object();
```

**Component Breakdown:**
- `key`: An identifier, string, or computed expression.
- `value`: Any JavaScript value.
- `computedKey`: An expression evaluated to produce the key.

**Syntax Rules:**
- Property keys are strings or symbols; numeric keys are coerced to strings.
- Properties can be accessed via dot notation (`obj.key`) or bracket notation (`obj["key"]`).
- `delete` removes a property (returns `true` if successful).
- `in` checks for property existence (including inherited properties).

**Constraints and Limitations:**
- Prototype pollution: Modifying `Object.prototype` affects all objects.
- `Object.keys()` only returns own enumerable string-keyed properties (not symbols, not inherited).
- Objects are not JSON-serializable if they contain functions or circular references.

### Annotated Code Examples

#### Example 1: Creating and Accessing Object Properties

```js
// Step 1: Create an object with various property types.
const user = {
  name: "Alice",
  age: 30,
  "favourite-color": "blue", // quoted key
  [Symbol("id")]: 123,       // symbol key (hidden from Object.keys)
};

// Step 2: Access properties.
console.log(user.name);            // "Alice" (dot notation)
console.log(user["favourite-color"]); // "blue" (bracket notation)

// Step 3: Add a new property.
user.email = "alice@example.com";
console.log(user.email); // "alice@example.com"

// Step 4: Delete a property.
delete user.age;
console.log(user.age); // undefined

// Step 5: Enumerate own string keys.
console.log(Object.keys(user)); // ["name", "favourite-color", "email"]
```

**Expected Output:**
```
Alice
blue
alice@example.com
undefined
[ 'name', 'favourite-color', 'email' ]
```

**Why This Output:** Dot and bracket notation access property values. `delete` removes the `age` property. `Object.keys()` excludes symbol keys and non-enumerable properties.

### Real-World Cases

- **API responses:** JSON objects represent server data.
- **Configuration:** `{ theme: "dark", language: "en" }`.
- **Namespaces:** Grouping related functions and constants.

---

## Array

### Definitions

**Core Definition:** An Array is a special type of object used to store an ordered collection of values, accessible by numeric index.

**Technical Definition:** Arrays are exotic objects with a special `length` property that automatically updates. They are zero-indexed, and elements are stored as properties with integer-like string keys. Arrays inherit from `Array.prototype`, which provides methods like `push`, `pop`, `map`, and `filter`.

**Beginner-Friendly Explanation:** An array is like a numbered list. You can access the first item with index 0, the second with index 1, and so on. Arrays can hold any type of value, including other arrays and objects.

### Purposes

- To store ordered collections of data (lists, sequences).
- To iterate over elements with loops and array methods.
- To represent stacks (LIFO) and queues (FIFO).
- To serve as the primary data structure for tabular data.

### Syntax Rules and Structure

**Array Literal:**
```js
const arr = [1, 2, 3];
const mixed = [1, "two", true, null, { key: "value" }];
```

**Constructor:**
```js
const arr = new Array(1, 2, 3); // [1, 2, 3]
const empty = new Array(5);     // [ <5 empty items> ]
```

**Component Breakdown:**
- Elements are separated by commas.
- Trailing commas are allowed and ignored.

**Syntax Rules:**
- `length` is one more than the highest index.
- Sparse arrays have "holes" (missing indices).
- Array indices are non-negative integers (or their string forms).
- `Array.isArray()` is the reliable way to check for arrays.

**Constraints and Limitations:**
- Arrays are objects; `typeof []` returns `"object"`.
- Sorting is lexicographic by default (`[10, 2].sort()` yields `[10, 2]`).
- `delete` on an array element creates a hole (does not change `length`).

### Annotated Code Examples

#### Example 1: Array Operations and Methods

```js
// Step 1: Create an array.
const fruits = ["apple", "banana"];

// Step 2: Add elements.
fruits.push("cherry");     // add to end
fruits.unshift("apricot"); // add to start
console.log(fruits);       // ["apricot", "apple", "banana", "cherry"]

// Step 3: Remove elements.
const last = fruits.pop(); // remove from end
const first = fruits.shift(); // remove from start
console.log(last, first);  // "cherry" "apricot"
console.log(fruits);       // ["apple", "banana"]

// Step 4: Transform elements.
const upper = fruits.map((f) => f.toUpperCase());
console.log(upper);        // ["APPLE", "BANANA"]
```

**Expected Output:**
```
[ 'apricot', 'apple', 'banana', 'cherry' ]
cherry apricot
[ 'apple', 'banana' ]
[ 'APPLE', 'BANANA' ]
```

**Why This Output:** `push` and `unshift` add to end/start; `pop` and `shift` remove from end/start. `map` returns a new array with transformed elements.

### Real-World Cases

- **Lists in UI:** Rendering todo items, messages, or products.
- **Data processing:** Filtering, mapping, and reducing datasets.
- **Matrix operations:** Nested arrays for grids and tables.

---

## Function

### Definitions

**Core Definition:** A Function is a callable object that encapsulates a block of code, optionally accepting parameters and returning a value.

**Technical Definition:** Every function in JavaScript is a `Function` object. Functions are first-class objects: they can be assigned to variables, passed as arguments, returned from other functions, and have properties and methods (e.g., `call()`, `apply()`, `bind()`).

**Beginner-Friendly Explanation:** A function is like a recipe: it has a name, ingredients (parameters), and steps (the body). You can "call" the recipe to execute the steps and get a result.

### Purposes

- To encapsulate reusable logic.
- To create abstractions and modular code.
- To serve as callbacks, event handlers, and higher-order functions.
- To enable closures and functional programming patterns.

### Syntax Rules and Structure

**Function Declaration:**
```js
function name(param1, param2) {
  return param1 + param2;
}
```

**Function Expression:**
```js
const name = function(param1, param2) {
  return param1 + param2;
};
```

**Arrow Function:**
```js
const name = (param1, param2) => param1 + param2;
```

**Component Breakdown:**
- `name`: The function's identifier (optional for expressions).
- `param1, param2`: Parameters (optional).
- `return`: Returns a value (defaults to `undefined`).

**Syntax Rules:**
- Function declarations are hoisted (callable before their definition).
- Function expressions and arrow functions are not hoisted.
- Arrow functions do not have their own `this`, `arguments`, or `super`.
- Functions have a `length` property (number of declared parameters) and a `name` property.

**Constraints and Limitations:**
- `Function` constructor (`new Function()`) has security and performance issues similar to `eval()` and should be avoided.
- Functions are objects; they can hold properties (e.g., `fn.cache = {}`).

### Annotated Code Examples

#### Example 1: Function Types and Hoisting

```js
// Step 1: Function declaration (hoisted).
console.log(add(2, 3)); // 5

function add(a, b) {
  return a + b;
}

// Step 2: Function expression (not hoisted).
try {
  console.log(subtract(5, 2));
} catch (err) {
  console.log(err.name); // "ReferenceError"
}

const subtract = function (a, b) {
  return a - b;
};

// Step 3: Arrow function.
const multiply = (a, b) => a * b;
console.log(multiply(4, 5)); // 20
```

**Expected Output:**
```
5
ReferenceError
20
```

**Why This Output:** Function declarations are fully hoisted. Function expressions are not; `subtract` is in the TDZ until its declaration. Arrow functions are concise and do not bind their own `this`.

### Real-World Cases

- **Event handling:** `button.addEventListener("click", () => { ... })`.
- **Array methods:** `arr.map(fn)`, `arr.filter(fn)`.
- **Factory functions:** `function createUser(name) { return { name }; }`.

---

## Date

### Definitions

**Core Definition:** The `Date` object represents a single moment in time in a platform-independent format, stored as milliseconds since the Unix epoch (1 January 1970 UTC).

**Technical Definition:** `Date` instances encapsulate an integer representing milliseconds since the epoch. The `Date` constructor can be called with various arguments (timestamp, date string, or individual components). Methods provide access to date components (year, month, day, etc.) in both local time and UTC.

**Beginner-Friendly Explanation:** A Date object is a timestamp — it holds a specific date and time. You can create one for "now", or for any date in the past or future, and extract parts like the year or hour.

### Purposes

- To record timestamps for events.
- To calculate differences between dates (e.g., age, duration).
- To format dates for display.
- To schedule operations based on time.

### Syntax Rules and Structure

**Creating Dates:**
```js
new Date()                  // current date/time
new Date(timestamp)         // milliseconds since epoch
new Date(dateString)        // ISO 8601 string
new Date(year, month, day)  // individual components (month is 0-indexed)
```

**Component Breakdown:**
- `timestamp`: Integer milliseconds since 1970-01-01T00:00:00Z.
- `dateString`: A string parsable by `Date.parse()`.
- `year, month, day`: Integers; month is 0-based (0 = January).

**Common Methods:**
- `getFullYear()`, `getMonth()`, `getDate()`, `getDay()` — local components.
- `getUTCFullYear()`, `getUTCMonth()`, etc. — UTC components.
- `getTime()` — milliseconds since epoch.
- `toISOString()` — ISO 8601 string.

**Constraints and Limitations:**
- Month is 0-indexed (a common source of bugs).
- Date parsing of non-ISO strings is implementation-dependent.
- `Date` objects are mutable (setter methods modify the object in place).
- Timezone handling can be tricky; use UTC methods for consistency.

### Annotated Code Examples

#### Example 1: Working with Dates

```js
// Step 1: Create a date for a specific moment.
const date = new Date(2025, 0, 15, 10, 30); // 15 Jan 2025, 10:30 local
console.log(date.getFullYear()); // 2025
console.log(date.getMonth());    // 0 (January)
console.log(date.getDate());     // 15

// Step 2: Get the timestamp.
console.log(date.getTime()); // e.g., 1736937000000

// Step 3: Format as ISO string.
console.log(date.toISOString()); // "2025-01-15T02:30:00.000Z" (UTC)

// Step 4: Calculate date difference.
const now = new Date();
const diffMs = now - date;
const diffDays = Math.floor(diffMs / (1000 * 60 * 60 * 24));
console.log(diffDays); // number of days between
```

**Expected Output:**
```
2025
0
15
1736937000000
2025-01-15T02:30:00.000Z
<number>
```

**Why This Output:** The constructor interprets local time. `getMonth()` is 0-based. `getTime()` returns milliseconds. `toISOString()` converts to UTC.

### Real-World Cases

- **Logging timestamps:** Recording when events occur.
- **Countdowns:** Calculating time remaining.
- **Scheduling:** Determining whether a deadline has passed.

---

## RegExp

### Definitions

**Core Definition:** A RegExp (regular expression) object represents a pattern used to match character combinations in strings.

**Technical Definition:** RegExp objects can be created via literal notation (`/pattern/flags`) or the `RegExp` constructor (`new RegExp("pattern", "flags")`). They support methods like `test()` and `exec()`, and can be used with string methods like `match()`, `replace()`, and `split()`.

**Beginner-Friendly Explanation:** A regular expression is a search pattern — like a "find and replace" with superpowers. You can use it to check if a string contains an email address, extract phone numbers, or replace all occurrences of a word.

### Purposes

- To validate input (emails, phone numbers, passwords).
- To search, extract, and replace substrings.
- To parse structured text (logs, CSV, markup).
- To split strings on complex delimiters.

### Syntax Rules and Structure

**Literal Notation:**
```js
const re = /pattern/flags;
```

**Constructor:**
```js
const re = new RegExp("pattern", "flags");
```

**Common Flags:**
- `g` — global match.
- `i` — case-insensitive.
- `m` — multiline.
- `u` — Unicode mode.

**Component Breakdown:**
- `pattern`: The regular expression pattern (character classes, quantifiers, groups).
- `flags`: Modifiers controlling matching behaviour.

**Common Methods:**
- `re.test(str)` — returns `true` if a match is found.
- `re.exec(str)` — returns an array with match information.
- `str.match(re)`, `str.replace(re, replacement)`, `str.split(re)`.

**Constraints and Limitations:**
- Patterns are compiled once for literals; constructor patterns compile at runtime.
- Global regexes with `exec()` are stateful (`lastIndex` advances).
- Catastrophic backtracking can cause performance issues with complex patterns.

### Annotated Code Examples

#### Example 1: Email Validation

```js
// Step 1: Define the pattern.
const emailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

// Step 2: Test strings.
console.log(emailPattern.test("alice@example.com")); // true
console.log(emailPattern.test("not-an-email"));      // false

// Step 3: Extract with a global pattern.
const text = "Contact: alice@example.com, bob@test.org";
const found = text.match(/[^\s@]+@[^\s@]+\.[^\s@]+/g);
console.log(found); // ["alice@example.com", "bob@test.org"]
```

**Expected Output:**
```
true
false
[ 'alice@example.com', 'bob@test.org' ]
```

**Why This Output:** The pattern checks for the basic shape of an email. The `g` flag finds all matches in the string.

### Real-World Cases

- **Form validation:** Checking email, phone, and password formats.
- **Text processing:** Highlighting search terms, parsing logs.
- **Data extraction:** Scraping structured data from HTML.

---

## Map

### Definitions

**Core Definition:** A `Map` is a collection of key-value pairs where keys can be of any type (primitive or object), and insertion order is preserved.

**Technical Definition:** `Map` objects hold key-value pairs and remember the original insertion order of keys. Any value (object or primitive) may be used as either a key or a value. A key may only occur once in a Map's collection; it is unique.

**Beginner-Friendly Explanation:** A Map is like a dictionary where the "word" (key) can be anything — a string, a number, even an object — and each key maps to a value. Unlike plain objects, Map keys can be non-strings.

### Purposes

- To store key-value data where keys are not strings.
- To maintain insertion order for iteration.
- To efficiently check key existence and delete entries.
- To avoid prototype pollution issues of plain objects.

### Syntax Rules and Structure

**Creating a Map:**
```js
const map = new Map();
map.set("key", "value");
map.set(42, "answer");
map.set({ id: 1 }, "object key");
```

**Constructor with Iterable:**
```js
const map = new Map([["a", 1], ["b", 2]]);
```

**Common Methods:**
- `set(key, value)` — adds or updates.
- `get(key)` — retrieves value.
- `has(key)` — checks existence.
- `delete(key)` — removes entry.
- `size` — number of entries.
- `keys()`, `values()`, `entries()` — iterators.

**Constraints and Limitations:**
- Map keys are compared by reference for objects (two different objects are different keys).
- Map is not directly JSON-serializable.
- Not as lightweight as plain objects for simple string-keyed data.

### Annotated Code Examples

#### Example 1: Map with Object Keys

```js
// Step 1: Create a Map.
const userRoles = new Map();

// Step 2: Use objects as keys.
const alice = { name: "Alice" };
const bob = { name: "Bob" };

userRoles.set(alice, "admin");
userRoles.set(bob, "editor");

// Step 3: Retrieve values.
console.log(userRoles.get(alice)); // "admin"
console.log(userRoles.get(bob));   // "editor"

// Step 4: Check size and existence.
console.log(userRoles.size);       // 2
console.log(userRoles.has(alice)); // true

// Step 5: Iterate.
for (const [user, role] of userRoles) {
  console.log(`${user.name}: ${role}`);
}
```

**Expected Output:**
```
admin
editor
2
true
Alice: admin
Bob: editor
```

**Why This Output:** Objects are compared by reference, so `alice` and `bob` are distinct keys. Iteration preserves insertion order.

### Real-World Cases

- **Caching:** Mapping request URLs to responses.
- **Metadata:** Associating extra data with objects without modifying them.
- **Frequency counting:** Counting occurrences of values.

---

## Set

### Definitions

**Core Definition:** A `Set` is a collection of unique values of any type, preserving insertion order.

**Technical Definition:** `Set` objects are collections of values. A value may only occur once; it is unique in the Set's collection. Values can be primitives or object references.

**Beginner-Friendly Explanation:** A Set is like a bag where duplicates are not allowed. If you try to add the same value twice, the second addition is ignored.

### Purposes

- To store unique values (deduplication).
- To efficiently check membership.
- To perform set operations (union, intersection, difference).
- To maintain insertion order of unique items.

### Syntax Rules and Structure

**Creating a Set:**
```js
const set = new Set([1, 2, 2, 3]); // {1, 2, 3}
```

**Common Methods:**
- `add(value)` — adds a value.
- `has(value)` — checks existence.
- `delete(value)` — removes a value.
- `size` — number of unique values.
- `values()`, `keys()`, `entries()` — iterators.

**Constraints and Limitations:**
- Objects are compared by reference; two different objects with the same content are distinct.
- `NaN` is considered equal to itself in Sets (unlike `===`).
- Sets are not indexable; use iteration or convert to an array.

### Annotated Code Examples

#### Example 1: Deduplication and Set Operations

```js
// Step 1: Deduplicate an array.
const numbers = [1, 2, 2, 3, 3, 4];
const unique = new Set(numbers);
console.log([...unique]); // [1, 2, 3, 4]

// Step 2: Set operations.
const setA = new Set([1, 2, 3]);
const setB = new Set([3, 4, 5]);

const union = new Set([...setA, ...setB]);
console.log([...union]); // [1, 2, 3, 4, 5]

const intersection = new Set([...setA].filter((x) => setB.has(x)));
console.log([...intersection]); // [3]
```

**Expected Output:**
```
[ 1, 2, 3, 4 ]
[ 1, 2, 3, 4, 5 ]
[ 3 ]
```

**Why This Output:** The Set constructor removes duplicates. Spreading into an array and filtering enables set operations.

### Real-World Cases

- **Unique visitors:** Tracking distinct user IDs.
- **Tag systems:** Storing unique tags.
- **Removing duplicates:** Cleaning datasets.

---

## WeakMap

### Definitions

**Core Definition:** A `WeakMap` is a collection of key-value pairs where keys must be objects (or non-registered symbols), and the keys are held weakly — they do not prevent garbage collection.

**Technical Definition:** A `WeakMap` does not create strong references to its keys. If the only remaining reference to a key object is from the WeakMap, the object can be garbage-collected, and the corresponding entry is removed.

**Beginner-Friendly Explanation:** A WeakMap is like a special sticky note attached to an object. If the object is thrown away, the sticky note goes with it automatically — you don't need to clean it up manually.

### Purposes

- To associate metadata with objects without preventing their garbage collection.
- To implement caches that do not leak memory.
- To store private data for objects in a way that is automatically cleaned up.
- To avoid memory leaks in long-lived collections.

### Syntax Rules and Structure

```js
const wm = new WeakMap();
const key = {};
wm.set(key, "value");
```

**Constraints and Limitations:**
- Keys must be objects or non-registered symbols (primitives are not allowed).
- Not enumerable: no `size` property, no iteration methods.
- No `clear()` method.
- Cannot inspect keys; only `get`, `set`, `has`, `delete`.

### Annotated Code Examples

#### Example 1: Metadata Without Memory Leaks

```js
// Step 1: Create a WeakMap.
const metadata = new WeakMap();

// Step 2: Attach data to an object.
let user = { name: "Alice" };
metadata.set(user, { lastLogin: new Date() });

// Step 3: Retrieve the data.
console.log(metadata.get(user).lastLogin instanceof Date); // true

// Step 4: Remove the only reference to the key object.
user = null;
// The entry in the WeakMap is now eligible for garbage collection.
console.log(metadata.has(user)); // false (user is null, not a key)
```

**Expected Output:**
```
true
false
```

**Why This Output:** The WeakMap does not prevent garbage collection. Once `user` is dereferenced, the entry becomes eligible for removal.

### Real-World Cases

- **DOM node metadata:** Associating data with elements without risking leaks.
- **Private class fields (polyfill pattern):** Storing internal state in a WeakMap.
- **Caching computed values per object.**

---

## WeakSet

### Definitions

**Core Definition:** A `WeakSet` is a collection of unique objects (and non-registered symbols) held weakly, meaning they do not prevent garbage collection.

**Technical Definition:** A WeakSet is a collection of garbage-collectable values. A value may only occur once. References to objects in a WeakSet are held weakly; if no other references exist, the object can be garbage-collected.

**Beginner-Friendly Explanation:** A WeakSet is like a guest list where you mark objects as "present." If an object is no longer needed elsewhere, it disappears from the list automatically.

### Purposes

- To mark objects without preventing their cleanup.
- To track object membership without memory leaks.
- To implement "visited" sets in graph traversal.

### Syntax Rules and Structure

```js
const ws = new WeakSet();
const obj = {};
ws.add(obj);
ws.has(obj); // true
```

**Constraints and Limitations:**
- Only objects and non-registered symbols are allowed (no primitives).
- Not enumerable; no `size`, no iteration.
- No `clear()` method.

### Annotated Code Examples

#### Example 1: Marking Visited Objects

```js
// Step 1: Create a WeakSet.
const visited = new WeakSet();

// Step 2: Mark objects.
const node1 = { id: 1 };
const node2 = { id: 2 };
visited.add(node1);

// Step 3: Check membership.
console.log(visited.has(node1)); // true
console.log(visited.has(node2)); // false

// Step 4: Object can be garbage-collected when no longer referenced.
```

**Expected Output:**
```
true
false
```

**Why This Output:** The WeakSet tracks membership without keeping objects alive.

### Real-World Cases

- **Graph algorithms:** Marking visited nodes.
- **Object tagging:** Flagging objects for special treatment without mutation.

---

## ArrayBuffer

### Definitions

**Core Definition:** An `ArrayBuffer` is a low-level object representing a fixed-length block of raw binary data.

**Technical Definition:** `ArrayBuffer` represents a generic, fixed-length raw binary data buffer. It is a byte array, but its contents cannot be read or written directly; you must use a Typed Array or `DataView` to access the memory.

**Beginner-Friendly Explanation:** An ArrayBuffer is like a blank USB drive: it holds raw bytes, but you need a special tool (a Typed Array or DataView) to read and write the data in a meaningful format.

### Purposes

- To allocate raw memory for binary data.
- To serve as the backing store for Typed Arrays.
- To transfer data between Web Workers without copying.
- To process audio, video, and network data.

### Syntax Rules and Structure

```js
const buffer = new ArrayBuffer(16); // 16 bytes, initialised to 0
```

**Key Properties:**
- `byteLength` — size in bytes.
- `slice(start, end)` — copies a portion.
- `transfer()` — transfers ownership (detaches the original).

**Constraints and Limitations:**
- Cannot read/write contents directly.
- Fixed size (though `resize()` exists for resizable buffers).
- `SharedArrayBuffer` allows sharing across threads (requires cross-origin isolation).

### Annotated Code Examples

#### Example 1: Creating and Slicing an ArrayBuffer

```js
// Step 1: Allocate a buffer.
const buffer = new ArrayBuffer(8);
console.log(buffer.byteLength); // 8

// Step 2: Create a view to write data.
const view = new Uint8Array(buffer);
view[0] = 255;
console.log(view[0]); // 255

// Step 3: Slice the buffer.
const sliced = buffer.slice(0, 4);
console.log(sliced.byteLength); // 4
```

**Expected Output:**
```
8
255
4
```

**Why This Output:** `byteLength` reports the size. A `Uint8Array` view allows byte-level access. `slice` copies a portion.

### Real-World Cases

- **File reading:** `FileReader.readAsArrayBuffer()`.
- **WebSocket binary data:** Receiving/sending binary frames.
- **WebAssembly memory:** Wasm modules use ArrayBuffer for linear memory.

---

## Typed Arrays

### Definitions

**Core Definition:** Typed Arrays are array-like objects that provide a mechanism for reading and writing raw binary data in memory buffers with a specific numeric type.

**Technical Definition:** Typed Arrays are views over an `ArrayBuffer`. Each type (e.g., `Int8Array`, `Uint8Array`, `Float64Array`) interprets the underlying bytes as a specific numeric format. Typed Arrays share many array methods but are not true Arrays (`Array.isArray()` returns `false`).

**Beginner-Friendly Explanation:** A Typed Array is like a grid laid over the raw bytes of an ArrayBuffer, where each cell has a fixed type (8-bit integer, 32-bit float, etc.). It lets you work with binary data as numbers.

### Purposes

- To read and write binary data with numeric types.
- To process audio, video, and image pixel data.
- To interface with WebGL, WebAssembly, and network protocols.
- To achieve performance benefits for numeric computation.

### Syntax Rules and Structure

**Common Typed Array Constructors:**
- `Int8Array`, `Uint8Array`, `Uint8ClampedArray`
- `Int16Array`, `Uint16Array`
- `Int32Array`, `Uint32Array`
- `Float32Array`, `Float64Array`
- `BigInt64Array`, `BigUint64Array`

**Creating Views:**
```js
const buffer = new ArrayBuffer(16);
const int32View = new Int32Array(buffer); // 4 elements (16 / 4)
const uint8View = new Uint8Array(buffer); // 16 elements
```

**Constraints and Limitations:**
- Not true arrays; `Array.isArray()` returns `false`.
- Fixed length; no `push`/`pop`.
- Element type is fixed (no mixing types).
- Out-of-range values are clamped or wrapped depending on the type.

### Annotated Code Examples

#### Example 1: Multiple Views on the Same Buffer

```js
// Step 1: Create a buffer.
const buffer = new ArrayBuffer(8);

// Step 2: Create two views.
const bytes = new Uint8Array(buffer);
const floats = new Float32Array(buffer);

// Step 3: Write through one view.
bytes[0] = 0x41; // 'A' in ASCII
console.log(bytes[0]); // 65

// Step 4: Read through another view (interpretation changes).
console.log(floats[0]); // a small float value (depends on bytes)
```

**Expected Output:**
```
65
<small float number>
```

**Why This Output:** Both views share the same underlying buffer. Writing bytes changes the raw data; reading as a float reinterprets those bytes.

### Real-World Cases

- **Image processing:** Reading pixel data from `<canvas>` via `getImageData()`.
- **Audio processing:** Manipulating PCM samples in `AudioBuffer`.
- **WebGL:** Uploading vertex data to GPU buffers.

---

## Reference vs. Value

### Definitions

**Core Definition:** Reference vs. value describes how JavaScript handles assignment and comparison: primitives are copied by value (the actual data is copied), while objects are copied by reference (the memory address is copied).

**Technical Definition:** When a primitive value is assigned to a variable, the value itself is stored in that variable. When a reference value is assigned, the variable stores a reference (pointer) to the object's location in memory. Copying a reference-type variable copies the reference, not the object. Consequently, two variables can refer to the same object, and mutations through one are visible through the other.

**Beginner-Friendly Explanation:** Primitives are like photocopies: each copy is independent. Objects are like shared Google Docs: if you share the link, everyone edits the same document.

### Purposes

- To understand why object mutations affect all references.
- To avoid unintended side effects when passing objects to functions.
- To decide when to copy (shallow or deep) versus share references.
- To reason about equality comparisons.

### Syntax Rules and Structure

**Value Assignment (Primitives):**
```js
let a = 10;
let b = a; // b gets a copy of 10
b = 20;
console.log(a); // 10 (unchanged)
```

**Reference Assignment (Objects):**
```js
let obj1 = { value: 10 };
let obj2 = obj1; // obj2 gets a copy of the reference
obj2.value = 20;
console.log(obj1.value); // 20 (changed!)
```

**Equality:**
- Primitives: `===` compares values.
- Objects: `===` compares references (same memory address).

### Annotated Code Examples

#### Example 1: Value vs. Reference in Functions

```js
// Step 1: Primitive parameter (passed by value).
function changePrimitive(x) {
  x = 99;
}
let num = 10;
changePrimitive(num);
console.log(num); // 10 (unchanged)

// Step 2: Object parameter (reference passed).
function changeObject(obj) {
  obj.value = 99;
}
let obj = { value: 10 };
changeObject(obj);
console.log(obj.value); // 99 (changed!)
```

**Expected Output:**
```
10
99
```

**Why This Output:** Primitives are copied; the function receives an independent copy. Objects pass the reference; the function mutates the same object.

#### Example 2: Equality Comparison

```js
// Step 1: Primitive equality.
console.log(10 === 10); // true

// Step 2: Object equality (reference comparison).
const a = { x: 1 };
const b = { x: 1 };
const c = a;

console.log(a === b); // false (different objects)
console.log(a === c); // true (same reference)
```

**Expected Output:**
```
true
false
true
```

**Why This Output:** `===` on objects checks whether both operands reference the exact same object.

### Real-World Cases

- **React state:** Immutable updates create new objects to trigger re-renders.
- **Redux:** Reducers return new state objects; mutating state directly is forbidden.
- **Function design:** Documenting whether a function mutates its arguments.

---

## Memory Model

### Definitions

**Core Definition:** The memory model describes how JavaScript engines allocate, access, and reclaim memory for values, distinguishing between the stack (for execution contexts and primitive-like data) and the heap (for objects).

**Technical Definition:** JavaScript engines (e.g., V8, SpiderMonkey, JavaScriptCore) use a heap for dynamic memory allocation. Primitive values are typically stored directly in the variable's slot (often on the stack for local variables), while objects are allocated on the heap, and variables hold references (pointers) to those heap locations. The exact strategy is implementation-dependent; modern engines may optimise storage based on scope, lifetime, and type.

**Beginner-Friendly Explanation:** Think of the stack as a small, fast notepad where you write simple values, and the heap as a large warehouse where objects live. A variable holding an object has the warehouse address written on the notepad, not the object itself.

### Purposes

- To explain why objects are shared and primitives are copied.
- To understand garbage collection and memory leaks.
- To reason about performance implications of object allocation.
- To make informed decisions about data structures.

### Syntax Rules and Structure

The memory model is not a syntax construct; it is an implementation detail. The observable behaviour is what matters:

- Primitives: independent copies.
- Objects: shared references.
- Garbage collection reclaims unreachable objects automatically.

**Constraints and Limitations:**
- The stack/heap distinction is an implementation detail; JavaScript does not expose memory addresses.
- Some engines allocate objects on the stack if they do not escape (escape analysis).
- `WeakMap` and `WeakSet` enable weak references that do not prevent garbage collection.

### Annotated Code Examples

#### Example 1: Demonstrating Reference Sharing

```js
// Step 1: Two variables referencing the same object.
const shared = { count: 0 };
const alias = shared;

alias.count += 1;
console.log(shared.count); // 1

// Step 2: Two variables with separate objects.
const separate = { count: 0 };
const copy = { ...separate }; // shallow copy (new object)
copy.count += 1;
console.log(separate.count); // 0 (unchanged)
```

**Expected Output:**
```
1
0
```

**Why This Output:** `shared` and `alias` point to the same heap object. The spread operator creates a new object with copied properties, so `copy` is independent.

### Real-World Cases

- **Performance:** Avoiding unnecessary deep copies of large objects.
- **Memory leaks:** Unintentional references keeping objects alive.
- **Immutable data patterns:** Ensuring predictable state changes.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – ECMAScript Data Types and Values — https://tc39.es/ecma262/
- MDN Web Docs – JavaScript data types and data structures — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Data_structures
- MDN Web Docs – Object — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object
- MDN Web Docs – Array — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array
- MDN Web Docs – Function — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function
- MDN Web Docs – Date — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date
- MDN Web Docs – RegExp — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp
- MDN Web Docs – Map — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map
- MDN Web Docs – Set — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set
- MDN Web Docs – WeakMap — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap
- MDN Web Docs – WeakSet — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakSet
- MDN Web Docs – ArrayBuffer — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer
- MDN Web Docs – JavaScript typed arrays — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Typed_arrays
- MDN Web Docs – Keyed collections — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Keyed_collections
- MDN Web Docs – Memory management — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Memory_management
- MDN Web Docs – Copying objects — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_objects#copying_objects
- Stack Overflow – Are primitive/reference types stored on the heap or stack? — https://stackoverflow.com/questions/45260121