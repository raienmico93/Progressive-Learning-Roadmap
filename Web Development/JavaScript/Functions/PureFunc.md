# Pure Functions — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A pure function is a function that, given the same input, always returns the same output and has no observable side effects. It does not depend on or modify any state outside its own scope.

**Technical Definition:** Per MDN, a pure function is a function that satisfies two conditions: (1) **Determinism** — given the same arguments, it always returns the same result; and (2) **No side effects** — it does not cause any observable changes outside its own scope (no mutation of external state, no I/O, no logging, no network requests). Pure functions are said to be **referentially transparent**: any expression containing a pure function call can be replaced with its computed value without changing the program's behaviour. In ECMAScript terms, a pure function is one whose execution does not depend on or modify the `this` binding, global variables, closures over mutable state, or any external resource.

**Beginner-Friendly Explanation:** A pure function is like a vending machine: you put in the same coins, you get the same snack every time, and nothing else in the world changes. It doesn't matter what time it is, who else is using the machine, or what happened before — the same input always produces the same output. Impure functions, by contrast, are like a coffee shop: the result depends on who's working, what's in stock, and the barista might spill something.

### Key Characteristics

- **Deterministic:** Same input → same output, every time.
- **No side effects:** Does not mutate external state, perform I/O, or interact with the outside world.
- **Referentially transparent:** Can be replaced with its return value without changing program behaviour.
- **Self-contained:** Depends only on its arguments, not on external state.
- **Idempotent (often):** Calling it multiple times with the same input produces the same result without additional effects.
- **Testable:** Easy to test because there are no external dependencies to mock.
- **Composable:** Pure functions compose cleanly because they have no hidden interactions.

### Prerequisites

- Understanding of functions, parameters, and return values.
- Familiarity with scope and closures.
- Basic knowledge of array methods (`map`, `filter`, `reduce`).
- Awareness of mutation and immutability concepts.

### Related Programming Areas

- **Functional programming:** Pure functions are the foundation of FP.
- **Immutability:** Avoiding mutation is closely related to purity.
- **Testing:** Pure functions are the easiest to unit test.
- **Concurrency:** Pure functions are safe to parallelise.
- **State management:** Redux reducers are required to be pure functions.

### Core Concepts / Features

---

## Pure Functions

### Definitions

**Core Definition:** A pure function is a function that always returns the same output for the same input and has no side effects.

**Technical Definition:** Per MDN, "A pure function is a function that satisfies the following two conditions: (1) Deterministic — given the same arguments, it always returns the same result; (2) No side effects — it does not cause any observable changes outside its own scope". In ECMAScript, purity means the function does not read or write to variables outside its own scope, does not use `this`, does not call `Date.now()`, `Math.random()`, or other impure functions, and does not perform I/O.

**Beginner-Friendly Explanation:** A pure function is a function that behaves like a mathematical formula. `add(2, 3)` always returns `5`, no matter when or where you call it. It doesn't change anything else in your program. If a function depends on the current time, a random number, or modifies a global variable, it's not pure.

### Purposes

- To make code predictable and easy to reason about.
- To enable safe testing without mocks or stubs.
- To support caching and memoization (same input → cached output).
- To enable safe parallelisation and concurrency.
- To facilitate function composition and refactoring.
- To reduce bugs caused by hidden state mutations.

### Syntax Rules and Structure

Purity is not a syntactic feature; it is a semantic property. The pattern is:

```js
// Pure function:
function add(a, b) {
  return a + b;
}

// Impure function (mutates external state):
let total = 0;
function addToTotal(n) {
  total += n; // Side effect
  return total;
}
```

**Component Breakdown:**
- **Pure:** Takes arguments, returns a value, touches nothing else.
- **Impure:** Reads or writes external state, performs I/O, or uses non-deterministic sources.

**Syntax Rules:**
- Pure functions should not use `this`, `arguments.callee`, or global variables.
- Pure functions should not call impure functions (e.g., `Math.random()`, `Date.now()`, `console.log`).
- Pure functions should not mutate their arguments.
- Pure functions may use local variables and constants.

**Constraints and Limitations:**
- Real-world applications require side effects (network calls, DOM updates, logging). Purity must be balanced with practicality.
- Some algorithms are inherently stateful or imperative.
- Pure functions cannot perform I/O, which is sometimes necessary.

### Annotated Code Examples

#### Example 1: Pure vs Impure Function

```js
// Step 1: Pure function — depends only on arguments.
function multiply(a, b) {
  return a * b;
}

// Step 2: Impure function — depends on external state.
let factor = 2;
function multiplyByFactor(a) {
  return a * factor; // Reads external 'factor'
}

// Step 3: Demonstrate the difference.
console.log(multiply(3, 4)); // 12 (always)
console.log(multiplyByFactor(3)); // 6

factor = 10; // Change external state
console.log(multiply(3, 4)); // 12 (unchanged)
console.log(multiplyByFactor(3)); // 30 (changed!)
```

**Expected Output:**
```
12
6
12
30
```

**Why This Output:** `multiply` is pure — its output depends only on its arguments. `multiplyByFactor` is impure — its output depends on the external `factor` variable, which can change. The same input (`3`) produces different outputs (`6` then `30`).

#### Example 2: Pure Function with No Mutation

```js
// Step 1: Impure function — mutates the input array.
function addItemImpure(array, item) {
  array.push(item); // Side effect: mutates the original array
  return array;
}

// Step 2: Pure function — returns a new array.
function addItemPure(array, item) {
  return [...array, item]; // Creates a new array
}

// Step 3: Compare behaviour.
const original = [1, 2, 3];
const impureResult = addItemImpure(original, 4);
console.log(original); // [1, 2, 3, 4] — original was mutated!

const original2 = [1, 2, 3];
const pureResult = addItemPure(original2, 4);
console.log(original2); // [1, 2, 3] — original unchanged
console.log(pureResult); // [1, 2, 3, 4]
```

**Expected Output:**
```
[ 1, 2, 3, 4 ]
[ 1, 2, 3 ]
[ 1, 2, 3, 4 ]
```

**Why This Output:** `addItemImpure` mutates the original array. `addItemPure` creates a new array using the spread operator, leaving the original untouched. The pure version has no side effects.

### Real-World Cases

- **Redux reducers:** Must be pure functions — `(state, action) => newState`.
- **Array methods:** `map`, `filter`, and `reduce` callbacks should be pure.
- **Mathematical utilities:** `Math.sqrt`, `Math.max`, and custom math functions.
- **Data transformation:** Parsing, formatting, and converting data.

---

## Side Effects

### Definitions

**Core Definition:** A side effect is any observable change to the state of the program or the outside world that occurs as a result of calling a function, beyond returning a value.

**Technical Definition:** Per MDN, "A side effect is any observable change to the state of the program or the outside world that occurs as a result of calling a function, beyond returning a value". Side effects include mutating external variables, modifying function arguments, writing to the DOM, logging to the console, making network requests, reading or writing files, and calling `Math.random()` or `Date.now()`. A function with side effects is impure.

**Beginner-Friendly Explanation:** A side effect is when a function does something besides just calculating and returning a value. It might change a global variable, update the screen, send data to a server, or write to a file. These actions are "side effects" because they affect things outside the function itself. Pure functions have no side effects — they just compute and return.

### Purposes

- To identify and isolate impure behaviour in a codebase.
- To distinguish between computation (pure) and action (impure).
- To make side effects explicit and controllable.
- To improve testability by minimising side effects.

### Syntax Rules and Structure

Side effects are not a syntax construct; they are behaviours. Common side effects include:

```js
// 1. Mutating external variables:
let count = 0;
function increment() {
  count++; // Side effect
}

// 2. Mutating function arguments:
function addItem(array, item) {
  array.push(item); // Side effect
}

// 3. DOM manipulation:
function updateUI(text) {
  document.getElementById("output").textContent = text; // Side effect
}

// 4. Logging:
function logMessage(msg) {
  console.log(msg); // Side effect
}

// 5. Network requests:
async function fetchData(url) {
  return await fetch(url); // Side effect
}

// 6. Non-deterministic sources:
function getRandom() {
  return Math.random(); // Impure: not deterministic
}
```

**Component Breakdown:**
- **Mutating external state:** Changing variables outside the function's scope.
- **Mutating arguments:** Modifying objects or arrays passed as parameters.
- **I/O operations:** Logging, DOM updates, network requests, file operations.
- **Non-deterministic sources:** `Math.random()`, `Date.now()`, `new Date()`.

**Syntax Rules:**
- Any operation that affects something outside the function's local scope is a side effect.
- Reading external mutable state also makes a function impure (its output depends on external factors).
- Side effects are not inherently bad — they are necessary for real applications.

**Constraints and Limitations:**
- Completely pure programs are impossible; they cannot interact with the world.
- The goal is to **isolate** side effects, not eliminate them entirely.
- Functional programming pushes side effects to the boundaries of the application.

### Annotated Code Examples

#### Example 1: Identifying Side Effects

```js
// Step 1: Function with multiple side effects.
let log = [];

function processUser(user) {
  // Side effect 1: Mutates the input object
  user.processed = true;

  // Side effect 2: Mutates external array
  log.push(`Processed ${user.name}`);

  // Side effect 3: Writes to the console
  console.log(`Done: ${user.name}`);

  // Returns a value (but also has side effects)
  return user;
}

// Step 2: Call the function.
const user = { name: "Alice" };
processUser(user);

console.log(user.processed); // true (mutated)
console.log(log);            // ["Processed Alice"] (mutated)
```

**Expected Output:**
```
Done: Alice
true
[ 'Processed Alice' ]
```

**Why This Output:** `processUser` has three side effects: it mutates `user`, pushes to `log`, and logs to the console. Even though it returns a value, it is impure.

#### Example 2: Pure Alternative

```js
// Step 1: Pure function — no side effects.
function processUserPure(user) {
  return { ...user, processed: true };
}

// Step 2: Call the pure function.
const user = { name: "Alice" };
const processed = processUserPure(user);

console.log(user.processed);      // undefined (unchanged)
console.log(processed.processed); // true
```

**Expected Output:**
```
undefined
true
```

**Why This Output:** `processUserPure` creates a new object with the `processed` property set to `true`. The original `user` object is unchanged. There are no side effects.

### Real-World Cases

- **Redux actions:** Side effects are handled by middleware (redux-thunk, redux-saga), keeping reducers pure.
- **React:** `useEffect` is used to isolate side effects from rendering.
- **Database access:** Repository functions encapsulate side effects.
- **Logging:** Logger functions are inherently impure and should be isolated.

---

## Referential Transparency

### Definitions

**Core Definition:** Referential transparency is the property of an expression that allows it to be replaced with its computed value without changing the program's behaviour.

**Technical Definition:** Per MDN, "A pure function is said to be referentially transparent: any expression containing a pure function call can be replaced with its computed value without changing the program's behaviour". This property is fundamental to equational reasoning and enables compiler optimisations such as common subexpression elimination and lazy evaluation. Referential transparency is a consequence of purity: because a pure function has no side effects and is deterministic, its call can always be replaced by its result.

**Beginner-Friendly Explanation:** Referential transparency means you can swap a function call with its answer, and nothing changes. If `add(2, 3)` always returns `5`, then anywhere you see `add(2, 3)`, you can write `5` instead. The program behaves exactly the same. This makes code easier to reason about because you don't have to worry about hidden effects.

### Purposes

- To enable equational reasoning about programs.
- To support compiler optimisations (caching, inlining, common subexpression elimination).
- To make code easier to refactor and understand.
- To enable memoization and lazy evaluation.
- To simplify debugging by eliminating hidden dependencies.

### Syntax Rules and Structure

Referential transparency is a property, not a syntax. It applies when:

```js
// Referentially transparent:
function add(a, b) {
  return a + b;
}

// Anywhere you see add(2, 3), you can replace it with 5:
const x = add(2, 3); // Same as:
const x = 5;
```

**Component Breakdown:**
- The function must be pure (deterministic and side-effect-free).
- The expression must not depend on mutable external state.

**Syntax Rules:**
- Referential transparency holds only for pure functions.
- Impure functions break referential transparency because their results depend on external state or produce side effects.

**Constraints and Limitations:**
- Referential transparency is an ideal; most real programs have some impure parts.
- JavaScript engines do not currently exploit referential transparency for optimisation (unlike Haskell).

### Annotated Code Examples

#### Example 1: Referentially Transparent Expression

```js
// Step 1: Pure function.
function double(n) {
  return n * 2;
}

// Step 2: These two statements are equivalent.
const a = double(21);
const b = 42;

console.log(a === b); // true
```

**Expected Output:**
```
true
```

**Why This Output:** Because `double` is pure, `double(21)` is referentially transparent and can be replaced with `42`. Both produce the same result.

#### Example 2: Broken Referential Transparency

```js
// Step 1: Impure function.
let counter = 0;
function next() {
  return ++counter;
}

// Step 2: These two statements are NOT equivalent.
const a = next(); // 1
const b = 1;

console.log(a === b); // true for the first call

const c = next(); // 2
const d = 1;

console.log(c === d); // false — impure function
```

**Expected Output:**
```
true
false
```

**Why This Output:** `next()` is impure because it depends on and modifies the external `counter` variable. Replacing `next()` with a constant value would change the program's behaviour, so referential transparency does not hold.

### Real-World Cases

- **Memoization:** Referentially transparent functions can be safely cached.
- **Compiler optimisation:** Functional languages like Haskell exploit this property.
- **Refactoring:** Referentially transparent code is easier to extract, inline, and reorder.
- **Testing:** Referentially transparent functions can be tested with simple input-output assertions.

---

## Idempotence

### Definitions

**Core Definition:** Idempotence is the property of an operation whereby applying it multiple times produces the same result as applying it once. In the context of pure functions, idempotence often refers to functions that return the same result when called repeatedly with the same input.

**Technical Definition:** Per MDN, idempotence is related to purity but distinct: an idempotent function can be called multiple times without changing the result beyond the initial call. For pure functions, idempotence is trivially satisfied for the output value (same input → same output), but idempotence becomes significant for impure operations like HTTP requests: a `GET` request is idempotent (calling it multiple times has the same effect as calling it once), while a `POST` request is not (calling it multiple times creates multiple resources). In functional programming, idempotence is often discussed alongside purity as a desirable property.

**Beginner-Friendly Explanation:** Idempotence means doing something twice has the same effect as doing it once. For example, pressing the "floor 3" button in an elevator multiple times doesn't make the elevator go to floor 3 any faster — it's idempotent. A pure function is always idempotent in terms of its output: `Math.abs(-5)` is `5` whether you call it once or a hundred times.

### Purposes

- To ensure that repeated operations do not cause unintended additional effects.
- To enable safe retries in distributed systems (idempotent HTTP methods).
- To simplify reasoning about repeated function calls.
- To support caching and memoization strategies.

### Syntax Rules and Structure

Idempotence is a property, not a syntax. For pure functions, it follows from determinism:

```js
// Idempotent pure function:
function absoluteValue(n) {
  return Math.abs(n);
}

console.log(absoluteValue(-5)); // 5
console.log(absoluteValue(-5)); // 5 (same result)
```

**Component Breakdown:**
- For pure functions, idempotence is guaranteed because the output depends only on the input.
- For impure operations, idempotence must be explicitly designed (e.g., `PUT` vs `POST`).

**Syntax Rules:**
- Pure functions are always idempotent with respect to their return value.
- Impure functions may or may not be idempotent.
- Idempotence does not imply purity (e.g., a function that logs and returns a constant is idempotent in output but impure).

**Constraints and Limitations:**
- Idempotence is about the effect, not the return value (for impure functions).
- A function can be idempotent but still impure (e.g., `console.log` returns `undefined` every time but has a side effect).

### Annotated Code Examples

#### Example 1: Pure Function Idempotence

```js
// Step 1: Pure function.
function toUpperCase(str) {
  return str.toUpperCase();
}

// Step 2: Call multiple times.
console.log(toUpperCase("hello")); // "HELLO"
console.log(toUpperCase("hello")); // "HELLO"
console.log(toUpperCase("hello")); // "HELLO"
```

**Expected Output:**
```
HELLO
HELLO
HELLO
```

**Why This Output:** `toUpperCase` is pure and idempotent. Calling it multiple times with the same input produces the same result every time.

#### Example 2: Idempotent vs Non-Idempotent Operations

```js
// Idempotent: setting a value.
function setName(obj, name) {
  return { ...obj, name };
}

// Non-idempotent: incrementing a value.
function increment(obj) {
  return { ...obj, count: obj.count + 1 };
}

const user = { name: "Alice", count: 0 };

// Idempotent: calling multiple times gives the same result.
console.log(setName(user, "Bob")); // { name: "Bob", count: 0 }
console.log(setName(user, "Bob")); // { name: "Bob", count: 0 }

// Non-idempotent: each call changes the result.
console.log(increment(user)); // { name: "Alice", count: 1 }
console.log(increment(increment(user))); // { name: "Alice", count: 2 }
```

**Expected Output:**
```
{ name: 'Bob', count: 0 }
{ name: 'Bob', count: 0 }
{ name: 'Alice', count: 1 }
{ name: 'Alice', count: 2 }
```

**Why This Output:** `setName` is idempotent — repeated calls with the same input produce the same output. `increment` is not idempotent — each call increases the count.

### Real-World Cases

- **HTTP methods:** `GET`, `PUT`, `DELETE` are idempotent; `POST` is not.
- **Database migrations:** Idempotent migrations can be run multiple times safely.
- **Configuration:** Setting a configuration value is idempotent.
- **Retry logic:** Idempotent operations can be safely retried.

---

## Deterministic Functions

### Definitions

**Core Definition:** A deterministic function is a function that always produces the same output for the same input, regardless of when or where it is called.

**Technical Definition:** Determinism is one of the two conditions for purity (the other being no side effects). Per MDN, "Deterministic — given the same arguments, it always returns the same result". A function that uses `Math.random()`, `Date.now()`, or any other non-deterministic source is not deterministic and therefore not pure. Determinism is a necessary but not sufficient condition for purity: a function can be deterministic but still impure if it has side effects.

**Beginner-Friendly Explanation:** A deterministic function is predictable: same input, same output, every time. If you call `add(2, 3)`, you always get `5`. But if you call `Math.random()`, you get a different number each time — that's non-deterministic. Pure functions must be deterministic.

### Purposes

- To make function behaviour predictable and reproducible.
- To enable testing with fixed expectations.
- To support caching and memoization.
- To simplify debugging and reasoning.
- To enable parallel execution without race conditions.

### Syntax Rules and Structure

```js
// Deterministic:
function add(a, b) {
  return a + b;
}

// Non-deterministic:
function randomAdd(a) {
  return a + Math.random(); // Different result each call
}

// Non-deterministic:
function timeBasedGreeting(name) {
  const hour = new Date().getHours();
  return hour < 12 ? `Good morning, ${name}` : `Good evening, ${name}`;
}
```

**Component Breakdown:**
- **Deterministic:** Output depends only on arguments.
- **Non-deterministic:** Output depends on external factors (time, randomness, external state).

**Syntax Rules:**
- A function is deterministic if it does not use non-deterministic sources.
- Non-deterministic sources include `Math.random()`, `Date.now()`, `performance.now()`, and external state.
- Determinism is required for purity but does not guarantee it (side effects can still make a function impure).

**Constraints and Limitations:**
- Some applications require non-determinism (e.g., random number generators, simulations).
- Non-deterministic functions can be made testable by injecting the non-deterministic source as a parameter.

### Annotated Code Examples

#### Example 1: Deterministic vs Non-Deterministic

```js
// Step 1: Deterministic function.
function square(n) {
  return n * n;
}

// Step 2: Non-deterministic function.
function randomSquare(n) {
  return n * n * Math.random();
}

// Step 3: Call both multiple times.
console.log(square(5)); // 25 (always)
console.log(square(5)); // 25 (always)
console.log(randomSquare(5)); // e.g., 18.34...
console.log(randomSquare(5)); // e.g., 7.92...
```

**Expected Output:**
```
25
25
18.34... (varies)
7.92... (varies)
```

**Why This Output:** `square` is deterministic — it always returns `25` for `5`. `randomSquare` multiplies by `Math.random()`, so it returns a different value each time.

#### Example 2: Making Non-Deterministic Functions Testable

```js
// Step 1: Non-deterministic function (hard to test).
function createIdBad() {
  return Math.random().toString(36).substring(2);
}

// Step 2: Deterministic version with injected dependency.
function createId(generator) {
  return generator().toString(36).substring(2);
}

// Step 3: Test with a predictable generator.
const fakeGenerator = () => 0.123456789;
console.log(createId(fakeGenerator)); // Always the same result

// Step 4: Use the real generator in production.
console.log(createId(Math.random)); // Random result
```

**Expected Output:**
```
4fzzzxjylrx (same every time with fakeGenerator)
(random result with Math.random)
```

**Why This Output:** By injecting the random number generator as a parameter, `createId` becomes deterministic when given a deterministic generator. This makes it testable while still allowing randomness in production.

### Real-World Cases

- **Testing:** Deterministic functions are easy to test.
- **Caching:** Deterministic functions can be memoized.
- **Distributed systems:** Deterministic functions can be safely retried.
- **Game development:** Deterministic simulations allow replay and debugging.

---

## Pure Function Benefits

### Definitions

**Core Definition:** Pure functions offer numerous advantages in software development, including predictability, testability, composability, and performance optimisations.

**Technical Definition:** Per MDN, pure functions are "much easier to reason about, test, and debug". They are the foundation of functional programming and enable techniques such as memoization, referential transparency, and equational reasoning. Because they have no side effects and are deterministic, pure functions can be safely cached, parallelised, and composed.

**Beginner-Friendly Explanation:** Pure functions make your code better in many ways. They're easy to test because you just check input and output. They're easy to understand because nothing hidden is going on. They're easy to reuse because they don't depend on anything external. And they're easy to optimise because the same input always gives the same output.

### Purposes

- To improve code predictability and maintainability.
- To simplify unit testing (no mocks or stubs needed).
- To enable safe caching and memoization.
- To support parallel execution without race conditions.
- To facilitate function composition and refactoring.
- To reduce bugs caused by hidden state mutations.

### Syntax Rules and Structure

Benefits are realised through consistent application of purity principles:

```js
// Testable: simple input-output assertion
function add(a, b) { return a + b; }
console.assert(add(2, 3) === 5);

// Cacheable: memoization is safe
const memoizedAdd = memoize(add);
memoizedAdd(2, 3); // Computes
memoizedAdd(2, 3); // Returns cache

// Composable: pure functions compose cleanly
const process = pipe(trim, toLowerCase, capitalize);
```

**Component Breakdown:**
- **Testability:** Pure functions need no mocks.
- **Cacheability:** Results can be stored and reused.
- **Composability:** Functions can be chained without side-effect interference.
- **Parallelisability:** No shared state means no race conditions.

**Syntax Rules:**
- Benefits are maximised when purity is consistently applied.
- Impure functions should be isolated at the edges of the application.

**Constraints and Limitations:**
- Real applications require side effects (I/O, DOM, network). Purity is a spectrum, not a binary.
- Overemphasising purity can lead to complex abstractions.

### Annotated Code Examples

#### Example 1: Testing Pure Functions

```js
// Step 1: Pure function.
function calculateDiscount(price, discountPercent) {
  return price * (1 - discountPercent / 100);
}

// Step 2: Simple tests — no mocks needed.
console.assert(calculateDiscount(100, 20) === 80, "20% off 100 should be 80");
console.assert(calculateDiscount(50, 0) === 50, "0% off 50 should be 50");
console.assert(calculateDiscount(200, 50) === 100, "50% off 200 should be 100");

console.log("All tests passed!");
```

**Expected Output:**
```
All tests passed!
```

**Why This Output:** Pure functions can be tested with simple input-output assertions. No setup, no mocks, no teardown. The tests are fast and deterministic.

#### Example 2: Memoization of Pure Functions

```js
// Step 1: Pure function.
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

// Step 2: Memoize it.
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = args.join(",");
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

const memoFib = memoize(fibonacci);
console.log(memoFib(40)); // Fast because of memoization
```

**Expected Output:**
```
102334155
```

**Why This Output:** Because `fibonacci` is pure, memoization is safe. The same input always produces the same output, so caching the result is correct.

### Real-World Cases

- **Redux:** Reducers must be pure for predictable state updates.
- **React:** Pure components can be optimised with `React.memo`.
- **Testing:** Pure functions are the easiest to unit test.
- **Caching:** Memoization relies on purity.
- **Parallelism:** Pure functions can run in parallel without locks.

---

## Testing Pure Functions

### Definitions

**Core Definition:** Testing pure functions involves verifying that the function returns the correct output for given inputs, without needing to mock or stub external dependencies.

**Technical Definition:** Per the testing literature, "A pure function is a function that will always return the same result given the same input. It's the simplest unit to test because it doesn't depend on external state". Pure functions are the easiest to test because they have no side effects, no external dependencies, and no hidden state. Tests are simple input-output assertions.

**Beginner-Friendly Explanation:** Testing pure functions is easy. You call the function with some input, check that the output is what you expect, and you're done. No need to set up databases, mock APIs, or clean up after the test. This is why pure functions are a favourite among developers who care about testing.

### Purposes

- To verify that a function produces the correct output for given inputs.
- To catch regressions when code changes.
- To document expected behaviour through executable examples.
- To enable test-driven development (TDD).
- To support property-based testing.

### Syntax Rules and Structure

```js
// Pure function:
function add(a, b) {
  return a + b;
}

// Test:
describe("add", () => {
  it("returns the sum of two numbers", () => {
    expect(add(2, 3)).toBe(5);
  });
  it("handles negative numbers", () => {
    expect(add(-1, -2)).toBe(-3);
  });
});
```

**Component Breakdown:**
- **Function under test:** The pure function.
- **Input:** The arguments passed to the function.
- **Expected output:** The value the function should return.
- **Assertion:** A comparison between actual and expected output.

**Syntax Rules:**
- Pure functions require no mocking or stubbing.
- Tests are simple: call the function, assert the result.
- Tests are fast and deterministic.

**Constraints and Limitations:**
- Pure functions are easy to test, but impure functions require more setup.
- Testing cannot prove the absence of bugs, only their presence.

### Annotated Code Examples

#### Example 1: Unit Testing a Pure Function

```js
// Step 1: Pure function.
function formatCurrency(amount, currency = "USD") {
  return new Intl.NumberFormat("en-US", {
    style: "currency",
    currency,
  }).format(amount);
}

// Step 2: Tests.
function testFormatCurrency() {
  const tests = [
    { input: [1234.56], expected: "$1,234.56" },
    { input: [0], expected: "$0.00" },
    { input: [1000, "EUR"], expected: "€1,000.00" },
  ];

  tests.forEach(({ input, expected }) => {
    const actual = formatCurrency(...input);
    console.assert(actual === expected, `Expected ${expected}, got ${actual}`);
  });

  console.log("All formatCurrency tests passed!");
}

testFormatCurrency();
```

**Expected Output:**
```
All formatCurrency tests passed!
```

**Why This Output:** `formatCurrency` is pure, so each test is a simple input-output assertion. No mocks, no setup, no teardown. The tests run instantly.

#### Example 2: Property-Based Testing

```js
// Step 1: Pure function.
function reverseString(str) {
  return str.split("").reverse().join("");
}

// Step 2: Property-based test — reversing twice returns the original.
function testReverseProperty() {
  const samples = ["hello", "world", "a", "", "racecar", "12345"];

  samples.forEach((s) => {
    const doubleReversed = reverseString(reverseString(s));
    console.assert(
      doubleReversed === s,
      `Expected ${s}, got ${doubleReversed}`
    );
  });

  console.log("Reverse property holds for all samples!");
}

testReverseProperty();
```

**Expected Output:**
```
Reverse property holds for all samples!
```

**Why This Output:** The property "reversing a string twice returns the original" holds for all inputs. Property-based testing leverages the purity of `reverseString` to make general assertions.

### Real-World Cases

- **Unit testing:** Pure functions are the easiest to test in isolation.
- **Test-driven development (TDD):** Pure functions fit naturally into the TDD cycle.
- **Property-based testing:** Libraries like fast-check use pure functions to generate test cases.
- **Regression testing:** Pure function tests catch regressions quickly.

---

## References

- MDN Web Docs – Pure function — https://developer.mozilla.org/en-US/docs/Glossary/Pure_function
- MDN Web Docs – Side effect — https://developer.mozilla.org/en-US/docs/Glossary/Side_effect
- MDN Web Docs – Idempotent — https://developer.mozilla.org/en-US/docs/Glossary/Idempotent
- MDN Web Docs – Referential transparency — https://developer.mozilla.org/en-US/docs/Glossary/Referential_transparency
- FreeCodeCamp – What are Pure Functions and Side Effects in JavaScript — https://www.freecodecamp.org/news/what-are-pure-functions-and-side-effects-in-javascript/
- JavaScript.info – Pure functions — https://javascript.info/function-basics
- Redux Documentation – Reducers must be pure — https://redux.js.org/usage/structuring-reducers/PrerequisiteConcepts
- React Documentation – Keeping Components Pure — https://react.dev/learn/keeping-components-pure
- Eloquent JavaScript – Pure Functions — https://eloquentjavascript.net/03_functions.html
- Functional Programming in JavaScript – Pure Functions — https://github.com/getify/Functional-Light-JS
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf
- Haskell Wiki – Referential Transparency — https://wiki.haskell.org/Referential_transparency