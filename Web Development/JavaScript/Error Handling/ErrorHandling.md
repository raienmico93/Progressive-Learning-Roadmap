# JavaScript Error Handling — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** JavaScript error handling is the set of language mechanisms and programming patterns used to detect, report, and recover from runtime failures during script execution. It centres on the `Error` object, the `throw` statement, and the `try...catch...finally` construct, with additional support for custom error types and structured error metadata.

**Technical Definition:** Per the ECMAScript specification, error handling is governed by the Error Object structure (ECMA-262 §20.5) and the `try`, `catch`, `finally`, and `throw` statements (ECMA-262 §14.15). The specification defines a hierarchy of native error constructors — `EvalError`, `RangeError`, `ReferenceError`, `SyntaxError`, `TypeError`, `URIError`, and `AggregateError` — each inheriting from `Error.prototype` and carrying an `[[ErrorData]]` internal slot. Errors propagate through the call stack until intercepted by a `catch` clause, and the specification mandates that implementations report most errors at the time the relevant language construct is evaluated.

**Beginner-Friendly Explanation:** When something goes wrong in your JavaScript code — a variable doesn't exist, a function is called with the wrong type of data, or a network request fails — JavaScript creates an "Error" object and starts "throwing" it. If nothing catches it, your program stops and the error appears in the console. Error handling lets you intercept these errors, decide what to do about them (show a message, retry, log the problem), and keep your application running smoothly.

### Key Characteristics

- **Runtime vs. parse-time:** Syntax errors are reported before execution; all other error types occur at runtime.
- **Prototype-based hierarchy:** All error types inherit from `Error.prototype`, enabling `instanceof` checks and shared properties (`message`, `name`, `stack`).
- **Propagation through the call stack:** An uncaught `throw` unwinds the stack until a `catch` clause is found, or the program terminates.
- **Structured metadata:** Modern error objects support `cause` (ES2022), `errors` (on `AggregateError`), and `stack` (de facto standard).
- **Extensible:** Custom error classes can be created via `class X extends Error` to encode domain-specific failure information.
- **Serializable:** `Error` objects can be cloned with `structuredClone()` and transferred between Workers via `postMessage()`.

### Prerequisites

- Familiarity with JavaScript syntax, functions, and scope.
- Understanding of the call stack and execution contexts.
- Basic knowledge of object-oriented programming (prototypes, classes, inheritance).
- Awareness of asynchronous patterns (`Promise`, `async/await`) because rejected promises produce unhandled errors.

### Related Programming Areas

- **Asynchronous programming:** Promise rejection and `unhandledrejection` events.
- **Debugging and observability:** Stack traces, logging, error monitoring services.
- **API design:** Throwing structured errors from library functions; documenting error contracts.
- **Security:** Preventing information leakage through error messages; handling DOMException from Web APIs.
- **Testing:** Asserting that specific errors are thrown (`expect().toThrow()`).

### Core Concepts / Features

The following sections cover every core concept and feature listed in the requested topic, each using the uniform structure: Definitions, Purposes, Syntax Rules and Structure, Annotated Code Examples with Expected Outputs, and Real-World Cases.

---

## Errors

### Definitions

**Core Definition:** An error is an exceptional condition that disrupts the normal flow of program execution. In JavaScript, errors are represented as objects created from the `Error` constructor or its subclasses.

**Technical Definition:** An error object is an ordinary object that has an `[[ErrorData]]` internal slot and inherits from `Error.prototype`. When a runtime error is detected, the implementation creates a new native error object and throws it. The specification distinguishes between early errors (detectable before evaluation) and runtime errors (detected during evaluation).

**Beginner-Friendly Explanation:** An error is JavaScript's way of saying "something went wrong and I can't continue normally." Instead of silently producing wrong results, JavaScript stops and creates an Error object that describes what happened. You can then decide whether to fix the problem, report it, or let it stop the program.

### Purposes

- To signal that a pre-condition, post-condition, or invariant has been violated.
- To provide structured diagnostic information (message, type, stack trace) for debugging.
- To allow programs to recover gracefully from expected failure modes.
- To distinguish between different categories of failure so that each can be handled appropriately.

### Syntax Rules and Structure

**General Syntax (Error creation):**
```js
new Error(message)
new Error(message, options)
Error(message)
Error(message, options)
```

**Component Breakdown:**
- `message` (optional): A human-readable description of the error. Converted to a string if not already one.
- `options` (optional, ES2022): An object that may contain a `cause` property. The value of `cause` is assigned to the new error's `cause` data property.
- `new` is optional; calling `Error()` without `new` behaves identically.

**Syntax Rules:**
- The `Error` constructor can be called with or without `new`; both produce an `Error` instance.
- If `message` is provided, it becomes a non-enumerable own property named `"message"` on the new object.
- The `cause` property is non-enumerable, writable, and configurable.

**Constraints and Limitations:**
- `Error` objects are not JSON-serializable by default because `message` and `stack` are non-enumerable. Use `structuredClone()` for cloning.
- The `stack` property is non-standard (though widely implemented) and its format varies between engines.
- `instanceof Error` is unreliable across realms (iframes, vm contexts); the `Error.isError()` proposal (Stage 4) addresses this.

### Annotated Code Examples

#### Example 1: Creating and Throwing a Basic Error

```js
// Step 1: Define a function that validates its input.
function divide(a, b) {
  // Step 2: Check for invalid input (division by zero).
  if (b === 0) {
    // Step 3: Throw a new Error with a descriptive message.
    throw new Error("Division by zero is not allowed.");
  }
  // Step 4: Return the result if no error occurred.
  return a / b;
}

// Step 5: Call the function with valid arguments.
console.log(divide(10, 2)); // Output: 5

// Step 6: Call the function with invalid arguments; an error will be thrown.
try {
  divide(10, 0);
} catch (err) {
  // Step 7: Log the error message to the console.
  console.error(err.message); // Output: "Division by zero is not allowed."
}
```

**Expected Output:**
```
5
Division by zero is not allowed.
```

**Why This Output:** The first call succeeds because `b` is not zero. The second call throws an `Error`; the `catch` block captures it and prints its `message` property. The program does not crash because the error was handled.

#### Example 2: Accessing Error Properties

```js
// Step 1: Create an Error with a message and a cause.
const originalError = new Error("Database connection failed.");
const wrappedError = new Error("Failed to load user profile.", {
  cause: originalError,
});

// Step 2: Inspect the properties.
console.log(wrappedError.message); // "Failed to load user profile."
console.log(wrappedError.name);    // "Error"
console.log(wrappedError.cause);   // Error: Database connection failed.
console.log(wrappedError.stack);   // (stack trace string)
```

**Expected Output:**
```
Failed to load user profile.
Error
Error: Database connection failed.
<stack trace>
```

**Why This Output:** The `message` property holds the human-readable description. `name` defaults to `"Error"` for the base constructor. `cause` holds the original error object, preserving the chain of causation. `stack` provides the call trace.

### Real-World Cases

- **Form validation:** Throwing an `Error` when user input does not meet criteria, then catching it to display a validation message.
- **API client libraries:** Throwing structured errors when HTTP responses indicate failure, so callers can inspect status codes and retry logic.
- **Node.js streams:** Emitting `'error'` events and throwing errors when stream operations fail, enabling pipeline error handling.

---

## `Error` Object

### Definitions

**Core Definition:** The `Error` object is the built-in JavaScript constructor used to create error instances. It serves as the base class for all standard error types and for user-defined error classes.

**Technical Definition:** `Error` is a global intrinsic object. Its prototype object provides the shared `message` and `name` properties, the `toString()` method, and the (non-standard but universally implemented) `stack` property. When called as a constructor, it creates an object with an `[[ErrorData]]` internal slot and installs a non-enumerable `message` property if a message argument is supplied.

**Beginner-Friendly Explanation:** The `Error` object is like a blank form for describing problems. It has fields for a message ("what went wrong"), a name ("what kind of error"), and a stack trace ("where did it happen"). You can fill it out and throw it to tell the rest of your program that something is wrong.

### Purposes

- To provide a standardised container for error information.
- To serve as the prototype for all native and custom error types.
- To enable `instanceof Error` checks for generic error detection.
- To support structured cloning and inter-worker messaging through serialization.

### Syntax Rules and Structure

**General Syntax (Constructor):**
```js
new Error(message, options)
Error(message, options)
```

**Component Breakdown:**
- `message` (string, optional): Human-readable description.
- `options` (object, optional): May contain `{ cause: any }`.

**Instance Properties:**
- `message`: The error description.
- `name`: The error type name (default `"Error"`).
- `stack` (non-standard): Call trace string.
- `cause` (ES2022): Original error that caused this one.

**Prototype Methods:**
- `Error.prototype.toString()`: Returns `"name: message"`.

**Syntax Rules:**
- `Error` can be called without `new`; the result is identical.
- Subclasses must call `super(message, options)` to initialise the `[[ErrorData]]` slot.

**Constraints and Limitations:**
- The `stack` property is not standardised; its format and availability differ across engines.
- `Error.prototype.toString()` returns `"Error"` if `name` is empty and `message` is empty.

### Annotated Code Examples

#### Example 1: Using `Error.prototype.toString()`

```js
// Step 1: Create an Error with a message.
const err = new Error("Something went wrong.");

// Step 2: Call toString().
console.log(err.toString()); // Output: "Error: Something went wrong."

// Step 3: Override the name property.
err.name = "CustomError";
console.log(err.toString()); // Output: "CustomError: Something went wrong."
```

**Expected Output:**
```
Error: Something went wrong.
CustomError: Something went wrong.
```

**Why This Output:** `toString()` concatenates the `name` and `message` properties with a colon and space. Changing `name` changes the output prefix.

#### Example 2: Structured Cloning of an Error

```js
// Step 1: Create an Error with a cause.
const original = new Error("Original failure.");
const wrapped = new Error("Wrapper failure.", { cause: original });

// Step 2: Clone the error using structuredClone().
const clone = structuredClone(wrapped);

// Step 3: Verify the clone.
console.log(clone.message);          // "Wrapper failure."
console.log(clone.cause.message);    // "Original failure."
console.log(clone instanceof Error); // true
```

**Expected Output:**
```
Wrapper failure.
Original failure.
true
```

**Why This Output:** `Error` objects are serializable, so `structuredClone()` produces a deep copy. The `cause` chain is preserved, and the clone remains an `Error` instance.

### Real-World Cases

- **Logging frameworks:** Creating `Error` objects with `cause` chains to preserve root-cause information across layers.
- **Worker communication:** Passing errors between the main thread and web workers via `postMessage()`.
- **Library APIs:** Throwing `Error` instances so that consumers can rely on `instanceof Error` and standard properties.

---

## Error Types

### Definitions

**Core Definition:** JavaScript defines several native error constructors that represent specific categories of runtime failure. Each inherits from `Error` and has a distinct `name`.

**Technical Definition:** The native error constructors — `EvalError`, `RangeError`, `ReferenceError`, `SyntaxError`, `TypeError`, `URIError`, and `AggregateError` — each have their own prototype object whose `[[Prototype]]` is `Error.prototype`. They differ only in the value of their `name` property and the conditions under which the engine throws them.

**Beginner-Friendly Explanation:** JavaScript has different "flavours" of errors for different problems. A `TypeError` means you used the wrong type of value; a `ReferenceError` means you referenced something that doesn't exist; a `RangeError` means a number was outside the allowed range. Knowing the type helps you handle the problem correctly.

### Purposes

- To categorise failures so that handlers can respond appropriately.
- To provide semantic meaning beyond a generic "something went wrong."
- To enable type-specific `instanceof` checks in `catch` blocks.

### Syntax Rules and Structure

All native error constructors follow the same pattern:

```js
new NativeError(message)
new NativeError(message, options)
```

Where `NativeError` is one of: `EvalError`, `RangeError`, `ReferenceError`, `SyntaxError`, `TypeError`, `URIError`, `AggregateError`.

**Sub-feature: `SyntaxError`**

**Definitions:**
- **Core Definition:** An error representing invalid JavaScript syntax.
- **Technical Definition:** Thrown when the parser encounters a token sequence that does not conform to the ECMAScript grammar.
- **Beginner-Friendly Explanation:** You made a typo or wrote code that JavaScript can't understand.

**Purposes:**
- To alert the developer to malformed code before execution.
- To prevent the engine from attempting to run code that cannot be parsed.

**Syntax Rules:** Cannot be caught with `try...catch` for parse-time syntax errors in the same script, because the script fails to parse before execution. Can be caught when using `eval()` or `new Function()`.

**Example:**
```js
try {
  eval("var x = ;"); // Invalid syntax
} catch (err) {
  console.log(err.name);    // "SyntaxError"
  console.log(err.message); // "Unexpected token ';'"
}
```

**Expected Output:**
```
SyntaxError
Unexpected token ';'
```

**Real-World Cases:** Linters and code editors use `SyntaxError` detection to highlight invalid code.

---

**Sub-feature: `ReferenceError`**

**Definitions:**
- **Core Definition:** An error thrown when a non-existent variable is referenced.
- **Technical Definition:** Thrown when a `Reference` record cannot be resolved in the current lexical environment.
- **Beginner-Friendly Explanation:** You tried to use a variable that hasn't been declared.

**Example:**
```js
try {
  console.log(nonExistentVariable);
} catch (err) {
  console.log(err.name);    // "ReferenceError"
  console.log(err.message); // "nonExistentVariable is not defined"
}
```

**Expected Output:**
```
ReferenceError
nonExistentVariable is not defined
```

---

**Sub-feature: `TypeError`**

**Definitions:**
- **Core Definition:** An error thrown when a value is not of the expected type.
- **Technical Definition:** Thrown when an operation receives an operand of an inappropriate type (e.g., calling a non-function, accessing a property of `null`).
- **Beginner-Friendly Explanation:** You tried to do something with a value that doesn't support that operation.

**Example:**
```js
try {
  null.foo;
} catch (err) {
  console.log(err.name);    // "TypeError"
  console.log(err.message); // "Cannot read properties of null (reading 'foo')"
}
```

**Expected Output:**
```
TypeError
Cannot read properties of null (reading 'foo')
```

---

**Sub-feature: `RangeError`**

**Definitions:**
- **Core Definition:** An error thrown when a numeric value is outside the allowable range.
- **Technical Definition:** Thrown when a numeric parameter exceeds its valid bounds (e.g., `new Array(-1)`, `toFixed(200)`).
- **Beginner-Friendly Explanation:** You used a number that is too big, too small, or otherwise not allowed.

**Example:**
```js
try {
  new Array(-1);
} catch (err) {
  console.log(err.name);    // "RangeError"
  console.log(err.message); // "Invalid array length"
}
```

**Expected Output:**
```
RangeError
Invalid array length
```

---

**Sub-feature: `URIError`**

**Definitions:**
- **Core Definition:** An error thrown when a URI-handling function receives an invalid parameter.
- **Technical Definition:** Thrown by `encodeURI()`, `decodeURI()`, `encodeURIComponent()`, or `decodeURIComponent()` when given malformed URI sequences.
- **Beginner-Friendly Explanation:** You gave a web address function a string it can't process.

**Example:**
```js
try {
  decodeURIComponent("%");
} catch (err) {
  console.log(err.name);    // "URIError"
  console.log(err.message); // "URI malformed"
}
```

**Expected Output:**
```
URIError
URI malformed
```

---

**Sub-feature: `EvalError`**

**Definitions:**
- **Core Definition:** An error historically associated with the `eval()` function.
- **Technical Definition:** The specification defines `EvalError` but does not currently throw it in any standard scenario; it is retained for backward compatibility.
- **Beginner-Friendly Explanation:** This error type exists but is essentially never thrown by modern JavaScript engines.

**Note:** `EvalError` is deprecated in practice. It is included in the specification but is not thrown by any current engine in standard operation.

---

**Sub-feature: `AggregateError`**

**Definitions:**
- **Core Definition:** An error that wraps multiple errors into a single error object.
- **Technical Definition:** A subclass of `Error` whose instances have an `errors` own property containing an array of aggregated errors. Thrown by `Promise.any()` when all input promises reject.
- **Beginner-Friendly Explanation:** When several things go wrong at once, `AggregateError` bundles them together so you can see all the problems in one place.

**Example:**
```js
Promise.any([
  Promise.reject(new Error("First failure")),
  Promise.reject(new Error("Second failure")),
]).catch((err) => {
  console.log(err.name);          // "AggregateError"
  console.log(err.errors.length); // 2
  console.log(err.errors[0].message); // "First failure"
});
```

**Expected Output:**
```
AggregateError
2
First failure
```

---

### Real-World Cases (Error Types)

- **`ReferenceError`:** Catching misspelled variable names during development; providing helpful suggestions in developer tools.
- **`TypeError`:** Defensive programming in library functions; validating argument types before use.
- **`RangeError`:** Input validation for numeric parameters (e.g., array sizes, precision values).
- **`URIError`:** Sanitising user-supplied URL parameters before decoding.
- **`AggregateError`:** Reporting all failures from parallel operations (e.g., bulk API calls) in a single response.

---

## `throw`

### Definitions

**Core Definition:** The `throw` statement raises a user-defined exception, immediately halting normal execution and transferring control to the nearest `catch` block.

**Technical Definition:** The `throw` statement evaluates its expression, creates an abrupt completion of type `throw` with the resulting value, and begins stack unwinding. If no `catch` clause intercepts the completion, the program terminates.

**Beginner-Friendly Explanation:** `throw` is how you deliberately trigger an error. When you write `throw new Error("...")`, you're telling JavaScript: "Stop what you're doing and report this problem."

### Purposes

- To signal that a function cannot fulfill its contract.
- To enforce preconditions and invariants.
- To transfer control to an error-handling block.
- To create custom error conditions specific to application logic.

### Syntax Rules and Structure

```js
throw expression;
```

**Component Breakdown:**
- `expression`: Any JavaScript expression. In practice, this should be an `Error` object or an instance of an `Error` subclass.

**Syntax Rules:**
- The `throw` statement must be followed by an expression; it cannot stand alone.
- A line terminator is not allowed between `throw` and the expression (ASI hazard). Wrap the expression in parentheses to avoid this.
- Valid in all contexts where statements are allowed.

**Constraints and Limitations:**
- Throwing non-`Error` values is technically allowed but discouraged because handlers expect standard properties (`message`, `name`, `stack`).
- `throw` inside a `finally` block overrides any previous exception.

### Annotated Code Examples

#### Example 1: Throwing a `TypeError`

```js
// Step 1: Define a validation function.
function assertNumber(value) {
  // Step 2: Check the type.
  if (typeof value !== "number") {
    // Step 3: Throw a TypeError if not a number.
    throw new TypeError(`Expected a number, got ${typeof value}.`);
  }
  // Step 4: Return the value if valid.
  return value;
}

// Step 5: Call with a valid argument.
console.log(assertNumber(42)); // Output: 42

// Step 6: Call with an invalid argument and catch the error.
try {
  assertNumber("hello");
} catch (err) {
  console.log(err.name);    // "TypeError"
  console.log(err.message); // "Expected a number, got string."
}
```

**Expected Output:**
```
42
TypeError
Expected a number, got string.
```

**Why This Output:** `assertNumber(42)` passes the type check and returns 42. `assertNumber("hello")` fails the check, throws a `TypeError`, which is then caught and its properties logged.

#### Example 2: ASI Hazard with `throw`

```js
// This is WRONG — ASI inserts a semicolon after throw.
try {
  throw
    new Error("This will not work as intended.");
} catch (err) {
  console.log(err); // Output: undefined (the throw statement throws undefined)
}
```

**Expected Output:**
```
undefined
```

**Why This Output:** The line terminator after `throw` triggers Automatic Semicolon Insertion, converting the code to `throw; new Error(...);`. The `throw` statement with no expression throws `undefined`.

**Correct version:**
```js
try {
  throw new Error("This works correctly.");
} catch (err) {
  console.log(err.message); // "This works correctly."
}
```

### Real-World Cases

- **Input validation:** Throwing `TypeError` or `RangeError` when function arguments are invalid.
- **State machines:** Throwing an `Error` when an illegal state transition is attempted.
- **Authentication:** Throwing a custom `AuthenticationError` when credentials are invalid.

---

## `try`

### Definitions

**Core Definition:** The `try` block encloses code that may throw an exception, allowing subsequent `catch` and/or `finally` blocks to handle or clean up after the error.

**Technical Definition:** The `try` statement is a compound statement that evaluates a block of statements. If an abrupt completion of type `throw` occurs within the block, the completion is passed to the `catch` clause (if present) or the `finally` clause (if present).

**Beginner-Friendly Explanation:** The `try` block is where you put code that might fail. If it succeeds, great — the `catch` block is skipped. If it fails, the `catch` block takes over.

### Purposes

- To isolate code that may throw an exception.
- To provide a controlled context for error handling.
- To enable cleanup operations through the `finally` clause.

### Syntax Rules and Structure

```js
try {
  tryStatements
} catch (exceptionVar) {
  catchStatements
} finally {
  finallyStatements
}
```

**Component Breakdown:**
- `tryStatements`: The block of statements to execute.
- `exceptionVar` (optional): An identifier or pattern that receives the thrown value.
- `catchStatements`: Code to execute if an exception is thrown.
- `finallyStatements`: Code that always executes before control leaves the construct.

**Syntax Rules:**
- The `try` block must be followed by either a `catch` block, a `finally` block, or both.
- `try`, `catch`, and `finally` blocks must be blocks (curly braces), not single statements.
- The `catch` binding can be omitted (`catch { ... }`) if the exception value is not needed.

**Constraints and Limitations:**
- Cannot catch syntax errors that occur during the initial parsing of the script.
- Exceptions thrown in the `finally` block override any exception from the `try` or `catch` blocks.

### Annotated Code Examples

#### Example 1: Basic `try...catch`

```js
// Step 1: Attempt to execute code that may throw.
try {
  // Step 2: This function does not exist, so a ReferenceError is thrown.
  undefinedFunction();
} catch (err) {
  // Step 3: Handle the error.
  console.log("Caught:", err.name);    // "ReferenceError"
  console.log("Message:", err.message); // "undefinedFunction is not defined"
}
```

**Expected Output:**
```
Caught: ReferenceError
Message: undefinedFunction is not defined
```

**Why This Output:** Calling `undefinedFunction()` throws a `ReferenceError`. The `catch` block receives the error object and logs its `name` and `message`.

#### Example 2: Nested `try` Blocks

```js
// Step 1: Outer try block.
try {
  // Step 2: Inner try block without a catch.
  try {
    throw new Error("Inner error");
  } finally {
    // Step 3: This finally block runs first.
    console.log("Inner finally");
  }
} catch (err) {
  // Step 4: The outer catch handles the inner error.
  console.log("Outer caught:", err.message);
}
```

**Expected Output:**
```
Inner finally
Outer caught: Inner error
```

**Why This Output:** The inner `try` has no `catch`, so the exception propagates to the outer `catch`. The inner `finally` executes before propagation continues.

### Real-World Cases

- **File I/O (Node.js):** Wrapping file read operations in `try...catch` to handle `ENOENT` errors.
- **JSON parsing:** Catching `SyntaxError` from `JSON.parse()` to handle malformed input gracefully.
- **Third-party API calls:** Isolating network calls so that failures do not crash the entire application.

---

## `catch`

### Definitions

**Core Definition:** The `catch` block receives and handles exceptions thrown within the corresponding `try` block.

**Technical Definition:** The `catch` clause contains a block of statements and an optional binding that is initialised with the thrown value when an abrupt completion of type `throw` reaches the `try` statement.

**Beginner-Friendly Explanation:** The `catch` block is where you decide what to do when something goes wrong. You can log the error, show a message to the user, retry the operation, or throw a different error.

### Purposes

- To intercept and handle exceptions before they terminate the program.
- To log diagnostic information about failures.
- To recover from errors by providing fallback behaviour.
- To translate low-level errors into domain-specific errors.

### Syntax Rules and Structure

```js
catch (exceptionVar) {
  catchStatements
}
```

**Component Breakdown:**
- `exceptionVar` (optional): Binds the thrown value to a local variable.
- `catchStatements`: Code to execute when an exception is caught.

**Syntax Rules:**
- The catch binding is optional in ES2019+; you may write `catch { ... }` without parentheses.
- The binding is block-scoped to the `catch` block.
- A `catch` block without a binding does not have access to the thrown value.

**Constraints and Limitations:**
- The `catch` block cannot catch errors thrown in its own body; those propagate outward.
- Re-throwing the caught error (or a new error) is common practice when the handler cannot fully recover.

### Annotated Code Examples

#### Example 1: Catching and Re-throwing with Context

```js
// Step 1: Simulate a low-level operation that fails.
function readConfig() {
  throw new Error("File not found");
}

// Step 2: Wrap the operation and add context.
try {
  readConfig();
} catch (err) {
  // Step 3: Re-throw with a more descriptive message and the original cause.
  throw new Error("Failed to initialise application.", { cause: err });
}
```

**Expected Output:**
```
Uncaught Error: Failed to initialise application.
  [cause]: Error: File not found
```

**Why This Output:** The original `Error` is preserved as the `cause` of the new error, maintaining the full chain of causation.

#### Example 2: Optional Catch Binding

```js
// Step 1: Try to parse malformed JSON.
try {
  JSON.parse("{invalid}");
} catch {
  // Step 2: No binding needed; just handle the error generically.
  console.log("Invalid JSON provided.");
}
```

**Expected Output:**
```
Invalid JSON provided.
```

**Why This Output:** The catch block does not need the error object; it only needs to know that an error occurred. The optional binding simplifies the syntax.

### Real-World Cases

- **API error translation:** Catching a low-level `TypeError` and throwing a domain-specific `ValidationError`.
- **Retry logic:** Catching a network error and retrying the request a limited number of times.
- **Graceful degradation:** Catching an error in a non-critical feature and continuing without it.

---

## `finally`

### Definitions

**Core Definition:** The `finally` block contains code that always executes when control leaves the `try...catch` construct, regardless of whether an exception was thrown or caught.

**Technical Definition:** The `finally` clause is evaluated after the `try` block and any `catch` clause, before control flow exits the entire construct. It executes even if the `try` or `catch` block contains a `return`, `break`, `continue`, or `throw` statement.

**Beginner-Friendly Explanation:** The `finally` block is for cleanup code that must run no matter what — like closing a file, releasing a lock, or hiding a loading spinner.

### Purposes

- To release resources (files, network connections, locks) reliably.
- To restore program state after a failed operation.
- To ensure that cleanup code runs even when an early `return` or `throw` occurs.

### Syntax Rules and Structure

```js
finally {
  finallyStatements
}
```

**Component Breakdown:**
- `finallyStatements`: Statements that always execute before control leaves the construct.

**Syntax Rules:**
- The `finally` block cannot appear without a preceding `try` block.
- If both `catch` and `finally` are present, `finally` executes after `catch`.
- A `return` statement inside `finally` overrides any `return` or `throw` from the `try` or `catch` blocks.

**Constraints and Limitations:**
- Code in `finally` should not throw an exception, as it would override the original error.
- Avoid `return` in `finally` unless you fully understand the control-flow implications.

### Annotated Code Examples

#### Example 1: Cleanup with `finally`

```js
// Step 1: Simulate acquiring a resource.
let resourceOpen = false;

function openResource() {
  resourceOpen = true;
  console.log("Resource opened.");
}

function closeResource() {
  resourceOpen = false;
  console.log("Resource closed.");
}

// Step 2: Use try...finally to guarantee cleanup.
try {
  openResource();
  throw new Error("Operation failed");
} catch (err) {
  console.log("Handled:", err.message);
} finally {
  closeResource();
}

// Step 3: Verify the resource is closed.
console.log("Resource open?", resourceOpen); // false
```

**Expected Output:**
```
Resource opened.
Handled: Operation failed
Resource closed.
Resource open? false
```

**Why This Output:** Even though an error is thrown and caught, the `finally` block executes and closes the resource. The final check confirms cleanup.

#### Example 2: `return` in `finally` Overrides

```js
function test() {
  try {
    return "from try";
  } finally {
    return "from finally";
  }
}

console.log(test()); // "from finally"
```

**Expected Output:**
```
from finally
```

**Why This Output:** The `return` in `finally` overrides the `return` in `try`. This is a dangerous pattern and should generally be avoided.

### Real-World Cases

- **Database transactions:** Committing or rolling back a transaction in `finally`.
- **UI state:** Hiding a loading spinner in `finally` regardless of whether the data fetch succeeded.
- **File handles:** Closing file descriptors in Node.js `fs` operations.

---

## Error Propagation

### Definitions

**Core Definition:** Error propagation is the process by which an exception travels up the call stack from the point where it was thrown to the nearest enclosing `catch` block.

**Technical Definition:** When a `throw` statement executes, the current execution context is terminated with an abrupt completion of type `throw`. The completion propagates to the caller's context. If the caller has a `try` statement with a `catch` clause, the completion is converted into a normal completion and the `catch` block executes. Otherwise, propagation continues to the next caller, and so on, until the global execution context, where the program terminates.

**Beginner-Friendly Explanation:** When an error is thrown, JavaScript doesn't just stop immediately — it "bubbles up" through the functions that called the failing function, looking for someone who can handle it. If no one catches it, the program crashes.

### Purposes

- To allow errors to be handled at the appropriate level of abstraction.
- To separate error detection (where the error occurs) from error handling (where the error is best dealt with).
- To enable centralised error handling at the application boundary.

### Syntax Rules and Structure

Error propagation is not a separate syntax construct; it is a behaviour of the `throw` statement and the `try...catch` statement. The relevant syntax is:

```js
// In a nested function:
function inner() {
  throw new Error("Inner failure");
}

function outer() {
  inner(); // The error propagates out of outer().
}

try {
  outer(); // The error propagates here and is caught.
} catch (err) {
  // Handle the error.
}
```

**Constraints and Limitations:**
- Errors cannot be caught by a `try` block that has already completed execution.
- Asynchronous errors (in callbacks, promise rejections) do not propagate synchronously through the call stack; they must be handled via promise chains or global handlers.

### Annotated Code Examples

#### Example 1: Propagation Through Multiple Frames

```js
// Step 1: Deepest function throws.
function level3() {
  throw new Error("Failure at level 3");
}

// Step 2: Middle function calls level3 without catching.
function level2() {
  level3();
}

// Step 3: Top function calls level2.
function level1() {
  level2();
}

// Step 4: Catch the propagated error at the top.
try {
  level1();
} catch (err) {
  console.log("Caught at top:", err.message);
}
```

**Expected Output:**
```
Caught at top: Failure at level 3
```

**Why This Output:** The error originates in `level3`, propagates through `level2` and `level1` (neither of which catches it), and is finally caught in the top-level `try` block.

#### Example 2: Asynchronous Error Propagation

```js
// Step 1: An async function that rejects.
async function fetchData() {
  throw new Error("Network error");
}

// Step 2: Handle the rejection with .catch().
fetchData().catch((err) => {
  console.log("Caught async:", err.message);
});

// Step 3: Unhandled rejection (would trigger unhandledrejection event).
fetchData(); // No handler attached.
```

**Expected Output:**
```
Caught async: Network error
```
(And potentially an unhandled rejection warning for the second call.)

**Why This Output:** The first call attaches a `.catch()` handler, so the rejection is handled. The second call has no handler, so the rejection is unhandled and reported to the global `unhandledrejection` event.

### Real-World Cases

- **Express.js middleware:** Throwing errors in route handlers; Express propagates them to error-handling middleware.
- **React error boundaries:** Catching errors in component trees and rendering a fallback UI.
- **Node.js domains:** Deprecated, but historically used to centralise error handling across asynchronous operations.

---

## Custom Errors

### Definitions

**Core Definition:** A custom error is a user-defined error type created by extending the built-in `Error` class, typically to represent a domain-specific failure condition.

**Technical Definition:** In ES2015+, custom errors are created using the `class` syntax: `class CustomError extends Error { ... }`. The subclass constructor must call `super(message, options)` to initialise the `[[ErrorData]]` slot and inherit `message`, `name`, and `stack` behaviour.

**Beginner-Friendly Explanation:** Sometimes the built-in error types aren't specific enough. If your application has a `PaymentError` or `ValidationError`, you can create your own error class that extends `Error` and add extra properties like `code` or `field`.

### Purposes

- To represent domain-specific failure conditions with meaningful names.
- To carry additional metadata (error codes, affected fields, HTTP status).
- To enable precise `instanceof` checks in `catch` blocks.
- To improve readability and maintainability of error-handling code.

### Syntax Rules and Structure

```js
class CustomError extends Error {
  constructor(message, options) {
    super(message, options);
    this.name = "CustomError";
    // Additional properties...
  }
}
```

**Component Breakdown:**
- `extends Error`: Inherits from the built-in `Error` class.
- `super(message, options)`: Calls the `Error` constructor to initialise the base object.
- `this.name = "CustomError"`: Sets the error name for better identification.
- Additional properties: Domain-specific metadata.

**Syntax Rules:**
- The subclass constructor must call `super()` before accessing `this`.
- Setting `this.name` is recommended so that `toString()` and logs show the custom name.
- The `stack` property is inherited automatically in modern engines.

**Constraints and Limitations:**
- In some older transpiled environments (Babel, TypeScript targeting ES5), extending built-ins can break `instanceof` checks.
- Custom error classes should not throw errors in their constructor (except for argument validation) to avoid recursive failures.

### Annotated Code Examples

#### Example 1: Basic Custom Error

```js
// Step 1: Define a custom error class.
class ValidationError extends Error {
  constructor(message, field) {
    // Step 2: Call the parent constructor with the message.
    super(message);
    // Step 3: Set the error name.
    this.name = "ValidationError";
    // Step 4: Add a domain-specific property.
    this.field = field;
  }
}

// Step 5: Use the custom error.
function validateAge(age) {
  if (typeof age !== "number" || age < 0) {
    throw new ValidationError("Age must be a non-negative number.", "age");
  }
  return true;
}

// Step 6: Catch and inspect.
try {
  validateAge(-5);
} catch (err) {
  console.log(err instanceof ValidationError); // true
  console.log(err instanceof Error);           // true
  console.log(err.name);                       // "ValidationError"
  console.log(err.field);                      // "age"
  console.log(err.message);                    // "Age must be a non-negative number."
}
```

**Expected Output:**
```
true
true
ValidationError
age
Age must be a non-negative number.
```

**Why This Output:** The `ValidationError` class extends `Error`, so instances are both `ValidationError` and `Error`. The `field` property carries additional context.

#### Example 2: Custom Error with `cause`

```js
class DatabaseError extends Error {
  constructor(message, options) {
    super(message, options);
    this.name = "DatabaseError";
  }
}

try {
  try {
    throw new Error("Connection timeout");
  } catch (originalErr) {
    throw new DatabaseError("Failed to fetch user.", { cause: originalErr });
  }
} catch (err) {
  console.log(err.name);              // "DatabaseError"
  console.log(err.cause.message);     // "Connection timeout"
}
```

**Expected Output:**
```
DatabaseError
Connection timeout
```

**Why This Output:** The `cause` option is passed through to the `Error` constructor, preserving the original error for debugging.

### Real-World Cases

- **Payment processing:** `PaymentError` with `code` and `transactionId` properties.
- **Form validation:** `ValidationError` with `field` and `value` properties.
- **HTTP clients:** `HttpError` with `status` and `response` properties.

---

## Error Subclassing

### Definitions

**Core Definition:** Error subclassing is the practice of creating new error classes that inherit from `Error` (or from another error subclass) to form a hierarchy of error types.

**Technical Definition:** JavaScript's prototype chain allows error subclasses to inherit the `[[ErrorData]]` internal slot, the `message` and `name` properties, and the `stack` accessor. Subclasses can define their own prototype methods and static properties.

**Beginner-Friendly Explanation:** Just as you can have a `Vehicle` class and a `Car` subclass, you can have an `Error` class and a `NetworkError` subclass. This lets you organise errors into categories and handle them differently.

### Purposes

- To create a taxonomy of errors for an application or library.
- To enable hierarchical `instanceof` checks (a `NetworkError` is also an `Error`).
- To share common behaviour among related error types.
- To support framework-level error handling based on error category.

### Syntax Rules and Structure

```js
class BaseError extends Error {
  constructor(message, options) {
    super(message, options);
    this.name = this.constructor.name;
  }
}

class NetworkError extends BaseError {
  constructor(message, options) {
    super(message, options);
    this.statusCode = options?.statusCode;
  }
}

class TimeoutError extends NetworkError {
  constructor(message, options) {
    super(message, options);
    this.timeoutMs = options?.timeoutMs;
  }
}
```

**Component Breakdown:**
- `BaseError`: An intermediate class that sets `name` automatically.
- `NetworkError`: Adds network-specific properties.
- `TimeoutError`: A more specific subclass of `NetworkError`.

**Syntax Rules:**
- Each subclass constructor must call `super()` before accessing `this`.
- `this.constructor.name` automatically resolves to the subclass name in modern engines.
- The prototype chain is: `TimeoutError.prototype` → `NetworkError.prototype` → `BaseError.prototype` → `Error.prototype`.

**Constraints and Limitations:**
- Deep inheritance chains can complicate error handling; prefer shallow hierarchies.
- `instanceof` checks fail across realms; `Error.isError()` (Stage 4) provides a cross-realm-safe alternative.

### Annotated Code Examples

#### Example 1: Hierarchical Error Handling

```js
// Step 1: Define the hierarchy.
class AppError extends Error {
  constructor(message, options) {
    super(message, options);
    this.name = this.constructor.name;
  }
}

class NetworkError extends AppError {
  constructor(message, options) {
    super(message, options);
    this.statusCode = options?.statusCode ?? null;
  }
}

class TimeoutError extends NetworkError {
  constructor(message, options) {
    super(message, options);
    this.timeoutMs = options?.timeoutMs ?? 5000;
  }
}

// Step 2: Simulate a timeout error.
try {
  throw new TimeoutError("Request timed out", { statusCode: 408, timeoutMs: 10000 });
} catch (err) {
  // Step 3: Handle based on hierarchy.
  if (err instanceof TimeoutError) {
    console.log("Timeout after", err.timeoutMs, "ms");
  } else if (err instanceof NetworkError) {
    console.log("Network error with status", err.statusCode);
  } else if (err instanceof AppError) {
    console.log("Application error:", err.message);
  } else {
    console.log("Unknown error");
  }
}
```

**Expected Output:**
```
Timeout after 10000 ms
```

**Why This Output:** The `instanceof` chain matches `TimeoutError` first because it is the most specific type. If the check order were reversed, `AppError` would match first.

#### Example 2: Automatic `name` Assignment

```js
class HttpError extends Error {
  constructor(message, status) {
    super(message);
    this.name = this.constructor.name;
    this.status = status;
  }
}

const err = new HttpError("Not Found", 404);
console.log(err.name);    // "HttpError"
console.log(err.message); // "Not Found"
console.log(err.status);  // 404
```

**Expected Output:**
```
HttpError
Not Found
404
```

**Why This Output:** Using `this.constructor.name` automatically sets the `name` property to the subclass name, avoiding manual string duplication.

### Real-World Cases

- **Framework error hierarchies:** Angular, React, and Vue define custom error types for framework-specific failures.
- **Database drivers:** PostgreSQL, MySQL, and MongoDB drivers define error subclasses for connection, query, and transaction failures.
- **HTTP clients:** Axios and Fetch wrappers define `HttpError` with status codes and response bodies.

---

## Error Messages

### Definitions

**Core Definition:** The error message is the human-readable string that describes what went wrong, stored in the `message` property of an `Error` object.

**Technical Definition:** The `message` property is a non-enumerable, writable, configurable own property of an `Error` instance. It is initialised from the first argument passed to the `Error` constructor (if any) and is included in the output of `Error.prototype.toString()`.

**Beginner-Friendly Explanation:** The error message is the part of the error that explains the problem in plain language. Good error messages tell you exactly what went wrong, where, and ideally how to fix it.

### Purposes

- To communicate the nature of the failure to developers.
- To provide actionable information for debugging.
- To enable programmatic parsing of error conditions (when combined with structured metadata).
- To inform end-users through translated, user-friendly messaging.

### Syntax Rules and Structure

The `message` property is set automatically when an argument is passed to the `Error` constructor:

```js
new Error("Your message here")
```

It can also be set or modified after creation:

```js
const err = new Error();
err.message = "Updated message";
```

**Syntax Rules:**
- If no message is provided, `message` defaults to an empty string.
- The message is converted to a string using `ToString()` if a non-string is passed.
- In `toString()`, the message is concatenated after the name with `": "`.

**Constraints and Limitations:**
- Error messages should not contain sensitive information (passwords, tokens, internal paths).
- Messages intended for end-users should be localised separately from developer-facing messages.
- Very long messages can truncate in some logging systems.

### Annotated Code Examples

#### Example 1: Constructing Descriptive Messages

```js
// Step 1: Create an error with a detailed message.
const err = new Error(
  `Failed to fetch user data: server responded with status 500 ` +
  `for URL "https://api.example.com/users/42".`
);

// Step 2: Log the message.
console.log(err.message);
```

**Expected Output:**
```
Failed to fetch user data: server responded with status 500 for URL "https://api.example.com/users/42".
```

**Why This Output:** The message includes the operation, the failure reason, and the specific resource involved, providing maximum diagnostic value.

#### Example 2: Message vs. Name in `toString()`

```js
const err = new Error("Something went wrong.");
console.log(err.toString()); // "Error: Something went wrong."

err.name = "DatabaseError";
console.log(err.toString()); // "DatabaseError: Something went wrong."
```

**Expected Output:**
```
Error: Something went wrong.
DatabaseError: Something went wrong.
```

**Why This Output:** `toString()` always combines `name` and `message`; changing either property changes the output.

### Real-World Cases

- **User-facing validation:** Translating technical messages into user-friendly language ("Please enter a valid email address.").
- **Developer debugging:** Including request IDs, URLs, and parameter values in error messages for production logs.
- **API responses:** Returning standardised error codes alongside human-readable messages.

---

## Error Stacks

### Definitions

**Core Definition:** The error stack is a string representation of the call stack at the moment the error was created, showing the sequence of function calls that led to the error.

**Technical Definition:** The `stack` property is a non-standard (but universally implemented) property of `Error` instances. It is an accessor property on `Error.prototype` in some engines and a data property in others. Its format varies between engines, but generally lists function names, file names, line numbers, and column numbers, starting from the most recent call.

**Beginner-Friendly Explanation:** The stack trace is like a receipt that shows the path your code took to reach the error. It tells you which function called which other function, and exactly where in the code the error happened.

### Purposes

- To identify the exact location where an error was thrown.
- To trace the sequence of function calls that led to the error.
- To debug complex, deeply nested or asynchronous code.

### Syntax Rules and Structure

The `stack` property is accessed directly:

```js
const err = new Error("Test");
console.log(err.stack);
```

There is no constructor option for setting the stack; it is captured automatically when the error is created.

**Syntax Rules:**
- `stack` is not enumerable, so it does not appear in `for...in` loops or `Object.keys()`.
- In V8 (Node.js, Chrome), it is a getter-setter pair on `Error.prototype`.
- In SpiderMonkey (Firefox) and JavaScriptCore (Safari), it is a data property on individual error instances.

**Constraints and Limitations:**
- The format is non-standard; parsing stack traces across engines requires normalisation.
- `Error.stackTraceLimit` (V8-specific) controls the number of frames collected (default: 10).
- `Error.captureStackTrace(targetObject, constructorOpt)` can install a stack on a custom object and omit frames above a given constructor.

### Annotated Code Examples

#### Example 1: Reading a Stack Trace

```js
function a() {
  b();
}

function b() {
  c();
}

function c() {
  throw new Error("Deep error");
}

try {
  a();
} catch (err) {
  // The stack trace shows the call chain: c -> b -> a -> (top level).
  console.log(err.stack);
}
```

**Expected Output (approximate, engine-dependent):**
```
Error: Deep error
    at c (<anonymous>:8:9)
    at b (<anonymous>:4:3)
    at a (<anonymous>:1:3)
    at <anonymous>:11:3
```

**Why This Output:** The stack trace lists the error message, then each function call in reverse chronological order (most recent first).

#### Example 2: `Error.captureStackTrace()` for Custom Errors

```js
class CustomError extends Error {
  constructor(message) {
    super(message);
    this.name = "CustomError";
    // Install a stack that omits the constructor frame.
    Error.captureStackTrace(this, CustomError);
  }
}

const err = new CustomError("Custom failure");
console.log(err.stack);
```

**Expected Output (approximate):**
```
CustomError: Custom failure
    at <anonymous>:10:13
```

**Why This Output:** `Error.captureStackTrace(this, CustomError)` omits the `CustomError` constructor from the trace, showing only the call site.

### Real-World Cases

- **Production error monitoring:** Sentry, Bugsnag, and Rollbar parse stack traces to group and prioritise errors.
- **Development debugging:** Browser DevTools display clickable stack traces for quick navigation.
- **Custom logging:** Including stack traces in structured log entries for post-mortem analysis.

---

## Error Causes

### Definitions

**Core Definition:** The `cause` property is an ES2022 feature that allows an error to reference the original error that triggered it, enabling error chaining.

**Technical Definition:** The `cause` data property is installed on an `Error` instance when the `options` argument contains a `cause` property. It is non-enumerable, writable, and configurable. The value can be any JavaScript value, not necessarily an `Error`.

**Beginner-Friendly Explanation:** Sometimes you catch an error and need to throw a new one with a better message. The `cause` property lets you attach the original error to the new one, so you don't lose the original information.

### Purposes

- To preserve the original error when wrapping it in a more descriptive one.
- To build a chain of errors that traces back to the root cause.
- To provide structured diagnostic data (not just strings) for machine parsing.

### Syntax Rules and Structure

```js
throw new Error("Wrapper message", { cause: originalError });
```

**Component Breakdown:**
- `"Wrapper message"`: The new, more descriptive error message.
- `{ cause: originalError }`: The options object with the original error.

**Syntax Rules:**
- The `cause` property is installed only if the `options` argument is an object and has a `cause` property.
- The value of `cause` can be any type: an `Error`, a string, a plain object, etc.
- The `cause` property is not inherited; it is an own property of the instance.

**Constraints and Limitations:**
- The `cause` property is non-enumerable, so it does not appear in `JSON.stringify()` by default.
- Not all environments support ES2022 features; transpilation or polyfills may be required for older runtimes.

### Annotated Code Examples

#### Example 1: Chaining Errors with `cause`

```js
// Step 1: A low-level function throws a specific error.
function readFile(path) {
  throw new Error(`File not found: ${path}`);
}

// Step 2: A higher-level function catches and wraps.
function loadConfig() {
  try {
    readFile("/etc/app/config.json");
  } catch (err) {
    throw new Error("Failed to load application configuration.", { cause: err });
  }
}

// Step 3: Catch the wrapped error and inspect the chain.
try {
  loadConfig();
} catch (err) {
  console.log(err.message);         // "Failed to load application configuration."
  console.log(err.cause.message);   // "File not found: /etc/app/config.json"
}
```

**Expected Output:**
```
Failed to load application configuration.
File not found: /etc/app/config.json
```

**Why This Output:** The `cause` property provides access to the original error, creating a chain of causation that is easy to follow.

#### Example 2: Structured Data as Cause

```js
function makeRSA(p, q) {
  if (!Number.isInteger(p) || !Number.isInteger(q)) {
    throw new Error("RSA key generation requires integer inputs.", {
      cause: { code: "NonInteger", values: [p, q] },
    });
  }
}

try {
  makeRSA(3.5, 7);
} catch (err) {
  console.log(err.cause.code);    // "NonInteger"
  console.log(err.cause.values);  // [3.5, 7]
}
```

**Expected Output:**
```
NonInteger
[ 3.5, 7 ]
```

**Why This Output:** The `cause` can carry structured data for machine parsing, separate from the human-readable message.

### Real-World Cases

- **Database connection wrappers:** Wrapping a low-level connection error with a higher-level `DatabaseError` while preserving the original.
- **API clients:** Wrapping network errors with request context (URL, method, status).
- **Configuration loaders:** Chaining file-read errors with parsing errors.

---

## Aggregate Errors

### Definitions

**Core Definition:** `AggregateError` is an `Error` subclass that bundles multiple errors into a single error object, accessible via the `errors` property.

**Technical Definition:** `AggregateError` is a native error constructor introduced in ES2021. Its instances have an own `errors` property containing an array of the aggregated errors. It is thrown by `Promise.any()` when all input promises reject, and can also be thrown manually.

**Beginner-Friendly Explanation:** When several things fail at once — like multiple API calls in parallel — `AggregateError` lets you collect all the individual errors into one object, so you can handle them together.

### Purposes

- To report multiple failures from a single operation.
- To avoid losing error information when parallel operations fail.
- To provide a structured way to inspect all errors in a batch.

### Syntax Rules and Structure

```js
new AggregateError(errors)
new AggregateError(errors, message)
new AggregateError(errors, message, options)
```

**Component Breakdown:**
- `errors`: An iterable (usually an array) of error values. These need not be actual `Error` instances.
- `message` (optional): A human-readable aggregate description.
- `options` (optional): May contain `cause`.

**Instance Properties:**
- `errors`: An array of the aggregated errors, in the same order as the input iterable.
- `message`: The aggregate message.
- `name`: `"AggregateError"`.

**Syntax Rules:**
- `AggregateError` can be called with or without `new`.
- The `errors` argument is required; omitting it throws a `TypeError`.
- The `errors` array is a copy; modifying it after construction does not affect the original iterable.

**Constraints and Limitations:**
- The `errors` array is not frozen; it can be mutated after creation.
- The individual errors in the array may not be `Error` instances; check their types before accessing `message` or `stack`.

### Annotated Code Examples

#### Example 1: Catching `AggregateError` from `Promise.any()`

```js
// Step 1: Create multiple rejecting promises.
const promises = [
  Promise.reject(new Error("First failure")),
  Promise.reject(new Error("Second failure")),
  Promise.reject(new Error("Third failure")),
];

// Step 2: Use Promise.any() and catch the AggregateError.
Promise.any(promises).catch((err) => {
  console.log(err instanceof AggregateError); // true
  console.log(err.name);                      // "AggregateError"
  console.log(err.errors.length);             // 3
  err.errors.forEach((e) => console.log(e.message));
});
```

**Expected Output:**
```
true
AggregateError
3
First failure
Second failure
Third failure
```

**Why This Output:** `Promise.any()` rejects with an `AggregateError` when all promises reject. The `errors` array contains the individual rejection reasons in the order the promises were passed.

#### Example 2: Manually Throwing an `AggregateError`

```js
try {
  throw new AggregateError(
    [new Error("Validation failed for field A"), new Error("Validation failed for field B")],
    "Multiple validation errors occurred."
  );
} catch (err) {
  console.log(err.message);   // "Multiple validation errors occurred."
  console.log(err.errors[0].message); // "Validation failed for field A"
  console.log(err.errors[1].message); // "Validation failed for field B"
}
```

**Expected Output:**
```
Multiple validation errors occurred.
Validation failed for field A
Validation failed for field B
```

**Why This Output:** The `AggregateError` constructor accepts an iterable of errors and an optional message. The `errors` property provides access to each individual error.

### Real-World Cases

- **Bulk API requests:** Aggregating failures from multiple parallel HTTP requests.
- **Form validation:** Collecting all validation errors across multiple fields into one object.
- **Database batch operations:** Reporting all failed rows in a bulk insert or update.

---

## Error Handling Best Practices

### Definitions

**Core Definition:** Error handling best practices are the established patterns and principles for writing robust, maintainable, and debuggable error-handling code.

**Technical Definition:** Best practices encompass throw discipline (only throw `Error` instances), catch specificity (handle only errors you can recover from), resource safety (`finally` for cleanup), error chaining (`cause`), and user-facing message localisation.

**Beginner-Friendly Explanation:** These are the rules of thumb that experienced developers follow to avoid common mistakes — like catching errors and doing nothing, throwing strings instead of `Error` objects, or showing technical messages to end-users.

### Purposes

- To ensure that errors are not silently swallowed.
- To provide maximum diagnostic information for developers.
- To protect end-users from confusing or sensitive technical details.
- To maintain predictable control flow in the presence of failures.

### Key Practices

**Practice 1: Always Throw `Error` Instances**

```js
// GOOD
throw new Error("Something failed");

// BAD
throw "Something failed"; // No stack trace, no message property
```

**Practice 2: Catch Only What You Can Handle**

```js
// GOOD: Handle specific error types.
try {
  JSON.parse(input);
} catch (err) {
  if (err instanceof SyntaxError) {
    return defaultValue;
  }
  throw err; // Re-throw errors you cannot handle.
}

// BAD: Swallow all errors.
try {
  riskyOperation();
} catch (err) {
  // Silent failure — debugging becomes impossible.
}
```

**Practice 3: Use `finally` for Cleanup**

```js
let connection;
try {
  connection = openConnection();
  connection.execute(query);
} finally {
  connection?.close();
}
```

**Practice 4: Preserve Error Chains with `cause`**

```js
try {
  await fetchData();
} catch (err) {
  throw new Error("Failed to load dashboard.", { cause: err });
}
```

**Practice 5: Localise User-Facing Messages**

```js
const USER_MESSAGES = {
  NetworkError: "Unable to connect. Please check your internet connection.",
  ValidationError: "Please correct the highlighted fields.",
};

function getUserMessage(err) {
  return USER_MESSAGES[err.name] ?? "An unexpected error occurred.";
}
```

**Practice 6: Log with Structured Data**

```js
console.error({
  timestamp: new Date().toISOString(),
  name: err.name,
  message: err.message,
  stack: err.stack,
  cause: err.cause?.message,
});
```

**Practice 7: Handle Unhandled Rejections Globally**

```js
// Browser
window.addEventListener("unhandledrejection", (event) => {
  console.error("Unhandled rejection:", event.reason);
  event.preventDefault(); // Prevent default console logging.
});

// Node.js
process.on("unhandledRejection", (reason, promise) => {
  console.error("Unhandled rejection at:", promise, "reason:", reason);
});
```

**Practice 8: Use `Error.isError()` for Cross-Realm Checks (Stage 4)**

```js
// Instead of `instanceof Error`, which fails across realms:
if (Error.isError(value)) {
  // Handle the error.
}
```

### Real-World Cases

- **Production applications:** Layered error handling with local `try/catch` at trust boundaries, framework-level middleware, and global handlers.
- **Libraries:** Documenting error contracts (which errors a function may throw) and using custom error classes.
- **React applications:** Error boundaries catch rendering errors and display fallback UI.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) – Error Objects — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf
- MDN Web Docs – Error — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error
- MDN Web Docs – try...catch — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/try...catch
- MDN Web Docs – throw — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/throw
- MDN Web Docs – Error: cause — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error/cause
- MDN Web Docs – AggregateError — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/AggregateError
- MDN Web Docs – Error.captureStackTrace() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error/captureStackTrace
- MDN Web Docs – Error.prototype.stack — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error/stack
- MDN Web Docs – JavaScript debugging and error handling — https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Scripting/Debugging_JavaScript
- TC39 Proposal – Error.isError — https://github.com/tc39/proposal-is-error
- TC39 Proposal – Promise.any + AggregateError — https://www.proposals.es
- Node.js Documentation – Errors — https://nodejs.org/api/errors.html
- V8 Documentation – Stack Trace API — https://v8.dev/docs/stack-trace-api
- MDN Web Docs – DOMException — https://developer.mozilla.org/en-US/docs/Web/API/DOMException