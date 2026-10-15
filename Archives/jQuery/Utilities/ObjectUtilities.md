# Object Manipulation and Cloning: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Object manipulation and cloning refers to the techniques for copying, merging, and transforming JavaScript objects, with a critical distinction between shallow copies (top-level properties only) and deep copies (recursive copying of nested structures).

**Technical Definition**
Shallow copying creates a new object with the same top-level properties, but nested objects and arrays are shared by reference. Deep copying recursively clones all nested structures, creating fully independent copies. jQuery's `$.extend()` provides both shallow merging (default) and deep merging (`$.extend(true, ...)`), while modern JavaScript offers `Object spread ({...obj})` for shallow copies and `structuredClone()` for deep copies .

**Beginner-Friendly Explanation**
When you copy an object, you have two options. A shallow copy is like photocopying the cover of a book — the cover is new, but the pages inside are the same pages. Changing a page in one book changes it in the other. A deep copy is like rewriting the entire book — everything is independent. jQuery's `$.extend()` does shallow copies by default, but you can make it deep with `$.extend(true, {}, obj)`. Modern JavaScript has `structuredClone()` for true deep copies.

### Key Characteristics

- **Reference Sharing**: Shallow copies share nested object references, leading to unintended mutations .
- **Recursive Copying**: Deep copies recursively clone all nested objects and arrays .
- **Mutation Warning**: `$.extend(target, source)` mutates the target object and returns it; use an empty object `{}` as target to avoid mutation .
- **Modern Alternatives**: `Object spread ({...obj})` for shallow copies; `structuredClone()` for deep copies of complex data .
- **Type Limitations**: `$.extend(true)` does not preserve `Date`, `RegExp`, or custom class instances; `structuredClone()` handles these but throws on functions .

### Prerequisites

- Basic JavaScript knowledge (objects, arrays, references).
- Understanding of the difference between primitive values and object references.
- Familiarity with jQuery's utility methods and modern ES6+ syntax.

### Related Programming Areas

- **Configuration Management**: Merging default settings with user options .
- **State Management**: Creating independent copies of state objects before mutation.
- **Data Processing**: Transforming API response data without modifying the original.
- **Plugin Development**: Using `$.extend()` to add methods to the jQuery namespace .

### Core Concepts / Features

1. `$.extend()` for Merging Configuration Objects
2. Shallow Copying vs. Deep Copying
3. Implementing Deep Copies with `$.extend(true, target, source)`
4. Modern Native Alternatives

---

## Core Concept 1: `$.extend()` for Merging Configuration Objects

### Definitions

**Core Definition**
`$.extend()` is jQuery's utility for merging the properties of two or more objects into a target object, commonly used to combine default configuration with user-supplied options.

**Technical Definition**
`jQuery.extend(target, object1 [, objectN])` merges properties from `object1` through `objectN` into `target`, modifying and returning `target`. If only one argument is supplied, the target is assumed to be the jQuery namespace itself, allowing plugin authors to add methods to jQuery . The merge is shallow by default: if a property in the target is an object or array, it is completely overwritten by a same-named property from a subsequent object, not merged .

**Beginner-Friendly Explanation**
`$.extend()` combines objects. You give it a target object and one or more source objects, and it copies all properties from the sources into the target. It's commonly used in plugins to merge user options with default settings.

### Purposes

- To merge default configuration with user-provided options.
- To combine multiple objects into one.
- To add methods to the jQuery namespace (plugin development) .
- To create a new object without modifying the originals (by passing `{}` as target) .

### Syntax Rules and Structure

**Complete General Syntaxes**

```javascript
// Basic merge (mutates target)
$.extend(target, object1 [, objectN])

// Preserve originals (pass empty object as target)
var result = $.extend({}, object1, object2)

// Deep merge (recursive)
$.extend(true, target, object1 [, objectN])

// Add to jQuery namespace
$.extend(object)
```

**Component Breakdown**

- `target` : The object that receives new properties (modified and returned) .
- `object1, objectN` : Source objects containing properties to merge .
- `deep` (optional): Boolean; if `true`, performs recursive merge .
- `{}` : Empty object used as target to avoid mutating originals .

**Syntax Rules**

1. The target object is always modified and returned .
2. `null` and `undefined` arguments are ignored .
3. Shallow merge: nested objects/arrays are overwritten, not merged .
4. Deep merge: nested objects are recursively merged .
5. Passing `false` for the deep argument is not supported .

**Constraints and Limitations**

- **Prototype Pollution**: Versions prior to 3.4 had a security issue with `__proto__` .
- **Type Loss**: Built-in types like `Date` and `RegExp` become plain objects in deep extends .
- **No Re-construction**: Objects created with `new MyCustomObject(args)` are not reconstructed .
- **Cyclic Structures**: Deep-extending cyclical data structures throws an error .

### Multiple Annotated Complete Code Examples

**Example 1: Merging Defaults and Options (Plugin Pattern)**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>$.extend() — Merging Defaults</title>
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
  <div id="log"></div>

  <script>
    $(function () {
      // Step 1: Define default settings
      var defaults = {
        validate: false,
        limit: 5,
        name: "foo"
      };

      // Step 2: Define user options
      var options = {
        validate: true,
        name: "bar"
      };

      // Step 3: Merge defaults and options without modifying defaults
      // Empty object {} as target preserves both originals
      var settings = $.extend({}, defaults, options);

      // Step 4: Log the merged result
      $("#log").append(JSON.stringify(settings));    // Result: {"validate":true,"limit":5,"name":"bar"}

      // Step 5: Verify defaults are unchanged
      console.log("Defaults name:", defaults.name);  // "foo"
      console.log("Settings name:", settings.name);  // "bar"
    });
  </script>
</body>
</html>
```

**Expected Output**
- The log displays 
```
{
    "validate":true,
    "limit":5,
    "name":"bar"
}
```
- Console shows:
```
"Defaults name: foo"
"Settings name: bar"
```

**Why This Output Occurs**
The empty object `{}` is used as the target, so neither `defaults` nor `options` is mutated. Properties from `options` override matching properties in `defaults`, while non-matching properties are preserved .

---

**Example 2: Shallow Merge Overwrites Nested Objects**

```javascript
var object1 = {
  apple: 0,
  banana: { weight: 52, price: 100 },
  cherry: 97
};

var object2 = {
  banana: { price: 200 },
  durian: 100
};

// Shallow merge: object2's banana completely replaces object1's
$.extend(object1, object2);

console.log(JSON.stringify(object1));
// {"apple":0,"banana":{"price":200},"cherry":97,"durian":100}
```

**Expected Output (Console)**
```
{"apple":0,"banana":{"price":200},"cherry":97,"durian":100}
```

**Why This Output Occurs**
The shallow merge replaces `object1.banana` entirely with `object2.banana`. The `weight` property is lost because it was not merged recursively .

### Real-World Cases

- **jQuery Plugins**: Merging user options with plugin defaults .
- **Configuration Systems**: Combining base config with environment-specific overrides.
- **API Wrappers**: Merging request options with default headers and settings.

### References

- jQuery API — jQuery.extend() – https://api.jquery.com/jquery.Extend/
- Learn jQuery — Utility Methods – https://learn.jquery.com/using-jquery-core/utility-methods/

---

## Core Concept 2: Shallow Copying vs. Deep Copying

### Definitions

**Core Definition**
A shallow copy creates a new object with copies of the top-level properties, but nested objects and arrays are shared by reference. A deep copy recursively clones all nested structures, creating a fully independent object .

**Technical Definition**
In JavaScript, assigning an object creates a reference, not a copy. A shallow copy (via spread `{...obj}`, `Object.assign()`, or `$.extend({}, obj)`) copies property values; for non-primitive values, the reference is copied, meaning the nested object is shared between original and copy . A deep copy recursively traverses the object, creating new instances of every nested object and array, ensuring complete independence .

**Beginner-Friendly Explanation**
A shallow copy is like copying a house's address — you get a new piece of paper with the same address, but both papers point to the same house. If you renovate the house, both papers see the change. A deep copy is like building a completely new house that looks identical — changes to one house don't affect the other.

### Purposes

- **Shallow Copy**: To create a new reference to an object while sharing nested data (when nested data should not be duplicated) .
- **Deep Copy**: To create a fully independent clone for safe mutation without affecting the original .
- **State Management**: Deep copies prevent accidental shared-state mutations in complex applications.

### Syntax Rules and Structure

**Comparison Table: Shallow vs. Deep Copy**

| Aspect | Shallow Copy | Deep Copy |
|---|---|---|
| **Top-level properties** | Copied | Copied |
| **Nested objects/arrays** | Shared reference | Recursively cloned |
| **Mutation of nested values** | Affects both | Affects only the copy |
| **Performance** | Fast | Slower (recursive) |
| **Methods** | `{...obj}`, `Object.assign()`, `$.extend({}, obj)` | `structuredClone()`, `$.extend(true, {}, obj)`, `_.cloneDeep()` |

**Syntax Rules**

1. Shallow copy only duplicates the first level of properties .
2. Deep copy recursively duplicates all levels .
3. `Object spread ({...obj})` performs a shallow copy .
4. `structuredClone()` performs a deep copy .

**Constraints and Limitations**

- **Shallow Copy**: Nested mutations affect both original and copy .
- **Deep Copy**: `$.extend(true)` does not preserve `Date`, `RegExp`, or class instances .
- **JSON Method**: `JSON.parse(JSON.stringify())` loses `Date`, `undefined`, `NaN`, `Map`, `Set`, `RegExp`, and functions .

### Multiple Annotated Complete Code Examples

**Example 1: Demonstrating Shallow vs. Deep Copy**

```javascript
// Original object with nested structure
var original = {
  name: "Alice",
  address: {
    city: "Springfield",
    zip: "62704"
  }
};

// SHALLOW COPY using spread operator
var shallowCopy = { ...original };

// DEEP COPY using structuredClone()
var deepCopy = structuredClone(original);

// Step 1: Modify nested property in shallow copy
shallowCopy.address.city = "Shelbyville";

// Step 2: Check original
console.log("Original city:", original.address.city);  // "Shelbyville" — MUTATED!
console.log("Shallow copy city:", shallowCopy.address.city);  // "Shelbyville"

// Step 3: Reset and test deep copy
original.address.city = "Springfield";
deepCopy.address.city = "Ogdenville";

console.log("\nAfter deep copy mutation:");
console.log("Original city:", original.address.city);  // "Springfield" — UNCHANGED!
console.log("Deep copy city:", deepCopy.address.city);  // "Ogdenville"
```

**Expected Output (Console)**
```
Original city: Shelbyville
Shallow copy city: Shelbyville

After deep copy mutation:
Original city: Springfield
Deep copy city: Ogdenville
```

**Why This Output Occurs**
The shallow copy shares the `address` object reference. Mutating `shallowCopy.address.city` also mutates `original.address.city`. The deep copy has its own independent `address` object, so mutations do not propagate .

---

**Example 2: jQuery Shallow Copy Shares Arrays**

```javascript
var obj1 = { items: [1, 2, 3] };
var obj2 = { items: [4, 5, 6] };

// Shallow merge — obj2's array replaces obj1's entirely
var shallowMerged = $.extend({}, obj1, obj2);
console.log("Shallow merged items:", shallowMerged.items);  // [4, 5, 6]

// The array is shared by reference
shallowMerged.items.push(7);
console.log("Original obj2 items:", obj2.items);  // [4, 5, 6, 7] — MUTATED!
```

**Expected Output (Console)**
```
Shallow merged items: [ 4, 5, 6 ]
Original obj2 items: [ 4, 5, 6, 7 ]
```

**Why This Output Occurs**
The shallow merge copies the array reference. Mutating the merged array also mutates the original `obj2.items` array .

### Real-World Cases

- **State Management**: Using deep copies to prevent shared-state bugs.
- **Configuration Merging**: Using shallow copies when nested overrides are intentional.
- **Data Caching**: Deep copying cached data before returning to prevent mutation.

### References

- web.dev — Deep copies with structuredClone – https://web.dev/articles/structured-clone
- GitHub — Shallow vs Deep Copy – https://github.com/greatfrontend/top-javascript-interview-questions

---

## Core Concept 3: Implementing Deep Copies with `$.extend(true, target, source)`

### Definitions

**Core Definition**
`$.extend(true, target, source)` performs a recursive (deep) merge, creating new instances of nested objects and arrays rather than sharing references.

**Technical Definition**
When the first argument to `$.extend()` is `true`, the merge becomes recursive. Nested objects are merged property by property, and nested arrays are extended (concatenated) rather than replaced. However, `$.extend(true)` does not reconstruct built-in types like `Date` or `RegExp`; they become plain objects. Objects created with custom constructors lose their prototype. Cyclical data structures throw an error .

**Beginner-Friendly Explanation**
`$.extend(true, {}, obj)` makes a deep copy of `obj`. It recursively copies everything, so changes to the copy don't affect the original. But it doesn't preserve special object types like `Date` or custom classes.

### Purposes

- To create a fully independent copy of a nested object.
- To merge configuration objects recursively.
- To clone data before mutation in jQuery-based applications .

### Syntax Rules and Structure

**Complete General Syntax**

```javascript
// Deep merge (mutates target)
$.extend(true, target, object1 [, objectN])

// Deep copy (preserves original)
var copy = $.extend(true, {}, originalObject)
```

**Component Breakdown**

- `true` : Enables recursive (deep) merge .
- `target` : The object to receive properties (modified and returned).
- `{}` : Empty object as target for a pure copy.

**Syntax Rules**

1. `true` must be the first argument; passing `false` is not supported .
2. Arrays are merged by index, not replaced .
3. `Date` and `RegExp` become plain objects .
4. Custom class instances lose their prototype .
5. Cyclical references cause an error .

**Constraints and Limitations**

- **No Type Preservation**: `Date`, `RegExp`, and class instances are converted to plain objects .
- **No Functions**: Functions are not deeply merged .
- **Performance**: Deep merging is slower than shallow merging.
- **Not a True Clone**: For perfect deep cloning, use `structuredClone()` or Lodash `_.cloneDeep()` .

### Multiple Annotated Complete Code Examples

**Example 1: Deep Copy with `$.extend(true, {}, obj)`**

```javascript
var original = {
  name: "Alice",
  settings: {
    theme: "dark",
    notifications: {
      email: true,
      sms: false
    }
  }
};

// Step 1: Deep copy using $.extend(true, {}, obj)
var deepCopy = $.extend(true, {}, original);

// Step 2: Modify nested property in the copy
deepCopy.settings.notifications.email = false;

// Step 3: Verify original is unchanged
console.log("Original email:", original.settings.notifications.email);  // true
console.log("Copy email:", deepCopy.settings.notifications.email);      // false
```

**Expected Output (Console)**
```
Original email: true
Copy email: false
```

**Why This Output Occurs**
The deep merge recursively cloned the nested `notifications` object. Mutating the copy's nested property does not affect the original .

---

**Example 2: Deep Copy Limitation with Date**

```javascript
var original = {
  name: "Event",
  date: new Date("2024-01-15")
};

// Deep copy with $.extend(true)
var copy = $.extend(true, {}, original);

// Step 1: Check the type of the copied date
console.log("Original date type:", original.date.constructor.name);  // "Date"
console.log("Copied date type:", copy.date.constructor.name);        // "Object"
console.log("Copied date value:", copy.date);  // {} or string representation

// The Date object was converted to a plain object
```

**Expected Output (Console)**
```
Original date type: Date
Copied date type: Object
Copied date value: {}
```

**Why This Output Occurs**
`$.extend(true)` does not reconstruct built-in `Date` objects. The `Date` becomes a plain object, losing its methods and type information .

### Real-World Cases

- **Plugin Defaults**: Deep merging nested configuration objects in jQuery plugins.
- **Data Cloning**: Creating independent copies of nested data structures.
- **Legacy Codebases**: Using `$.extend(true)` when `structuredClone()` is not available.

### References

- jQuery API — jQuery.extend() – https://api.jquery.com/jquery.Extend/
- Stack Overflow — jQuery.extend(true) limitations – https://stackoverflow.com/revisions/d4fed311-c9c4-4f4f-8107-9d4c2f7f5335/view-source

---

## Core Concept 4: Modern Native Alternatives

### Definitions

**Core Definition**
Modern JavaScript provides native alternatives for object manipulation: `Object spread ({...obj})` for shallow copies and `structuredClone()` for deep copies, with `Object.assign()` as a legacy shallow-copy alternative.

**Technical Definition**
`Object spread ({...obj})` creates a new object by iterating the enumerable own properties of the source and assigning them to a new object. It performs a shallow copy . `structuredClone(value)` uses the HTML structured clone algorithm to create a deep copy, supporting `Date`, `Map`, `Set`, `RegExp`, typed arrays, `ArrayBuffer`, and circular references, but throwing on functions and symbols . `Object.assign(target, ...sources)` copies enumerable own properties from sources to target, also performing a shallow copy .

**Beginner-Friendly Explanation**
`{...obj}` is the modern way to make a shallow copy — short and readable. `structuredClone(obj)` is the modern way to make a deep copy — it handles dates, maps, sets, and even circular references, but it can't copy functions.

### Purposes

- **Object Spread**: To create a shallow copy without jQuery dependency .
- **structuredClone()**: To create a robust deep copy of complex data structures .
- **Object.assign()**: To merge objects into a target (legacy alternative to spread).

### Syntax Rules and Structure

**Complete General Syntaxes**

**Object Spread (Shallow Copy):**
```javascript
const shallowCopy = { ...originalObject };
```

**Object.assign() (Shallow Copy):**
```javascript
const shallowCopy = Object.assign({}, originalObject);
```

**structuredClone() (Deep Copy):**
```javascript
const deepCopy = structuredClone(originalObject);
```

**Component Breakdown**

- `{ ...obj }` : Spreads enumerable own properties into a new object .
- `Object.assign({}, obj)` : Copies enumerable own properties into the target .
- `structuredClone(obj)` : Deep clones using the structured clone algorithm .

**Syntax Rules**

1. Spread and `Object.assign()` perform **shallow** copies .
2. `structuredClone()` performs a **deep** copy .
3. `structuredClone()` throws `DataCloneError` on functions and symbols .
4. `structuredClone()` is baseline available since March 2022 .
5. `structuredClone()` does not preserve prototypes (class instances become plain objects) .

**Constraints and Limitations**

- **structuredClone Limitations**: Cannot clone functions, DOM nodes, or symbols; does not preserve prototypes .
- **Object Spread Limitations**: Shared references for nested objects .
- **Browser Support**: `structuredClone()` requires modern browsers (Firefox 94+, Safari 15.4+, Chrome 98+) .

### Multiple Annotated Complete Code Examples

**Example 1: Object Spread vs. structuredClone**

```javascript
const original = {
  name: "Alice",
  address: {
    city: "Springfield",
    zip: "62704"
  },
  hobbies: ["reading", "gaming"]
};

// SHALLOW COPY with spread
const shallow = { ...original };

// DEEP COPY with structuredClone
const deep = structuredClone(original);

// Step 1: Modify nested array in shallow copy
shallow.hobbies.push("hiking");
console.log("Original hobbies:", original.hobbies);  // ["reading", "gaming", "hiking"]
console.log("Shallow hobbies:", shallow.hobbies);    // ["reading", "gaming", "hiking"]

// Step 2: Modify nested array in deep copy
deep.hobbies.push("cooking");
console.log("\nAfter deep copy mutation:");
console.log("Original hobbies:", original.hobbies);  // ["reading", "gaming", "hiking"]
console.log("Deep hobbies:", deep.hobbies);          // ["reading", "gaming", "hiking", "cooking"]
```

**Expected Output (Console)**
```
Original hobbies: [ 'reading', 'gaming', 'hiking' ]
Shallow hobbies: [ 'reading', 'gaming', 'hiking' ]

After deep copy mutation:
Original hobbies: [ 'reading', 'gaming', 'hiking' ]
Deep hobbies: [ 'reading', 'gaming', 'hiking', 'cooking' ]
```

**Why This Output Occurs**
The spread copy shares the `hobbies` array reference, so pushing to `shallow.hobbies` mutates the original. The `structuredClone()` copy has its own independent array .

---

**Example 2: structuredClone Preserves Date and Map**

```javascript
const original = {
  created: new Date("2024-01-15"),
  tags: new Map([["priority", "high"], ["status", "active"]]),
  regex: /test/i
};

const deep = structuredClone(original);

// Step 1: Check types are preserved
console.log("Date preserved:", deep.created instanceof Date);  // true
console.log("Map preserved:", deep.tags instanceof Map);       // true
console.log("RegExp preserved:", deep.regex instanceof RegExp); // true

// Step 2: Verify independence
deep.tags.set("priority", "low");
console.log("Original priority:", original.tags.get("priority")); // "high"
console.log("Deep priority:", deep.tags.get("priority"));         // "low"
```

**Expected Output (Console)**
```
Date preserved: true
Map preserved: true
RegExp preserved: true
Original priority: high
Deep priority: low
```

**Why This Output Occurs**
`structuredClone()` supports `Date`, `Map`, and `RegExp`, preserving their types and creating independent copies .

### Real-World Cases

- **State Management**: Using `structuredClone()` to create independent state snapshots .
- **Configuration Merging**: Using `{...defaults, ...options}` for shallow merges .
- **Data Processing**: Deep cloning API responses before transformation.

### References

- web.dev — Deep copies with structuredClone – https://web.dev/articles/structured-clone
- WHATWG HTML Standard — Structured cloning API – https://html.spec.whatwg.org/dev/structured-data.html
- MDN Web Docs — structuredClone() – https://developer.mozilla.org/en-US/docs/Web/API/structuredClone
- GitHub — Shallow vs Deep Copy – https://github.com/greatfrontend/top-javascript-interview-questions

---

## Summary Comparison Table

| Method | Type | Nested Objects | Date/RegExp | Circular Refs | Functions | Browser Support |
|---|---|---|---|---|---|---|
| `$.extend({}, obj)` | Shallow | Shared | Shared | Shared | Shared | All jQuery |
| `$.extend(true, {}, obj)` | Deep | Cloned | Plain Object | Throws | Shared | All jQuery |
| `{...obj}` | Shallow | Shared | Shared | Shared | Shared | ES2018+ |
| `Object.assign({}, obj)` | Shallow | Shared | Shared | Shared | Shared | ES2015+ |
| `structuredClone(obj)` | Deep | Cloned | Cloned | Supported | Throws | 2022+  |
| `JSON.parse(JSON.stringify())` | Deep | Cloned | Becomes String | Throws | Dropped | ES5+ |

---

## Important Notes on Version-Specific Behaviour and Limitations

1. **jQuery `$.extend()` Security**: Versions prior to 3.4 had a prototype pollution vulnerability with `__proto__` .
2. **structuredClone Baseline**: Available since March 2022 in all major browsers .
3. **structuredClone Limitations**: Cannot clone functions, DOM nodes, or symbols; does not preserve prototypes .
4. **jQuery `$.extend(true)` Limitations**: Does not reconstruct `Date`, `RegExp`, or custom class instances .
5. **Performance**: Spread operator is significantly faster than `structuredClone()` for shallow copies (40M+ vs 300K ops/sec in benchmarks) .
6. **JSON Method Pitfalls**: `JSON.parse(JSON.stringify())` loses `Date`, `undefined`, `NaN`, `Map`, `Set`, `RegExp`, and functions .

---

## References

- jQuery API — jQuery.extend() – https://api.jquery.com/jquery.Extend/
- Learn jQuery — Utility Methods – https://learn.jquery.com/using-jquery-core/utility-methods/
- web.dev — Deep copies with structuredClone – https://web.dev/articles/structured-clone
- WHATWG HTML Standard — Structured cloning API – https://html.spec.whatwg.org/dev/structured-data.html
- MDN Web Docs — structuredClone() – https://developer.mozilla.org/en-US/docs/Web/API/structuredClone
- GitHub — Shallow vs Deep Copy – https://github.com/greatfrontend/top-javascript-interview-questions
- jQuery Bug Tracker — Extended objects share single array – https://bugs.jquery.com/ticket/14613/
- BenchmarkLab — JavaScript fastest way to clone an object – https://benchmarklab.azurewebsites.net/Benchmarks/ShowResult/625992
- Stack Overflow — jQuery.extend(true) deep copy limitations – https://stackoverflow.com/revisions/d4fed311-c9c4-4f4f-8107-9d4c2f7f5335/view-source