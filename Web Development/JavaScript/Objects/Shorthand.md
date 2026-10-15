# JavaScript Property Shorthand and Computed Properties — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Property shorthand and computed properties are ES2015+ object literal enhancements that reduce repetition and enable dynamic property names when constructing objects.

**Technical Definition:** Per ECMAScript 2025 (ECMA-262, §13.2.5.1), an `ObjectLiteral` supports three enhancements over ES5 syntax: (1) **Shorthand properties** — `{ foo }` is syntactic sugar for `{ foo: foo }`, where the property name is derived from the `IdentifierReference` and the value is the result of evaluating that identifier; (2) **Shorthand methods** — `{ method() {} }` is equivalent to `{ method: function() {} }`, defined via a `MethodDefinition` production with `[[Kind]]` set to `"method"`; (3) **Computed property names** — `{ [expr]: value }` evaluates `expr` at object creation time, coerces the result to a property key (string or symbol), and uses it as the property name. These features work together to make object literals more expressive and concise.

**Beginner-Friendly Explanation:** When you build objects in JavaScript, you often find yourself writing the same name twice: `{ name: name, age: age }`. Shorthand lets you just write `{ name, age }`. Similarly, if you want a method, you can write `{ greet() {} }` instead of `{ greet: function() {} }`. And computed property names let you use a variable or expression as a property key: `{ [userId]: userData }`. Together, these features make object creation cleaner, shorter, and more dynamic.

### Key Characteristics

- **ES2015+ features:** Available in all modern JavaScript environments.
- **Shorthand properties:** The property name is taken from the variable name.
- **Shorthand methods:** Methods defined without the `function` keyword.
- **Computed property names:** Property keys are determined by expressions at runtime.
- **Composable:** All three can be combined in a single object literal.
- **Expressions evaluated once:** Computed names are evaluated at object creation, once per property.
- **Symbol-friendly:** Computed names can be symbols, enabling well-known symbol usage.

### Prerequisites

- Basic JavaScript syntax (objects, properties, variables).
- Understanding of object literals and property access.
- Familiarity with functions and function expressions.
- Awareness of symbols and property keys.
- Basic knowledge of expressions and evaluation order.

### Related Programming Areas

- **Object construction:** Building objects dynamically.
- **Functional programming:** Returning objects from functions concisely.
- **React/Redux:** Returning state objects from reducers.
- **Module patterns:** Building exports dynamically.
- **DOM manipulation:** Setting element properties dynamically.
- **Reactive frameworks:** Vue's `data` and `computed` use these features.

---

## Shorthand Properties

### Definitions

**Core Definition:** Shorthand property syntax allows you to omit the value when the property name matches a variable name in scope.

**Technical Definition:** Per ECMA-262 §13.2.5.1, an `ObjectLiteral` `PropertyDefinition` can be a single `IdentifierReference`. In this case, the property name is the identifier's name (a string), and the value is the result of evaluating the identifier. This is equivalent to `IdentifierReference : IdentifierReference`.

**Beginner-Friendly Explanation:** Shorthand properties let you write `{ name }` instead of `{ name: name }` when you have a variable named `name`.

### Purposes

- To reduce repetition when property names match variable names.
- To improve readability when building objects from variables.
- To make object literals more concise.
- To simplify returning objects from functions.

### Syntax Rules and Structure

**General Syntax:**
```js
const obj = { variableName };
```

**Component Breakdown:**
- `variableName`: An in-scope identifier.
- Property name = identifier name (as a string).
- Property value = the identifier's current value.

**Equivalent to:**
```js
const obj = { variableName: variableName };
```

**Syntax Rules:**
- The identifier must be in scope.
- Can be mixed with regular properties.
- Can be combined with computed names and methods.
- Works with `let`, `const`, and `var` bindings.
- Cannot be used with reserved words unless they are valid identifiers.

**Constraints and Limitations:**
- Only works when the property name matches the variable name.
- The identifier must exist; otherwise `ReferenceError`.
- Does not work with member expressions (`obj.prop`).

### Annotated Code Examples

#### Example 1: Basic Shorthand

```js
// Step 1: Define variables.
const name = "Alice";
const age = 30;
const city = "Paris";

// Step 2: Create an object using shorthand.
const person = { name, age, city };

console.log(person);
// { name: "Alice", age: 30, city: "Paris" }
```

**Expected Output:**
```
{ name: 'Alice', age: 30, city: 'Paris' }
```

**Why This Output:** Each shorthand identifier becomes a property with the same name and value.

#### Example 2: Mixing Shorthand with Regular Properties

```js
const firstName = "Alice";
const lastName = "Smith";

const user = {
  firstName,
  lastName,
  role: "admin", // Regular property
  active: true,
};

console.log(user);
// { firstName: "Alice", lastName: "Smith", role: "admin", active: true }
```

**Expected Output:**
```
{ firstName: 'Alice', lastName: 'Smith', role: 'admin', active: true }
```

**Why This Output:** Shorthand and regular properties can be freely mixed.

#### Example 3: Returning Objects from Functions

```js
function createPoint(x, y) {
  return { x, y }; // Shorthand
}

const p = createPoint(3, 4);
console.log(p); // { x: 3, y: 4 }
```

**Expected Output:**
```
{ x: 3, y: 4 }
```

**Why This Output:** Shorthand makes returning objects from functions concise.

#### Example 4: Shorthand in Destructuring Round-Trips

```js
const config = { host: "localhost", port: 3000 };

// Destructure
const { host, port } = config;

// Reconstruct with shorthand
const copy = { host, port };
console.log(copy); // { host: "localhost", port: 3000 }
```

**Expected Output:**
```
{ host: 'localhost', port: 3000 }
```

**Why This Output:** Destructuring extracts values into variables; shorthand builds a new object from them.

### Real-World Cases

- **Function returns:** `return { data, error, loading };`.
- **Redux reducers:** `return { ...state, user };`.
- **Module exports:** `export { foo, bar };`.
- **Component state:** `const [state, setState] = useState({ count, name });`.

---

## Shorthand Methods

### Definitions

**Core Definition:** Shorthand method syntax allows you to define methods in object literals without the `function` keyword or a colon.

**Technical Definition:** Per ECMA-262 §13.2.5.1, a `MethodDefinition` inside an object literal has the form `PropertyName ( UniqueFormalParameters ) { FunctionBody }`. The method is defined as a function-valued property. Unlike `function` expressions, methods have a `[[HomeObject]]` internal slot, which enables `super` references. Methods are non-constructable (cannot be used with `new`), and their `name` property is set to the method name.

**Beginner-Friendly Explanation:** Shorthand methods let you write `{ greet() {} }` instead of `{ greet: function() {} }`. It's shorter and slightly different in behaviour (methods can use `super`).

### Purposes

- To define methods concisely.
- To improve readability of object literals.
- To visually distinguish methods from data properties.
- To enable `super` references in object literals.
- To define generators and async methods cleanly.

### Syntax Rules and Structure

**Basic method:**
```js
const obj = {
  methodName(params) {
    // body
  },
};
```

**Async method:**
```js
const obj = {
  async fetchData() {
    // await ...
  },
};
```

**Generator method:**
```js
const obj = {
  *gen() {
    yield 1;
  },
};
```

**Computed method name:**
```js
const obj = {
  ["method" + "Name"]() {
    return "called";
  },
};
```

**Component Breakdown:**
- `methodName`: The method name.
- `(params)`: The parameter list.
- `{ }`: The method body.

**Syntax Rules:**
- No `function` keyword.
- No colon after the method name.
- Methods are non-enumerable in classes but enumerable in object literals.
- Can be async (`async method() {}`), generators (`*method() {}`), or async generators (`async *method() {}`).
- Can be combined with computed names (`[expr]() {}`).
- Methods have a `[[HomeObject]]` enabling `super`.

**Constraints and Limitations:**
- Methods cannot be used as constructors (`new obj.method()` throws `TypeError`).
- Methods do not have a `prototype` property.
- The `super` keyword only works in methods (not in arrow functions defined as properties).

### Annotated Code Examples

#### Example 1: Basic Method Shorthand

```js
// Step 1: Define an object with shorthand methods.
const calculator = {
  value: 0,

  add(n) {
    this.value += n;
    return this;
  },

  subtract(n) {
    this.value -= n;
    return this;
  },

  get result() {
    return this.value;
  },
};

// Step 2: Use the methods.
calculator.add(10).subtract(3);
console.log(calculator.result); // 7
```

**Expected Output:**
```
7
```

**Why This Output:** The methods use `this` to access and modify `value`. Returning `this` enables chaining.

#### Example 2: Async and Generator Methods

```js
const api = {
  async fetchUser(id) {
    // Simulated fetch
    return { id, name: "Alice" };
  },

  *range(n) {
    for (let i = 0; i < n; i++) yield i;
  },
};

// Async method
api.fetchUser(1).then((user) => console.log(user.name)); // "Alice"

// Generator method
console.log([...api.range(3)]); // [0, 1, 2]
```

**Expected Output:**
```
Alice
[ 0, 1, 2 ]
```

**Why This Output:** Shorthand syntax supports `async` and `*` modifiers.

#### Example 3: Computed Method Name

```js
const actionType = "increment";

const reducer = {
  [`handle${actionType}`](state) {
    return { ...state, count: state.count + 1 };
  },
};

console.log(reducer.handleincrement({ count: 0 })); // { count: 1 }
```

**Expected Output:**
```
{ count: 1 }
```

**Why This Output:** The method name is computed from `actionType`.

#### Example 4: Method with `super`

```js
const base = {
  greet() {
    return "Hello from base";
  },
};

const derived = {
  __proto__: base,
  greet() {
    return `${super.greet()} and derived`;
  },
};

console.log(derived.greet()); // "Hello from base and derived"
```

**Expected Output:**
```
Hello from base and derived
```

**Why This Output:** Shorthand methods have a `[[HomeObject]]`, enabling `super` to reference the prototype.

### Real-World Cases

- **Object APIs:** `const api = { getUser() {}, createUser() {} };`.
- **Vue methods:** `methods: { handleClick() {} }`.
- **Redux action creators:** `{ increment() { return { type: "INCREMENT" }; } }`.
- **Utility modules:** `{ formatDate() {}, parseDate() {} }`.

---

## Computed Property Names

### Definitions

**Core Definition:** Computed property names allow the property name in an object literal to be determined by an expression, enclosed in square brackets.

**Technical Definition:** Per ECMA-262 §13.2.5.1, a `PropertyName` in an `ObjectLiteral` can be a `ComputedPropertyName` of the form `[ AssignmentExpression ]`. During object literal evaluation, the expression is evaluated, its result is converted to a property key via `ToPropertyKey` (string or symbol), and the property is created with that key. The expression is evaluated exactly once, in source order.

**Beginner-Friendly Explanation:** Computed property names let you use a variable or expression as a property name: `{ [key]: value }`.

### Purposes

- To create properties with dynamic names.
- To avoid repetitive code when property names follow a pattern.
- To use variables as keys.
- To use symbols as property keys.
- To build objects from data.

### Syntax Rules and Structure

**Basic computed name:**
```js
const key = "dynamic";
const obj = { [key]: "value" };
```

**Expression as key:**
```js
const obj = { ["pre" + "fix"]: "value" };
```

**Symbol as key:**
```js
const sym = Symbol("meta");
const obj = { [sym]: "symbol value" };
```

**Computed method name:**
```js
const obj = { ["method" + "Name"]() { return "called"; } };
```

**Computed getter/setter:**
```js
const obj = {
  get ["prop" + "Name"]() { return "computed"; },
  set ["prop" + "Name"](v) { /* ... */ },
};
```

**Component Breakdown:**
- `[ ]`: Brackets enclose the expression.
- `AssignmentExpression`: Any expression evaluating to a string or symbol.
- Result is coerced to a property key.

**Syntax Rules:**
- Expression is evaluated at object creation time.
- Evaluated exactly once per property.
- Can be combined with shorthand, methods, and accessors.
- Result is coerced via `ToPropertyKey` (string or symbol).
- Evaluation order follows source order.

**Constraints and Limitations:**
- The expression must be evaluated at creation time; cannot be deferred.
- Computed keys cannot use shorthand property syntax (`{ [key] }` is invalid; use `{ [key]: key }`).
- Numeric keys are coerced to strings.

### Annotated Code Examples

#### Example 1: Basic Computed Property

```js
// Step 1: Define a variable for the key.
const fieldName = "email";
const fieldValue = "alice@example.com";

// Step 2: Create an object with a computed property.
const formData = { [fieldName]: fieldValue };

console.log(formData);
// { email: "alice@example.com" }
```

**Expected Output:**
```
{ email: 'alice@example.com' }
```

**Why This Output:** The value of `fieldName` ("email") becomes the property name.

#### Example 2: Expression as Key

```js
const index = 3;

const items = {
  [`item_${index}`]: "third item",
  [`item_${index + 1}`]: "fourth item",
};

console.log(items);
// { item_3: "third item", item_4: "fourth item" }
```

**Expected Output:**
```
{ item_3: 'third item', item_4: 'fourth item' }
```

**Why This Output:** Template literal expressions compute the property names.

#### Example 3: Symbol Keys

```js
const metaSymbol = Symbol("metadata");

const obj = {
  name: "Alice",
  [metaSymbol]: { created: "2025-01-01" },
};

console.log(obj[metaSymbol]); // { created: "2025-01-01" }
console.log(Object.keys(obj)); // ["name"] (symbol not included)
```

**Expected Output:**
```
{ created: '2025-01-01' }
[ 'name' ]
```

**Why This Output:** Symbol keys are valid computed property names but are not returned by `Object.keys`.

#### Example 4: Computed Methods and Accessors

```js
const prefix = "user";

const api = {
  [`get${prefix}`](id) {
    return { id, name: "Alice" };
  },

  get [`${prefix}Count`]() {
    return 42;
  },
};

console.log(api.getuser(1));  // { id: 1, name: "Alice" }
console.log(api.userCount);   // 42
```

**Expected Output:**
```
{ id: 1, name: 'Alice' }
42
```

**Why This Output:** Computed names work with methods and getters.

### Real-World Cases

- **Form handling:** `{ [event.target.name]: event.target.value }`.
- **Redux reducers:** `{ [action.type]: handler }`.
- **API normalisation:** `{ [user.id]: user }`.
- **Localisation:** `{ [locale]: translations }`.
- **Symbol-based metadata:** `{ [Symbol.iterator]() {} }`.

---

## Dynamic Property Names

### Definitions

**Core Definition:** Dynamic property names are property keys determined at runtime by variables, expressions, or user input, typically using computed property syntax.

**Technical Definition:** Dynamic property names arise when the property key is not statically known at parse time. This can be achieved via computed property names in object literals, bracket notation for property access, or `Object.defineProperty` with a computed key. The key is coerced to a string or symbol at creation or access time.

**Beginner-Friendly Explanation:** Dynamic property names let you build objects whose keys aren't known until the program runs — like when you're building an object from user input or a list of fields.

### Purposes

- To build objects from dynamic data.
- To create maps of IDs to values.
- To handle form fields with unknown names.
- To implement caches keyed by computed values.
- To normalise data from external sources.

### Syntax Rules and Structure

**Computed property in literal:**
```js
const obj = { [dynamicKey]: value };
```

**Bracket notation for access:**
```js
obj[dynamicKey] = value;
const v = obj[dynamicKey];
```

**`Object.defineProperty` with computed key:**
```js
Object.defineProperty(obj, dynamicKey, { value });
```

**Constraints and Limitations:**
- Keys are coerced to strings (or used as symbols).
- Object keys are always strings or symbols; numbers are coerced.
- Object keys must be unique; duplicates overwrite.
- Beware of prototype pollution when keys come from user input.

### Annotated Code Examples

#### Example 1: Building an Object from Form Data

```js
function buildFormData(fields) {
  const data = {};
  for (const { name, value } of fields) {
    data[name] = value;
  }
  return data;
}

const formData = buildFormData([
  { name: "username", value: "alice" },
  { name: "email", value: "alice@example.com" },
  { name: "age", value: "30" },
]);

console.log(formData);
// { username: "alice", email: "alice@example.com", age: "30" }
```

**Expected Output:**
```
{ username: 'alice', email: 'alice@example.com', age: '30' }
```

**Why This Output:** Each field's name becomes a key via bracket notation assignment.

#### Example 2: Normalising an Array to a Keyed Object

```js
const users = [
  { id: 1, name: "Alice" },
  { id: 2, name: "Bob" },
  { id: 3, name: "Carol" },
];

// Step 1: Normalise to an object keyed by id.
const usersById = {};
for (const user of users) {
  usersById[user.id] = user;
}

console.log(usersById[2]); // { id: 2, name: "Bob" }
```

**Expected Output:**
```
{ id: 2, name: 'Bob' }
```

**Why This Output:** Bracket notation with `user.id` creates dynamic keys.

#### Example 3: Computed Property in Object Literal

```js
function createEvent(type, payload) {
  return {
    type,
    payload,
    [`${type}_at`]: Date.now(),
  };
}

const event = createEvent("click", { x: 10, y: 20 });
console.log(event);
// { type: "click", payload: { x: 10, y: 20 }, click_at: <timestamp> }
```

**Expected Output (approximate):**
```
{ type: 'click', payload: { x: 10, y: 20 }, click_at: 1729000000000 }
```

**Why This Output:** The computed key combines `type` with `"_at"`.

#### Example 4: Symbol-Based Dynamic Keys

```js
const iterable = {
  data: [1, 2, 3],
  [Symbol.iterator]() {
    let i = 0;
    const data = this.data;
    return {
      next() {
        return i < data.length
          ? { value: data[i++], done: false }
          : { done: true };
      },
    };
  },
};

console.log([...iterable]); // [1, 2, 3]
```

**Expected Output:**
```
[ 1, 2, 3 ]
```

**Why This Output:** `Symbol.iterator` is a well-known symbol used as a dynamic key to make the object iterable.

### Real-World Cases

- **Redux:** `{ [action.type]: handler }` in reducers.
- **Form handling:** Building state from input names.
- **Caching:** `cache[key] = value`.
- **Localisation:** `translations[locale]`.
- **Normalised stores:** `{ [entity.id]: entity }`.

---

## Use Cases

### Definitions

**Core Definition:** Use cases for property shorthand and computed properties are the practical scenarios where these features provide concise, readable, and dynamic object construction.

**Technical Definition:** These features are applicable whenever object literals need to be built from variables, when methods need to be defined concisely, when property names are dynamic, or when symbols are used as keys.

**Beginner-Friendly Explanation:** These features are useful any time you're building objects from variables, defining methods, or using dynamic keys.

### Common Use Cases

1. **Building Objects from Variables**
   ```js
   const name = "Alice", age = 30;
   const user = { name, age };
   ```

2. **Returning Multiple Values from Functions**
   ```js
   function minMax(arr) {
     return { min: Math.min(...arr), max: Math.max(...arr) };
   }
   ```

3. **Defining Object APIs**
   ```js
   const mathUtils = {
     add(a, b) { return a + b; },
     subtract(a, b) { return a - b; },
   };
   ```

4. **Dynamic Form Data**
   ```js
   const formData = { [event.target.name]: event.target.value };
   ```

5. **Redux Action Handlers**
   ```js
   const handlers = {
     [ActionTypes.INCREMENT](state) { return { ...state, count: state.count + 1 }; },
     [ActionTypes.DECREMENT](state) { return { ...state, count: state.count - 1 }; },
   };
   ```

6. **Normalising Data**
   ```js
   const byId = {};
   for (const item of items) byId[item.id] = item;
   ```

7. **Well-Known Symbols**
   ```js
   class MyCollection {
     [Symbol.iterator]() { /* ... */ }
     get [Symbol.toStringTag]() { return "MyCollection"; }
   }
   ```

8. **Localisation**
   ```js
   const messages = {
     [locale]: { greeting: "Hello" },
   };
   ```

9. **Dynamic Method Names**
   ```js
   const prefix = "handle";
   const handler = {
     [`${prefix}Click`]() { console.log("clicked"); },
   };
   ```

10. **Combining All Three**
    ```js
    const key = "dynamic";
    const name = "Alice";
    const obj = {
      name,
      [key]: "value",
      greet() { return `Hi, ${this.name}`; },
    };
    ```

### Annotated Code Examples

#### Example 1: Redux-Style Reducer with Computed Handlers

```js
const initialState = { count: 0 };

const handlers = {
  INCREMENT(state) {
    return { ...state, count: state.count + 1 };
  },
  DECREMENT(state) {
    return { ...state, count: state.count - 1 };
  },
  RESET(state) {
    return { ...state, count: 0 };
  },
};

function reducer(state = initialState, action) {
  const handler = handlers[action.type];
  return handler ? handler(state, action) : state;
}

console.log(reducer(undefined, { type: "INCREMENT" })); // { count: 1 }
console.log(reducer({ count: 5 }, { type: "DECREMENT" })); // { count: 4 }
console.log(reducer({ count: 10 }, { type: "RESET" })); // { count: 0 }
```

**Expected Output:**
```
{ count: 1 }
{ count: 4 }
{ count: 0 }
```

**Why This Output:** Shorthand methods define handlers; bracket notation looks them up dynamically.

#### Example 2: Dynamic Form State

```js
function handleInput(state, event) {
  return {
    ...state,
    [event.target.name]: event.target.value,
  };
}

const state = { username: "", email: "" };
const newState = handleInput(state, {
  target: { name: "username", value: "alice" },
});

console.log(newState);
// { username: "alice", email: "" }
```

**Expected Output:**
```
{ username: 'alice', email: '' }
```

**Why This Output:** The computed property updates the correct field dynamically.

#### Example 3: Combining Shorthand and Computed Names

```js
function createUser(name, role, id) {
  return {
    name,
    role,
    [`user_${id}`]: true,
    describe() {
      return `${this.name} (${this.role})`;
    },
  };
}

const user = createUser("Alice", "admin", 42);
console.log(user);
// { name: "Alice", role: "admin", user_42: true, describe: [Function: describe] }
console.log(user.describe()); // "Alice (admin)"
```

**Expected Output:**
```
{ name: 'Alice', role: 'admin', user_42: true, describe: [Function: describe] }
Alice (admin)
```

**Why This Output:** All three features combine in a single object literal.

### Real-World Cases

- **React:** `const [state, setState] = useState({ name, age });`.
- **Redux:** Reducers with dynamic action handlers.
- **Vue:** `data() { return { name, age }; }`.
- **Node.js:** Module exports with shorthand.
- **Libraries:** Well-known symbol implementations.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Object Literals — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-object-initializer
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Method Definitions — https://tc39.es/ecma262/multipage/ecmascript-language-functions-and-classes.html#sec-method-definitions
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Computed Property Names — https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-object-initializer-runtime-semantics-propertydefinitionevaluation
- MDN Web Docs – Object initializer — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer
- MDN Web Docs – Shorthand property names — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer#property_definitions
- MDN Web Docs – Shorthand method names — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer#method_definitions
- MDN Web Docs – Computed property names — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer#computed_property_names
- MDN Web Docs – Symbol — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol
- MDN Web Docs – Symbol.iterator — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/iterator
- Redux Docs – Reducing Boilerplate — https://redux.js.org/usage/reducing-boilerplate
- Vue.js Docs – Data and Methods — https://vuejs.org/guide/essentials/reactivity-fundamentals.html
- React Docs – useState — https://react.dev/reference/react/useState
- TC39 Proposal – Object Initializer Enhancements (ES2015) — https://github.com/tc39/ecma262
- Exploring JS – Object literals — https://exploringjs.com/impatient-js/ch_objects.html
- You Don't Know JS – Object Literals — https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/objects-classes/ch1.md