# JavaScript Object Destructuring — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Object destructuring is a JavaScript expression that unpacks properties from objects into distinct variables using a syntax that mirrors object literals.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §13.15.5), object destructuring uses `ObjectAssignmentPattern` and `ObjectBindingPattern` grammar productions. The pattern `{ a, b }` extracts properties `a` and `b` from the source object using `GetV` and binds them to variables of the same name. Patterns support default values (`= default`), renaming (`a: x`), nested patterns, rest properties (`...rest`), and computed property names (`[key]: value`). Destructuring works with `let`, `const`, `var`, or as a plain assignment expression (which requires parentheses around the pattern).

**Beginner-Friendly Explanation:** Object destructuring is a shortcut for pulling values out of an object and putting them into variables. Instead of writing `const name = user.name; const age = user.age;`, you write `const { name, age } = user;`. It's like unpacking a suitcase — you take out exactly what you need and give each item a name.

### Key Characteristics

- **Mirrors object literal syntax:** Uses `{ }` to match properties by name.
- **Name-based, not position-based:** Properties are matched by key, not order.
- **Supports defaults:** Fallback values when a property is `undefined`.
- **Supports renaming:** Extract a property into a differently-named variable.
- **Supports nesting:** Destructure deeply nested objects in one pattern.
- **Supports rest:** Collect remaining properties into a new object.
- **Works in declarations, assignments, parameters, and loops.**

### Prerequisites

- Basic JavaScript syntax (objects, properties, variables).
- Understanding of property access (dot and bracket notation).
- Familiarity with `let`, `const`, and `var`.
- Awareness of `undefined` vs `null`.
- Basic knowledge of function parameters.

### Related Programming Areas

- **Function parameters:** Destructuring in function signatures.
- **React props:** Destructuring props in components.
- **API response handling:** Extracting fields from JSON.
- **Module imports:** `import { useState } from "react"`.
- **Array destructuring:** Complementary syntax for arrays.
- **Spread/rest:** Related `...` syntax for collecting values.

---

## Object Destructuring

### Definitions

**Core Definition:** Object destructuring is the process of extracting properties from an object into individual variables using a pattern that mirrors object literal syntax.

**Technical Definition:** Per ECMA-262 §13.15.5, an `ObjectAssignmentPattern` consists of zero or more `AssignmentProperty` productions enclosed in `{ }`. Each `AssignmentProperty` can be a simple identifier (shorthand), a `PropertyName : AssignmentElement` (renaming), or a computed property. During evaluation, `DestructuringAssignmentEvaluation` iterates the pattern, calls `GetV` on the source for each property key, and assigns the result to the target.

**Beginner-Friendly Explanation:** Object destructuring lets you "unpack" an object into variables in one line. You list the property names you want inside curly braces, and JavaScript creates variables with those names and values.

### Purposes

- To extract multiple properties from an object concisely.
- To improve readability by avoiding repetitive property access.
- To provide default values for missing properties.
- To rename properties during extraction.
- To destructure nested objects in one expression.
- To simplify function parameters and return values.

### Syntax Rules and Structure

**General Syntax:**
```js
const { prop1, prop2 } = object;
let { prop1, prop2 } = object;
var { prop1, prop2 } = object;
({ prop1, prop2 } = object); // Assignment (needs parentheses)
```

**Component Breakdown:**
- `{ }`: The destructuring pattern.
- `prop1, prop2`: Property names to extract.
- `object`: The source object.
- Variables are created with the same names as the properties.

**Syntax Rules:**
- Properties are matched by name, not position.
- Missing properties are assigned `undefined`.
- In assignment context (not declaration), the pattern must be wrapped in parentheses: `({ a, b } = obj);`.
- Cannot destructure `null` or `undefined` (throws `TypeError`).
- Default values only apply when the value is `undefined` (not `null`).

**Constraints and Limitations:**
- Property names must be valid identifiers unless renamed.
- Destructuring `null` or `undefined` throws `TypeError`.
- Defaults do not trigger on `null`.

### Annotated Code Examples

#### Example 1: Basic Destructuring

```js
// Step 1: Define a source object.
const user = {
  name: "Alice",
  age: 30,
  city: "Paris",
};

// Step 2: Destructure into variables.
const { name, age, city } = user;

// Step 3: Use the variables.
console.log(name); // "Alice"
console.log(age);  // 30
console.log(city); // "Paris"
```

**Expected Output:**
```
Alice
30
Paris
```

**Why This Output:** The pattern `{ name, age, city }` extracts the corresponding properties from `user` and creates variables with those names.

#### Example 2: Destructuring Without Declaration

```js
let name, age;
const user = { name: "Bob", age: 25 };

// Step 1: Assignment destructuring needs parentheses.
({ name, age } = user);

console.log(name); // "Bob"
console.log(age);  // 25
```

**Expected Output:**
```
Bob
25
```

**Why This Output:** In assignment context, the parentheses prevent the `{` from being interpreted as a block statement.

#### Example 3: Destructuring from Function Return Values

```js
function getUser() {
  return { id: 1, name: "Alice", role: "admin" };
}

// Step 1: Destructure the return value directly.
const { id, name, role } = getUser();

console.log(id, name, role); // 1 "Alice" "admin"
```

**Expected Output:**
```
1 Alice admin
```

**Why This Output:** The function returns an object, which is immediately destructured.

### Real-World Cases

- **API responses:** `const { data, status } = await fetch(url);`.
- **React props:** `function Button({ label, onClick }) { ... }`.
- **Configuration:** `const { host, port } = config;`.
- **Module imports:** `import { useState, useEffect } from "react";`.

---

## Default Values

### Definitions

**Core Definition:** Default values in destructuring provide a fallback value when a property is `undefined` or missing from the source object.

**Technical Definition:** Per ECMA-262 §13.15.5, an `AssignmentElement` can include `= AssignmentExpression` after the target. During `DestructuringAssignmentEvaluation`, if the extracted value is `undefined`, the default expression is evaluated and used instead.

**Beginner-Friendly Explanation:** Default values let you say "if this property is missing, use this value instead."

### Purposes

- To provide fallbacks for optional properties.
- To avoid `undefined` values in variables.
- To make destructuring robust against incomplete objects.
- To define configuration defaults inline.

### Syntax Rules and Structure

```js
const { prop = defaultValue } = object;
```

**Component Breakdown:**
- `prop`: The property name.
- `= defaultValue`: The fallback expression.
- `object`: The source object.

**Syntax Rules:**
- Default applies only when the extracted value is `undefined`.
- `null` does NOT trigger the default.
- Default expressions are evaluated lazily (only if needed).
- Defaults can reference earlier bindings in the same pattern.

**Constraints and Limitations:**
- `null` values are not replaced by defaults.
- Defaults cannot reference later bindings in the same pattern.
- Default expressions can have side effects (called only when needed).

### Annotated Code Examples

#### Example 1: Basic Defaults

```js
const settings = { theme: "dark" };

// Step 1: Destructure with defaults.
const { theme = "light", fontSize = 14, language = "en" } = settings;

console.log(theme);    // "dark" (present)
console.log(fontSize); // 14 (default)
console.log(language); // "en" (default)
```

**Expected Output:**
```
dark
14
en
```

**Why This Output:** `theme` exists, so its value is used. `fontSize` and `language` are missing, so defaults apply.

#### Example 2: `null` Does Not Trigger Defaults

```js
const config = { timeout: null };

const { timeout = 5000 } = config;
console.log(timeout); // null (not 5000!)
```

**Expected Output:**
```
null
```

**Why This Output:** Defaults only trigger for `undefined`, not `null`. This is a common pitfall.

#### Example 3: Defaults with Expressions

```js
const { count = 0, total = count * 2 } = {};
console.log(count); // 0
console.log(total); // 0 (references count)
```

**Expected Output:**
```
0
0
```

**Why This Output:** Default expressions can reference earlier bindings in the same pattern.

### Real-World Cases

- **Configuration:** `const { timeout = 5000 } = options;`.
- **Component props:** `function Card({ title = "Untitled" }) { ... }`.
- **API parsing:** `const { items = [], total = 0 } = response;`.

---

## Renaming

### Definitions

**Core Definition:** Renaming in destructuring extracts a property into a variable with a different name.

**Technical Definition:** Per ECMA-262, a `PropertyName : AssignmentElement` production in an `ObjectAssignmentPattern` allows the source property name to differ from the target variable name. The value of `source[propertyName]` is assigned to `target`.

**Beginner-Friendly Explanation:** Renaming lets you pull a property out under a different name. For example, `{ name: userName }` extracts `name` and stores it in `userName`.

### Purposes

- To avoid naming conflicts with existing variables.
- To give properties more meaningful names in a specific context.
- To rename reserved words or invalid identifiers.
- To improve readability.

### Syntax Rules and Structure

```js
const { sourceProp: targetVar } = object;
```

**Component Breakdown:**
- `sourceProp`: The property name in the object.
- `targetVar`: The new variable name.
- The colon separates source and target.

**Syntax Rules:**
- Can be combined with defaults: `{ a: x = 1 }`.
- Can be combined with nesting: `{ a: { b: c } }`.
- Property name can be a string: `{ "prop with spaces": x }`.
- Can use computed property names: `{ [key]: value }`.

**Constraints and Limitations:**
- The original property name is not available unless also destructured separately.
- Cannot rename to an invalid identifier.

### Annotated Code Examples

#### Example 1: Basic Renaming

```js
const user = { name: "Alice", age: 30 };

// Step 1: Rename name to userName.
const { name: userName, age: userAge } = user;

console.log(userName); // "Alice"
console.log(userAge);  // 30
// console.log(name);  // ReferenceError (name is not defined)
```

**Expected Output:**
```
Alice
30
```

**Why This Output:** The pattern `name: userName` extracts the `name` property and binds it to `userName`.

#### Example 2: Renaming with Defaults

```js
const settings = { theme: "dark" };

const { theme: selectedTheme = "light", fontSize: size = 14 } = settings;
console.log(selectedTheme); // "dark"
console.log(size);          // 14
```

**Expected Output:**
```
dark
14
```

**Why This Output:** Renaming and defaults can be combined: `sourceProp: targetVar = default`.

#### Example 3: Renaming with Reserved Words

```js
const data = { class: "A", default: true };

const { class: className, default: isDefault } = data;
console.log(className); // "A"
console.log(isDefault); // true
```

**Expected Output:**
```
A
true
```

**Why This Output:** Renaming allows extraction of properties named with reserved words.

### Real-World Cases

- **API normalisation:** `const { user_name: userName } = response;`.
- **Avoiding conflicts:** `const { name: propName } = props;`.
- **Reserved words:** `const { class: className } = element;`.

---

## Nested Destructuring

### Definitions

**Core Definition:** Nested destructuring extracts values from objects nested within other objects using a pattern that mirrors the nested structure.

**Technical Definition:** Per ECMA-262, an `AssignmentProperty` can contain a nested `AssignmentPattern` after the colon, allowing recursive destructuring. The engine evaluates the outer property, then applies the inner pattern to the result.

**Beginner-Friendly Explanation:** Nested destructuring lets you pull values out of objects inside objects — like unpacking a suitcase that has smaller bags inside.

### Purposes

- To extract deeply nested values in one expression.
- To avoid intermediate variables.
- To simplify code that accesses `obj.a.b.c`.
- To destructure complex API responses.

### Syntax Rules and Structure

```js
const { outer: { inner } } = object;
const { a: { b: { c } } } = object;
```

**Component Breakdown:**
- Outer pattern matches the outer property.
- Colon introduces the inner pattern.
- Inner pattern matches properties of the nested object.

**Syntax Rules:**
- Can combine with defaults, renaming, and rest.
- If an intermediate property is `undefined`, destructuring throws `TypeError`.
- Use defaults at each level to guard against missing intermediates.

**Constraints and Limitations:**
- Accessing a property of `undefined` or `null` throws.
- Deeply nested patterns can be hard to read.

### Annotated Code Examples

#### Example 1: Basic Nested Destructuring

```js
const user = {
  id: 1,
  profile: {
    name: "Alice",
    address: {
      city: "Paris",
      country: "France",
    },
  },
};

// Step 1: Destructure nested properties.
const {
  id,
  profile: {
    name,
    address: { city, country },
  },
} = user;

console.log(id);      // 1
console.log(name);    // "Alice"
console.log(city);    // "Paris"
console.log(country); // "France"
```

**Expected Output:**
```
1
Alice
Paris
France
```

**Why This Output:** The pattern mirrors the nested structure, extracting values at each level.

#### Example 2: Nested Destructuring with Defaults

```js
const config = { server: { port: 8080 } };

const {
  server: { host = "localhost", port = 3000 } = {},
} = config;

console.log(host); // "localhost"
console.log(port); // 8080
```

**Expected Output:**
```
localhost
8080
```

**Why This Output:** The `= {}` default on the `server` pattern prevents a `TypeError` if `server` is missing.

#### Example 3: Nested Destructuring Pitfall

```js
const data = { user: null };

try {
  const { user: { name } } = data; // TypeError: Cannot destructure property 'name' of 'null'
} catch (err) {
  console.log(err.name); // "TypeError"
}
```

**Expected Output:**
```
TypeError
```

**Why This Output:** Destructuring a property of `null` throws. Use `user: { name } = {}` to guard.

### Real-World Cases

- **API responses:** `const { data: { user: { name } } } = response;`.
- **Configuration objects:** `const { database: { host, port } } = config;`.
- **React props:** `const { user: { name, email } } = props;`.

---

## Rest in Destructuring

### Definitions

**Core Definition:** The rest property in destructuring collects all remaining own enumerable properties into a new object.

**Technical Definition:** Per ES2018, the `...rest` syntax in an object destructuring pattern creates a new object containing all own enumerable properties of the source that were not matched by the pattern. The rest target must be a valid binding identifier.

**Beginner-Friendly Explanation:** Rest lets you say "give me these specific properties, and put everything else in a new object."

### Purposes

- To extract specific properties while keeping the rest together.
- To exclude properties from an object.
- To create a shallow copy without certain keys.
- To handle optional extra properties.

### Syntax Rules and Structure

```js
const { a, b, ...rest } = object;
```

**Component Breakdown:**
- `a, b`: Properties to extract.
- `...rest`: Collects remaining own enumerable properties.
- Must be the last element in the pattern.

**Syntax Rules:**
- Rest must be last.
- Only own enumerable properties are collected.
- Inherited and non-enumerable properties are excluded.
- Cannot have a default value on rest.
- Can be nested.

**Constraints and Limitations:**
- Rest must be last in the pattern.
- Symbol-keyed properties are included if enumerable.
- Creates a shallow copy of remaining properties.

### Annotated Code Examples

#### Example 1: Basic Rest

```js
const user = { id: 1, name: "Alice", age: 30, city: "Paris" };

// Step 1: Extract id and name; collect the rest.
const { id, name, ...rest } = user;

console.log(id);   // 1
console.log(name); // "Alice"
console.log(rest); // { age: 30, city: "Paris" }
```

**Expected Output:**
```
1
Alice
{ age: 30, city: 'Paris' }
```

**Why This Output:** `id` and `name` are extracted; the remaining properties are collected into `rest`.

#### Example 2: Excluding a Property

```js
const config = { host: "localhost", port: 3000, secret: "abc123" };

// Step 1: Exclude secret.
const { secret, ...safeConfig } = config;
console.log(safeConfig); // { host: "localhost", port: 3000 }
```

**Expected Output:**
```
{ host: 'localhost', port: 3000 }
```

**Why This Output:** Extracting `secret` and collecting the rest effectively excludes it.

#### Example 3: Nested Rest

```js
const data = {
  user: { name: "Alice", age: 30, role: "admin" },
  status: "active",
};

const {
  user: { name, ...userRest },
  ...dataRest
} = data;

console.log(name);     // "Alice"
console.log(userRest); // { age: 30, role: "admin" }
console.log(dataRest); // { status: "active" }
```

**Expected Output:**
```
Alice
{ age: 30, role: 'admin' }
{ status: 'active' }
```

**Why This Output:** Rest works at multiple levels of nesting.

### Real-World Cases

- **Excluding sensitive data:** Removing `password` from user objects.
- **React props spreading:** `const { children, ...props } = this.props;`.
- **Immutability:** Creating new objects with some properties removed.
- **Redux reducers:** Extracting action payload and spreading the rest.

---

## Destructuring in Parameters

### Definitions

**Core Definition:** Destructuring in parameters allows a function to receive an object and immediately unpack its properties into named parameters.

**Technical Definition:** Per ECMA-262, a `BindingElement` in a function's `FormalParameters` can be an `ObjectBindingPattern`. When the function is called, the argument is destructured according to the pattern, and the extracted values are bound to local parameters.

**Beginner-Friendly Explanation:** Instead of accepting an object and then unpacking it inside the function, you can unpack it right in the parameter list.

### Purposes

- To make function signatures self-documenting.
- To avoid repetitive `options.foo` access.
- To provide default values for parameters.
- To enable named parameters (via object destructuring).

### Syntax Rules and Structure

```js
function fn({ prop1, prop2 }) { ... }
function fn({ prop1 = default1, prop2 } = {}) { ... }
const fn = ({ prop1, prop2 }) => { ... };
```

**Component Breakdown:**
- `{ prop1, prop2 }`: Destructuring pattern as parameter.
- `= {}`: Default for the entire parameter object.
- Inner defaults: `prop1 = default1`.

**Syntax Rules:**
- The parameter must be an object (or default to one).
- Calling without arguments when no `= {}` default throws `TypeError`.
- Can combine with renaming, nesting, and rest.
- Arrow functions support parameter destructuring.

**Constraints and Limitations:**
- Without a `= {}` default, calling with no argument throws.
- Deeply nested patterns can reduce readability.
- Destructuring in parameters creates new bindings; original object is not modified.

### Annotated Code Examples

#### Example 1: Basic Parameter Destructuring

```js
// Step 1: Define a function with destructured parameters.
function greet({ name, greeting = "Hello" }) {
  return `${greeting}, ${name}!`;
}

console.log(greet({ name: "Alice" }));               // "Hello, Alice!"
console.log(greet({ name: "Bob", greeting: "Hi" })); // "Hi, Bob!"
```

**Expected Output:**
```
Hello, Alice!
Hi, Bob!
```

**Why This Output:** The parameter is destructured, extracting `name` and `greeting` with a default.

#### Example 2: Default Parameter Object

```js
function createUser({ name = "Anonymous", age = 0, role = "user" } = {}) {
  return { name, age, role };
}

console.log(createUser());                          // { name: "Anonymous", age: 0, role: "user" }
console.log(createUser({ name: "Alice" }));          // { name: "Alice", age: 0, role: "user" }
console.log(createUser({ age: 30, role: "admin" })); // { name: "Anonymous", age: 30, role: "admin" }
```

**Expected Output:**
```
{ name: 'Anonymous', age: 0, role: 'user' }
{ name: 'Alice', age: 0, role: 'user' }
{ name: 'Anonymous', age: 30, role: 'admin' }
```

**Why This Output:** The `= {}` default allows calling without arguments; inner defaults fill missing properties.

#### Example 3: Arrow Function Destructuring

```js
const renderUser = ({ name, email }) => `${name} <${email}>`;
console.log(renderUser({ name: "Alice", email: "alice@example.com" }));
// "Alice <alice@example.com>"
```

**Expected Output:**
```
Alice <alice@example.com>
```

**Why This Output:** Arrow functions support destructured parameters directly.

### Real-World Cases

- **React components:** `function Card({ title, children }) { ... }`.
- **Express handlers:** `(req, { body: { name } }) => { ... }`.
- **Configuration functions:** `function setup({ host = "localhost", port = 3000 } = {}) { ... }`.
- **Utility functions:** `function pick({ id, name }) { ... }`.

---

## Destructuring in Loops

### Definitions

**Core Definition:** Destructuring in loops allows each iteration to unpack properties from array elements or object values into local variables.

**Technical Definition:** Per ECMA-262, the `ForBinding` and `ForDeclaration` grammar productions support `ObjectBindingPattern` and `ArrayBindingPattern`. In `for...of` and `for...in` loops, the loop variable can be a destructuring pattern.

**Beginner-Friendly Explanation:** When iterating over an array of objects, you can unpack each object's properties right in the loop header.

### Purposes

- To iterate over arrays of objects and access properties directly.
- To simplify loop bodies.
- To destructure entries from Maps or `Object.entries()`.
- To improve readability of data processing loops.

### Syntax Rules and Structure

```js
for (const { prop1, prop2 } of arrayOfObjects) { ... }
for (const [key, value] of Object.entries(obj)) { ... }
for (const { name } of users) { ... }
```

**Component Breakdown:**
- `const { prop1, prop2 }`: Destructuring pattern in loop header.
- `of arrayOfObjects`: Iterates the array.
- Each element is destructured.

**Syntax Rules:**
- Works with `for...of` and `for...in`.
- Works with `const`, `let`, and `var`.
- Can destructure nested properties.
- Can use defaults, renaming, and rest.

**Constraints and Limitations:**
- Each element must be destructure-able (object or iterable).
- Destructuring `null` or `undefined` elements throws.
- `for...in` yields string keys, not values.

### Annotated Code Examples

#### Example 1: Iterating Over an Array of Objects

```js
const users = [
  { name: "Alice", age: 30 },
  { name: "Bob", age: 25 },
  { name: "Carol", age: 35 },
];

// Step 1: Destructure each user in the loop.
for (const { name, age } of users) {
  console.log(`${name} is ${age} years old`);
}
```

**Expected Output:**
```
Alice is 30 years old
Bob is 25 years old
Carol is 35 years old
```

**Why This Output:** Each object in the array is destructured in the loop header.

#### Example 2: Iterating Over Object.entries()

```js
const scores = { math: 90, science: 85, history: 78 };

for (const [subject, score] of Object.entries(scores)) {
  console.log(`${subject}: ${score}`);
}
```

**Expected Output:**
```
math: 90
science: 85
history: 78
```

**Why This Output:** `Object.entries` returns `[key, value]` pairs, which are destructured in the loop.

#### Example 3: Nested Destructuring in Loops

```js
const orders = [
  { id: 1, customer: { name: "Alice", email: "alice@example.com" } },
  { id: 2, customer: { name: "Bob", email: "bob@example.com" } },
];

for (const { id, customer: { name, email } } of orders) {
  console.log(`Order ${id}: ${name} (${email})`);
}
```

**Expected Output:**
```
Order 1: Alice (alice@example.com)
Order 2: Bob (bob@example.com)
```

**Why This Output:** Nested destructuring works in loop headers too.

### Real-World Cases

- **Data processing:** Iterating API responses.
- **React lists:** Rendering arrays of items.
- **Map iteration:** `for (const [key, value] of map) { ... }`.
- **CSV parsing:** Processing rows of data.

---

## Destructuring Pitfalls

### Definitions

**Core Definition:** Destructuring pitfalls are the common mistakes and surprising behaviours that developers encounter when using object destructuring.

**Technical Definition:** Pitfalls arise from the interaction of destructuring with `undefined`, `null`, default values, assignment context, and nested patterns. The ECMAScript specification defines specific behaviours that can surprise developers.

**Beginner-Friendly Explanation:** Destructuring is powerful but has some gotchas. Knowing them helps you avoid bugs.

### Common Pitfalls

#### 1. Destructuring `null` or `undefined` Throws

```js
const { name } = null; // TypeError: Cannot destructure property 'name' of 'null'
```

**Fix:** Use a default: `const { name } = null ?? {};` or `const { name } = obj || {};`.

#### 2. Defaults Do Not Trigger on `null`

```js
const { timeout = 5000 } = { timeout: null };
console.log(timeout); // null (not 5000!)
```

**Fix:** Use `??` explicitly: `const timeout = obj.timeout ?? 5000;`.

#### 3. Assignment Context Needs Parentheses

```js
let name;
{ name } = { name: "Alice" }; // SyntaxError: Unexpected token '='
({ name } = { name: "Alice" }); // Correct
```

**Fix:** Wrap the pattern in parentheses in assignment context.

#### 4. Nested Destructuring Without Guards

```js
const data = {};
const { user: { name } } = data; // TypeError: Cannot destructure property 'name' of 'undefined'
```

**Fix:** `const { user: { name } = {} } = data;`.

#### 5. Missing Property Becomes `undefined`

```js
const { missing } = {};
console.log(missing); // undefined (not an error)
```

**Fix:** Provide defaults or check for `undefined`.

#### 6. Rest Must Be Last

```js
const { ...rest, a } = obj; // SyntaxError: Rest element must be last element
```

**Fix:** Always put rest last.

#### 7. Computed Keys Need Brackets

```js
const key = "name";
const { key } = obj;      // Extracts "key" property, not "name"
const { [key]: value } = obj; // Extracts "name" property
```

**Fix:** Use `[key]: value` for computed keys.

#### 8. Shadowing Variables

```js
const name = "Global";
const { name } = obj; // SyntaxError: Identifier 'name' has already been declared
```

**Fix:** Rename: `const { name: localName } = obj;`.

#### 9. Destructuring Function Parameters Without Defaults

```js
function greet({ name }) { return name; }
greet(); // TypeError: Cannot destructure property 'name' of 'undefined'
```

**Fix:** `function greet({ name } = {}) { ... }`.

#### 10. Order of Properties in Rest

```js
const obj = { a: 1, b: 2, c: 3 };
const { c, ...rest } = obj;
console.log(rest); // { a: 1, b: 2 } — order preserved from source
```

**Fix:** Remember that rest preserves source order, not pattern order.

### Annotated Code Examples

#### Example 1: `null` vs `undefined` Defaults

```js
const config = { timeout: null, retries: undefined };

const { timeout = 5000, retries = 3 } = config;
console.log(timeout); // null (not 5000!)
console.log(retries); // 3 (undefined triggers default)
```

**Expected Output:**
```
null
3
```

**Why This Output:** Defaults only trigger on `undefined`. `null` is a valid value.

**Fix:**
```js
const timeout = config.timeout ?? 5000; // null → 5000
```

#### Example 2: Nested Destructuring Guard

```js
const data = { user: {} };

// BAD: Throws if user is undefined
// const { user: { name } } = data;

// GOOD: Guard with default
const { user: { name = "Anonymous" } = {} } = data;
console.log(name); // "Anonymous"
```

**Expected Output:**
```
Anonymous
```

**Why This Output:** The `= {}` default guards against `user` being `undefined`.

#### Example 3: Assignment Parentheses

```js
let name, age;

// BAD: SyntaxError
// { name, age } = { name: "Alice", age: 30 };

// GOOD: Wrapped in parentheses
({ name, age } = { name: "Alice", age: 30 });
console.log(name, age); // "Alice" 30
```

**Expected Output:**
```
Alice 30
```

**Why This Output:** Parentheses disambiguate the destructuring pattern from a block statement.

### Real-World Cases

- **API integration:** Guarding against missing nested fields.
- **Configuration:** Using `??` instead of destructuring defaults for `null`.
- **React props:** Defaulting the entire props object.
- **Code review:** Catching common destructuring mistakes.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Destructuring Assignment — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-destructuring-assignment
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Object Destructuring — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-destructuring-assignment
- MDN Web Docs – Destructuring assignment — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment
- MDN Web Docs – Object destructuring — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#object_destructuring
- MDN Web Docs – Default values — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#default_value
- MDN Web Docs – Rest properties — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#rest_property
- MDN Web Docs – Nested object and array destructuring — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#nested_object_and_array_destructuring
- MDN Web Docs – Destructuring in function parameters — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#destructuring_function_arguments
- MDN Web Docs – for...of — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of
- MDN Web Docs – Object.entries() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/entries
- TC39 Proposal – Object Rest/Spread Properties — https://github.com/tc39/proposal-object-rest-spread
- Exploring JS – Destructuring — https://exploringjs.com/impatient-js/ch_destructuring.html
- You Don't Know JS – Destructuring — https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/es-next&beyond/ch3.md