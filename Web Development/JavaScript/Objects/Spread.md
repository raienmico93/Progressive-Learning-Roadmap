# JavaScript Object Spread — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Object spread is a syntax that expands the own enumerable properties of an object into a new object literal, enabling concise shallow copying, merging, and property overriding.

**Technical Definition:** Per ES2018 (ECMA-262, §13.2.5.1), the `...` syntax within an object literal is a `SpreadElement` in a `PropertyDefinitionList`. During object literal evaluation, `CopyDataProperties(target, source, excludedItems)` is invoked for each spread source, copying all own enumerable properties (both string-keyed and symbol-keyed) from the source to the newly created target object using `CreateDataPropertyOrThrow`. The resulting object is a new object with a new identity; the source objects are not modified.

**Beginner-Friendly Explanation:** Object spread is a shortcut for copying properties from one object into another. When you write `{ ...original, newProp: "value" }`, JavaScript makes a fresh object, copies all of `original`'s visible properties into it, then adds or overrides `newProp`. It's like photocopying a document and then writing something extra on the copy — the original is untouched.

### Key Characteristics

- **Creates a new object:** The result is a fresh object with a new reference.
- **Shallow copy:** Only the top-level properties are copied; nested objects are shared by reference.
- **Own enumerable only:** Inherited and non-enumerable properties are not copied.
- **String and symbol keys:** Both string-keyed and symbol-keyed enumerable properties are copied.
- **Later properties win:** Duplicate keys from later spreads or explicit properties override earlier ones.
- **Non-mutating:** The source objects are never modified.
- **Syntactic sugar:** Equivalent to `Object.assign({}, source)` for simple cases, with important differences.

### Prerequisites

- Basic JavaScript syntax (objects, properties, literals).
- Understanding of property attributes (enumerable, own).
- Familiarity with reference semantics and shallow vs deep copying.
- Awareness of `Object.assign()` and its behaviour.
- Basic knowledge of symbols.

### Related Programming Areas

- **Immutability:** Creating modified copies without mutating the original.
- **State management:** Redux and React use spread for state updates.
- **Configuration merging:** Combining defaults with user options.
- **Functional programming:** Non-mutating transformations.
- **Component props:** Spreading props in React and Vue.
- **Array spread:** Complementary syntax for arrays.

---

## Spread in Objects

### Definitions

**Core Definition:** Spread in objects is the use of `...source` inside an object literal to copy the own enumerable properties of `source` into the new object being created.

**Technical Definition:** Per ECMA-262 §13.2.5.1, an object literal may contain `SpreadElement` productions of the form `... AssignmentExpression`. When evaluated, the object literal creates a new object, then for each spread element, `CopyDataProperties(obj, source, [])` is called. This abstract operation retrieves the source's own property keys (both string and symbol), filters for enumerable properties, and creates corresponding data properties on the target using `CreateDataPropertyOrThrow`.

**Beginner-Friendly Explanation:** When you write `{ ...someObject }`, JavaScript copies all the visible properties of `someObject` into a brand-new object. You can combine this with explicit properties and other spreads.

### Purposes

- To create a shallow copy of an object.
- To merge multiple objects into one.
- To override specific properties while keeping the rest.
- To create new objects without mutating the original.
- To conditionally include properties in an object.

### Syntax Rules and Structure

**General Syntax:**
```js
const newObj = { ...source };
const merged = { ...obj1, ...obj2 };
const overridden = { ...original, key: newValue };
const conditional = { ...base, ...(condition ? { extra: true } : {}) };
```

**Component Breakdown:**
- `...`: The spread operator.
- `source`: Any expression that evaluates to an object.
- `{ }`: The object literal that becomes the new object.

**Syntax Rules:**
- Spread can appear anywhere in an object literal.
- Later properties override earlier ones with the same key.
- Explicit properties can appear before or after spreads.
- `null` and `undefined` sources are ignored (no error).
- Primitives are coerced to objects (their own enumerable properties, if any, are copied).
- Source objects are never modified.

**Constraints and Limitations:**
- Shallow copy: nested objects are shared by reference.
- Only own enumerable properties are copied.
- Inherited properties are not copied.
- Non-enumerable properties are not copied.
- Getter results are copied as values (not descriptors).

### Annotated Code Examples

#### Example 1: Basic Shallow Copy

```js
// Step 1: Define an original object.
const original = { a: 1, b: 2, c: 3 };

// Step 2: Create a shallow copy using spread.
const copy = { ...original };

// Step 3: Verify the copy is a new object.
console.log(copy);              // { a: 1, b: 2, c: 3 }
console.log(copy === original); // false
console.log(copy.a);            // 1
```

**Expected Output:**
```
{ a: 1, b: 2, c: 3 }
false
1
```

**Why This Output:** `{ ...original }` creates a new object with the same top-level properties. The two objects are distinct references.

#### Example 2: Adding Properties During Spread

```js
const user = { name: "Alice", age: 30 };

// Step 1: Spread and add a new property.
const enriched = { ...user, role: "admin", active: true };

console.log(enriched);
// { name: "Alice", age: 30, role: "admin", active: true }
```

**Expected Output:**
```
{ name: 'Alice', age: 30, role: 'admin', active: true }
```

**Why This Output:** The spread copies existing properties, then the explicit properties are added.

#### Example 3: Ignoring `null` and `undefined`

```js
const obj = { ...null, ...undefined, a: 1 };
console.log(obj); // { a: 1 }
```

**Expected Output:**
```
{ a: 1 }
```

**Why This Output:** Spreading `null` or `undefined` is a no-op (no error), unlike destructuring which throws.

### Real-World Cases

- **React state updates:** `setState({ ...state, count: state.count + 1 })`.
- **Redux reducers:** `return { ...state, loading: false, data };`.
- **Configuration defaults:** `const config = { ...defaults, ...userOptions };`.
- **Props spreading:** `<Button {...buttonProps} />`.

---

## Shallow Copy

### Definitions

**Core Definition:** A shallow copy is a new object whose top-level properties are copied from the source, but nested objects are shared by reference.

**Technical Definition:** Object spread performs a shallow copy: for each own enumerable property of the source, a new data property is created on the target with the same value. If the value is a primitive, the value is copied; if the value is an object, the reference is copied. The nested object is not duplicated.

**Beginner-Friendly Explanation:** A shallow copy copies the "top layer" of an object. If a property holds another object, both the original and the copy point to the same nested object. Changing the nested object affects both.

### Purposes

- To create a new top-level object without deep duplication.
- To update state immutably at the top level.
- To avoid the cost of deep copying when not needed.

### Syntax Rules and Structure

```js
const copy = { ...original };
// Top-level primitives are copied.
// Nested objects are shared.
```

**Constraints and Limitations:**
- Nested objects are shared references.
- Mutating a nested object affects both original and copy.
- For deep copies, use `structuredClone()` or a recursive copy function.

### Annotated Code Examples

#### Example 1: Demonstrating Shallow Copy

```js
const original = {
  name: "Alice",
  address: { city: "Paris", country: "France" },
};

// Step 1: Shallow copy.
const copy = { ...original };

// Step 2: Modify a top-level property (independent).
copy.name = "Bob";
console.log(original.name); // "Alice" (unchanged)
console.log(copy.name);     // "Bob"

// Step 3: Modify a nested property (shared!).
copy.address.city = "London";
console.log(original.address.city); // "London" (changed!)
console.log(copy.address.city);     // "London"
```

**Expected Output:**
```
Alice
Bob
London
London
```

**Why This Output:** Top-level properties are independent, but nested objects are shared references.

#### Example 2: Deep Copy with `structuredClone`

```js
const original = {
  name: "Alice",
  address: { city: "Paris" },
};

// Step 1: Deep copy with structuredClone.
const deepCopy = structuredClone(original);

// Step 2: Modify nested property.
deepCopy.address.city = "London";
console.log(original.address.city); // "Paris" (unchanged)
console.log(deepCopy.address.city); // "London"
```

**Expected Output:**
```
Paris
London
```

**Why This Output:** `structuredClone` performs a deep copy, so nested objects are independent.

### Real-World Cases

- **State management:** Top-level state updates via shallow copy.
- **Performance:** Avoiding deep copies when not needed.
- **Immutability:** Understanding when shallow copy is sufficient.

---

## Merging Objects

### Definitions

**Core Definition:** Merging objects is the practice of combining properties from multiple source objects into a single new object using spread.

**Technical Definition:** When multiple spread elements appear in an object literal, they are evaluated left to right. Each subsequent spread copies its own enumerable properties to the target, overriding any existing properties with the same keys. The final object contains the union of all properties, with later sources winning conflicts.

**Beginner-Friendly Explanation:** Merging with spread is like layering transparencies: each layer adds or replaces properties, and the final image shows the topmost version of each property.

### Purposes

- To combine configuration objects.
- To merge state updates.
- To compose objects from multiple sources.
- To apply defaults with overrides.

### Syntax Rules and Structure

```js
const merged = { ...source1, ...source2, ...source3 };
```

**Component Breakdown:**
- Sources are spread in order.
- Later sources override earlier ones for duplicate keys.
- Explicit properties override spreads based on position.

**Syntax Rules:**
- Order matters: later wins.
- Can mix spreads and explicit properties.
- `null` and `undefined` sources are ignored.

### Annotated Code Examples

#### Example 1: Merging Two Objects

```js
const defaults = { theme: "light", fontSize: 14, language: "en" };
const userPrefs = { theme: "dark", fontSize: 16 };

// Step 1: Merge with user preferences overriding defaults.
const config = { ...defaults, ...userPrefs };

console.log(config);
// { theme: "dark", fontSize: 16, language: "en" }
```

**Expected Output:**
```
{ theme: 'dark', fontSize: 16, language: 'en' }
```

**Why This Output:** `userPrefs` is spread after `defaults`, so its values win for duplicate keys. `language` comes only from `defaults`.

#### Example 2: Merging Multiple Sources with Explicit Overrides

```js
const a = { x: 1, y: 2 };
const b = { y: 3, z: 4 };
const c = { z: 5, w: 6 };

// Step 1: Merge in order with an explicit override.
const merged = { ...a, ...b, ...c, w: 100 };

console.log(merged); // { x: 1, y: 3, z: 5, w: 100 }
```

**Expected Output:**
```
{ x: 1, y: 3, z: 5, w: 100 }
```

**Why This Output:** Later spreads override earlier ones; the explicit `w: 100` overrides `c`'s `w: 6` because it comes after.

#### Example 3: Conditional Merging

```js
function createConfig(isProduction) {
  const base = { apiUrl: "/api", timeout: 5000 };
  const prodOverrides = isProduction ? { apiUrl: "https://api.example.com", timeout: 10000 } : {};

  return { ...base, ...prodOverrides };
}

console.log(createConfig(false)); // { apiUrl: "/api", timeout: 5000 }
console.log(createConfig(true));  // { apiUrl: "https://api.example.com", timeout: 10000 }
```

**Expected Output:**
```
{ apiUrl: '/api', timeout: 5000 }
{ apiUrl: 'https://api.example.com', timeout: 10000 }
```

**Why This Output:** Spreading an empty object when `isProduction` is `false` leaves the base unchanged.

### Real-World Cases

- **Configuration merging:** Defaults + environment overrides.
- **State updates:** Redux reducers merging state.
- **Component composition:** Combining default and custom props.

---

## Overriding Properties

### Definitions

**Core Definition:** Overriding properties is the practice of replacing the value of an existing property with a new value using spread and explicit properties.

**Technical Definition:** When an explicit property or a later spread contains a key that already exists in the target, the new value replaces the old. This follows the `CreateDataPropertyOrThrow` semantics: the property is redefined with the new value. The position of the property in the object's insertion order is determined by its first occurrence.

**Beginner-Friendly Explanation:** Overriding means "use this new value instead of the old one." You can override properties by placing them after the spread.

### Purposes

- To update specific properties while keeping the rest.
- To apply user options over defaults.
- To create variations of an object.
- To implement immutable updates.

### Syntax Rules and Structure

```js
const updated = { ...original, key: newValue };
const overridden = { ...original, ...overrides };
```

**Syntax Rules:**
- Later properties override earlier ones.
- The property's position is determined by its first appearance.
- Overriding a property does not change its position in iteration order.

### Annotated Code Examples

#### Example 1: Overriding a Single Property

```js
const user = { name: "Alice", age: 30, role: "user" };

// Step 1: Override the role.
const admin = { ...user, role: "admin" };

console.log(admin);
// { name: "Alice", age: 30, role: "admin" }
console.log(user.role); // "user" (unchanged)
```

**Expected Output:**
```
{ name: 'Alice', age: 30, role: 'admin' }
user
```

**Why This Output:** The explicit `role: "admin"` after the spread overrides the original value. The original object is unchanged.

#### Example 2: Position Preservation

```js
const obj = { a: 1, b: 2, c: 3 };
const updated = { ...obj, b: 99 };

console.log(Object.keys(updated)); // ["a", "b", "c"] (position preserved)
console.log(updated.b);            // 99
```

**Expected Output:**
```
[ 'a', 'b', 'c' ]
99
```

**Why This Output:** Overriding `b` does not change its position in the object's key order.

#### Example 3: Immutable State Update

```js
const state = {
  user: { name: "Alice" },
  posts: [1, 2, 3],
  loading: false,
};

// Step 1: Immutable update.
const newState = { ...state, loading: true };

console.log(newState.loading); // true
console.log(state.loading);    // false
console.log(newState.user === state.user); // true (shallow copy)
```

**Expected Output:**
```
true
false
true
```

**Why This Output:** Top-level `loading` is overridden; nested `user` is shared by reference.

### Real-World Cases

- **Redux:** `return { ...state, ...action.payload };`.
- **React:** `setState(prev => ({ ...prev, count: prev.count + 1 }))`.
- **Configuration:** `const config = { ...defaults, timeout: 10000 };`.

---

## Spread vs `Object.assign()`

### Definitions

**Core Definition:** Object spread and `Object.assign()` both copy own enumerable properties to a target, but they differ in syntax, target handling, and edge-case behaviour.

**Technical Definition:** `Object.assign(target, ...sources)` mutates and returns the target object. Object spread `{ ...source }` always creates a new object. Both use `[[Set]]` vs `CreateDataPropertyOrThrow` semantics: `Object.assign` uses `[[Set]]` (which triggers setters on the target), while spread uses `CreateDataPropertyOrThrow` (which defines properties directly, bypassing setters). Spread also creates a null-prototype-free object literal.

**Beginner-Friendly Explanation:** Spread and `Object.assign()` do similar things, but spread is cleaner syntax and always makes a new object. `Object.assign()` can mutate an existing object and behaves slightly differently with setters.

### Purposes

- **Spread:** Preferred for creating new objects and merging.
- **Object.assign:** Preferred when mutating an existing object is desired, or when the target has setters.

### Comparison Table

| Feature | Object Spread | `Object.assign()` |
|---|---|---|
| Syntax | `{ ...src }` | `Object.assign({}, src)` |
| Target | Always a new object | Can mutate an existing target |
| Return | The new object | The target object |
| Setter behaviour | Bypasses setters (defines properties) | Triggers setters on target |
| Getter behaviour | Reads getter values | Reads getter values |
| `null`/`undefined` sources | Ignored | Ignored |
| Symbol properties | Copied | Copied |
| Non-enumerable properties | Not copied | Not copied |
| Prototype | `Object.prototype` | Target's prototype |
| Nested objects | Shared (shallow) | Shared (shallow) |

### Annotated Code Examples

#### Example 1: Equivalent Behaviour

```js
const source = { a: 1, b: 2 };

// Spread
const spreadCopy = { ...source };
console.log(spreadCopy); // { a: 1, b: 2 }

// Object.assign
const assignCopy = Object.assign({}, source);
console.log(assignCopy); // { a: 1, b: 2 }
```

**Expected Output:**
```
{ a: 1, b: 2 }
{ a: 1, b: 2 }
```

**Why This Output:** For simple cases, both produce equivalent results.

#### Example 2: Setter Difference

```js
const target = {
  set value(v) { this._value = v * 2; },
  get value() { return this._value; },
};

// Object.assign triggers the setter.
Object.assign(target, { value: 10 });
console.log(target.value); // 20

// Spread bypasses the setter (defines property directly).
const newObj = { ...target, value: 10 };
console.log(newObj.value); // 10 (no setter called)
```

**Expected Output:**
```
20
10
```

**Why This Output:** `Object.assign` uses `[[Set]]`, invoking the setter. Spread uses `CreateDataPropertyOrThrow`, defining the property directly without calling the setter.

#### Example 3: Mutating vs Non-Mutating

```js
const original = { a: 1 };

// Object.assign mutates the target.
const target = { b: 2 };
Object.assign(target, original);
console.log(target); // { b: 2, a: 1 }
console.log(target === original); // false (target is separate)

// Spread creates a new object.
const spreadCopy = { ...original };
console.log(spreadCopy === original); // false
```

**Expected Output:**
```
{ b: 2, a: 1 }
false
false
```

**Why This Output:** `Object.assign` mutates the first argument; spread always creates a new object.

### Real-World Cases

- **Spread:** Preferred in modern code for readability and immutability.
- **Object.assign:** Used when mutating a target is intentional or when the target has setters.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Object Literals — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-object-initializer
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – CopyDataProperties — https://tc39.es/ecma262/multipage/abstract-operations.html#sec-copydataproperties
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Object.assign — https://tc39.es/ecma262/multipage/fundamental-objects.html#sec-object.assign
- MDN Web Docs – Spread syntax (...) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax
- MDN Web Docs – Object.assign() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign
- MDN Web Docs – Object initializer — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer
- MDN Web Docs – Rest parameters — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters
- MDN Web Docs – structuredClone() — https://developer.mozilla.org/en-US/docs/Web/API/Window/structuredClone
- TC39 Proposal – Object Rest/Spread Properties — https://github.com/tc39/proposal-object-rest-spread
- Exploring JS – Spreading into object literals — https://exploringjs.com/impatient-js/ch_objects.html#spreading-into-object-literals
- Redux Docs – Immutable Update Patterns — https://redux.js.org/usage/structuring-reducers/immutable-update-patterns
- React Docs – Updating Objects in State — https://react.dev/learn/updating-objects-in-state