# Parameters and Arguments — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Parameters are the named variables declared in a function's definition, while arguments are the actual values passed to the function when it is called. Together, they form the input mechanism through which data flows into functions.

**Technical Definition:** In ECMAScript, `FormalParameters` are part of a function's syntax and are bound in the function's scope upon invocation. They are initialised to the values of the arguments supplied by the caller in a `CallExpression`'s `Arguments` production. The specification supports several parameter types: simple positional parameters, default parameters (ES2015+), rest parameters (ES2015+), and destructured parameters (ES2015+). Arguments are always passed by value; for objects, the value is a reference to the object, meaning mutations to the object's properties are visible to the caller, but reassignment of the parameter itself is not.

**Beginner-Friendly Explanation:** Think of a function as a recipe and its parameters as the ingredient slots listed on the recipe card. When you actually cook (call the function), you put real ingredients (arguments) into those slots. If you forget an ingredient, the recipe might use a default substitute (default parameter). If you have extra ingredients, you can collect them all in a basket (rest parameter). And if your recipe takes a box of assorted items, you can unpack it right at the counter (destructured parameter).

### Key Characteristics

- **Positional binding:** JavaScript parameters are positional — the first argument fills the first parameter, the second fills the second, and so on.
- **Pass-by-value semantics:** Primitive arguments are copied; object arguments pass a copy of the reference.
- **ES2015+ features:** Default, rest, and destructured parameters were introduced in ES2015 (ES6) and are now widely available across all modern browsers.
- **Dedicated parameter scope:** When default, rest, or destructured parameters are present, a separate scope is created for parameters, distinct from the function body's scope.
- **Temporal Dead Zone (TDZ):** Parameters with default values are subject to the TDZ; accessing a later parameter in an earlier parameter's default initialiser throws a `ReferenceError`.
- **`arguments` object:** Available in all non-arrow functions, providing array-like access to all passed arguments, though rest parameters are the modern preferred alternative.

### Prerequisites

- Basic understanding of JavaScript functions (declarations, expressions, arrow functions).
- Familiarity with variable scope and hoisting.
- Awareness of array and object data structures.

### Related Programming Areas

- **Functional programming:** Higher-order functions, currying, partial application.
- **API design:** Designing clean function signatures and option objects.
- **Data processing:** Handling variable-length inputs and configuration objects.
- **Event handling:** Passing callback arguments and event data.

### Core Concepts / Features

---

## Positional Parameters

### Definitions

**Core Definition:** Positional parameters are the standard form of function parameters where arguments are assigned to parameters based solely on their order in the function call.

**Technical Definition:** In ECMAScript, when a function is called, the `Arguments` list is evaluated and bound to the `FormalParameters` in left-to-right order. The first argument is assigned to the first parameter, the second to the second, and so on. If fewer arguments are supplied than there are parameters, the remaining parameters are initialised to `undefined`. If more arguments are supplied than parameters, the excess arguments are accessible via the `arguments` object or rest parameters but have no named binding.

**Beginner-Friendly Explanation:** Positional parameters mean that the position of the argument matters. If your function is `function greet(firstName, lastName)`, then `greet("John", "Doe")` assigns `"John"` to `firstName` and `"Doe"` to `lastName`. If you swap them, you get the wrong result. JavaScript doesn't have named parameters like some other languages — the order is everything.

### Purposes

- To define the required inputs of a function in a simple, direct manner.
- To enable straightforward function signatures for small, focused functions.
- To support JavaScript's fundamental function-calling mechanism without additional syntax overhead.
- To allow the engine to optimise function calls based on predictable argument positioning.

### Syntax Rules and Structure

```js
function functionName(param1, param2, param3) {
  // param1 receives the first argument
  // param2 receives the second argument
  // param3 receives the third argument
}
```

**Component Breakdown:**
- `param1, param2, param3`: Comma-separated identifiers in the parameter list.
- Each identifier becomes a local variable in the function's scope.
- No type annotations or default values are required (though they may be added).

**Syntax Rules:**
- Parameters are separated by commas.
- Trailing commas in the parameter list are allowed in ES2017+.
- If an argument is omitted, the corresponding parameter is `undefined`.
- Parameters cannot have the same name in strict mode (a `SyntaxError` is thrown).

**Constraints and Limitations:**
- JavaScript does not support named parameters or keyword arguments natively.
- Functions with many positional parameters become difficult to call correctly; an options object is often preferred for functions with more than two or three parameters.
- Positional parameters provide no self-documenting information at the call site.

### Annotated Code Examples

#### Example 1: Basic Positional Parameter Assignment

```js
// Step 1: Define a function with three positional parameters.
function describePet(species, name, age) {
  return `${name} is a ${age}-year-old ${species}.`;
}

// Step 2: Call with arguments in the correct order.
console.log(describePet("dog", "Rex", 5));
// Output: "Rex is a 5-year-old dog."

// Step 3: Call with swapped arguments (produces incorrect result).
console.log(describePet("Rex", "dog", 5));
// Output: "dog is a 5-year-old Rex."
```

**Expected Output:**
```
Rex is a 5-year-old dog.
dog is a 5-year-old Rex.
```

**Why This Output:** The function assigns arguments strictly by position. In the first call, `"dog"` fills `species`, `"Rex"` fills `name`, and `5` fills `age`. In the second call, `"Rex"` fills `species` and `"dog"` fills `name`, producing a semantically incorrect but syntactically valid result.

#### Example 2: Missing Arguments Become `undefined`

```js
// Step 1: Define a function with two parameters.
function formatName(first, last) {
  return `${first} ${last}`;
}

// Step 2: Call with both arguments.
console.log(formatName("Alice", "Smith")); // "Alice Smith"

// Step 3: Call with only one argument.
console.log(formatName("Alice")); // "Alice undefined"

// Step 4: Call with no arguments.
console.log(formatName()); // "undefined undefined"
```

**Expected Output:**
```
Alice Smith
Alice undefined
undefined undefined
```

**Why This Output:** When an argument is not provided, the corresponding parameter is `undefined`. Template literal interpolation converts `undefined` to the string `"undefined"`, which is why the output shows `"Alice undefined"` rather than an error.

### Real-World Cases

- **Math utilities:** `function add(a, b) { return a + b; }` — simple, predictable positional parameters.
- **DOM manipulation:** `function setStyle(element, property, value) { ... }` — clear positional roles.
- **Array methods:** `array.map((element, index, array) => { ... })` — the callback receives arguments in a defined positional order.

---

## Default Parameters

### Definitions

**Core Definition:** Default parameters allow named parameters to be initialised with default values if no value or `undefined` is passed for that parameter.

**Technical Definition:** A `DefaultParameter` is a `BindingElement` that includes an `Initializer` expression. When the function is called, if the argument for that parameter is `undefined` (either because it was omitted or explicitly passed as `undefined`), the `Initializer` expression is evaluated and its result is used as the parameter's value. The default initialiser is evaluated at call time in the parameter scope, which is a separate scope from the function body. Default parameters were introduced in ES2015.

**Beginner-Friendly Explanation:** A default parameter is a fallback value for when the caller doesn't provide an argument. Instead of writing `if (name === undefined) name = "Guest";` inside the function, you write `function greet(name = "Guest")`. If no name is passed, `"Guest"` is used automatically.

### Purposes

- To provide sensible fallback values for optional parameters.
- To eliminate boilerplate `undefined` checks inside function bodies.
- To make function signatures more self-documenting and expressive.
- To support optional configuration without requiring callers to pass every argument.

### Syntax Rules and Structure

```js
function functionName(param1 = defaultValue1, param2 = defaultValue2) {
  // function body
}
```

**Component Breakdown:**
- `param1 = defaultValue1`: The parameter and its default expression.
- The default expression is evaluated only when the argument is `undefined`.
- The default expression can be any valid JavaScript expression (except `await` or `yield`).

**Syntax Rules:**
- Parameters are still set left-to-right; a later parameter can reference an earlier parameter in its default value.
- A default parameter and all parameters after it do not contribute to the function's `length` property.
- The default initialiser lives in its own scope, which is a parent of the function body scope.
- `var`-declared variables in the function body are not visible to default parameter initialisers.

**Constraints and Limitations:**
- `null` does not trigger the default value; only `undefined` does. Passing `null` explicitly sets the parameter to `null`.
- Default initialisers cannot contain `await` or `yield` if they would pause evaluation.
- Functions and variables declared in the function body cannot be referenced from default parameter initialisers; doing so throws a `ReferenceError`.

### Annotated Code Examples

#### Example 1: Basic Default Parameter

```js
// Step 1: Define a function with a default parameter.
function greet(name = "Guest") {
  return `Hello, ${name}!`;
}

// Step 2: Call with an argument.
console.log(greet("Alice")); // "Hello, Alice!"

// Step 3: Call without an argument (default is used).
console.log(greet()); // "Hello, Guest!"

// Step 4: Call with undefined (default is used).
console.log(greet(undefined)); // "Hello, Guest!"

// Step 5: Call with null (default is NOT used).
console.log(greet(null)); // "Hello, null!"
```

**Expected Output:**
```
Hello, Alice!
Hello, Guest!
Hello, Guest!
Hello, null!
```

**Why This Output:** The default `"Guest"` is applied when the argument is omitted or explicitly `undefined`. Passing `null` does not trigger the default because `null` is a defined value, not `undefined`.

#### Example 2: Default Referencing an Earlier Parameter

```js
// Step 1: Define a function where a later parameter references an earlier one.
function createRectangle(width, height = width) {
  return { width, height, area: width * height };
}

// Step 2: Call with both dimensions.
console.log(createRectangle(5, 10));
// { width: 5, height: 10, area: 50 }

// Step 3: Call with only width (height defaults to width).
console.log(createRectangle(5));
// { width: 5, height: 5, area: 25 }
```

**Expected Output:**
```
{ width: 5, height: 10, area: 50 }
{ width: 5, height: 5, area: 25 }
```

**Why This Output:** When `height` is not provided, its default expression `width` is evaluated in the parameter scope. Since `width` has already been initialised to `5`, `height` becomes `5`, creating a square. This works because parameters are set left-to-right.

### Real-World Cases

- **Configuration objects:** `function createServer(port = 3000, host = "localhost")`.
- **API clients:** `function fetchData(url, method = "GET", timeout = 5000)`.
- **Utility functions:** `function debounce(fn, delay = 300)`.

---

## Rest Parameters

### Definitions

**Core Definition:** The rest parameter syntax allows a function to accept an indefinite number of arguments as an array, providing a way to represent variadic functions in JavaScript.

**Technical Definition:** A `RestParameter` is a `FormalParameter` prefixed with `...` (three U+002E FULL STOP characters). When a function with a rest parameter is called, all arguments that do not have a corresponding named parameter are collected into a new `Array` instance bound to the rest parameter. A function definition can have only one rest parameter, and it must be the last parameter in the parameter list. Trailing commas are not allowed after the rest parameter, and it cannot have a default value.

**Beginner-Friendly Explanation:** A rest parameter is like a catch-all basket for extra arguments. If your function is `function sum(...numbers)`, then `sum(1, 2, 3, 4)` puts all four numbers into an array called `numbers`. You can then loop through them, sum them, or do whatever you need. The rest parameter collects everything that doesn't have a named slot.

### Purposes

- To create variadic functions that accept any number of arguments.
- To replace the legacy `arguments` object with a true array.
- To collect trailing arguments without declaring each one individually.
- To enable destructuring of remaining arguments in parameter lists.

### Syntax Rules and Structure

```js
function functionName(param1, ...restParam) {
  // restParam is an array containing all remaining arguments
}
```

**Component Breakdown:**
- `...restParam`: The rest parameter, always the last in the list.
- `restParam` is a real `Array` instance, so array methods (`map`, `filter`, `reduce`, etc.) can be used directly on it.

**Syntax Rules:**
- Only one rest parameter is allowed per function.
- The rest parameter must be the last parameter.
- Trailing commas are not allowed after the rest parameter.
- The rest parameter cannot have a default value.
- The rest parameter does not contribute to the function's `length` property.

**Constraints and Limitations:**
- The rest parameter bundles only arguments that do not have a corresponding named parameter. Named parameters before the rest are not included in the array.
- The rest parameter array is a new array each time the function is called; it is not a live view of the arguments.

### Annotated Code Examples

#### Example 1: Basic Rest Parameter

```js
// Step 1: Define a function with a rest parameter.
function sum(...numbers) {
  // Step 2: numbers is a real array.
  return numbers.reduce((total, n) => total + n, 0);
}

// Step 3: Call with varying numbers of arguments.
console.log(sum(1, 2, 3));    // 6
console.log(sum(10, 20));     // 30
console.log(sum());           // 0
console.log(sum(1, 2, 3, 4, 5)); // 15
```

**Expected Output:**
```
6
30
0
15
```

**Why This Output:** The rest parameter collects all arguments into an array. `reduce` sums the array starting from `0`. With no arguments, the array is empty and the result is `0`. The function works with any number of arguments.

#### Example 2: Rest Parameter with Named Parameters

```js
// Step 1: Define a function with a named parameter and a rest parameter.
function logWithPrefix(prefix, ...messages) {
  messages.forEach((msg) => console.log(`[${prefix}] ${msg}`));
}

// Step 2: Call with multiple arguments.
logWithPrefix("INFO", "Server started", "Listening on port 3000");
// [INFO] Server started
// [INFO] Listening on port 3000

// Step 3: Call with only the prefix.
logWithPrefix("WARN");
// (no output — messages is an empty array)
```

**Expected Output:**
```
[INFO] Server started
[INFO] Listening on port 3000
```

**Why This Output:** The first argument `"INFO"` fills the `prefix` parameter. The rest parameter `messages` collects the remaining arguments into an array. When only the prefix is passed, `messages` is an empty array, so `forEach` does nothing.

### Real-World Cases

- **Logging utilities:** `function log(...messages) { console.log(...messages); }`.
- **Math functions:** `function max(...numbers) { return Math.max(...numbers); }`.
- **Event emitters:** `emit(eventName, ...args)` — passing variable arguments to listeners.
- **Array construction:** `function concat(...arrays) { return [].concat(...arrays); }`.

---

## Destructured Parameters

### Definitions

**Core Definition:** Destructured parameters allow a function to unpack values from arrays or properties from objects directly in the parameter list, using JavaScript's destructuring syntax.

**Technical Definition:** A `DestructuringBindingPattern` in a parameter position uses the same syntax as destructuring assignment. For object destructuring, the parameter is written as `{ property1, property2 }`, which extracts those properties from the passed object into local variables. For array destructuring, the parameter is written as `[element1, element2]`, which extracts elements from the passed array. Default values can be combined with destructuring for both the entire parameter and individual properties.

**Beginner-Friendly Explanation:** Instead of writing `function greet(user) { console.log(user.name); console.log(user.age); }`, you can write `function greet({ name, age }) { console.log(name); console.log(age); }`. The function automatically unpacks the object properties into separate variables. It's like opening a package at the front door instead of carrying the whole box inside.

### Purposes

- To extract only the needed properties from an object argument.
- To make function signatures self-documenting by listing expected properties.
- To simplify code that would otherwise require multiple property accesses.
- To provide default values for individual properties within destructured parameters.

### Syntax Rules and Structure

```js
// Object destructuring:
function functionName({ prop1, prop2, prop3 = defaultValue }) {
  // prop1, prop2, prop3 are local variables
}

// Array destructuring:
function functionName([first, second, third]) {
  // first, second, third are local variables
}

// With default for the entire parameter:
function functionName({ prop1, prop2 } = {}) {
  // Handles the case where no argument is passed
}
```

**Component Breakdown:**
- `{ prop1, prop2 }`: Object destructuring pattern in the parameter list.
- `= defaultValue`: Default value for an individual property or the entire parameter.
- `[first, second]`: Array destructuring pattern.

**Syntax Rules:**
- The entire destructured parameter can have a default value: `function f({ a, b } = {}) {}`. This prevents a `TypeError` when no argument is passed.
- Individual properties can have default values: `function f({ a = 1, b = 2 }) {}`.
- Destructured parameters are evaluated left-to-right in the parameter scope.

**Constraints and Limitations:**
- If no argument is passed and no default is provided for the entire destructured parameter, a `TypeError` is thrown because you cannot destructure `undefined`.
- Destructuring a large object extracts only the listed properties; the original object reference is not retained unless you also capture it (e.g., using a rest property).

### Annotated Code Examples

#### Example 1: Object Destructuring with Defaults

```js
// Step 1: Define a function with a destructured object parameter.
function createUser({ name = "Anonymous", age = 18, role = "viewer" } = {}) {
  return { name, age, role };
}

// Step 2: Call with a full object.
console.log(createUser({ name: "Alice", age: 30, role: "admin" }));
// { name: 'Alice', age: 30, role: 'admin' }

// Step 3: Call with partial object.
console.log(createUser({ name: "Bob" }));
// { name: 'Bob', age: 18, role: 'viewer' }

// Step 4: Call with no arguments.
console.log(createUser());
// { name: 'Anonymous', age: 18, role: 'viewer' }
```

**Expected Output:**
```
{ name: 'Alice', age: 30, role: 'admin' }
{ name: 'Bob', age: 18, role: 'viewer' }
{ name: 'Anonymous', age: 18, role: 'viewer' }
```

**Why This Output:** The `= {}` default for the entire parameter ensures that calling `createUser()` does not throw. Individual property defaults fill in missing values. The function extracts only the properties it needs, ignoring any extra properties in the passed object.

#### Example 2: Array Destructuring and Nested Destructuring

```js
// Step 1: Array destructuring in parameters.
function getFirstTwo([first, second]) {
  return { first, second };
}
console.log(getFirstTwo([10, 20, 30]));
// { first: 10, second: 20 }

// Step 2: Nested object destructuring.
function displayUser({ name, address: { city, country = "Unknown" } }) {
  return `${name} lives in ${city}, ${country}.`;
}
console.log(displayUser({
  name: "Alice",
  address: { city: "Paris", country: "France" }
}));
// "Alice lives in Paris, France."

// Step 3: Missing nested property with default.
console.log(displayUser({
  name: "Bob",
  address: { city: "London" }
}));
// "Bob lives in London, Unknown."
```

**Expected Output:**
```
{ first: 10, second: 20 }
Alice lives in Paris, France.
Bob lives in London, Unknown.
```

**Why This Output:** Array destructuring extracts elements by position. Nested destructuring drills into the `address` property of the user object. The default `"Unknown"` is applied when `country` is missing from the `address` object.

### Real-World Cases

- **React props:** `function Button({ label, onClick, variant = "primary" }) { ... }`.
- **API request handlers:** `function handleRequest({ method, url, body, headers = {} }) { ... }`.
- **Configuration loading:** `function createConnection({ host = "localhost", port = 5432, database }) { ... }`.

---

## Arguments Object

### Definitions

**Core Definition:** The `arguments` object is an array-like object accessible inside non-arrow functions that contains the values of all arguments passed to that function.

**Technical Definition:** The `arguments` object is a local variable available in all non-arrow functions. It has a `length` property indicating the number of arguments passed, and indexed properties (`arguments[0]`, `arguments[1]`, etc.) for each argument. In non-strict functions with simple parameters, the `arguments` object is *linked* to the named parameters: changing `arguments[0]` also changes the first parameter, and vice versa. However, if the function has rest, default, or destructured parameters, this linkage is broken, and the `arguments` object always reflects the values passed at call time, regardless of later reassignments. In strict mode, the `arguments` object is not linked to the parameters at all. Modern code should prefer rest parameters over the `arguments` object.

**Beginner-Friendly Explanation:** The `arguments` object is a built-in list inside every regular function that holds everything the caller passed in. It's like a receipt listing all the items someone gave you. You can check how many items there are with `arguments.length`, and access each one with `arguments[0]`, `arguments[1]`, and so on. But it's not a real array — it's "array-like" — so you can't use `.map()` or `.filter()` directly on it. Most developers today use rest parameters (`...args`) instead.

### Purposes

- To access all arguments passed to a function when the number is not known in advance.
- To inspect the number of arguments passed via `arguments.length`.
- To support legacy variadic functions (though rest parameters are now preferred).
- To allow arguments to be accessed or reassigned by index.

### Syntax Rules and Structure

```js
function functionName() {
  // arguments is available automatically
  console.log(arguments.length);    // number of arguments passed
  console.log(arguments[0]);        // first argument
  console.log(arguments[1]);        // second argument
}
```

**Component Breakdown:**
- `arguments`: An array-like object available in non-arrow functions.
- `arguments.length`: The number of arguments passed.
- `arguments[i]`: The argument at index `i`.

**Syntax Rules:**
- The `arguments` object is available in all non-arrow functions.
- It is not a real array; it lacks array methods like `map`, `filter`, `reduce`, etc.
- In non-strict functions with simple parameters, `arguments` is linked to the named parameters.
- In strict mode, or with rest/default/destructured parameters, the linkage is broken.
- Arrow functions do not have their own `arguments` object; they inherit it from the enclosing scope.

**Constraints and Limitations:**
- The `arguments` object is not a true `Array`; use `Array.from(arguments)` or `[...arguments]` to convert it.
- The `callee` property is deprecated and throws in strict mode.
- Modern code should use rest parameters instead of `arguments` for better performance and array-like behaviour.

### Annotated Code Examples

#### Example 1: Accessing Arguments by Index

```js
// Step 1: Define a function with no named parameters.
function listArguments() {
  // Step 2: arguments contains all passed values.
  console.log("Count:", arguments.length);
  for (let i = 0; i < arguments.length; i++) {
    console.log(`arguments[${i}] =`, arguments[i]);
  }
}

// Step 3: Call with various arguments.
listArguments("apple", "banana", "cherry");
```

**Expected Output:**
```
Count: 3
arguments[0] = apple
arguments[1] = banana
arguments[2] = cherry
```

**Why This Output:** The `arguments` object captures all passed values, even though the function declares no parameters. `arguments.length` is `3`, and the loop iterates over each argument.

#### Example 2: Linkage Between `arguments` and Parameters (Non-Strict)

```js
// Step 1: Non-strict function with simple parameters.
function linked(a) {
  // Step 2: Modify arguments[0].
  arguments[0] = 99;
  // Step 3: The parameter a is also updated.
  console.log("a:", a); // 99
}

function linked2(a) {
  // Step 4: Modify the parameter a.
  a = 99;
  // Step 5: arguments[0] is also updated.
  console.log("arguments[0]:", arguments[0]); // 99
}

linked(10);
linked2(10);
```

**Expected Output:**
```
a: 99
arguments[0]: 99
```

**Why This Output:** In non-strict functions with simple parameters, the `arguments` object and the named parameters share their values bidirectionally. Modifying one updates the other. This linkage does not exist in strict mode or when complex parameters are used.

#### Example 3: Linkage Broken with Default Parameters

```js
// Step 1: Function with a default parameter.
function withDefault(a = 55) {
  // Step 2: Modify arguments[0].
  arguments[0] = 99;
  // Step 3: The parameter a is NOT updated.
  console.log("a:", a); // 10 (the value passed, not 99)
}

withDefault(10);
```

**Expected Output:**
```
a: 10
```

**Why This Output:** When a function has default (or rest, or destructured) parameters, the `arguments` object is no longer linked to the named parameters. Modifying `arguments[0]` does not affect `a`.

### Real-World Cases

- **Legacy libraries:** Older codebases use `arguments` extensively for variadic functions; understanding it is necessary for maintenance.
- **Polyfills:** Some polyfills use `arguments` to forward all arguments to native methods.
- **Debugging:** Inspecting `arguments` in the console to see exactly what a function received.

---

## Spread Arguments

### Definitions

**Core Definition:** Spread syntax (`...`) in a function call expands an iterable (such as an array or string) into individual arguments.

**Technical Definition:** The spread syntax in a function call's argument list (`fn(...iterable)`) causes the iterable to be expanded, with each element passed as a separate argument. Any argument in the argument list can use spread syntax, and it can be used multiple times in the same call. Only iterable values can be spread in argument lists; non-iterable objects (like plain objects) cannot. Spread syntax looks identical to rest syntax but performs the opposite operation: spread *expands* an array into elements, while rest *collects* elements into an array.

**Beginner-Friendly Explanation:** Spread syntax is like taking a box of items and dumping them all out onto a table so each item can be used individually. If you have an array `[1, 2, 3]` and a function `add(a, b, c)`, you can call `add(...[1, 2, 3])` instead of `add(1, 2, 3)`. It "spreads" the array into separate arguments.

### Purposes

- To pass an array as individual arguments to a function.
- To combine multiple arrays or values into a single function call.
- To replace the legacy `Function.prototype.apply()` method for argument spreading.
- To forward arguments from one function to another.

### Syntax Rules and Structure

```js
functionName(...iterableObject);
functionName(a, ...iterableObject, b);
functionName(...array1, ...array2);
```

**Component Breakdown:**
- `...iterableObject`: The spread element, which must be an iterable (array, string, Set, Map, etc.).
- Spread can appear at any position in the argument list.
- Multiple spreads are allowed in a single call.

**Syntax Rules:**
- The spread operand must be iterable. Plain objects (without `Symbol.iterator`) cannot be spread in argument lists.
- Spread syntax can be used in function calls, array literals, and object literals (for object literals, the semantics differ slightly).
- Spread syntax in argument lists is evaluated left-to-right.

**Constraints and Limitations:**
- Spreading a very large array can cause a `RangeError` due to exceeding the maximum number of arguments (engine-dependent, typically around 65,535 in V8).
- Plain objects cannot be spread in argument lists; use `Object.values()` or rest parameters with destructuring instead.

### Annotated Code Examples

#### Example 1: Spreading an Array into Function Arguments

```js
// Step 1: Define a function with three parameters.
function add(a, b, c) {
  return a + b + c;
}

// Step 2: Create an array of values.
const numbers = [10, 20, 30];

// Step 3: Spread the array into the function call.
console.log(add(...numbers)); // 60

// Step 4: Compare with apply() (legacy approach).
console.log(add.apply(null, numbers)); // 60
```

**Expected Output:**
```
60
60
```

**Why This Output:** The spread operator expands `[10, 20, 30]` into three separate arguments: `10`, `20`, and `30`. This is equivalent to calling `add(10, 20, 30)`. The `apply()` method achieves the same result but with more verbose syntax.

#### Example 2: Combining Spread with Regular Arguments

```js
// Step 1: Define a function with four parameters.
function introduce(greeting, firstName, lastName, title) {
  return `${greeting}, ${title} ${firstName} ${lastName}!`;
}

// Step 2: Create an array for some arguments.
const nameParts = ["John", "Doe"];

// Step 3: Spread the array in the middle of the argument list.
console.log(introduce("Hello", ...nameParts, "Dr."));
// "Hello, Dr. John Doe!"

// Step 4: Spread multiple iterables.
const part1 = ["Hello"];
const part2 = ["Jane", "Smith"];
console.log(introduce(...part1, ...part2, "Prof."));
// "Hello, Prof. Jane Smith!"
```

**Expected Output:**
```
Hello, Dr. John Doe!
Hello, Prof. Jane Smith!
```

**Why This Output:** Spread can appear at any position and multiple times. In the first call, `"Hello"` fills `greeting`, the spread fills `firstName` and `lastName`, and `"Dr."` fills `title`. In the second call, two spreads combine to fill the first three parameters.

### Real-World Cases

- **Math functions:** `Math.max(...arrayOfNumbers)` — finding the maximum value in an array.
- **Array concatenation:** `[...array1, ...array2, extraItem]` — creating a new combined array.
- **Function forwarding:** `function wrapper(...args) { return original(...args); }` — passing all arguments through.
- **React props spreading:** `<Component {...props} />` — passing an object's properties as individual props.

---

## Parameter Scope

### Definitions

**Core Definition:** Parameter scope is the lexical environment in which function parameters are defined and evaluated, which is separate from the function body's scope when default, rest, or destructured parameters are used.

**Technical Definition:** Per ECMAScript §9.2.12 (FunctionDeclarationInstantiation), when a function's formal parameters contain any default initializers (or are otherwise complex), a separate `Environment Record` is created for the parameters. This parameter environment is a parent of the function body's environment. The specification states that this separate environment is needed to ensure that closures created by expressions in the formal parameter list do not have visibility of declarations in the function body. If the function's formal parameters do not include any default initializers, the body declarations are instantiated in the same `Environment Record` as the parameters.

**Beginner-Friendly Explanation:** When a function has default values or other complex parameters, JavaScript creates a special "parameter area" that is separate from the main function body. This means that code in the parameter list cannot see variables declared inside the function body. It also means that earlier parameters can be used in later parameters' default values, but later parameters cannot be used in earlier ones.

### Purposes

- To ensure that parameter initialisers are evaluated in a predictable, isolated environment.
- To prevent default parameter expressions from accidentally accessing function body variables.
- To allow earlier parameters to be referenced in later parameters' default values.
- To support closures over parameter values without exposing body declarations.

### Syntax Rules and Structure

Parameter scope is not a syntactic construct but a runtime behaviour. The relevant scenario is:

```js
function example(a, b = a + 1, c = b * 2) {
  // a, b, c are in the parameter scope
  // function body has its own scope
  var bodyVar = "body";
}
```

**Component Breakdown:**
- `a`, `b`, `c`: Parameters bound in the parameter environment record.
- `b = a + 1`: Default initialiser that can access `a` (earlier parameter).
- `c = b * 2`: Default initialiser that can access `b` (earlier parameter).
- `bodyVar`: Declared in the function body scope, which is a child of the parameter scope.

**Syntax Rules:**
- Parameters are initialised left-to-right in the parameter scope.
- Later parameters cannot be referenced in earlier parameters' default values (TDZ violation).
- Variables declared in the function body (`var`, `let`, `const`) are not accessible from parameter default initialisers.
- Functions declared in the function body are also not accessible from parameter initialisers.

**Constraints and Limitations:**
- Accessing a later parameter in an earlier parameter's default throws a `ReferenceError` due to the Temporal Dead Zone.
- Closures created in the parameter list cannot capture function body declarations, which can lead to unexpected behaviour if not understood.

### Annotated Code Examples

#### Example 1: Parameter Scope Isolation

```js
// Step 1: Define a function with a default parameter.
function testScope(a = bodyVar) {
  var bodyVar = "body";
  return a;
}

// Step 2: Calling throws a ReferenceError.
try {
  testScope();
} catch (err) {
  console.log(err.name); // "ReferenceError"
  console.log(err.message); // "bodyVar is not defined"
}
```

**Expected Output:**
```
ReferenceError
bodyVar is not defined
```

**Why This Output:** The default parameter `a = bodyVar` attempts to access `bodyVar`, which is declared inside the function body. Because the parameter scope is a parent of the body scope, `bodyVar` is not visible from the parameter scope. The `var` declaration is hoisted to the top of the body scope, not the parameter scope.

#### Example 2: Earlier Parameters Visible in Later Defaults

```js
// Step 1: Define a function with sequential default parameters.
function createRange(start, end = start + 10, step = end > start ? 1 : -1) {
  return { start, end, step };
}

// Step 2: Call with only start.
console.log(createRange(5));
// { start: 5, end: 15, step: 1 }

// Step 3: Call with start and end.
console.log(createRange(100, 50));
// { start: 100, end: 50, step: -1 }
```

**Expected Output:**
```
{ start: 5, end: 15, step: 1 }
{ start: 100, end: 50, step: -1 }
```

**Why This Output:** The default for `end` can reference `start` because `start` is initialised first (left-to-right). The default for `step` can reference both `start` and `end` because both have been initialised by the time `step` is evaluated.

### Real-World Cases

- **Currying and partial application:** Using parameter defaults that depend on earlier parameters.
- **Configuration builders:** Functions where later settings depend on earlier ones.
- **Framework internals:** React and other libraries rely on parameter scope semantics for their APIs.

---

## Parameter Defaults (Continued)

This section has been covered comprehensively under "Default Parameters." The key additional details regarding parameter defaults are:

- **Evaluation timing:** Default parameters are evaluated at call time, not at function definition time. This means that a new value is computed for each call if the default expression involves dynamic values such as `Date.now()` or object creation.
- **Scope:** Default parameter initialisers live in their own scope, which is a parent of the function body scope.
- **TDZ:** Parameters are subject to the Temporal Dead Zone; accessing a parameter before it is initialised throws a `ReferenceError`.

### Annotated Code Example: Evaluation at Call Time

```js
// Step 1: Define a function with a default that creates a new object each call.
function createItem(id, metadata = { createdAt: new Date() }) {
  return { id, metadata };
}

// Step 2: Each call creates a new metadata object.
const item1 = createItem(1);
const item2 = createItem(2);

console.log(item1.metadata === item2.metadata); // false
console.log(item1.metadata.createdAt instanceof Date); // true
```

**Expected Output:**
```
false
true
```

**Why This Output:** The default expression `{ createdAt: new Date() }` is evaluated each time the function is called with `undefined` for `metadata`. Each call creates a new object with a new timestamp. This demonstrates late binding — the default is not a single object created once at definition time.

---

## Parameter Evaluation

### Definitions

**Core Definition:** Parameter evaluation is the process by which JavaScript initialises function parameters with the values of the arguments passed during a function call.

**Technical Definition:** Per ECMAScript §9.2.12 (FunctionDeclarationInstantiation), when a function is called, the following steps occur: (1) the `FunctionDeclarationInstantiation` algorithm is invoked; (2) if the function has parameter expressions (default, rest, or destructured parameters), a new `Environment Record` is created for the parameters; (3) parameters are bound in left-to-right order; (4) for each parameter with an initializer, if the corresponding argument is `undefined`, the initializer is evaluated in the parameter environment; (5) after parameter initialisation, the function body's declarations are instantiated in either the same environment (if no parameter expressions) or a child environment (if parameter expressions exist).

**Beginner-Friendly Explanation:** Parameter evaluation is the process JavaScript goes through every time you call a function. It lines up your arguments with the parameters, fills in default values where needed, and makes sure everything is ready before the function body runs. It happens left to right, so if a later parameter needs an earlier one, that's fine — but not the other way around.

### Purposes

- To bind incoming argument values to named parameters.
- To evaluate default expressions only when necessary (when the argument is `undefined`).
- To establish the parameter scope before the function body executes.
- To detect and report TDZ violations when parameters are accessed out of order.

### Syntax Rules and Structure

Parameter evaluation follows this order:

```js
// Conceptual evaluation order:
// 1. Create parameter environment (if needed)
// 2. Evaluate arg1 → bind to param1
// 3. Evaluate arg2 → bind to param2 (or evaluate default if undefined)
// 4. Evaluate arg3 → bind to param3 (or evaluate default if undefined)
// ... and so on
// 5. Create function body environment
// 6. Instantiate body declarations
// 7. Execute body statements
```

**Component Breakdown:**
- **Argument evaluation:** The arguments in the call expression are evaluated left-to-right before the function is invoked.
- **Parameter binding:** Each parameter is bound to the corresponding argument value.
- **Default evaluation:** If an argument is `undefined` and the parameter has a default, the default expression is evaluated in the parameter scope.
- **Body environment creation:** After all parameters are initialised, the function body's environment is created (if parameter expressions exist) or reused.

**Syntax Rules:**
- Arguments are evaluated eagerly (left-to-right) before the function body executes.
- Default initialisers are evaluated conditionally — only when the argument is `undefined`.
- Parameters are initialised left-to-right, so earlier parameters are available to later ones.

**Constraints and Limitations:**
- Later parameters are in the TDZ when earlier parameters' defaults are evaluated.
- `await` and `yield` cannot be used in default parameter initialisers if they would pause evaluation.
- The evaluation order means that side effects in default expressions occur at call time, not definition time.

### Annotated Code Examples

#### Example 1: Left-to-Right Evaluation

```js
// Step 1: Define a function with defaults that reference earlier parameters.
function calculate(a, b = a * 2, c = b + 1) {
  return { a, b, c };
}

// Step 2: Call with only 'a'.
console.log(calculate(5));
// { a: 5, b: 10, c: 11 }

// Step 3: Call with 'a' and 'b'.
console.log(calculate(3, 7));
// { a: 3, b: 7, c: 8 }
```

**Expected Output:**
```
{ a: 5, b: 10, c: 11 }
{ a: 3, b: 7, c: 8 }
```

**Why This Output:** In the first call, `a` is `5`. Since `b` is `undefined`, its default `a * 2` evaluates to `10`. Then `c`'s default `b + 1` evaluates to `11`. In the second call, `b` is explicitly `7`, so its default is not evaluated, and `c` becomes `8`.

#### Example 2: Temporal Dead Zone Violation

```js
// Step 1: Define a function where an earlier parameter references a later one.
function broken(a = b, b = 2) {
  return { a, b };
}

// Step 2: Calling throws a ReferenceError.
try {
  broken();
} catch (err) {
  console.log(err.name); // "ReferenceError"
  console.log(err.message); // "Cannot access 'b' before initialization"
}

// Step 3: Providing an argument avoids the TDZ issue.
console.log(broken(1)); // { a: 1, b: 2 }
```

**Expected Output:**
```
ReferenceError
Cannot access 'b' before initialization
{ a: 1, b: 2 }
```

**Why This Output:** When `broken()` is called with no arguments, `a`'s default expression `b` is evaluated before `b` has been initialised. Since `b` is in the TDZ, accessing it throws a `ReferenceError`. When `1` is passed for `a`, the default expression is not evaluated, so the TDZ is never entered.

### Real-World Cases

- **Framework configuration:** Libraries that use default parameters to provide sensible defaults while allowing overrides.
- **API design:** Functions that derive later configuration values from earlier ones (e.g., `timeout = retries * baseDelay`).
- **Functional programming:** Curried functions where later parameters depend on earlier ones.

---

## References

- MDN Web Docs – Default parameters — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters
- MDN Web Docs – Rest parameters — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters
- MDN Web Docs – Destructuring assignment — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring
- MDN Web Docs – The arguments object — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments
- MDN Web Docs – Spread syntax (...) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax
- MDN Web Docs – Functions — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions
- MDN Web Docs – Function: length — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/length
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – §9.2.12 FunctionDeclarationInstantiation — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf
- TC39 – Scope of Default function parameters in javascript (Bugzilla #4419) — https://tc39.es/archives/bugzilla/4419/
- Stack Overflow – Scope of Default function parameters in javascript — https://browse.library.kiwix.org/content/stackoverflow.com_en_all_nopic_2022-07/questions/44896829/scope-of-default-function-parameters-in-javascript
- Stack Overflow – default parameters in ES6 throws error — https://stackoverflow.com/questions/39722854/default-parameters-in-es6-throws-error
- Stack Overflow – Where are arguments positioned in the lexical environment? — https://browse.library.kiwix.org/content/stackoverflow.com_en_all_2023-11/questions/61208843/where-are-arguments-positioned-in-the-lexical-environment