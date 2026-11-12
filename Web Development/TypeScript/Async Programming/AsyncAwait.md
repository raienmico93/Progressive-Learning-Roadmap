# Async/Await in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **`async`/`await`** is a language feature that provides **syntactic sugar over promise-based asynchronous programming**, allowing asynchronous code to be written in a **synchronous-looking style**. An **async function** always returns a `Promise<T>`, and the **`await` operator** pauses execution of the async function until the awaited promise settles, returning its resolved value or throwing its rejection reason. TypeScript types both constructs precisely: `async` functions have `Promise<T>` return types, and `await` unwraps `Promise<T>` to `T`.

**Technical Definition.** An `async` function is a function declared with the `async` keyword whose body is executed in an implicit promise context. Its return type is always `Promise<T>`, where `T` is the inferred or annotated return type. The `await` operator may appear only inside `async` functions (or at the top level of ES modules with `module: "esnext"`/`"nodenext"`), and it suspends the function until the operand settles. Rejections propagate as **thrown exceptions**, which can be caught with `try`/`catch`. Since TypeScript 4.4, `catch` variables are typed `unknown` under `strict`, requiring narrowing before use. `Awaited<T>` models `await`'s recursive unwrapping at the type level.

**Beginner-Friendly Explanation.** `async`/`await` lets you **write asynchronous code that reads like normal synchronous code**. You mark a function as `async`, and inside it you can `await` a promise—the function **pauses** until the promise resolves, then continues with the value. If the promise rejects, it **throws** an error that you can catch with `try`/`catch`. TypeScript knows that an `async` function returns a `Promise<T>` and that `await` gives you back `T`, so you get full type safety without manually chaining `.then()` calls.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **`async` functions return `Promise<T>`** | The return type is always a promise, even if the body returns a plain value. |
| **`await` unwraps promises** | `await Promise<T>` yields `T`; nested promises are recursively unwrapped. |
| **Sequential-looking flow** | Async code reads top-to-bottom, unlike `.then()` chains. |
| **Errors become exceptions** | Rejections are thrown and caught with `try`/`catch`. |
| **`catch` is `unknown`** | Under `strict`, catch variables must be narrowed before use. |
| **Top-level `await`** | Supported in ES modules with `module: "esnext"` or `"nodenext"`. |
| **`Awaited<T>`** | Models `await`'s recursive unwrapping at the type level. |
| **Performance-sensitive** | Sequential `await` calls slow down independent operations; use `Promise.all` for concurrency. |

---

### Prerequisites

- **TypeScript basics**: functions, parameters, return types, and generics.
- **Promises**: `Promise<T>`, `.then()`, `.catch()`, and combinators.
- **`try`/`catch`/`finally`** syntax.
- **The `unknown` type** and type narrowing with `instanceof Error`.
- **`Awaited<T>`** and `ReturnType<T>`.
- **A TypeScript environment** with `target: "es2017"` or later (or downleveling).

---

### Related Programming Areas

- **Promises** — the foundation that `async`/`await` builds on.
- **Error handling** — `try`/`catch` and the `unknown` catch variable.
- **Concurrency** — `Promise.all`, `Promise.allSettled`, and `Promise.race`.
- **Cancellation** — `AbortController` for fetch and long-running operations.
- **Top-level `await`** — module-level asynchronous initialization.
- **`Awaited<T>`** — type-level promise unwrapping.

---

## Core Concepts / Features

---

## 1. Async Functions

### Definitions

**Core Definition.** An **async function** is a function declared with the `async` keyword that **always returns a promise** and may contain `await` expressions.

**Technical Definition.** An `async` function's return type is always `Promise<T>`, where `T` is the annotated or inferred return type. If the body returns a value of type `T`, the function returns `Promise<T>`; if it throws, the returned promise is rejected. The `async` keyword enables the use of `await` inside the function body.

**Beginner-Friendly Explanation.** An async function is **a function that always gives you a promise**. You write `async function fetchUser(): Promise<User>`, and inside you can `await` other promises. Callers always get a promise back, even if you return a plain value.

### Purposes

- **To write asynchronous functions** with synchronous-looking control flow.
- **To enable `await` expressions** inside the function body.
- **To propagate resolved types automatically** as `Promise<T>`.
- **To convert synchronous logic** into asynchronous logic without changing the structure.
- **To integrate with promise-based APIs** such as `fetch` and database clients.

### Syntax Rules and Structure

**General syntax:**

```typescript
async function functionName(params: Type): Promise<T> {
  // await expressions allowed here
  return value; // T is wrapped in Promise<T>
}
```

**Arrow function:**

```typescript
const fetchUser = async (id: string): Promise<User> => {
  return { id, name: "Ada" };
};
```

**Component breakdown:**

- `async` — the keyword that makes the function return a promise and enables `await`.
- `: Promise<T>` — the return type annotation (inferred if omitted).
- `return value` — the value is wrapped in a resolved promise of type `T`.

**Syntax rules.**

- The return type is **always** `Promise<T>`, even if the body returns `T` synchronously.
- If no return type is annotated, TypeScript infers `Promise<T>` from the returned value.
- `async` functions may throw; thrown errors reject the returned promise.
- `async` may be combined with arrow functions, methods, and class methods.

**Constraints and limitations.**

- `await` can only be used inside `async` functions (or top-level modules).
- `async` functions cannot be constructors.
- An `async` function that never returns has return type `Promise<never>`.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic async function

```typescript
// example_async_basic.ts
// Step 1: Declare an async function with a typed return.
async function fetchUser(id: string): Promise<{ id: string; name: string }> {
  // Step 2: Simulate async work.
  return { id, name: "Ada" };
}

// Step 3: Call the function and await the result.
async function main() {
  const user = await fetchUser("u1");
  console.log(user.name); // Expected: Ada
}

main();
```

**Setup.** Compile with `npx tsc example_async_basic.ts --strict --target es2017` and run `node example_async_basic.js`.

**Expected Output:**
```
Ada
```

**Why this output occurs.** `fetchUser` is `async`, so it returns `Promise<{ id: string; name: string }>`. `await` unwraps it to the object type, so `user.name` is typed as `string`.

---

#### Example 2: Async arrow function with inferred return

```typescript
// example_async_arrow.ts
const fetchNumber = async (): Promise<number> => 42;

async function main() {
  const n = await fetchNumber();
  console.log(n.toFixed(1)); // Expected: 42.0
}

main();
```

**Expected Output:**
```
42.0
```

**Why this output occurs.** The async arrow function returns `Promise<number>`; `await` unwraps it to `number`, enabling `.toFixed(1)`.

---

### Real-World Cases

- **HTTP clients**: `async function fetchUser(id: string): Promise<User>`.
- **Database access**: `async function findById(id: string): Promise<Entity | null>`.
- **File system operations**: `async function readConfig(path: string): Promise<Config>`.
- **API route handlers**: `async function handler(req: Request): Promise<Response>`.

---

## 2. `async` Keyword

### Definitions

**Core Definition.** The **`async` keyword** marks a function as asynchronous, making it return a promise and enabling the use of `await` inside its body.

**Technical Definition.** `async` transforms a function's return type into `Promise<T>`, where `T` is the annotated or inferred return type. It also enables the `await` operator within the function body. The keyword may be applied to function declarations, function expressions, arrow functions, and class methods.

**Beginner-Friendly Explanation.** `async` is **the word that makes a function asynchronous**. It tells TypeScript "this function returns a promise, and you can `await` inside it."

### Purposes

- **To mark a function as returning a promise** without manually constructing one.
- **To enable `await`** inside the function body.
- **To propagate return types automatically** as `Promise<T>`.
- **To signal asynchronous intent** to callers and tooling.

### Syntax Rules and Structure

**Function declaration:**

```typescript
async function f(): Promise<T> { ... }
```

**Arrow function:**

```typescript
const f = async (): Promise<T> => { ... };
```

**Class method:**

```typescript
class Service {
  async fetch(id: string): Promise<User> { ... }
}
```

**Component breakdown:**

- `async` — the keyword.
- `Promise<T>` — the resulting return type (inferred if omitted).
- The body may contain `await` expressions.

**Syntax rules.**

- `async` must precede the `function` keyword or the arrow function's parameter list.
- The return type is always `Promise<T>`.
- `async` methods are supported in classes and object literals.

**Constraints and limitations.**

- `async` functions cannot be constructors.
- `async` functions do not run synchronously; they return a promise immediately.

### Annotated Complete Step by Step Code Examples

#### Example 1: Async class method

```typescript
// example_async_method.ts
class UserService {
  async findById(id: string): Promise<{ id: string; name: string }> {
    return { id, name: "Ada" };
  }
}

async function main() {
  const service = new UserService();
  const user = await service.findById("u1");
  console.log(user.name); // Expected: Ada
}

main();
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** The `async` method returns `Promise<{ id: string; name: string }>`, which `await` unwraps to the object type.

---

### Real-World Cases

- **Service classes**: `async` methods for database and HTTP operations.
- **Event handlers**: `async` callbacks for form submission.
- **Module initialization**: `async` setup functions.

---

## 3. `await` Operator

### Definitions

**Core Definition.** The **`await` operator** pauses the execution of an async function until the awaited promise settles, returning its resolved value or throwing its rejection reason.

**Technical Definition.** `await expr` evaluates `expr`, and if it is a promise-like value, suspends the async function until it settles. If fulfilled, `await` returns the resolved value with type `Awaited<T>`; if rejected, the rejection reason is thrown. `await` can only be used inside `async` functions or at the top level of ES modules with `module: "esnext"` or `"nodenext"`.

**Beginner-Friendly Explanation.** `await` means **"pause here until the promise is done."** It gives you the resolved value directly, so you can write `const user = await fetchUser(id)` instead of `fetchUser(id).then(user => ...)`.

### Purposes

- **To unwrap promises** into their resolved values.
- **To write sequential asynchronous code** in a readable, synchronous style.
- **To propagate rejections as exceptions** caught by `try`/`catch`.
- **To use promise results directly** without nested callbacks.

### Syntax Rules and Structure

**General syntax:**

```typescript
const value = await promise;
```

**Top-level await (ES modules):**

```typescript
const config = await loadConfig();
```

**Component breakdown:**

- `await` — the operator.
- `promise` — any promise-like value.
- `value` — the resolved value, typed as `Awaited<typeof promise>`.

**Syntax rules.**

- `await` may appear only inside `async` functions or top-level ES modules.
- The operand is typically `Promise<T>`, but non-promise values are allowed and returned as-is.
- Rejections are thrown as exceptions.

**Constraints and limitations.**

- Sequential `await` on independent operations is slower than `Promise.all`.
- `await` does not cancel the underlying operation.

### Annotated Complete Step by Step Code Examples

#### Example 1: Sequential `await`

```typescript
// example_await_sequential.ts
async function fetchName(): Promise<string> {
  return "Ada";
}

async function fetchAge(): Promise<number> {
  return 36;
}

async function main() {
  const name = await fetchName(); // string
  const age = await fetchAge();   // number
  console.log(`${name} is ${age}`); // Expected: Ada is 36
}

main();
```

**Expected Output:**
```
Ada is 36
```

**Why this output occurs.** Each `await` unwraps the promise to its resolved type, so `name` is `string` and `age` is `number`.

---

#### Example 2: `await` in a loop

```typescript
// example_await_loop.ts
async function processItem(item: string): Promise<string> {
  return item.toUpperCase();
}

async function main() {
  const items = ["a", "b", "c"];
  const results: string[] = [];
  for (const item of items) {
    results.push(await processItem(item));
  }
  console.log(results); // Expected: [ 'A', 'B', 'C' ]
}

main();
```

**Expected Output:**
```
[ 'A', 'B', 'C' ]
```

**Why this output occurs.** Each iteration awaits `processItem` before continuing, processing items sequentially. For concurrency, use `Promise.all(items.map(processItem))`.

---

### Real-World Cases

- **API calls**: `const response = await fetch(url)`.
- **Database queries**: `const user = await db.findUser(id)`.
- **Sequential workflows**: `await saveUser(user); await sendEmail(user.email)`.
- **Top-level initialization**: `const config = await loadConfig()` in ES modules.

---

## 4. Error Handling

### Definitions

**Core Definition.** **Error handling** in async functions uses `try`/`catch`/`finally` to catch promise rejections as thrown exceptions.

**Technical Definition.** When a promise awaited with `await` rejects, the rejection reason is **thrown** at the `await` expression. A surrounding `try`/`catch` catches it. Since TypeScript 4.4, `useUnknownInCatchVariables` (part of `strict`) types catch variables as `unknown`, requiring narrowing with `instanceof Error` or other checks before accessing properties. `finally` blocks run regardless of outcome.

**Beginner-Friendly Explanation.** Error handling in async code works **just like synchronous error handling**: wrap `await` calls in `try`, and catch failures in `catch`. The caught error is `unknown`, so you must check what it is before using it.

### Purposes

- **To catch rejections** from awaited promises.
- **To centralize error handling** at the appropriate level of the call stack.
- **To guarantee cleanup** with `finally` blocks.
- **To narrow error types safely** with `instanceof Error`.
- **To propagate or recover from errors** deliberately.

### Syntax Rules and Structure

**Basic `try`/`catch`:**

```typescript
try {
  const user = await fetchUser(id);
} catch (error: unknown) {
  if (error instanceof Error) console.error(error.message);
}
```

**With `finally`:**

```typescript
try {
  await save();
} finally {
  cleanup();
}
```

**Custom error classes:**

```typescript
class NotFoundError extends Error {
  constructor(message: string) {
    super(message);
    this.name = "NotFoundError";
  }
}

try {
  await fetchUser(id);
} catch (error: unknown) {
  if (error instanceof NotFoundError) { ... }
  else if (error instanceof Error) { ... }
  else { ... }
}
```

**Syntax rules.**

- `catch` variables are `unknown` under `strict`.
- Narrow with `instanceof Error` before accessing `.message` or `.stack`.
- `finally` runs regardless of whether the `try` block succeeded or threw.
- `throw` inside `catch` propagates the error to the caller.

**Constraints and limitations.**

- Non-`Error` rejection reasons are common in JavaScript; always handle the `unknown` case.
- `try`/`catch` does not catch errors from promises that are not awaited.

### Annotated Complete Step by Step Code Examples

#### Example 1: `try`/`catch` with `unknown`

```typescript
// example_error_try_catch.ts
async function fetchUser(id: string): Promise<{ id: string; name: string }> {
  if (id === "missing") throw new Error("User not found");
  return { id, name: "Ada" };
}

async function main() {
  try {
    const user = await fetchUser("missing");
    console.log(user.name);
  } catch (error: unknown) {
    if (error instanceof Error) {
      console.error("Error:", error.message); // Expected: Error: User not found
    } else {
      console.error("Unknown error:", error);
    }
  }
}

main();
```

**Expected Output:**
```
Error: User not found
```

**Why this output occurs.** The rejected promise throws at `await`, and `catch` receives the `Error` instance. The `instanceof Error` check narrows `unknown` to `Error`, enabling `.message`.

---

#### Example 2: Custom error classes

```typescript
// example_error_custom.ts
class ValidationError extends Error {
  constructor(public field: string, message: string) {
    super(message);
    this.name = "ValidationError";
  }
}

async function validate(value: string): Promise<void> {
  if (value.length < 3) throw new ValidationError("value", "Too short");
}

async function main() {
  try {
    await validate("ab");
  } catch (error: unknown) {
    if (error instanceof ValidationError) {
      console.error(`Invalid ${error.field}: ${error.message}`);
      // Expected: Invalid value: Too short
    } else if (error instanceof Error) {
      console.error(error.message);
    }
  }
}

main();
```

**Expected Output:**
```
Invalid value: Too short
```

**Why this output occurs.** The custom error class preserves the `field` property, and the `instanceof ValidationError` check narrows the `unknown` error to the custom type.

---

### Real-World Cases

- **API clients**: catching `NotFoundError` and `ValidationError` for different handling.
- **Form submission**: showing field-specific errors.
- **Retry logic**: catching transient errors and retrying.
- **Cleanup**: `finally` blocks for closing connections and releasing resources.

---

## 5. Async Best Practices

### Definitions

**Core Definition.** **Async best practices** are guidelines for writing `async`/`await` code that is **safe, readable, performant, and type-correct**, avoiding unhandled rejections, sequential bottlenecks, and swallowed errors.

**Technical Definition.** Best practices center on **always handling rejections**, **using `Promise.all` for concurrent independent operations**, **avoiding sequential `await` in loops**, **narrowing `unknown` catch variables**, **preferring `async`/`await` over `.then()` chains**, **using `AbortController` for cancellation**, and **enabling lint rules** such as `no-floating-promises` and `no-misused-promises`.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep async code reliable. The big rules are: always handle errors; run independent operations concurrently; do not await in loops; narrow catch errors; and prefer `async`/`await` over `.then()`.

### Purposes

- **To prevent unhandled rejections** that crash processes or log errors.
- **To maximize performance** by running independent operations concurrently.
- **To keep error handling consistent** and type-safe.
- **To avoid sequential bottlenecks** from `await` in loops.
- **To enable cancellation** for long-running operations.

### Syntax Rules and Structure

**Concurrency with `Promise.all`:**

```typescript
const [user, posts] = await Promise.all([fetchUser(id), fetchPosts(id)]);
```

**Avoid `await` in loops for independent work:**

```typescript
// ❌ Sequential.
for (const id of ids) results.push(await fetchUser(id));

// ✅ Concurrent.
const results = await Promise.all(ids.map(fetchUser));
```

**Narrow catch variables:**

```typescript
catch (error: unknown) {
  if (error instanceof Error) console.error(error.message);
}
```

**Best-practice rules.**

1. **Always handle rejections.** Use `try`/`catch` with `await` or `.catch()` with `.then()`. Enable `@typescript-eslint/no-floating-promises`.
2. **Use `Promise.all` for independent concurrent operations.** Sequential `await` multiplies latency.
3. **Avoid `await` in loops for independent work.** Use `Promise.all(items.map(fn))` instead; use sequential `await` only when each iteration depends on the previous.
4. **Narrow `unknown` catch variables.** Use `instanceof Error` or custom error classes before accessing properties.
5. **Prefer `async`/`await` over `.then()` chains.** It is more readable, supports `try`/`catch`, and preserves stack traces better.
6. **Use `AbortController` for cancellation.** Pass `signal` to `fetch` and abort when the operation is no longer needed.
7. **Return promises directly when possible.** Avoid `return await promise` unless inside a `try` block; `return promise` is equivalent and avoids an extra microtask.
8. **Avoid mixing `async`/`await` and `.then()` unnecessarily.** Choose one style per function.
9. **Enable `@typescript-eslint/no-misused-promises`.** It catches promises passed to void-returning callbacks (e.g., event handlers).

**Component breakdown (async decision tree):**

```
Do you have multiple independent async operations?
  ├── Yes → Should they all succeed?
  │        ├── Yes → Use `Promise.all`.
  │        └── No → Use `Promise.allSettled`.
  └── No → Do you need to handle errors?
           ├── Yes → Use `try`/`catch` with `unknown` narrowing.
           └── No → Use `await` directly.
```

**Constraints and limitations.**

- TypeScript cannot enforce rejection handling; lint rules are required.
- `return await` inside `try` is correct for error handling; outside, it adds a microtask.
- `AbortController` is supported in modern browsers and Node.js 15+.

### Annotated Complete Step by Step Code Examples

#### Example 1: Concurrent vs. sequential await

```typescript
// example_best_concurrent.ts
async function fetchUser(id: string): Promise<{ id: string; name: string }> {
  return new Promise((resolve) => setTimeout(() => resolve({ id, name: "Ada" }), 10));
}

async function fetchPosts(id: string): Promise<string[]> {
  return new Promise((resolve) => setTimeout(() => resolve(["post1"]), 10));
}

async function main() {
  // ❌ Sequential: ~20ms total.
  // const user = await fetchUser("u1");
  // const posts = await fetchPosts("u1");

  // ✅ Concurrent: ~10ms total.
  const [user, posts] = await Promise.all([fetchUser("u1"), fetchPosts("u1")]);
  console.log(user.name, posts.length); // Expected: Ada 1
}

main();
```

**Expected Output:**
```
Ada 1
```

**Why this output occurs.** `Promise.all` starts both promises immediately, running them in parallel. Sequential `await` would wait for the first before starting the second.

---

#### Example 2: Avoiding `await` in loops

```typescript
// example_best_loop.ts
async function fetchItem(id: string): Promise<string> {
  return `item-${id}`;
}

async function main() {
  const ids = ["1", "2", "3"];

  // ❌ Sequential: each iteration waits.
  // const sequential: string[] = [];
  // for (const id of ids) sequential.push(await fetchItem(id));

  // ✅ Concurrent: all run in parallel.
  const concurrent = await Promise.all(ids.map(fetchItem));
  console.log(concurrent); // Expected: [ 'item-1', 'item-2', 'item-3' ]
}

main();
```

**Expected Output:**
```
[ 'item-1', 'item-2', 'item-3' ]
```

**Why this output occurs.** `ids.map(fetchItem)` starts all promises immediately, and `Promise.all` waits for all of them concurrently. This is much faster than sequential `await` in a loop.

---

#### Example 3: Narrowing `unknown` errors

```typescript
// example_best_unknown.ts
class NetworkError extends Error {
  constructor(public status: number) {
    super(`HTTP ${status}`);
    this.name = "NetworkError";
  }
}

async function fetchData(url: string): Promise<string> {
  if (url === "fail") throw new NetworkError(500);
  return "data";
}

async function main() {
  try {
    await fetchData("fail");
  } catch (error: unknown) {
    if (error instanceof NetworkError) {
      console.error(`Network error ${error.status}`); // Expected: Network error 500
    } else if (error instanceof Error) {
      console.error(error.message);
    } else {
      console.error("Unknown error:", error);
    }
  }
}

main();
```

**Expected Output:**
```
Network error 500
```

**Why this output occurs.** The `unknown` catch variable is narrowed with `instanceof NetworkError`, giving access to the `status` property. The fallback handles non-`Error` rejection reasons.

---

#### Example 4: `return await` inside `try` vs. outside

```typescript
// example_best_return_await.ts
async function fetchValue(): Promise<number> {
  return 42;
}

// ✅ `return await` inside try: errors are caught.
async function insideTry() {
  try {
    return await fetchValue();
  } catch (error: unknown) {
    console.error("Caught:", error);
    return 0;
  }
}

// ✅ `return` without await outside try: equivalent and avoids an extra microtask.
async function outsideTry() {
  return fetchValue();
}

insideTry().then(console.log);  // Expected: 42
outsideTry().then(console.log); // Expected: 42
```

**Expected Output:**
```
42
42
```

**Why this output occurs.** `return await` inside `try` is necessary to catch errors from `fetchValue`. Outside `try`, `return fetchValue()` is equivalent and slightly more efficient because it avoids an extra microtask.

---

### Real-World Cases

- **API clients**: `Promise.all` for concurrent fetches; `try`/`catch` with `unknown` narrowing.
- **Batch processing**: `Promise.all` for independent items; sequential `await` only for dependent steps.
- **Cancellation**: `AbortController` with `fetch` for user-initiated cancellations.
- **Lint enforcement**: `no-floating-promises` and `no-misused-promises` in CI.

---

## References

- TypeScript 4.5 Release Notes — The `Awaited` Type and `Promise` Improvements — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-5.html#the-awaited-type-and-promise-improvements
- TypeScript 4.4 Release Notes — Defaulting to the `unknown` Type in Catch Variables — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-4.html#defaulting-to-the-unknown-type-in-catch-variables
- TypeScript 2.1 Release Notes — `async`/`await` Support — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-1.html#asyncawait-support
- TypeScript Handbook — Utility Types (`Awaited<T>`) — https://www.typescriptlang.org/docs/handbook/utility-types.html#awaitedtype
- MDN — `async` Function — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function
- MDN — `await` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await
- MDN — `try...catch` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/try...catch
- MDN — Top-Level `await` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await#top_level_await
- TypeScript ESLint — `no-floating-promises` Rule — https://typescript-eslint.io/rules/no-floating-promises/
- TypeScript ESLint — `no-misused-promises` Rule — https://typescript-eslint.io/rules/no-misused-promises/
- TypeScript ESLint — `await-thenable` Rule — https://typescript-eslint.io/rules/await-thenable/
- MDN — `AbortController` — https://developer.mozilla.org/en-US/docs/Web/API/AbortController