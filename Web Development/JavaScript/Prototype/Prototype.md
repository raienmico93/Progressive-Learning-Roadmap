# JavaScript Prototype Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Prototypes are the mechanism by which JavaScript objects inherit features from one another, forming the foundation of JavaScript's prototype-based inheritance model.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §10.1), every ordinary object has an internal slot called `[[Prototype]]` that is either `null` or an object. When a property is accessed on an object and not found as an own property, the engine follows the object's `[[Prototype]]` reference and searches there, continuing up the chain until the property is found or `null` is reached. Functions have a `prototype` property (a data property) that becomes the `[[Prototype]]` of objects created via `new`. The `__proto__` accessor on `Object.prototype` provides a way to get and set `[[Prototype]]` in most implementations, though `Object.getPrototypeOf()` and `Object.setPrototypeOf()` are the standardised alternatives.

**Beginner-Friendly Explanation:** Think of prototypes as a "family tree" for objects. When you ask an object for a property it doesn't have, it checks with its parent (its prototype). If the parent doesn't have it, the grandparent is checked, and so on. This is how JavaScript objects share behaviour — they inherit from their prototypes.

### Key Characteristics

- **Every object has a prototype:** Except for objects created with `Object.create(null)`.
- **Prototype chain:** A linked list of objects connected by `[[Prototype]]` references.
- **Lookup walks the chain:** Property access follows the chain until found or `null`.
- **Prototype vs `prototype` property:** `[[Prototype]]` is the internal link; `prototype` is a property on functions.
- **Dynamic:** Prototypes can be modified at runtime (though this is discouraged).
- **Shadowing:** Own properties override inherited ones.
- **Rooted at `Object.prototype`:** Most chains end at `Object.prototype`, then `null`.

### Prerequisites

- Basic JavaScript syntax (objects, functions, properties).
- Understanding of object literals and property access.
- Familiarity with functions and the `new` operator.
- Awareness of `this` binding.
- Basic knowledge of `Object` static methods.

### Related Programming Areas

- **Inheritance:** Prototype chains implement inheritance in JavaScript.
- **Classes:** ES6 classes are syntactic sugar over prototypes.
- **Object.create():** The standard way to set prototypes explicitly.
- **Object methods:** `Object.getPrototypeOf`, `Object.setPrototypeOf`, `Object.create`.
- **Performance:** Deep prototype chains have lookup overhead.
- **Metaprogramming:** `Proxy`, `Reflect`, and `__proto__` for dynamic behaviour.

### Core Concepts / Features

---

## Prototypes

### Definitions

**Core Definition:** A prototype is an object from which other objects inherit properties and methods.

**Technical Definition:** Per ECMA-262 §10.1, an object's prototype is the value of its `[[Prototype]]` internal slot. In JavaScript's prototype-based inheritance, when a property is accessed and not found as an own property, the engine follows the `[[Prototype]]` chain. The `prototype` property on constructor functions is a separate concept: it is a data property that becomes the `[[Prototype]]` of objects created by calling the function with `new`.

**Beginner-Friendly Explanation:** A prototype is a "parent" object that shares its properties with "child" objects. When you access a property that a child doesn't have, JavaScript looks at the parent.

### Purposes

- To share properties and methods across many objects.
- To implement inheritance without classes.
- To save memory by avoiding duplication of shared behaviour.
- To enable dynamic extension of objects at runtime.
- To support the `instanceof` operator and prototype-based type checking.

### Syntax Rules and Structure

```js
// Object literal: [[Prototype]] is Object.prototype
const obj = { a: 1 };

// Constructor: instances' [[Prototype]] is Constructor.prototype
function Person(name) { this.name = name; }
Person.prototype.greet = function() { return `Hi, ${this.name}`; };
const alice = new Person("Alice");
alice.greet(); // "Hi, Alice"

// Object.create: explicit [[Prototype]]
const proto = { greet() { return "hi"; } };
const child = Object.create(proto);
child.greet(); // "hi"
```

**Syntax Rules:**
- Every ordinary object has a `[[Prototype]]` (may be `null`).
- The `prototype` property exists only on functions (and classes).
- Constructor `prototype` defaults to an object with a `constructor` property pointing back to the constructor.
- Setting `prototype` on a function changes the `[[Prototype]]` of future instances.

**Constraints and Limitations:**
- The `prototype` property of arrow functions does not exist; arrow functions cannot be constructors.
- Built-in prototypes (`Object.prototype`, `Array.prototype`) are shared globally.
- Modifying built-in prototypes can break other code.

### Annotated Code Examples

#### Example 1: Constructor Prototype

```js
// Step 1: Define a constructor.
function Animal(name) {
  this.name = name;
}

// Step 2: Add a method to the prototype.
Animal.prototype.speak = function() {
  return `${this.name} makes a sound.`;
};

// Step 3: Create instances.
const dog = new Animal("Rex");
const cat = new Animal("Whiskers");

// Step 4: Both share the method.
console.log(dog.speak()); // "Rex makes a sound."
console.log(cat.speak()); // "Whiskers makes a sound."

// Step 5: The method is inherited, not own.
console.log(Object.hasOwn(dog, "speak")); // false
```

**Expected Output:**
```
Rex makes a sound.
Whiskers makes a sound.
false
```

**Why This Output:** `speak` is defined on `Animal.prototype`, so all instances share it. `Object.hasOwn(dog, "speak")` is `false` because it's inherited, not own.

#### Example 2: Prototype is Shared

```js
function Counter() {}
Counter.prototype.count = 0;
Counter.prototype.increment = function() { this.count++; };

const c1 = new Counter();
const c2 = new Counter();

c1.increment();
console.log(c1.count); // 1
console.log(c2.count); // 0 (independent)
```

**Expected Output:**
```
1
0
```

**Why This Output:** Each instance has its own `count` after `increment` creates an own property (shadowing the prototype's `count`).

### Real-World Cases

- **Class methods:** All ES6 class methods live on the prototype.
- **Array methods:** `Array.prototype.push`, `Array.prototype.map`.
- **String methods:** `String.prototype.toUpperCase`.
- **Library inheritance:** Backbone.js, older frameworks.

---

## `[[Prototype]]`

### Definitions

**Core Definition:** `[[Prototype]]` is the internal slot of an object that holds a reference to its prototype.

**Technical Definition:** Per ECMA-262 §6.1.7.2, every object has an internal slot called `[[Prototype]]` whose value is either `null` or an object. This slot is not directly accessible from JavaScript; it can be read and written via `Object.getPrototypeOf()` / `Object.setPrototypeOf()` or the `__proto__` accessor.

**Beginner-Friendly Explanation:** `[[Prototype]]` is the "secret pointer" inside every object that tells JavaScript where to look next when a property isn't found. You can't touch it directly, but special methods let you read and change it.

### Purposes

- To establish inheritance relationships between objects.
- To enable property lookup through the prototype chain.
- To support the `instanceof` operator and `isPrototypeOf`.

### Syntax Rules and Structure

```js
// Read
Object.getPrototypeOf(obj);

// Write (discouraged)
Object.setPrototypeOf(obj, newProto);

// Accessor (legacy)
obj.__proto__;
```

**Syntax Rules:**
- `[[Prototype]]` is not a JavaScript property; it's an internal slot.
- `Object.getPrototypeOf` is the standard reader.
- `Object.setPrototypeOf` is the standard writer.
- The value must be an object or `null`.

**Constraints and Limitations:**
- Direct manipulation is discouraged for performance reasons.
- `Object.setPrototypeOf` deoptimises the object's hidden class in V8.
- Setting `[[Prototype]]` to a primitive throws `TypeError`.

### Annotated Code Examples

```js
const proto = { greet() { return "hi"; } };
const obj = Object.create(proto);

// Step 1: Read [[Prototype]].
console.log(Object.getPrototypeOf(obj) === proto); // true

// Step 2: [[Prototype]] of proto is Object.prototype.
console.log(Object.getPrototypeOf(proto) === Object.prototype); // true

// Step 3: [[Prototype]] of Object.prototype is null.
console.log(Object.getPrototypeOf(Object.prototype)); // null
```

**Expected Output:**
```
true
true
null
```

**Why This Output:** The prototype chain ends at `Object.prototype`, whose `[[Prototype]]` is `null`.

### Real-World Cases

- **Inheritance:** Every `class` hierarchy uses `[[Prototype]]`.
- **Polyfills:** Setting prototypes for older environments.
- **Frameworks:** React, Vue, Angular all use prototypes.

---

## `__proto__`

### Definitions

**Core Definition:** `__proto__` is an accessor property on `Object.prototype` that gets and sets an object's `[[Prototype]]`.

**Technical Definition:** Per ECMA-262 §B.2.2.1 (Annex B), `Object.prototype.__proto__` is an accessor property with a getter that returns `Object.getPrototypeOf(this)` and a setter that calls `Object.setPrototypeOf(this, value)`. It is normative-optional: implementations are not required to support it, though all major engines do.

**Beginner-Friendly Explanation:** `__proto__` is a special property that lets you read or change an object's prototype. It's the "old way" — the modern methods are `Object.getPrototypeOf` and `Object.setPrototypeOf`.

### Purposes

- To quickly inspect an object's prototype in the console.
- To set an object's prototype in legacy code.
- To create objects with a specific prototype inline in object literals.

### Syntax Rules and Structure

```js
// Get
const proto = obj.__proto__;

// Set (discouraged)
obj.__proto__ = newProto;

// In object literal (special case — sets [[Prototype]])
const obj = { __proto__: proto, a: 1 };
```

**Syntax Rules:**
- `__proto__` is an accessor inherited from `Object.prototype`.
- In object literals, `__proto__:` is a special syntax that sets `[[Prototype]]` (not a property).
- `Object.create(null)` objects do not have `__proto__`.

**Constraints and Limitations:**
- Deprecated in favour of `Object.getPrototypeOf` / `Object.setPrototypeOf`.
- Performance penalty (deoptimises hidden classes).
- `__proto__` in object literals is a special case that sets the prototype, not a regular property.

### Annotated Code Examples

```js
const proto = { greet() { return "hi"; } };

// Step 1: Set via object literal.
const obj = { __proto__: proto, name: "Alice" };
console.log(obj.greet()); // "hi"
console.log(Object.getPrototypeOf(obj) === proto); // true

// Step 2: Set via __proto__ accessor.
const obj2 = {};
obj2.__proto__ = proto;
console.log(obj2.greet()); // "hi"

// Step 3: Object literal __proto__ is special.
const obj3 = { __proto__: proto };
console.log(Object.keys(obj3)); // [] (no own properties)
```

**Expected Output:**
```
hi
true
hi
[]
```

**Why This Output:** In object literals, `__proto__:` sets the prototype; it does not create an own property.

### Real-World Cases

- **Console debugging:** Inspecting prototypes in DevTools.
- **Legacy code:** Older codebases.
- **JSON parsing pitfall:** `JSON.parse('{"__proto__": {"x": 1}}')` creates a real `__proto__` property in some engines.

---

## `Object.getPrototypeOf()`

### Definitions

**Core Definition:** `Object.getPrototypeOf()` returns the prototype (the value of `[[Prototype]]`) of a specified object.

**Technical Definition:** Per ECMA-262 §20.1.2.12, `Object.getPrototypeOf(O)` returns `O.[[GetPrototypeOf]]()`. For ordinary objects, this returns the value of the `[[Prototype]]` internal slot. `null` and `undefined` throw `TypeError`; other primitives are coerced to objects.

**Beginner-Friendly Explanation:** This method tells you what an object's prototype is — the object it inherits from.

### Purposes

- To inspect an object's prototype.
- To walk the prototype chain.
- To check inheritance relationships.
- To compare prototypes.

### Syntax Rules and Structure

```js
Object.getPrototypeOf(obj)
```

**Returns:** The prototype object, or `null`.

**Constraints and Limitations:**
- Throws `TypeError` for `null` or `undefined`.
- Returns `Object.prototype` for most ordinary objects.
- In some engines, `Object.getPrototypeOf` on a primitive returns the wrapper prototype.

### Annotated Code Examples

```js
// Step 1: Object literal.
const obj = { a: 1 };
console.log(Object.getPrototypeOf(obj) === Object.prototype); // true

// Step 2: Array literal.
const arr = [1, 2, 3];
console.log(Object.getPrototypeOf(arr) === Array.prototype); // true

// Step 3: Custom prototype.
const proto = { greet() {} };
const child = Object.create(proto);
console.log(Object.getPrototypeOf(child) === proto); // true

// Step 4: Walking the chain.
let current = arr;
const chain = [];
while (current !== null) {
  chain.push(current.constructor?.name || "null");
  current = Object.getPrototypeOf(current);
}
console.log(chain);
```

**Expected Output:**
```
true
true
true
[ 'Array', 'Object' ]
```

**Why This Output:** `arr` → `Array.prototype` → `Object.prototype` → `null`. The chain lists the constructors.

### Real-World Cases

- **Debugging:** Tracing inheritance.
- **Framework internals:** Vue, React use it.
- **Testing:** Verifying prototype relationships.

---

## `Object.setPrototypeOf()`

### Definitions

**Core Definition:** `Object.setPrototypeOf()` sets the prototype of a specified object to another object or `null`.

**Technical Definition:** Per ECMA-262 §20.1.2.22, `Object.setPrototypeOf(O, proto)` performs `O.[[SetPrototypeOf]](proto)`. If `O` is not extensible and `proto` differs from the current prototype, it throws `TypeError`. Setting the prototype deoptimises the object in V8.

**Beginner-Friendly Explanation:** This method changes what an object inherits from. It's the modern replacement for `obj.__proto__ = proto`.

### Purposes

- To change an object's prototype at runtime.
- To implement dynamic inheritance.
- To polyfill `__proto__`.

### Syntax Rules and Structure

```js
Object.setPrototypeOf(obj, proto)
```

**Returns:** The object `obj`.

**Constraints and Limitations:**
- Very slow in V8 (deoptimises hidden class).
- Throws `TypeError` if `obj` is non-extensible.
- `proto` must be an object or `null`.
- Prefer `Object.create(proto)` for creating objects with a specific prototype.

### Annotated Code Examples

```js
const proto = { greet() { return "hi"; } };
const obj = { name: "Alice" };

// Step 1: Set prototype.
Object.setPrototypeOf(obj, proto);
console.log(obj.greet()); // "hi"

// Step 2: Cannot set on non-extensible object.
const frozen = Object.freeze({});
try {
  Object.setPrototypeOf(frozen, proto);
} catch (err) {
  console.log(err.name); // "TypeError"
}
```

**Expected Output:**
```
hi
TypeError
```

**Why This Output:** Setting the prototype on a non-extensible object throws.

### Real-World Cases

- **Polyfills:** Implementing `__proto__`.
- **Dynamic inheritance:** Rare cases of runtime prototype changes.
- **Avoid when possible:** Use `Object.create` instead.

---

## Prototype Chain

### Definitions

**Core Definition:** The prototype chain is the sequence of objects linked by `[[Prototype]]` references, used for property lookup.

**Technical Definition:** Per ECMA-262, the prototype chain is traversed by the `[[Get]]` and `[[Set]]` internal methods. When a property is accessed, the engine searches the object's own properties, then follows the `[[Prototype]]` chain until the property is found or `null` is reached. The chain terminates at an object whose `[[Prototype]]` is `null` (typically `Object.prototype`).

**Beginner-Friendly Explanation:** The prototype chain is the "family tree" JavaScript follows when looking for a property. It starts at the object, then its parent, grandparent, and so on until the property is found or the end of the chain is reached.

### Purposes

- To enable inheritance and shared behaviour.
- To provide a predictable lookup order.
- To support `instanceof` and `isPrototypeOf`.

### Syntax Rules and Structure

```js
// Example chain
const grandparent = { a: 1 };
const parent = Object.create(grandparent);
parent.b = 2;
const child = Object.create(parent);
child.c = 3;

child.a; // 1 (grandparent)
child.b; // 2 (parent)
child.c; // 3 (own)
child.d; // undefined (not in chain)
```

**Chain for a typical object:**
```
child → parent → grandparent → Object.prototype → null
```

**Constraints and Limitations:**
- Deep chains have lookup overhead.
- Modifying the chain affects all objects that reference it.
- `in` operator checks the entire chain.

### Annotated Code Examples

#### Example 1: Walking the Chain

```js
const grandparent = { a: 1 };
const parent = Object.create(grandparent);
parent.b = 2;
const child = Object.create(parent);
child.c = 3;

// Step 1: Access inherited properties.
console.log(child.a); // 1
console.log(child.b); // 2
console.log(child.c); // 3
console.log(child.d); // undefined

// Step 2: in operator checks the chain.
console.log("a" in child); // true
console.log("d" in child); // false

// Step 3: hasOwnProperty only checks own.
console.log(Object.hasOwn(child, "a")); // false
console.log(Object.hasOwn(child, "c")); // true
```

**Expected Output:**
```
1
2
3
undefined
true
false
false
true
```

**Why This Output:** Property access walks the chain; `in` checks the chain; `Object.hasOwn` only checks own properties.

#### Example 2: Typical Array Chain

```js
const arr = [1, 2, 3];

const chain = [];
let obj = arr;
while (obj !== null) {
  chain.push(obj.constructor?.name || "null");
  obj = Object.getPrototypeOf(obj);
}
console.log(chain); // ["Array", "Object"]
```

**Expected Output:**
```
[ 'Array', 'Object' ]
```

**Why This Output:** Arrays inherit from `Array.prototype`, which inherits from `Object.prototype`.

### Real-World Cases

- **Class hierarchies:** Every `class` creates a prototype chain.
- **DOM elements:** HTMLButtonElement → HTMLElement → Element → Node → EventTarget.
- **Library design:** Prototype chains for extensibility.

---

## Prototype Lookup

### Definitions

**Core Definition:** Prototype lookup is the process by which JavaScript searches the prototype chain to find a property when it is accessed on an object.

**Technical Definition:** Per ECMA-262, the `[[Get]]` internal method searches the object's own property keys, then recursively calls the prototype's `[[Get]]`. The `[[Set]]` internal method is more nuanced: it searches for a writable data property or a setter. If it finds a data property on the prototype, it creates an own property on the receiver (shadowing); if it finds a setter, it calls the setter; if it finds a non-writable data property, it fails (or throws in strict mode).

**Beginner-Friendly Explanation:** When you read a property, JavaScript looks at the object first, then its prototype, and so on. When you write a property, it creates an own property on the object (unless the prototype has a setter).

### Purposes

- To resolve property accesses predictably.
- To understand shadowing behaviour.
- To implement getters and setters correctly.

### Syntax Rules and Structure

**Read (`[[Get]]`):**
1. Check own properties.
2. If found, return value.
3. Else, follow `[[Prototype]]`.
4. Repeat until found or `null`.
5. If not found, return `undefined`.

**Write (`[[Set]]`):**
1. Check own properties for a data property.
2. If found and writable, update value.
3. If found and non-writable, fail/throw.
4. If not own, search prototype chain.
5. If a setter is found, call it.
6. If a writable data property is found, create own property on receiver.
7. If a non-writable data property is found, fail/throw.

### Annotated Code Examples

#### Example 1: Read Lookup

```js
const proto = { a: 1 };
const obj = Object.create(proto);
obj.b = 2;

console.log(obj.a); // 1 (from proto)
console.log(obj.b); // 2 (own)
console.log(obj.c); // undefined (not found)
```

**Expected Output:**
```
1
2
undefined
```

**Why This Output:** Reads walk the chain until found or `null`.

#### Example 2: Write Creates Own Property

```js
const proto = { value: 10 };
const obj = Object.create(proto);

// Step 1: Read inherited value.
console.log(obj.value); // 10

// Step 2: Write creates own property (shadowing).
obj.value = 20;
console.log(obj.value);          // 20 (own)
console.log(proto.value);        // 10 (unchanged)
console.log(Object.hasOwn(obj, "value")); // true
```

**Expected Output:**
```
10
20
10
true
```

**Why This Output:** Assignment creates an own property on `obj`, shadowing the prototype's `value`.

#### Example 3: Setter in Prototype

```js
const proto = {
  set value(v) { this._value = v * 2; },
  get value() { return this._value; },
};
const obj = Object.create(proto);

obj.value = 10;
console.log(obj.value);          // 20 (setter doubled it)
console.log(Object.hasOwn(obj, "_value")); // true (setter created own)
console.log(Object.hasOwn(obj, "value"));  // false (accessor on proto)
```

**Expected Output:**
```
20
true
false
```

**Why This Output:** The prototype's setter is invoked, and it creates an own `_value` property on `obj`.

### Real-World Cases

- **Getters/setters:** Inherited accessors work via prototype lookup.
- **Class methods:** Inherited via prototype lookup.
- **Framework reactivity:** Vue uses setters defined on prototypes.

---

## Prototype Shadowing

### Definitions

**Core Definition:** Prototype shadowing occurs when an own property on an object has the same name as a property on its prototype, hiding the inherited one.

**Technical Definition:** Per ECMA-262, when `[[Set]]` creates an own property on the receiver with the same key as an inherited property, the own property "shadows" the inherited one. Subsequent `[[Get]]` operations find the own property first and stop searching.

**Beginner-Friendly Explanation:** Shadowing is like having a note on your desk that covers up a note on the wall behind it. When you look for the information, you find the closest one first.

### Purposes

- To override inherited behaviour for specific objects.
- To customise shared methods.
- To understand why inherited properties may appear "changed."

### Syntax Rules and Structure

```js
const proto = { value: "proto" };
const obj = Object.create(proto);

obj.value;         // "proto" (inherited)
obj.value = "own"; // Creates own property
obj.value;         // "own" (shadows)
delete obj.value;  // Removes own property
obj.value;         // "proto" (inherited again)
```

**Syntax Rules:**
- Own properties shadow inherited ones.
- Deleting the own property reveals the inherited one.
- Shadowing a non-writable inherited data property fails (or throws in strict mode).
- Shadowing an inherited setter calls the setter.

**Constraints and Limitations:**
- Shadowing a non-writable inherited property fails.
- Shadowing with `=` may not work as expected if the prototype has a setter.
- `Object.defineProperty` can bypass shadowing rules for own properties.

### Annotated Code Examples

#### Example 1: Basic Shadowing

```js
const proto = { name: "proto" };
const obj = Object.create(proto);

console.log(obj.name); // "proto"

obj.name = "own";
console.log(obj.name);          // "own"
console.log(proto.name);        // "proto" (unchanged)
console.log(Object.hasOwn(obj, "name")); // true

delete obj.name;
console.log(obj.name); // "proto" (revealed again)
```

**Expected Output:**
```
proto
own
proto
true
proto
```

**Why This Output:** Assignment creates an own property; deletion removes it and reveals the inherited one.

#### Example 2: Non-Writable Inherited Property

```js
const proto = {};
Object.defineProperty(proto, "value", { value: 10, writable: false });
const obj = Object.create(proto);

obj.value = 20; // Fails silently (non-strict) or TypeError (strict)
console.log(obj.value);          // 10
console.log(Object.hasOwn(obj, "value")); // false
```

**Expected Output:**
```
10
false
```

**Why This Output:** Non-writable inherited data properties block assignment.

#### Example 3: Shadowing an Inherited Method

```js
const proto = { greet() { return "proto greeting"; } };
const obj = Object.create(proto);

console.log(obj.greet()); // "proto greeting"

obj.greet = function() { return "own greeting"; };
console.log(obj.greet()); // "own greeting"
console.log(proto.greet()); // "proto greeting"
```

**Expected Output:**
```
proto greeting
own greeting
proto greeting
```

**Why This Output:** The own method shadows the inherited one for `obj`, but `proto` is unaffected.

### Real-World Cases

- **Method overriding:** Subclasses override parent methods.
- **Instance customisation:** Per-object method replacement.
- **Testing:** Mocking methods on instances.
- **Polyfill conflicts:** Built-in prototype modifications can shadow.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Ordinary Object Internal Methods — https://tc39.es/ecma262/multipage/ordinary-and-exotic-objects-behaviours.html#sec-ordinary-object-internal-methods-and-internal-slots
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Object.getPrototypeOf — https://tc39.es/ecma262/multipage/fundamental-objects.html#sec-object.getprototypeof
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Object.setPrototypeOf — https://tc39.es/ecma262/multipage/fundamental-objects.html#sec-object.setprototypeof
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Object.prototype.__proto__ — https://tc39.es/ecma262/multipage/additional-ecmascript-features-for-web-browsers.html#sec-object.prototype.__proto__
- MDN Web Docs – Object prototypes — https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Object_prototypes
- MDN Web Docs – Inheritance and the prototype chain — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain
- MDN Web Docs – Object.getPrototypeOf() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getPrototypeOf
- MDN Web Docs – Object.setPrototypeOf() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf
- MDN Web Docs – Object.create() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create
- MDN Web Docs – __proto__ — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/proto
- MDN Web Docs – instanceof — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof
- MDN Web Docs – Object.prototype.isPrototypeOf() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/isPrototypeOf
- MDN Web Docs – Classes — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes
- Exploring JS – Prototypes — https://exploringjs.com/impatient-js/ch_proto-chains-classes.html
- You Don't Know JS – Prototypes — https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/objects-classes/ch2.md
- V8 Blog – Fast Properties in V8 — https://v8.dev/blog/fast-properties
- V8 Blog – Setting Prototypes in V8 — https://v8.dev/blog/setting-prototypes