# JavaScript Object Properties — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Object properties are the named data slots and accessor functions attached to a JavaScript object, each governed by a set of attributes that control whether it can be read, written, deleted, or enumerated.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §6.1.7), an object is a collection of properties, each with zero or more attributes that determine how each property can be used. A property is either a **data property** (with a `[[Value]]` and a `[[Writable]]` attribute) or an **accessor property** (with `[[Get]]` and `[[Set]]` function attributes). Both kinds also carry `[[Enumerable]]` and `[[Configurable]]` attributes. Properties are either **own properties** (stored directly on the object) or **inherited properties** (resolved through the prototype chain). The `Object` constructor and its static methods provide programmatic access to property descriptors, own property enumeration, and symbol-keyed properties.

**Beginner-Friendly Explanation:** Every object is like a filing cabinet full of drawers. Each drawer (property) has a label (the name), contents (the value), and rules about whether you can open it, change what's inside, or remove it (the attributes). Some drawers are visible when you scan the cabinet (enumerable), and some are hidden (non-enumerable). Some drawers are inherited from a "parent" cabinet (prototype), and some belong directly to this cabinet (own properties). JavaScript gives you special tools to inspect and control all of this precisely.

### Key Characteristics

- **Two property kinds:** Data properties (hold a value) and accessor properties (run getter/setter functions).
- **Four attributes:** `value`, `writable`, `enumerable`, `configurable` for data properties; `get`, `set`, `enumerable`, `configurable` for accessor properties.
- **Own vs inherited:** Own properties belong to the object itself; inherited properties come from the prototype chain.
- **Enumerable vs non-enumerable:** Enumerable properties appear in `for...in`, `Object.keys`, and `JSON.stringify`; non-enumerable ones do not.
- **Configurable vs non-configurable:** Configurable properties can be deleted or redefined; non-configurable ones cannot (with limited exceptions).
- **Writable vs non-writable:** Writable data properties can be reassigned; non-writable ones throw in strict mode.
- **String and Symbol keys:** Property keys are either strings or symbols; symbol-keyed properties are invisible to many enumeration operations.

### Prerequisites

- Basic JavaScript syntax (variables, objects, functions).
- Understanding of object literals and property access.
- Familiarity with the prototype chain and inheritance.
- Awareness of strict mode.
- Basic knowledge of `Object` static methods.

### Related Programming Areas

- **Prototypes and inheritance:** Inherited properties come from the prototype chain.
- **Metaprogramming:** Property descriptors enable introspection and reflection.
- **Immutability:** `writable: false` and `configurable: false` enforce immutability.
- **Serialisation:** `enumerable` controls what appears in JSON output.
- **Security:** Non-writable and non-configurable properties prevent tampering.
- **Symbols:** Symbol-keyed properties enable unique, non-string keys.

---

## Own Properties

### Definitions

**Core Definition:** Own properties are properties stored directly on an object itself, as opposed to inherited properties that come from the prototype chain.

**Technical Definition:** Per ECMA-262, own properties are properties whose keys are in the object's own `[[OwnPropertyKeys]]` internal method result. The `Object.hasOwn(obj, key)` static method (and the legacy `Object.prototype.hasOwnProperty()`) tests whether a property is an own property. Own properties are created by object literals, assignment, `Object.defineProperty()`, or the `Object.create()` properties argument.

**Beginner-Friendly Explanation:** An own property is a drawer that belongs to this specific filing cabinet. When you look inside the cabinet, you'll find it right there — not borrowed from a parent cabinet.

### Purposes

- To distinguish properties that belong to an object from those inherited from its prototype.
- To safely iterate or copy only an object's own data.
- To avoid accidentally modifying inherited behaviour.
- To check ownership before reading or writing.

### Syntax Rules and Structure

```js
const obj = { ownProp: 1 };

Object.hasOwn(obj, "ownProp");       // true
Object.hasOwn(obj, "toString");      // false (inherited)
obj.hasOwnProperty("ownProp");       // true (legacy)
```

**Syntax Rules:**
- Own properties are created by literals, assignment, `defineProperty`, or `create`.
- `Object.hasOwn()` is the modern, preferred check (ES2022).
- `Object.prototype.hasOwnProperty()` is the legacy method but can be shadowed.
- `in` operator checks both own and inherited properties.

**Constraints and Limitations:**
- `hasOwnProperty` can be overridden by a property with that name; prefer `Object.hasOwn()`.
- The `in` operator does not distinguish own from inherited.

### Annotated Code Examples

#### Example 1: Own vs Inherited

```js
// Step 1: Create a prototype object.
const proto = { inherited: "from proto" };

// Step 2: Create an object that inherits from proto.
const obj = Object.create(proto);
obj.own = "my own";

// Step 3: Check ownership.
console.log(Object.hasOwn(obj, "own"));         // true
console.log(Object.hasOwn(obj, "inherited"));   // false
console.log("inherited" in obj);                // true (inherited)
console.log("own" in obj);                      // true
```

**Expected Output:**
```
true
false
true
true
```

**Why This Output:** `own` is a direct property; `inherited` is resolved through the prototype chain. `Object.hasOwn` distinguishes them; `in` does not.

#### Example 2: Copying Only Own Properties

```js
const source = { a: 1, b: 2 };
const copy = {};

for (const key of Object.keys(source)) {
  copy[key] = source[key];
}
console.log(copy); // { a: 1, b: 2 }
```

**Expected Output:**
```
{ a: 1, b: 2 }
```

**Why This Output:** `Object.keys` returns only own enumerable string-keyed properties.

### Real-World Cases

- **Serialisation:** Copying only own properties to JSON.
- **Merging objects:** Combining own properties from multiple sources.
- **Prototype-safe checks:** Using `Object.hasOwn` instead of `in`.

---

## Inherited Properties

### Definitions

**Core Definition:** Inherited properties are properties that an object resolves through its prototype chain rather than holding directly.

**Technical Definition:** Per ECMA-262, when a property is accessed and not found as an own property, the engine follows the object's `[[Prototype]]` reference and searches there, continuing up the chain until the property is found or the chain ends at `null`. Inherited properties are not in the object's own property keys but are accessible via the `in` operator and direct access.

**Beginner-Friendly Explanation:** Inherited properties are "borrowed" from a parent object. They work as if they were on your object, but they actually live on the prototype.

### Purposes

- To share methods and default values across many objects.
- To enable prototype-based inheritance.
- To reduce memory usage by sharing common behaviour.
- To allow objects to "override" inherited properties with their own.

### Syntax Rules and Structure

```js
const parent = { greet() { return "hi"; } };
const child = Object.create(parent);

child.greet();                   // "hi" (inherited)
Object.hasOwn(child, "greet");   // false
"greet" in child;                // true
```

**Syntax Rules:**
- Inherited properties are accessible via dot/bracket notation.
- `for...in` includes inherited enumerable properties.
- `Object.keys` and `Object.getOwnPropertyNames` exclude inherited properties.
- Assignment creates an own property even if an inherited one exists.

**Constraints and Limitations:**
- Inherited properties cannot be deleted from the child.
- Shadowing an inherited property creates an own property.
- Modifying `Object.prototype` affects all objects.

### Annotated Code Examples

#### Example 1: Accessing Inherited Properties

```js
const animal = { breathe() { return "breathing"; } };
const dog = Object.create(animal);
dog.bark = function () { return "woof"; };

console.log(dog.bark());    // "woof" (own)
console.log(dog.breathe()); // "breathing" (inherited)
console.log(Object.hasOwn(dog, "breathe")); // false
```

**Expected Output:**
```
woof
breathing
false
```

**Why This Output:** `bark` is an own property; `breathe` is resolved via the prototype chain.

#### Example 2: Shadowing Inherited Properties

```js
const parent = { value: "parent" };
const child = Object.create(parent);

child.value = "child"; // Creates own property, shadows inherited
console.log(child.value);                 // "child"
console.log(Object.hasOwn(child, "value")); // true
console.log(parent.value);                // "parent" (unchanged)
```

**Expected Output:**
```
child
true
parent
```

**Why This Output:** Assignment creates an own property on `child`, shadowing the inherited one.

### Real-World Cases

- **Class methods:** Methods on `prototype` are inherited by instances.
- **Built-in methods:** `toString`, `valueOf` from `Object.prototype`.
- **Mixin patterns:** Shared behaviour via prototype chains.

---

## Enumerable Properties

### Definitions

**Core Definition:** Enumerable properties are properties whose `[[Enumerable]]` attribute is `true`, meaning they are included in enumeration operations like `for...in`, `Object.keys`, and `JSON.stringify`.

**Technical Definition:** Per ECMA-262, `[[Enumerable]]` is a boolean attribute of every property. Properties created by simple assignment or object literals default to `enumerable: true`. Properties created by `Object.defineProperty()` default to `enumerable: false`.

**Beginner-Friendly Explanation:** Enumerable properties are the "visible" ones — they show up when you loop through an object or convert it to JSON.

### Purposes

- To control which properties are visible in loops and serialisation.
- To hide internal or helper properties from casual inspection.
- To make objects behave predictably with `Object.keys`, `for...in`, and `JSON.stringify`.

### Syntax Rules and Structure

```js
const obj = { visible: 1 };
Object.defineProperty(obj, "hidden", {
  value: 2,
  enumerable: false,
});

console.log(Object.keys(obj));     // ["visible"]
for (const k in obj) console.log(k); // "visible"
console.log(JSON.stringify(obj));  // '{"visible":1}'
```

**Syntax Rules:**
- Literal and assignment properties default to enumerable.
- `defineProperty` defaults to non-enumerable.
- `Object.keys`, `Object.values`, `Object.entries`, `for...in`, spread, and `JSON.stringify` respect enumerability.
- `Object.getOwnPropertyNames` and `Object.getOwnPropertySymbols` ignore enumerability.

### Annotated Code Examples

#### Example 1: Hiding a Property

```js
const user = { name: "Alice" };
Object.defineProperty(user, "password", {
  value: "secret",
  enumerable: false,
});

console.log(Object.keys(user));          // ["name"]
console.log(JSON.stringify(user));       // '{"name":"Alice"}'
console.log(user.password);              // "secret" (still accessible)
```

**Expected Output:**
```
[ 'name' ]
{"name":"Alice"}
secret
```

**Why This Output:** Non-enumerable properties are skipped by enumeration and serialisation but remain directly accessible.

### Real-World Cases

- **Serialisation control:** Hiding internal state from JSON.
- **Built-in methods:** `Object.prototype.toString` is non-enumerable.
- **Class fields:** Methods on prototypes are non-enumerable by default.

---

## Non-Enumerable Properties

### Definitions

**Core Definition:** Non-enumerable properties are properties whose `[[Enumerable]]` attribute is `false`, so they are excluded from enumeration and serialisation.

**Technical Definition:** Per ECMA-262, a non-enumerable property is skipped by `Object.keys`, `Object.values`, `Object.entries`, `for...in`, spread, and `JSON.stringify`, but is still accessible via direct access, `Object.getOwnPropertyNames`, `Object.getOwnPropertySymbols`, and `Object.getOwnPropertyDescriptor`.

**Beginner-Friendly Explanation:** Non-enumerable properties are "hidden" from loops and JSON, but you can still access them if you know their names.

### Purposes

- To hide internal implementation details.
- To avoid polluting JSON output.
- To match the behaviour of built-in methods (which are non-enumerable).

### Syntax Rules and Structure

```js
Object.defineProperty(obj, "hidden", {
  value: 42,
  enumerable: false,
});
```

**Constraints and Limitations:**
- Non-enumerable properties are still accessible and writable (unless `writable: false`).
- `Object.assign` copies enumerable own properties only.
- Spread `{...obj}` copies enumerable own properties only.

### Annotated Code Examples

```js
const obj = {};
Object.defineProperty(obj, "visible", { value: 1, enumerable: true });
Object.defineProperty(obj, "hidden", { value: 2, enumerable: false });

console.log(Object.keys(obj));               // ["visible"]
console.log(Object.getOwnPropertyNames(obj)); // ["visible", "hidden"]
console.log({ ...obj });                     // { visible: 1 }
```

**Expected Output:**
```
[ 'visible' ]
[ 'visible', 'hidden' ]
{ visible: 1 }
```

**Why This Output:** Enumeration methods respect `enumerable`; `getOwnPropertyNames` ignores it; spread copies only enumerable properties.

### Real-World Cases

- **Library internals:** Hiding helper methods.
- **Serialisation:** Excluding sensitive fields.
- **Built-in parity:** Methods are non-enumerable by default.

---

## Configurable Properties

### Definitions

**Core Definition:** Configurable properties are properties whose `[[Configurable]]` attribute is `true`, meaning they can be deleted, redefined, or converted between data and accessor properties.

**Technical Definition:** Per ECMA-262, when `[[Configurable]]` is `false`, the property cannot be deleted, its attributes cannot be changed (except `[[Value]]` if `[[Writable]]` is `true`), and it cannot be converted between data and accessor kinds. Properties created by literals and assignment default to `configurable: true`; properties created by `Object.defineProperty()` default to `configurable: false`.

**Beginner-Friendly Explanation:** A configurable property is one you can delete or change the rules for. A non-configurable property is "locked" in place — you can't delete it or redefine its attributes.

### Purposes

- To prevent accidental deletion or redefinition of critical properties.
- To enforce immutability at the attribute level.
- To protect built-in methods from tampering.

### Syntax Rules and Structure

```js
Object.defineProperty(obj, "locked", {
  value: 42,
  configurable: false, // Cannot be deleted or redefined
});

delete obj.locked; // Fails silently (non-strict) or TypeError (strict)
```

**Constraints and Limitations:**
- Non-configurable does not mean non-writable.
- `delete` on a non-configurable property returns `false` in non-strict mode and throws in strict mode.
- Once a property is non-configurable, it cannot be made configurable again.

### Annotated Code Examples

```js
const obj = {};
Object.defineProperty(obj, "locked", {
  value: 42,
  configurable: false,
});

console.log(delete obj.locked); // false
console.log(obj.locked);        // 42

try {
  Object.defineProperty(obj, "locked", { value: 100 });
  // TypeError in strict mode (or silently fails in non-strict)
} catch (err) {
  console.log(err.name); // "TypeError"
}
```

**Expected Output:**
```
false
42
TypeError
```

**Why This Output:** Non-configurable properties cannot be deleted or redefined.

### Real-World Cases

- **Immutable constants:** Locking configuration values.
- **Built-ins:** `Object.prototype` methods are non-configurable.
- **Security:** Preventing tampering with critical properties.

---

## Writable Properties

### Definitions

**Core Definition:** Writable properties are data properties whose `[[Writable]]` attribute is `true`, meaning their value can be changed by assignment.

**Technical Definition:** Per ECMA-262, when `[[Writable]]` is `false`, assignment to the property fails: silently in non-strict mode, or by throwing `TypeError` in strict mode. Accessor properties do not have `[[Writable]]`; instead, they rely on the presence of a setter.

**Beginner-Friendly Explanation:** A writable property is one you can change. A non-writable property is "read-only."

### Purposes

- To create read-only constants.
- To prevent accidental modification of critical values.
- To enforce immutability at the property level.

### Syntax Rules and Structure

```js
Object.defineProperty(obj, "readOnly", {
  value: 42,
  writable: false,
});

obj.readOnly = 100; // Fails silently (non-strict) or TypeError (strict)
```

**Constraints and Limitations:**
- Non-writable does not mean non-configurable.
- A non-writable, non-configurable property is fully locked.
- Accessor properties use `get`/`set` instead of `writable`.

### Annotated Code Examples

```js
const obj = {};
Object.defineProperty(obj, "constant", {
  value: 42,
  writable: false,
  configurable: false,
});

obj.constant = 100;
console.log(obj.constant); // 42 (unchanged)

"use strict";
try {
  obj.constant = 100;
} catch (err) {
  console.log(err.name); // "TypeError"
}
```

**Expected Output:**
```
42
TypeError
```

**Why This Output:** Non-writable properties ignore assignment; strict mode throws.

### Real-World Cases

- **Constants:** `Object.defineProperty(Math, "PI", { writable: false })` (already non-writable).
- **Configuration:** Locking config values after initialisation.
- **Immutability:** Building frozen objects.

---

## Property Descriptors

### Definitions

**Core Definition:** A property descriptor is an object that describes the attributes of a property, including whether it's a data or accessor property and its enumerable, configurable, writable, get, and set attributes.

**Technical Definition:** Per ECMA-262 §6.2.6, a Property Descriptor is a record with fields: `[[Value]]`, `[[Writable]]`, `[[Get]]`, `[[Set]]`, `[[Enumerable]]`, `[[Configurable]]`. A descriptor is either a data descriptor (has `value` or `writable`), an accessor descriptor (has `get` or `set`), or generic (has neither). Descriptors can be complete or partial.

**Beginner-Friendly Explanation:** A property descriptor is like a "spec sheet" for a property. It tells you the value, whether it's writable, enumerable, configurable, and whether it uses getters/setters.

### Purposes

- To inspect the full state of a property.
- To precisely define or modify property behaviour.
- To enable metaprogramming and reflection.
- To implement immutability and encapsulation.

### Syntax Rules and Structure

**Data descriptor:**
```js
{
  value: 42,
  writable: true,
  enumerable: true,
  configurable: true,
}
```

**Accessor descriptor:**
```js
{
  get() { return this._x; },
  set(v) { this._x = v; },
  enumerable: true,
  configurable: true,
}
```

**Constraints and Limitations:**
- Cannot mix data and accessor fields in one descriptor.
- When using `defineProperty`, unspecified fields default to `false`/`undefined`.
- Descriptors returned by `getOwnPropertyDescriptor` are complete.

### Annotated Code Examples

```js
const obj = { name: "Alice" };
const descriptor = Object.getOwnPropertyDescriptor(obj, "name");

console.log(descriptor);
// { value: 'Alice', writable: true, enumerable: true, configurable: true }
```

**Expected Output:**
```
{ value: 'Alice', writable: true, enumerable: true, configurable: true }
```

**Why This Output:** The descriptor describes all four attributes of the data property.

### Real-World Cases

- **Library design:** Defining non-enumerable methods.
- **Immutability:** Creating frozen or sealed objects.
- **Reflection:** Inspecting object structure at runtime.

---

## `Object.defineProperty()`

### Definitions

**Core Definition:** `Object.defineProperty()` defines a new property or modifies an existing property on an object, with precise control over its descriptor.

**Technical Definition:** Per ECMA-262 §20.1.2.4, `Object.defineProperty(O, P, Attributes)` returns `O` after defining or modifying the property `P` with the given descriptor. If the property exists and is non-configurable, only `[[Value]]` and `[[Writable]]` can be changed (and only if `[[Writable]]` is `true`). Otherwise, a `TypeError` is thrown.

**Beginner-Friendly Explanation:** `Object.defineProperty()` is the precise tool for creating or changing a property with exact control over its behaviour.

### Purposes

- To define non-enumerable, non-writable, or non-configurable properties.
- To create accessor properties with getters and setters.
- To modify property attributes after creation.
- To implement advanced object patterns.

### Syntax Rules and Structure

```js
Object.defineProperty(obj, propertyName, descriptor)
```

**Component Breakdown:**
- `obj`: The target object.
- `propertyName`: A string or Symbol.
- `descriptor`: A property descriptor object.
- Returns: The target object.

**Constraints and Limitations:**
- Throws `TypeError` for invalid descriptors.
- Non-configurable properties can only be modified in limited ways.
- Cannot mix data and accessor descriptor fields.

### Annotated Code Examples

#### Example 1: Defining a Non-Enumerable Property

```js
const user = { name: "Alice" };

Object.defineProperty(user, "password", {
  value: "secret",
  writable: false,
  enumerable: false,
  configurable: false,
});

console.log(user.password);          // "secret"
console.log(Object.keys(user));      // ["name"]
console.log(JSON.stringify(user));   // '{"name":"Alice"}'
```

**Expected Output:**
```
secret
[ 'name' ]
{"name":"Alice"}
```

**Why This Output:** The `password` property is hidden from enumeration and serialisation.

#### Example 2: Defining an Accessor Property

```js
const temp = { _celsius: 25 };

Object.defineProperty(temp, "fahrenheit", {
  get() { return this._celsius * 9 / 5 + 32; },
  set(f) { this._celsius = (f - 32) * 5 / 9; },
  enumerable: true,
  configurable: true,
});

console.log(temp.fahrenheit); // 77
temp.fahrenheit = 212;
console.log(temp._celsius);   // 100
```

**Expected Output:**
```
77
100
```

**Why This Output:** The getter and setter convert between Celsius and Fahrenheit.

### Real-World Cases

- **Library internals:** Hiding methods from `for...in`.
- **Computed properties:** Getters for derived values.
- **Immutability:** Locking critical properties.

---

## `Object.defineProperties()`

### Definitions

**Core Definition:** `Object.defineProperties()` defines multiple properties on an object simultaneously, each with its own descriptor.

**Technical Definition:** Per ECMA-262 §20.1.2.3, `Object.defineProperties(O, Properties)` iterates the own enumerable properties of `Properties`, each of which is a descriptor, and calls `Object.defineProperty` for each. Returns `O`.

**Beginner-Friendly Explanation:** `Object.defineProperties()` is like `Object.defineProperty()` but for multiple properties at once.

### Purposes

- To define multiple properties efficiently.
- To keep descriptor definitions grouped.
- To reduce boilerplate when configuring several properties.

### Syntax Rules and Structure

```js
Object.defineProperties(obj, {
  prop1: descriptor1,
  prop2: descriptor2,
});
```

**Constraints and Limitations:**
- Same rules as `defineProperty` per property.
- Order of definition is the order of the descriptor object's own enumerable keys.

### Annotated Code Examples

```js
const person = {};

Object.defineProperties(person, {
  name: {
    value: "Alice",
    writable: true,
    enumerable: true,
    configurable: true,
  },
  age: {
    value: 30,
    writable: false,
    enumerable: true,
    configurable: false,
  },
  secret: {
    value: "hidden",
    enumerable: false,
  },
});

console.log(person.name);        // "Alice"
console.log(person.age);         // 30
console.log(Object.keys(person)); // ["name", "age"]
```

**Expected Output:**
```
Alice
30
[ 'name', 'age' ]
```

**Why This Output:** Multiple properties are defined with individual descriptors.

### Real-World Cases

- **Class-like definitions:** Defining instance properties.
- **Configuration objects:** Grouped property setup.
- **Library APIs:** Precise control over many properties.

---

## `Object.getOwnPropertyDescriptor()`

### Definitions

**Core Definition:** `Object.getOwnPropertyDescriptor()` returns the property descriptor for a specific own property of an object.

**Technical Definition:** Per ECMA-262 §20.1.2.8, `Object.getOwnPropertyDescriptor(O, P)` returns a complete property descriptor for the own property `P` of `O`, or `undefined` if `P` is not an own property. The returned object is a new object, not a live reference.

**Beginner-Friendly Explanation:** This method tells you everything about a specific property: its value, whether it's writable, enumerable, configurable, and whether it uses getters/setters.

### Purposes

- To inspect a property's attributes.
- To debug unexpected property behaviour.
- To check whether a property is enumerable, writable, or configurable.

### Syntax Rules and Structure

```js
Object.getOwnPropertyDescriptor(obj, propertyName)
```

**Returns:** A descriptor object, or `undefined`.

**Constraints and Limitations:**
- Only returns own properties, not inherited.
- Returns a snapshot; changes to the descriptor do not affect the object.

### Annotated Code Examples

```js
const obj = { name: "Alice" };

const desc = Object.getOwnPropertyDescriptor(obj, "name");
console.log(desc);
// { value: 'Alice', writable: true, enumerable: true, configurable: true }

const missing = Object.getOwnPropertyDescriptor(obj, "toString");
console.log(missing); // undefined (inherited, not own)
```

**Expected Output:**
```
{ value: 'Alice', writable: true, enumerable: true, configurable: true }
undefined
```

**Why This Output:** `getOwnPropertyDescriptor` only returns own properties; inherited ones return `undefined`.

### Real-World Cases

- **Debugging:** Checking why a property behaves unexpectedly.
- **Reflection:** Inspecting library objects.
- **Cloning:** Copying property descriptors to a new object.

---

## `Object.getOwnPropertyDescriptors()`

### Definitions

**Core Definition:** `Object.getOwnPropertyDescriptors()` returns an object containing all own property descriptors of a given object.

**Technical Definition:** Per ECMA-262 §20.1.2.9, `Object.getOwnPropertyDescriptors(O)` returns a new object whose own properties are the descriptors of all own properties of `O`, including non-enumerable and symbol-keyed properties.

**Beginner-Friendly Explanation:** This method returns a "spec sheet" for every own property of an object, all in one object.

### Purposes

- To inspect all own properties at once.
- To copy objects with their full descriptors (including non-enumerable properties).
- To enable precise object cloning.

### Syntax Rules and Structure

```js
Object.getOwnPropertyDescriptors(obj)
```

**Returns:** An object mapping property names to descriptors.

### Annotated Code Examples

```js
const obj = {};
Object.defineProperty(obj, "a", { value: 1, enumerable: true });
Object.defineProperty(obj, "b", { value: 2, enumerable: false });

const descriptors = Object.getOwnPropertyDescriptors(obj);
console.log(Object.keys(descriptors)); // ["a", "b"]

// Precise clone preserving descriptors.
const clone = Object.create(Object.getPrototypeOf(obj));
Object.defineProperties(clone, descriptors);
console.log(clone.b); // 2 (non-enumerable preserved)
```

**Expected Output:**
```
[ 'a', 'b' ]
2
```

**Why This Output:** `getOwnPropertyDescriptors` includes non-enumerable properties; the clone preserves them.

### Real-World Cases

- **Deep cloning:** Preserving descriptors during copy.
- **Reflection:** Full introspection of an object.
- **Library design:** Understanding property configuration.

---

## `Object.getOwnPropertyNames()`

### Definitions

**Core Definition:** `Object.getOwnPropertyNames()` returns an array of all own property names (string-keyed) of an object, including non-enumerable ones.

**Technical Definition:** Per ECMA-262 §20.1.2.10, `Object.getOwnPropertyNames(O)` returns an array of all own property keys of `O` that are strings, in the order defined by `[[OwnPropertyKeys]]`: integer-like keys ascending, then other string keys in insertion order.

**Beginner-Friendly Explanation:** This method gives you a list of all the property names on an object — even the hidden ones — but not symbols.

### Purposes

- To list all string-keyed own properties, including non-enumerable ones.
- To inspect objects that hide properties.
- To enumerate an object's full string-keyed structure.

### Syntax Rules and Structure

```js
Object.getOwnPropertyNames(obj)
```

**Returns:** An array of strings.

**Constraints and Limitations:**
- Does not include symbol-keyed properties.
- Does not include inherited properties.
- Includes non-enumerable properties.

### Annotated Code Examples

```js
const obj = { visible: 1 };
Object.defineProperty(obj, "hidden", { value: 2, enumerable: false });

console.log(Object.keys(obj));                  // ["visible"]
console.log(Object.getOwnPropertyNames(obj));   // ["visible", "hidden"]
```

**Expected Output:**
```
[ 'visible' ]
[ 'visible', 'hidden' ]
```

**Why This Output:** `getOwnPropertyNames` includes non-enumerable string-keyed properties.

### Real-World Cases

- **Debugging:** Finding hidden properties.
- **Reflection:** Full string-keyed introspection.
- **Serialisation:** Custom serialisers that include non-enumerable properties.

---

## `Object.getOwnPropertySymbols()`

### Definitions

**Core Definition:** `Object.getOwnPropertySymbols()` returns an array of all own symbol-keyed properties of an object.

**Technical Definition:** Per ECMA-262 §20.1.2.11, `Object.getOwnPropertySymbols(O)` returns an array of all own property keys of `O` that are symbols, in insertion order. This is the only way to enumerate symbol-keyed properties.

**Beginner-Friendly Explanation:** Symbol keys are invisible to most enumeration methods. This method is the way to find them.

### Purposes

- To enumerate symbol-keyed properties.
- To inspect objects that use symbols for metadata.
- To implement well-known symbol behaviour.

### Syntax Rules and Structure

```js
Object.getOwnPropertySymbols(obj)
```

**Returns:** An array of Symbols.

**Constraints and Limitations:**
- Includes non-enumerable symbol properties.
- Does not include inherited symbols.
- Symbols are not coerced to strings.

### Annotated Code Examples

```js
const sym = Symbol("meta");
const obj = { [sym]: "symbol value", name: "Alice" };

console.log(Object.keys(obj));                    // ["name"]
console.log(Object.getOwnPropertyNames(obj));     // ["name"]
console.log(Object.getOwnPropertySymbols(obj));   // [Symbol(meta)]

const symbols = Object.getOwnPropertySymbols(obj);
console.log(obj[symbols[0]]);                     // "symbol value"
```

**Expected Output:**
```
[ 'name' ]
[ 'name' ]
[ Symbol(meta) ]
symbol value
```

**Why This Output:** `getOwnPropertySymbols` is the only way to enumerate symbol-keyed properties.

### Real-World Cases

- **Well-known symbols:** `Symbol.iterator`, `Symbol.toStringTag`.
- **Metadata:** Library-internal symbol keys.
- **Reflection:** Full introspection including symbols.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Property Descriptors — https://tc39.es/ecma262/multipage/ecmascript-data-types-and-values.html#sec-property-descriptors
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Object.defineProperty — https://tc39.es/ecma262/multipage/fundamental-objects.html#sec-object.defineproperty
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Object.defineProperties — https://tc39.es/ecma262/multipage/fundamental-objects.html#sec-object.defineproperties
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Object.getOwnPropertyDescriptor — https://tc39.es/ecma262/multipage/fundamental-objects.html#sec-object.getownpropertydescriptor
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Object.getOwnPropertyDescriptors — https://tc39.es/ecma262/multipage/fundamental-objects.html#sec-object.getownpropertydescriptors
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Object.getOwnPropertyNames — https://tc39.es/ecma262/multipage/fundamental-objects.html#sec-object.getownpropertynames
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Object.getOwnPropertySymbols — https://tc39.es/ecma262/multipage/fundamental-objects.html#sec-object.getownpropertysymbols
- MDN Web Docs – Object.defineProperty() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty
- MDN Web Docs – Object.defineProperties() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperties
- MDN Web Docs – Object.getOwnPropertyDescriptor() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor
- MDN Web Docs – Object.getOwnPropertyDescriptors() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptors
- MDN Web Docs – Object.getOwnPropertyNames() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyNames
- MDN Web Docs – Object.getOwnPropertySymbols() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertySymbols
- MDN Web Docs – Object.hasOwn() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwn
- MDN Web Docs – Enumerability and ownership of properties — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Enumerability_and_ownership_of_properties
- MDN Web Docs – Property descriptors — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty#description