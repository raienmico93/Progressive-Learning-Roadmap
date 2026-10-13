# Generators — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A generator is a special type of function that can be paused and resumed, producing a sequence of values over time. Generator functions are declared with `function*` and use the `yield` keyword to produce values.

**Technical Definition:** Per the ECMAScript specification, a generator function is a function that returns a `Generator` object, which conforms to both the iterable protocol and the iterator protocol. The `Generator` object is a subclass of the hidden `Iterator` class and has a `next()` method that returns an `IteratorResult` object with `value` and `done` properties. When a generator function is called, its body does not execute immediately; instead, the function's execution is suspended at the very beginning. Each call to `next()` resumes execution until the next `yield` expression, which pauses the function and returns the yielded value to the caller. The `yield` keyword can be thought of as a generator-based version of `return`, but unlike `return`, it can be used multiple times and the function can be resumed afterward.

**Beginner-Friendly Explanation:** A generator is a function that can pause itself in the middle of execution and resume later. Instead of running all at once and returning a single value, a generator can produce a series of values one at a time, on demand. Think of it like a book with bookmarks: you read a few pages, put in a bookmark, and later pick up exactly where you left off.

### Key Characteristics

- **Pausable execution:** Generator functions can be suspended at `yield` expressions and resumed later.
- **State preservation:** Variable bindings and execution context are saved across pauses.
- **Lazy evaluation:** Values are computed only when requested, not in advance.
- **Iterator protocol:** Generator objects conform to the iterator protocol (`next()` method returning `{ value, done }`).
- **Iterable protocol:** Generator objects conform to the iterable protocol (usable in `for...of`, spread, etc.).
- **Bidirectional communication:** Values can be passed into the generator via `next(value)` and out via `yield`.
- **Delegation:** `yield*` delegates iteration to another iterable.
- **No arrow function equivalent:** Generator functions cannot be written as arrow functions.
- **Infinite sequences:** Generators can produce potentially infinite sequences without running out of memory.

### Prerequisites

- Understanding of JavaScript functions, scope, and closures.
- Familiarity with the iterator and iterable protocols.
- Basic knowledge of `for...of` loops and the spread operator.
- Awareness of Promises and `async/await` (for comparison).

### Related Programming Areas

- **Iteration:** Generators are a way to create custom iterators.
- **Lazy evaluation:** Computing values on demand for performance and memory efficiency.
- **Asynchronous programming:** Generators can model asynchronous flows (pre-`async/await`).
- **State machines:** Generators naturally model state transitions.
- **Data streaming:** Processing large or infinite datasets incrementally.

### Core Concepts / Features

---

## Generator Functions

### Definitions

**Core Definition:** A generator function is a function declared with the `function*` syntax that returns a `Generator` object and can be paused and resumed using `yield`.

**Technical Definition:** Per MDN, "The `function*` declaration creates a binding of a new generator function to a given name. A generator function can be exited and later re-entered, with its context (variable bindings) saved across re-entrances". Each call to a generator function returns a new `Generator` object, and multiple generators can be created from the same function, each maintaining its own independent execution context. Generator functions do not have arrow function counterparts.

**Beginner-Friendly Explanation:** A generator function is like a normal function but with a superpower: it can pause in the middle and continue later. You declare it with `function*` and use `yield` to pause and produce values.

### Purposes

- To create functions that can produce a sequence of values over time.
- To enable lazy evaluation, computing values only when needed.
- To implement custom iterators with minimal boilerplate.
- To model state machines and sequential processes.
- To generate infinite sequences without memory issues.

### Syntax Rules and Structure

```js
function* name(param0, param1, ...) {
  // statements
  yield value;
  // more statements
}
```

**Component Breakdown:**
- `function*`: The declaration keyword (the `*` can be separated from `function` by whitespace).
- `name`: The function name.
- `yield value`: Pauses execution and returns `value` to the caller.
- The function body contains statements that execute between pauses.

**Syntax Rules:**
- `function` and `*` are separate tokens and can be separated by whitespace or line terminators.
- Generator functions have no arrow function equivalent.
- Calling a generator function does not execute its body; it returns a `Generator` object.
- The generator body begins execution only when `next()` is called.

**Constraints and Limitations:**
- `yield` can only be used directly inside the generator function, not in nested functions or callbacks.
- Generators cannot be used as constructors.
- Generator functions are not hoisted in the same way as regular function declarations.

### Annotated Code Examples

#### Example 1: Basic Generator Function

```js
// Step 1: Define a generator function.
function* generateNumbers() {
  console.log("Start");
  yield 1;
  console.log("After first yield");
  yield 2;
  console.log("After second yield");
  yield 3;
  console.log("End");
}

// Step 2: Call the generator function — returns a Generator object.
const gen = generateNumbers();
console.log("Generator created");

// Step 3: Call next() to resume execution.
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: false }
console.log(gen.next()); // { value: undefined, done: true }
```

**Expected Output:**
```
Generator created
Start
{ value: 1, done: false }
After first yield
{ value: 2, done: false }
After second yield
{ value: 3, done: false }
End
{ value: undefined, done: true }
```

**Why This Output:** The generator body does not execute when `generateNumbers()` is called. It executes step-by-step each time `next()` is called. Each `yield` pauses execution and returns a value. When the function completes, `done` becomes `true`.

#### Example 2: Passing Values into a Generator

```js
// Step 1: Define a generator that receives values.
function* echo() {
  const a = yield "First";
  console.log("Received:", a);
  const b = yield "Second";
  console.log("Received:", b);
  return "Done";
}

// Step 2: Use the generator.
const gen = echo();
console.log(gen.next());        // { value: 'First', done: false }
console.log(gen.next("hello")); // Logs "Received: hello"
                                // { value: 'Second', done: false }
console.log(gen.next("world")); // Logs "Received: world"
                                // { value: 'Done', done: true }
```

**Expected Output:**
```
{ value: 'First', done: false }
Received: hello
{ value: 'Second', done: false }
Received: world
{ value: 'Done', done: true }
```

**Why This Output:** The value passed to `next()` becomes the result of the paused `yield` expression. The first `next()` call starts execution and pauses at the first `yield`. The second `next("hello")` passes `"hello"` as the value of the first `yield` expression and resumes until the next `yield`.

### Real-World Cases

- **Fibonacci sequence:** Generating Fibonacci numbers lazily.
- **ID generators:** Producing unique IDs on demand.
- **Pagination:** Fetching pages one at a time.
- **State machines:** Managing state transitions in a workflow.

---

## `function*`

### Definitions

**Core Definition:** `function*` is the syntax used to declare a generator function.

**Technical Definition:** Per MDN, "The `function*` declaration creates a binding of a new generator function to a given name". It creates a `GeneratorFunction` object. The `function` keyword and the `*` token are separate, so they can be separated by whitespace or line terminators.

**Beginner-Friendly Explanation:** `function*` is just the way you tell JavaScript "this is a generator function." The star (`*`) is the magic symbol.

### Purposes

- To declare a generator function.
- To create a function that returns a `Generator` object.
- To enable the use of `yield` inside the function body.

### Syntax Rules and Structure

```js
function* name(param0, param1, ...) {
  statements
}
```

**Component Breakdown:**
- `function`: The keyword.
- `*`: The generator marker.
- `name`: The function name.
- `statements`: The function body, which may contain `yield` expressions.

**Syntax Rules:**
- The `*` is part of the syntax and must be present.
- `function` and `*` can be separated by whitespace.
- Generator functions can also be defined as expressions: `const gen = function*() { ... }`.

**Constraints and Limitations:**
- Generator functions cannot be arrow functions.
- Generator functions cannot be constructors.

### Annotated Code Examples

#### Example 1: Generator Function Declaration

```js
// Step 1: Declare a generator function.
function* simpleGenerator() {
  yield "a";
  yield "b";
}

// Step 2: Call it.
const gen = simpleGenerator();
console.log(gen.next().value); // "a"
console.log(gen.next().value); // "b"
```

**Expected Output:**
```
a
b
```

**Why This Output:** The generator function is declared with `function*`. Calling it returns a `Generator` object. Each `next()` call resumes execution until the next `yield`.

#### Example 2: Generator Function Expression

```js
// Step 1: Assign a generator function expression to a variable.
const genFunc = function*() {
  yield 1;
  yield 2;
};

// Step 2: Use it.
const g = genFunc();
console.log(g.next().value); // 1
console.log(g.next().value); // 2
```

**Expected Output:**
```
1
2
```

**Why This Output:** Generator functions can be defined as expressions, just like regular functions.

### Real-World Cases

- **Utility libraries:** Defining reusable generator functions.
- **Iterators:** Creating custom iterators for data structures.
- **Lazy sequences:** Generating values on demand.

---

## `yield`

### Definitions

**Core Definition:** `yield` is an operator used inside generator functions to pause execution and return a value to the caller.

**Technical Definition:** Per MDN, "The `yield` keyword is used to pause and resume a generator function". The `yield` expression returns an `IteratorResult` object with `value` and `done` properties. The value of the `yield` expression (when resumed) is the value passed to `next()`. `yield` can only be called directly from the generator function that contains it, not from nested functions or callbacks.

**Beginner-Friendly Explanation:** `yield` is like a `return` that doesn't end the function. It gives a value back to the caller and then pauses, waiting to be resumed.

### Purposes

- To produce a value from a generator.
- To pause execution and save state.
- To receive values from the caller via `next(value)`.
- To create sequences of values lazily.

### Syntax Rules and Structure

```js
[rv] = yield [expression];
```

**Component Breakdown:**
- `expression` (optional): The value to return to the caller. Defaults to `undefined`.
- `rv` (optional): The value passed to `next()` when resuming, assigned to a variable.

**Syntax Rules:**
- `yield` can only be used inside generator functions.
- `yield` pauses execution and returns control to the caller.
- The generator resumes when `next()` is called.

**Constraints and Limitations:**
- `yield` cannot be used in nested functions or callbacks.
- `yield` cannot be used in arrow functions.

### Annotated Code Examples

#### Example 1: Basic `yield`

```js
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

const g = gen();
console.log(g.next()); // { value: 1, done: false }
console.log(g.next()); // { value: 2, done: false }
console.log(g.next()); // { value: 3, done: false }
console.log(g.next()); // { value: undefined, done: true }
```

**Expected Output:**
```
{ value: 1, done: false }
{ value: 2, done: false }
{ value: 3, done: false }
{ value: undefined, done: true }
```

**Why This Output:** Each `yield` pauses the generator and returns the yielded value. When no more yields remain, `done` becomes `true`.

#### Example 2: `yield` with Expression Value

```js
function* gen() {
  const x = yield "First";
  console.log("x =", x);
  const y = yield "Second";
  console.log("y =", y);
}

const g = gen();
g.next();        // Starts, pauses at first yield
g.next("hello"); // x = "hello", pauses at second yield
g.next("world"); // y = "world", function ends
```

**Expected Output:**
```
x = hello
y = world
```

**Why This Output:** The value passed to `next()` becomes the result of the paused `yield` expression, assigned to the variable on the left.

### Real-World Cases

- **ID generators:** `yield ++id;` for unique IDs.
- **Pagination:** `yield await fetchPage(page);`.
- **State machines:** `yield state;` for each state transition.

---

## `yield*`

### Definitions

**Core Definition:** `yield*` is an operator used inside generator functions to delegate iteration to another iterable object, such as a `Generator`, array, or string.

**Technical Definition:** Per MDN, "The `yield*` operator can be used within generator (sync or async) functions to delegate to another iterable object, such as a `Generator`". It iterates over the operand and yields each value returned by it. The `yield*` expression first gets the iterator from the operand by calling its `[Symbol.iterator]()` method. Then, each time the generator's `next()` is called, `yield*` calls the iterator's `next()` method and yields the same result object. When the iterator's result has `done: true`, the `yield*` expression stops and returns the value of that result. The `yield*` operator also forwards the current generator's `throw()` and `return()` methods to the underlying iterator.

**Beginner-Friendly Explanation:** `yield*` means "yield everything from this other iterable." Instead of yielding values one by one, you can delegate to another generator, array, or string, and it will yield all of their values.

### Purposes

- To delegate iteration to another generator or iterable.
- To compose generators hierarchically.
- To reuse generator logic without duplication.
- To flatten nested iterables.

### Syntax Rules and Structure

```js
yield* expression;
```

**Component Breakdown:**
- `expression`: An iterable object (e.g., another generator, an array, a string).

**Syntax Rules:**
- `yield*` can only be used inside generator functions.
- It iterates over the operand and yields each value.
- It returns the value returned by the underlying iterator when it's done.

**Constraints and Limitations:**
- The operand must be iterable (have a `[Symbol.iterator]()` method).
- `yield*` cannot be used outside a generator function.

### Annotated Code Examples

#### Example 1: Delegating to Another Generator

```js
function* inner() {
  yield 1;
  yield 2;
}

function* outer() {
  yield "a";
  yield* inner();
  yield "b";
}

const g = outer();
console.log(g.next().value); // "a"
console.log(g.next().value); // 1
console.log(g.next().value); // 2
console.log(g.next().value); // "b"
```

**Expected Output:**
```
a
1
2
b
```

**Why This Output:** `yield* inner()` delegates to the `inner` generator, yielding all of its values before continuing with `outer`.

#### Example 2: Delegating to an Array

```js
function* gen() {
  yield* [1, 2, 3];
  yield* "ab";
}

const g = gen();
console.log([...g]); // [1, 2, 3, "a", "b"]
```

**Expected Output:**
```
[ 1, 2, 3, 'a', 'b' ]
```

**Why This Output:** `yield*` works with any iterable, including arrays and strings, yielding each element.

### Real-World Cases

- **Tree traversal:** Delegating to child generators for each subtree.
- **Composing iterators:** Combining multiple generators into one.
- **Flattening:** Flattening nested arrays or iterables.

---

## Generator Objects

### Definitions

**Core Definition:** A generator object is the object returned by calling a generator function. It conforms to both the iterable and iterator protocols.

**Technical Definition:** Per MDN, "The `Generator` object is returned by a generator function and it conforms to both the iterable protocol and the iterator protocol". `Generator` is a subclass of the hidden `Iterator` class. There is no `Generator` constructor available globally; instances must be returned from generator functions. The `Generator` object has `next()`, `return()`, and `throw()` methods.

**Beginner-Friendly Explanation:** A generator object is what you get when you call a generator function. It's an object you can call `next()` on to get values, and you can also use it in `for...of` loops.

### Purposes

- To represent the state of a generator function's execution.
- To provide methods for resuming, returning, and throwing into the generator.
- To serve as an iterator for `for...of` loops and other iteration contexts.

### Syntax Rules and Structure

```js
function* gen() { yield 1; yield 2; }
const g = gen(); // g is a Generator object
```

**Component Breakdown:**
- `gen()`: Calling a generator function returns a `Generator` object.
- `g`: The generator object, which has `next()`, `return()`, and `throw()` methods.

**Syntax Rules:**
- Generator objects are iterable (can be used in `for...of`, spread, etc.).
- Generator objects are iterators (have a `next()` method returning `{ value, done }`).
- The `[Symbol.toStringTag]` is `"Generator"`.

**Constraints and Limitations:**
- Generator objects are single-use; once exhausted (`done: true`), they cannot be restarted.
- Generator objects do not store a reference to the generator function that created them.

### Annotated Code Examples

#### Example 1: Generator Object Properties

```js
function* gen() { yield 1; }
const g = gen();

console.log(Object.prototype.toString.call(g)); // "[object Generator]"
console.log(typeof g.next); // "function"
console.log(typeof g.return); // "function"
console.log(typeof g.throw); // "function"
console.log(g[Symbol.iterator]() === g); // true
```

**Expected Output:**
```
[object Generator]
function
function
function
true
```

**Why This Output:** The generator object has `next`, `return`, and `throw` methods. It is also iterable (its `[Symbol.iterator]()` returns itself).

### Real-World Cases

- **Iteration:** Using generator objects in `for...of` loops.
- **Manual control:** Calling `next()`, `return()`, and `throw()` manually.
- **Lazy sequences:** Generating values on demand.

---

## `next()`

### Definitions

**Core Definition:** `next()` is a method on generator objects that resumes the generator's execution and returns an `IteratorResult` object.

**Technical Definition:** Per MDN, "The `next()` method returns an object with two properties: `done` and `value`". If the generator is not yet done, `done` is `false` and `value` is the yielded value. If the generator is done, `done` is `true` and `value` is the return value (or `undefined`). The argument passed to `next()` becomes the value of the paused `yield` expression.

**Beginner-Friendly Explanation:** `next()` tells the generator to continue running until the next `yield`. It returns an object with the yielded value and a flag indicating whether the generator has finished.

### Purposes

- To resume a generator's execution.
- To retrieve the next yielded value.
- To pass values into the generator.
- To check whether the generator is done.

### Syntax Rules and Structure

```js
generator.next(value);
```

**Component Breakdown:**
- `value` (optional): The value to pass into the generator, becoming the result of the paused `yield` expression.

**Syntax Rules:**
- Returns `{ value, done }`.
- `done` is `false` if a `yield` was reached; `true` if the generator completed.
- The first `next()` call starts execution from the beginning of the function body.

**Constraints and Limitations:**
- Calling `next()` on a completed generator returns `{ value: undefined, done: true }`.
- The value passed to `next()` is ignored if the generator is not currently paused at a `yield`.

### Annotated Code Examples

#### Example 1: Basic `next()`

```js
function* gen() {
  yield "a";
  yield "b";
}

const g = gen();
console.log(g.next()); // { value: 'a', done: false }
console.log(g.next()); // { value: 'b', done: false }
console.log(g.next()); // { value: undefined, done: true }
```

**Expected Output:**
```
{ value: 'a', done: false }
{ value: 'b', done: false }
{ value: undefined, done: true }
```

**Why This Output:** Each `next()` call resumes the generator until the next `yield` or the end of the function.

#### Example 2: Passing Values with `next()`

```js
function* gen() {
  const x = yield "start";
  console.log("Received:", x);
}

const g = gen();
g.next();          // Starts, pauses at first yield
g.next("hello");   // Resumes, x = "hello"
```

**Expected Output:**
```
Received: hello
```

**Why This Output:** The value passed to `next()` becomes the result of the paused `yield` expression.

### Real-World Cases

- **Manual iteration:** Stepping through a sequence one value at a time.
- **Interactive generators:** Passing user input into a generator.
- **Testing:** Verifying generator behaviour step by step.

---

## `return()`

### Definitions

**Core Definition:** `return()` is a method on generator objects that forces the generator to complete, as if a `return` statement were executed at the current suspended position.

**Technical Definition:** Per MDN, `return()` "Acts as if a `return` statement is inserted in the generator's body at the current suspended position, which finishes the generator and allows the generator to perform any cleanup tasks when combined with a `try...finally` block". The value passed to `return()` becomes the `value` of the returned `IteratorResult` object, and `done` is `true`.

**Beginner-Friendly Explanation:** `return()` tells the generator to stop and return immediately, as if you had written `return` at the current position.

### Purposes

- To terminate a generator early.
- To perform cleanup operations via `finally` blocks.
- To return a final value from a generator.

### Syntax Rules and Structure

```js
generator.return(value);
```

**Component Breakdown:**
- `value` (optional): The value to return. Defaults to `undefined`.

**Syntax Rules:**
- Returns `{ value, done: true }`.
- Triggers any `finally` blocks in the generator.
- Once called, the generator cannot be resumed.

**Constraints and Limitations:**
- If the generator is already done, `return()` returns `{ value, done: true }` with the value passed.

### Annotated Code Examples

#### Example 1: Early Return

```js
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

const g = gen();
console.log(g.next());       // { value: 1, done: false }
console.log(g.return("early")); // { value: 'early', done: true }
console.log(g.next());       // { value: undefined, done: true }
```

**Expected Output:**
```
{ value: 1, done: false }
{ value: 'early', done: true }
{ value: undefined, done: true }
```

**Why This Output:** `return("early")` terminates the generator with the value `"early"`. Subsequent `next()` calls return `done: true`.

#### Example 2: `return()` with `finally`

```js
function* gen() {
  try {
    yield 1;
    yield 2;
  } finally {
    console.log("Cleanup");
  }
}

const g = gen();
g.next();       // { value: 1, done: false }
g.return();     // Logs "Cleanup", returns { value: undefined, done: true }
```

**Expected Output:**
```
Cleanup
{ value: undefined, done: true }
```

**Why This Output:** The `finally` block executes when `return()` is called, allowing cleanup.

### Real-World Cases

- **Resource cleanup:** Releasing resources when a generator is terminated early.
- **Cancellation:** Stopping a generator when it's no longer needed.
- **Testing:** Verifying generator cleanup behaviour.

---

## `throw()`

### Definitions

**Core Definition:** `throw()` is a method on generator objects that throws an exception into the generator at the current suspended position.

**Technical Definition:** Per MDN, `throw()` "Acts as if a `throw` statement is inserted in the generator's body at the current suspended position, which informs the generator of an error condition and allows it to handle the error, or perform cleanup and close itself". If the generator catches the error, execution continues and `throw()` returns the next `IteratorResult`. If the error is not caught, the generator closes and the exception propagates to the caller.

**Beginner-Friendly Explanation:** `throw()` lets you send an error into a generator from the outside. The generator can catch it with a `try...catch` block, or it will propagate out.

### Purposes

- To signal error conditions to a generator.
- To allow generators to handle errors and continue.
- To trigger cleanup via `try...finally`.

### Syntax Rules and Structure

```js
generator.throw(exception);
```

**Component Breakdown:**
- `exception`: The exception to throw into the generator.

**Syntax Rules:**
- If the generator catches the exception, `throw()` returns the next `IteratorResult`.
- If the generator does not catch the exception, it propagates to the caller.
- The generator is closed if the exception is not caught.

**Constraints and Limitations:**
- Throwing into a completed generator propagates the exception immediately.

### Annotated Code Examples

#### Example 1: Catching an Error in a Generator

```js
function* gen() {
  try {
    yield 1;
    yield 2;
  } catch (e) {
    console.log("Caught inside:", e.message);
    yield "recovered";
  }
}

const g = gen();
console.log(g.next());              // { value: 1, done: false }
console.log(g.throw(new Error("Oops"))); // Logs "Caught inside: Oops"
                                    // { value: 'recovered', done: false }
```

**Expected Output:**
```
{ value: 1, done: false }
Caught inside: Oops
{ value: 'recovered', done: false }
```

**Why This Output:** `throw()` injects an error at the paused `yield`. The generator's `catch` block handles it and yields a recovery value.

### Real-World Cases

- **Error handling:** Injecting errors into generators for testing.
- **State machines:** Handling invalid state transitions.
- **Cleanup:** Triggering error paths for cleanup.

---

## Generator Iteration

### Definitions

**Core Definition:** Generator iteration is the process of consuming a generator's values using iteration protocols such as `for...of`, spread syntax, or manual `next()` calls.

**Technical Definition:** Per MDN, generator objects conform to the iterable protocol, meaning they have a `[Symbol.iterator]()` method that returns the generator object itself. This allows them to be used in `for...of` loops, spread syntax, destructuring, and other contexts that accept iterables.

**Beginner-Friendly Explanation:** You can loop over a generator's values using `for...of`, just like you would with an array. The generator produces values one at a time as the loop requests them.

### Purposes

- To consume generator values with standard iteration syntax.
- To integrate generators with array methods and other iterable-consuming APIs.
- To simplify code that would otherwise require manual `next()` calls.

### Syntax Rules and Structure

```js
for (const value of generator) {
  // process value
}
```

**Component Breakdown:**
- `generator`: A generator object.
- `value`: Each yielded value.

**Syntax Rules:**
- Generators can be used in `for...of`, spread (`[...gen]`), destructuring, `Array.from()`, etc.
- Iteration stops when the generator is done.

**Constraints and Limitations:**
- `for...of` automatically calls `next()` until `done` is `true`.
- Breaking out of a `for...of` loop calls `return()` on the generator, triggering cleanup.

### Annotated Code Examples

#### Example 1: `for...of` with a Generator

```js
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

for (const value of gen()) {
  console.log(value);
}
```

**Expected Output:**
```
1
2
3
```

**Why This Output:** `for...of` calls `next()` repeatedly until the generator is done, logging each yielded value.

#### Example 2: Spread Syntax

```js
function* gen() {
  yield "a";
  yield "b";
}

const arr = [...gen()];
console.log(arr); // ["a", "b"]
```

**Expected Output:**
```
[ 'a', 'b' ]
```

**Why This Output:** The spread operator consumes the generator and collects all yielded values into an array.

### Real-World Cases

- **Data processing:** Iterating over generated sequences.
- **Lazy evaluation:** Only computing values as needed by the loop.
- **Integration:** Using generators with array methods via `Array.from()`.

---

## Lazy Evaluation

### Definitions

**Core Definition:** Lazy evaluation is the technique of deferring computation until the result is actually needed, rather than computing values eagerly.

**Technical Definition:** Per CoreUI, "Lazy evaluation defers computation until values are actually needed, reducing memory usage and improving performance for large datasets. The most effective approach uses JavaScript generators to create lazy iterables that compute values on demand". Generators are inherently lazy: the function body does not execute until `next()` is called, and each `yield` produces a value only when requested.

**Beginner-Friendly Explanation:** Lazy evaluation means "don't do the work until someone asks for the result." Generators are lazy by nature — they compute values one at a time, only when needed.

### Purposes

- To reduce memory usage by not storing entire datasets.
- To improve performance by avoiding unnecessary computation.
- To enable infinite sequences.
- To process large datasets incrementally.

### Syntax Rules and Structure

```js
function* lazyRange(start, end) {
  for (let i = start; i <= end; i++) {
    yield i;
  }
}

const numbers = lazyRange(1, 1000000);
console.log(numbers.next().value); // 1 (only computes first value)
```

**Component Breakdown:**
- `lazyRange`: A generator that produces numbers on demand.
- `numbers.next().value`: Only the first value is computed.

**Syntax Rules:**
- Generator functions do not execute until `next()` is called.
- Each `yield` suspends execution until the next value is requested.

**Constraints and Limitations:**
- Lazy evaluation can make debugging harder because values are computed at different times.
- Not all operations benefit from laziness (e.g., operations that need the entire dataset).

### Annotated Code Examples

#### Example 1: Lazy Map and Filter

```js
function* lazyMap(iterable, fn) {
  for (const item of iterable) {
    yield fn(item);
  }
}

function* lazyFilter(iterable, predicate) {
  for (const item of iterable) {
    if (predicate(item)) yield item;
  }
}

function* lazyRange(start, end) {
  for (let i = start; i <= end; i++) yield i;
}

const result = lazyFilter(
  lazyMap(lazyRange(1, 1000000), (x) => x * 2),
  (x) => x > 10
);

console.log(result.next().value); // 12 (only computes up to 12)
```

**Expected Output:**
```
12
```

**Why This Output:** The pipeline is lazy: only the values needed to produce `12` are computed.

#### Example 2: Infinite Generator

```js
function* naturals() {
  let n = 1;
  while (true) {
    yield n++;
  }
}

const g = naturals();
console.log(g.next().value); // 1
console.log(g.next().value); // 2
console.log(g.next().value); // 3
```

**Expected Output:**
```
1
2
3
```

**Why This Output:** The generator can produce infinite values because it computes them lazily, one at a time.

### Real-World Cases

- **Data streaming:** Processing large files line by line.
- **Infinite sequences:** Generating Fibonacci numbers.
- **Pagination:** Fetching pages on demand.
- **Game loops:** Generating game state frames.

---

## Infinite Generators

### Definitions

**Core Definition:** An infinite generator is a generator that can produce an unbounded sequence of values, potentially forever, without running out of memory.

**Technical Definition:** Per MDN, "Using generator functions, values are not evaluated until needed. Therefore, generators allow us to define potentially infinite data structures". An infinite generator uses a loop that never terminates (e.g., `while (true)`) combined with `yield` to produce values on demand. Since values are computed lazily, the generator does not consume memory for unproduced values.

**Beginner-Friendly Explanation:** An infinite generator is a generator that can keep producing values forever, as long as you keep asking for them. It doesn't run out of memory because it only computes values when you need them.

### Purposes

- To model infinite sequences (e.g., natural numbers, Fibonacci).
- To generate values on demand without precomputing.
- To implement lazy data structures.
- To provide an unbounded source of values for consumers that stop when satisfied.

### Syntax Rules and Structure

```js
function* infiniteSequence() {
  let i = 0;
  while (true) {
    yield i++;
  }
}
```

**Component Breakdown:**
- `while (true)`: An infinite loop.
- `yield i++`: Produces values indefinitely.

**Syntax Rules:**
- Infinite generators must be consumed carefully; `for...of` over an infinite generator will loop forever unless broken.
- Manual `next()` calls give control over how many values are produced.

**Constraints and Limitations:**
- Spreading an infinite generator (`[...gen]`) will cause an infinite loop.
- `for...of` over an infinite generator requires a `break` condition.

### Annotated Code Examples

#### Example 1: Infinite Natural Numbers

```js
function* naturals() {
  let n = 1;
  while (true) {
    yield n++;
  }
}

const g = naturals();
console.log(g.next().value); // 1
console.log(g.next().value); // 2
console.log(g.next().value); // 3
```

**Expected Output:**
```
1
2
3
```

**Why This Output:** The generator produces natural numbers indefinitely, but only computes each value when `next()` is called.

#### Example 2: Taking from an Infinite Generator

```js
function* naturals() {
  let n = 1;
  while (true) yield n++;
}

function* take(iterable, count) {
  let taken = 0;
  for (const item of iterable) {
    if (taken >= count) break;
    yield item;
    taken++;
  }
}

console.log([...take(naturals(), 5)]); // [1, 2, 3, 4, 5]
```

**Expected Output:**
```
[ 1, 2, 3, 4, 5 ]
```

**Why This Output:** The `take` generator limits the infinite generator to 5 values, which are then collected into an array.

### Real-World Cases

- **Fibonacci sequence:** Generating Fibonacci numbers infinitely.
- **Prime numbers:** Generating primes on demand.
- **Unique IDs:** Producing unique IDs indefinitely.
- **Simulations:** Generating infinite event streams.

---

## Generator Use Cases

### Definitions

**Core Definition:** Generator use cases are the practical scenarios where generators provide unique value, including lazy evaluation, custom iteration, state machines, and asynchronous control flow.

**Technical Definition:** Per Chrome for Developers, "Generators are special functions for creating iterators, and iterators are objects with a `next()` method that can be called to get values. Generator functions have several typical use cases, such as using them to iterate over numbers in a Fibonacci sequence". Other use cases include pagination, data streaming, state machines, and custom iterators.

**Beginner-Friendly Explanation:** Generators are useful in many situations: when you need to produce a sequence of values lazily, when you want to create a custom iterator, when you need to model a state machine, or when you want to process data one piece at a time.

### Purposes

- To create custom iterators for data structures.
- To implement lazy evaluation for performance and memory efficiency.
- To model state machines and workflows.
- To generate infinite or large sequences on demand.
- To process streams of data incrementally.

### Syntax Rules and Structure

Generators are used in various patterns. Common patterns include:

```js
// 1. Custom iterator:
function* range(start, end) {
  for (let i = start; i < end; i++) yield i;
}

// 2. Lazy pipeline:
function* map(iterable, fn) {
  for (const item of iterable) yield fn(item);
}

// 3. State machine:
function* trafficLight() {
  while (true) {
    yield "green";
    yield "yellow";
    yield "red";
  }
}
```

**Component Breakdown:**
- **Custom iterator:** Generators provide a concise way to create iterators.
- **Lazy pipeline:** Generators compose lazily for efficient data processing.
- **State machine:** Generators model state transitions naturally.

**Syntax Rules:**
- Generators are ideal when you need to produce values on demand.
- They can be composed using `yield*` or by passing generators to other generators.

**Constraints and Limitations:**
- Generators are single-use; once exhausted, they cannot be restarted.
- Generators add some overhead compared to simple loops.

### Annotated Code Examples

#### Example 1: Custom Iterator for a Tree

```js
function* traverseTree(node) {
  yield node.value;
  for (const child of node.children) {
    yield* traverseTree(child);
  }
}

const tree = {
  value: 1,
  children: [
    { value: 2, children: [] },
    { value: 3, children: [{ value: 4, children: [] }] },
  ],
};

console.log([...traverseTree(tree)]); // [1, 2, 3, 4]
```

**Expected Output:**
```
[ 1, 2, 3, 4 ]
```

**Why This Output:** The generator recursively traverses the tree, yielding each node's value. `yield*` delegates to child generators.

#### Example 2: State Machine

```js
function* trafficLight() {
  while (true) {
    yield "green";
    yield "yellow";
    yield "red";
  }
}

const light = trafficLight();
console.log(light.next().value); // "green"
console.log(light.next().value); // "yellow"
console.log(light.next().value); // "red"
console.log(light.next().value); // "green"
```

**Expected Output:**
```
green
yellow
red
green
```

**Why This Output:** The generator models a traffic light state machine, cycling through states indefinitely.

### Real-World Cases

- **Pagination:** Fetching pages one at a time with `yield`.
- **Data streaming:** Processing file chunks lazily.
- **State machines:** Managing UI states or workflow transitions.
- **Testing:** Generating test data on demand.
- **Game development:** Generating enemy waves or level layouts.

---

## Generators vs Async/Await

### Definitions

**Core Definition:** Generators and `async/await` are both mechanisms for controlling asynchronous flow, but they differ in syntax, purpose, and native support.

**Technical Definition:** Per the comparison, "The most important difference between async/await and generators is that generators are natively supported all the way back to Node.js 4.x, whereas async/await requires Node.js >= 7.6.0". Generators are a general-purpose control-flow mechanism that can be used for asynchronous programming (with a runner like `co`), whereas `async/await` is specifically designed for asynchronous operations built on Promises. Generators use `yield` to pause execution, while `async/await` uses `await` to pause until a Promise settles.

**Beginner-Friendly Explanation:** Generators and `async/await` can both pause and resume code, but they serve different purposes. Generators are a general tool for producing sequences and controlling flow. `async/await` is specifically for working with Promises. In modern JavaScript, `async/await` is the preferred way to write asynchronous code, while generators are used for lazy evaluation, custom iteration, and state machines.

### Purposes

- To understand when to use generators vs `async/await`.
- To recognise that `async/await` is built on Promises, while generators are a general control-flow mechanism.
- To choose the right tool for the task.

### Syntax Rules and Structure

```js
// Generator with yield:
function* gen() {
  const data = yield fetchData();
  console.log(data);
}

// Async/await:
async function asyncFn() {
  const data = await fetchData();
  console.log(data);
}
```

**Component Breakdown:**
- **Generator:** `yield` pauses the function; the caller must resume it with `next()`.
- **Async/await:** `await` pauses the function until the Promise settles; the engine resumes automatically.

**Syntax Rules:**
- Generators are declared with `function*`; async functions with `async function`.
- `yield` can produce multiple values; `await` unwraps a Promise.
- Generators require a runner to handle Promises; `async/await` has built-in support.

**Constraints and Limitations:**
- Generators are more general but require more manual control.
- `async/await` is simpler for asynchronous flows but cannot produce multiple values like generators.
- Async generators (`async function*`) combine both features.

### Annotated Code Examples

#### Example 1: Generator-Based Async Flow

```js
function* fetchUserFlow() {
  const user = yield fetch("/api/user");
  const posts = yield fetch(`/api/posts/${user.id}`);
  console.log(posts);
}

// A simple runner:
function run(gen) {
  const g = gen();
  function step(value) {
    const { value: promise, done } = g.next(value);
    if (done) return;
    promise.then(step);
  }
  step();
}

run(fetchUserFlow);
```

**Expected Output:** The posts are logged after the fetches complete.

**Why This Output:** The generator yields Promises, and the runner resumes it when each Promise resolves. This is how generators were used for async flow before `async/await`.

#### Example 2: Async/Await Equivalent

```js
async function fetchUserFlow() {
  const user = await fetch("/api/user");
  const posts = await fetch(`/api/posts/${user.id}`);
  console.log(posts);
}

fetchUserFlow();
```

**Expected Output:** The posts are logged after the fetches complete.

**Why This Output:** `async/await` achieves the same result with less boilerplate. The engine handles Promise resolution automatically.

### Real-World Cases

- **Generators:** Lazy evaluation, custom iteration, state machines, infinite sequences.
- **`async/await`:** API calls, file I/O, database queries, any Promise-based asynchronous operation.
- **Async generators:** Streaming data from APIs, paginated fetches, real-time data processing.

---

## References

- MDN Web Docs – function* — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*
- MDN Web Docs – yield — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/yield
- MDN Web Docs – yield* — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/yield*
- MDN Web Docs – Generator — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator
- MDN Web Docs – Generator.prototype.next() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator/next
- MDN Web Docs – Generator.prototype.return() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator/return
- MDN Web Docs – Generator.prototype.throw() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator/throw
- MDN Web Docs – Iteration protocols — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – §27.5 Generator Objects — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf
- Chrome for Developers – Generators: The Gnarly Bits — https://developer.chrome.com/blog/generators-the-gnarly-bits
- CoreUI – How to implement lazy evaluation in JavaScript — https://coreui.io/answers/how-to-implement-lazy-evaluation-in-javascript/
- CoreUI – How to create a generator function in JavaScript — https://coreui.io/answers/how-to-create-a-generator-function-in-javascript/
- The Code Barbarian – Async Generators — https://thecodebarbarian.com/async-generators-in-nodejs.html
- Promises.org – Generators — https://www.promisejs.org/generators/