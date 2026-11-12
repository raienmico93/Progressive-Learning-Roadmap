# Promises in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **Promise** is a JavaScript object representing the **eventual result of an asynchronous operation**, which may resolve with a value or reject with a reason. TypeScript adds a **generic `Promise<T>` type** that statically tracks the type of the resolved value, allowing the compiler to enforce correct handling of asynchronous results and prevent common errors such as treating a promise as its resolved value.

**Technical Definition.** `Promise<T>` is a generic interface representing a promise that resolves to `T`. A promise is in one of three states: **pending**, **fulfilled** (resolved with `T`), or **rejected** (with a reason, typically an `Error`). TypeScript's `Promise<T>` models the resolved value type while rejection reasons are typed as `any` by default. The `async`/`await` syntax is syntactic sugar over promise chains, and `Awaited<T>` recursively unwraps nested promises at the type level. Promise methods (`then`, `catch`, `finally`, `all`, `allSettled`, `race`, `any`) are typed to preserve resolved value types.

**Beginner-Friendly Explanation.** A promise is **a placeholder for a value that will arrive later**—like a receipt for a meal that is still cooking. TypeScript's `Promise<T>` tells you **what type the value will be when it arrives**. If a function returns `Promise<User>`, you know that when the promise resolves, you will have a `User`. TypeScript prevents you from using the `User` before the promise resolves and helps you handle the failure case explicitly.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Generic resolved type** | `Promise<T>` tracks the type `T` of the resolved value. |
| **Three states** | Pending, fulfilled, or rejected; only pending is observable synchronously. |
| **Typed methods** | `then`, `catch`, `finally` are typed to preserve or transform the resolved type. |
| **Combinator methods** | `all`, `allSettled`, `race`, `any` have precise tuple and union types. |
| **`async`/`await`** | Syntactic sugar for promise chains; `await` unwraps `Promise<T>` to `T`. |
| **`Awaited<T>`** | Recursively unwraps nested promises at the type level (TS 4.5+). |
| **Unhandled rejections** | TypeScript cannot enforce rejection handling; runtime tools and lint rules help. |
| **Erasure** | Promise types are compile-time only; the runtime behavior is standard JavaScript. |

---

### Prerequisites

- **TypeScript basics**: generics, type annotations, and interfaces.
- **JavaScript promises**: the event loop, `.then()`, and `.catch()`.
- **Async/await syntax**: `async` functions and `await` expressions.
- **Union types** and **type narrowing**.
- **`Awaited<T>`** and `ReturnType<T>`.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Asynchronous programming** — the event loop, microtasks, and concurrency.
- **Async/await** — the modern syntax for working with promises.
- **Promise combinators** — `all`, `allSettled`, `race`, `any`.
- **`Awaited<T>`** — the type-level unwrapping of promises.
- **Error handling** — rejection types, `try`/`catch`, and `unknown` catch variables.
- **AbortController** — cancellation for fetch and other async operations.

---

## Core Concepts / Features

---

## 1. Promises

### Definitions

**Core Definition.** A **Promise** is an object representing an asynchronous operation's eventual value or failure. In TypeScript, `Promise<T>` is the generic type of a promise resolving to `T`.

**Technical Definition.** The `Promise<T>` interface provides `then`, `catch`, and `finally` methods, along with static combinators `Promise.all`, `Promise.allSettled`, `Promise.race`, and `Promise.any`. The resolved value type is `T`; the rejection reason is typed as `any`. `async` functions return `Promise<T>`, and `await` unwraps the promise at the type level.

**Beginner-Friendly Explanation.** A promise is **a box that will eventually contain a value or an error**. `Promise<User>` is a box that will contain a `User` when it opens.

### Purposes

- **To model asynchronous computations** with typed results.
- **To prevent using unresolved values** as if they were synchronous.
- **To handle success and failure paths explicitly** with typed methods.
- **To coordinate multiple asynchronous operations** with combinators.
- **To enable `async`/`await`** with precise type inference.

### Syntax Rules and Structure

**Creating a promise:**

```typescript
const promise = new Promise<string>((resolve, reject) => {
  resolve("done");
});
```

**Async function:**

```typescript
async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/users/${id}`);
  return response.json();
}
```

**Component breakdown:**

- `new Promise<T>((resolve, reject) => ...)` — the executor with typed `resolve: (value: T | PromiseLike<T>) => void` and `reject: (reason?: any) => void`.
- `async function ...: Promise<T>` — an async function returning a promise of `T`.
- `await` — unwraps `Promise<T>` to `T` in an async context.

**Syntax rules.**

- `resolve` accepts a value of type `T` (or `PromiseLike<T>`); `reject` accepts any reason.
- `async` functions always return `Promise<T>`; returning `T` is wrapped automatically.
- `await` can only be used inside `async` functions or at the top level of ES modules.
- Promise methods preserve or transform the resolved type.

**Constraints and limitations.**

- TypeScript does **not** enforce rejection handling; unhandled rejections are runtime errors.
- Promise types are erased; they provide no runtime validation.
- Rejection reasons are typed `any`; use `catch` with `unknown` and narrow.

### Annotated Complete Step by Step Code Examples

#### Example 1: Creating and resolving a promise

```typescript
// example_promise_basic.ts
// Step 1: Create a typed promise.
const promise = new Promise<string>((resolve, reject) => {
  setTimeout(() => resolve("Hello, TypeScript!"), 100);
});

// Step 2: Consume the resolved value.
promise.then((value) => {
  console.log(value); // Expected: Hello, TypeScript!
});
```

**Setup.** Compile with `npx tsc example_promise_basic.ts --strict` and run `node example_promise_basic.js`.

**Expected Output:**
```
Hello, TypeScript!
```

**Why this output occurs.** `Promise<string>` types the resolved value as `string`. The `.then((value) => ...)` callback receives `value: string`, so `console.log` is typed safely.

---

#### Example 2: Async function with typed return

```typescript
// example_promise_async.ts
interface User {
  id: string;
  name: string;
}

async function fetchUser(id: string): Promise<User> {
  // Simulated async fetch.
  return { id, name: "Ada" };
}

async function main() {
  const user = await fetchUser("u1"); // user is User
  console.log(user.name); // Expected: Ada
}

main();
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** `fetchUser` returns `Promise<User>`, and `await` unwraps it to `User`, so `user.name` is typed as `string`.

---

### Real-World Cases

- **HTTP requests**: `fetch()` returns `Promise<Response>`, and `.json()` returns `Promise<any>` (or a typed wrapper).
- **File system operations**: `fs.promises.readFile()` returns `Promise<Buffer>`.
- **Database queries**: ORM methods return `Promise<Entity[]>`.
- **Timers and delays**: `new Promise<void>((resolve) => setTimeout(resolve, 1000))`.

---

## 2. `Promise<T>`

### Definitions

**Core Definition.** `Promise<T>` is the **generic TypeScript type** for a promise that resolves to a value of type `T`.

**Technical Definition.** The `Promise<T>` interface includes `then`, `catch`, and `finally` methods, and the `PromiseConstructor` provides static combinators. The resolved value type is `T`; the rejection reason is `any` by default. `Awaited<T>` recursively unwraps nested `Promise<T>` types, modeling the behavior of `await`.

**Beginner-Friendly Explanation.** `Promise<T>` is **a promise that will eventually give you a `T`**. The `T` is the type of the value inside the box.

### Purposes

- **To type asynchronous return values** precisely.
- **To propagate resolved types** through promise chains and `async`/`await`.
- **To enable type-safe combinators** that preserve tuple and union types.
- **To integrate with `Awaited<T>`** for promise unwrapping in type-level code.

### Syntax Rules and Structure

**Type annotations:**

```typescript
function fetchUser(): Promise<User> { ... }
const p: Promise<number> = Promise.resolve(42);
```

**Awaited unwrapping:**

```typescript
type Resolved = Awaited<Promise<string>>; // string
type Nested = Awaited<Promise<Promise<number>>>; // number
```

**Component breakdown:**

- `Promise<T>` — the promise type.
- `Awaited<T>` — recursively unwraps nested promises.

**Syntax rules.**

- `async` functions return `Promise<T>` automatically.
- `Promise.resolve(value)` returns `Promise<T>` where `T` is the type of `value`.
- `Promise.reject(reason)` returns `Promise<never>`.

**Constraints and limitations.**

- `Promise<T>` is **covariant** in `T`; `Promise<Dog>` is assignable to `Promise<Animal>`.
- Rejection reasons are not typed; use `catch` with `unknown`.

### Annotated Complete Step by Step Code Examples

#### Example 1: `Promise<T>` annotations

```typescript
// example_promise_type.ts
function delay(ms: number): Promise<void> {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

const numberPromise: Promise<number> = Promise.resolve(42);

async function run() {
  await delay(10);
  const n = await numberPromise; // n is number
  console.log(n.toFixed(2)); // Expected: 42.00
}

run();
```

**Expected Output:**
```
42.00
```

**Why this output occurs.** `Promise<number>` resolves to `number`, so `n` is typed as `number` and `.toFixed(2)` is available.

---

#### Example 2: `Awaited<T>` for nested promises

```typescript
// example_awaited.ts
type A = Awaited<Promise<string>>;                // string
type B = Awaited<Promise<Promise<number>>>;        // number
type C = Awaited<boolean | Promise<number>>;       // boolean | number

const value: B = 42;
console.log(value); // Expected: 42
```

**Expected Output:**
```
42
```

**Why this output occurs.** `Awaited` recursively unwraps promises, modeling the result of `await` at the type level.

---

### Real-World Cases

- **API clients**: `Promise<ApiResponse<T>>` for typed responses.
- **Repository methods**: `Promise<User | null>` for lookups.
- **Async utilities**: `Promise.all` with typed tuple results.

---

## 3. Promise Creation

### Definitions

**Core Definition.** **Promise creation** is the act of producing a new promise, typically with the `Promise` constructor, `Promise.resolve`, `Promise.reject`, or an `async` function.

**Technical Definition.** The `new Promise<T>((resolve, reject) => ...)` constructor wraps an executor. `Promise.resolve(value)` and `Promise.reject(reason)` create already-settled promises. `async` functions implicitly return promises. TypeScript infers `T` from the `resolve` call or the async return type.

**Beginner-Friendly Explanation.** Creating a promise means **making a new box that will hold a value later**. You can create it manually with `new Promise`, use `Promise.resolve` for an already-known value, or let an `async` function create it for you.

### Purposes

- **To wrap callback-based APIs** in promises.
- **To create already-resolved or already-rejected promises** for testing and control flow.
- **To define async functions** that return promises implicitly.
- **To model delays and timers** with promises.

### Syntax Rules and Structure

**Constructor:**

```typescript
new Promise<T>((resolve, reject) => { ... });
```

**Static factories:**

```typescript
Promise.resolve(42);        // Promise<number>
Promise.reject(new Error()); // Promise<never>
```

**Async function:**

```typescript
async function f(): Promise<T> { return value; }
```

**Component breakdown:**

- `resolve: (value: T | PromiseLike<T>) => void` — fulfills the promise.
- `reject: (reason?: any) => void` — rejects the promise.
- `Promise.resolve` — returns a promise resolved with the value.
- `Promise.reject` — returns a rejected promise.

**Syntax rules.**

- `resolve` may accept a promise-like value, which is adopted.
- `reject` accepts any reason; use `Error` instances for consistency.
- `async` functions wrap returned values in promises automatically.

**Constraints and limitations.**

- The executor runs synchronously; throwing inside it rejects the promise.
- `Promise.resolve` flattens `PromiseLike` values but does not deep-unwrap nested promises.

### Annotated Complete Step by Step Code Examples

#### Example 1: Wrapping a callback API

```typescript
// example_promise_wrap.ts
function readFileAsync(path: string): Promise<string> {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (path === "missing.txt") {
        reject(new Error("File not found"));
      } else {
        resolve("file contents");
      }
    }, 10);
  });
}

readFileAsync("data.txt").then((contents) => {
  console.log(contents); // Expected: file contents
});
```

**Expected Output:**
```
file contents
```

**Why this output occurs.** The promise wraps the callback-based `setTimeout`, resolving with the file contents or rejecting with an error.

---

#### Example 2: `Promise.resolve` and `Promise.reject`

```typescript
// example_promise_factories.ts
const resolved = Promise.resolve("hello");
const rejected = Promise.reject(new Error("failure"));

resolved.then((value) => console.log(value)); // Expected: hello
rejected.catch((error: unknown) => {
  if (error instanceof Error) console.log(error.message); // Expected: failure
});
```

**Expected Output:**
```
hello
failure
```

**Why this output occurs.** `Promise.resolve` creates an already-resolved promise; `Promise.reject` creates an already-rejected one. The `catch` handler narrows `unknown` to `Error`.

---

### Real-World Cases

- **Wrapping Node.js callbacks**: `fs.readFile` → `Promise<Buffer>`.
- **Test doubles**: `Promise.resolve(mockValue)` for stubs.
- **Delays**: `new Promise<void>((resolve) => setTimeout(resolve, ms))`.

---

## 4. Promise Methods

### Definitions

**Core Definition.** **Promise methods** are the instance methods (`then`, `catch`, `finally`) and static combinators (`all`, `allSettled`, `race`, `any`) that consume, transform, or coordinate promises.

**Technical Definition.** Instance methods chain on a promise: `then` transforms the resolved value; `catch` handles rejection; `finally` runs cleanup. Static combinators take iterables of promises and return promises of aggregated results with precise tuple or union types.

**Beginner-Friendly Explanation.** Promise methods are **the tools you use to work with promises**. `.then()` runs code when the value arrives; `.catch()` handles errors; `.finally()` runs cleanup. The static methods combine multiple promises.

### Purposes

- **To chain asynchronous operations** with type preservation.
- **To handle errors** at the appropriate level.
- **To run cleanup logic** regardless of outcome.
- **To coordinate concurrent operations** with precise result types.
- **To implement timeouts and racing** with `race` and `any`.

### Syntax Rules and Structure

**Instance methods:**

```typescript
promise.then((value) => transformed);
promise.catch((error) => recovered);
promise.finally(() => cleanup);
```

**Combinators:**

```typescript
Promise.all([p1, p2]);          // Promise<[T1, T2]>
Promise.allSettled([p1, p2]);   // Promise<SettledResult<T1 | T2>[]>
Promise.race([p1, p2]);         // Promise<T1 | T2>
Promise.any([p1, p2]);          // Promise<T1 | T2>
```

**Syntax rules.**

- `then` returns a new promise of the transformed type.
- `catch` returns a promise of the original type or the recovery type.
- `finally` preserves the original resolved/rejected type.
- `all` preserves tuple types; `race` and `any` return unions.

**Constraints and limitations.**

- `Promise.all` rejects immediately on the first rejection.
- `Promise.any` rejects with `AggregateError` if all promises reject.
- `Promise.race` settles with the first promise to settle, whether fulfilled or rejected.

### Annotated Complete Step by Step Code Examples

#### Example 1: `then`, `catch`, `finally`

```typescript
// example_then_catch_finally.ts
function fetchUser(id: string): Promise<{ id: string; name: string }> {
  return id === "missing"
    ? Promise.reject(new Error("User not found"))
    : Promise.resolve({ id, name: "Ada" });
}

fetchUser("u1")
  .then((user) => {
    console.log(user.name); // Expected: Ada
    return user.name;
  })
  .catch((error: unknown) => {
    if (error instanceof Error) console.error(error.message);
    return "unknown";
  })
  .finally(() => {
    console.log("done"); // Expected: done
  });
```

**Expected Output:**
```
Ada
done
```

**Why this output occurs.** `.then` transforms the resolved user into the user's name; `.catch` handles rejection; `.finally` runs cleanup regardless of outcome.

---

#### Example 2: `Promise.all` with tuple types

```typescript
// example_promise_all.ts
async function fetchName(): Promise<string> {
  return "Ada";
}

async function fetchAge(): Promise<number> {
  return 36;
}

async function main() {
  const [name, age] = await Promise.all([fetchName(), fetchAge()]);
  // name: string, age: number
  console.log(`${name} is ${age}`); // Expected: Ada is 36
}

main();
```

**Expected Output:**
```
Ada is 36
```

**Why this output occurs.** `Promise.all` preserves the tuple types of its input, so the destructured `[name, age]` is typed as `[string, number]`.

---

#### Example 3: `Promise.allSettled`

```typescript
// example_promise_allsettled.ts
async function main() {
  const results = await Promise.allSettled([
    Promise.resolve("ok"),
    Promise.reject(new Error("failed")),
  ]);

  for (const result of results) {
    if (result.status === "fulfilled") {
      console.log("Value:", result.value); // Expected: Value: ok
    } else {
      console.log("Reason:", result.reason.message); // Expected: Reason: failed
    }
  }
}

main();
```

**Expected Output:**
```
Value: ok
Reason: failed
```

**Why this output occurs.** `allSettled` returns `PromiseSettledResult<T>[]`, where each result is either `fulfilled` with a `value` or `rejected` with a `reason`. The `status` discriminant narrows each result.

---

#### Example 4: `Promise.race` and `Promise.any`

```typescript
// example_promise_race_any.ts
async function main() {
  // Race: first to settle wins.
  const race = await Promise.race([
    new Promise<string>((resolve) => setTimeout(() => resolve("slow"), 50)),
    new Promise<string>((resolve) => setTimeout(() => resolve("fast"), 10)),
  ]);
  console.log("Race:", race); // Expected: Race: fast

  // Any: first to fulfill wins; rejects only if all reject.
  const any = await Promise.any([
    Promise.reject(new Error("fail1")),
    Promise.resolve("success"),
  ]);
  console.log("Any:", any); // Expected: Any: success
}

main();
```

**Expected Output:**
```
Race: fast
Any: success
```

**Why this output occurs.** `race` settles with the first promise to settle (here, the fast one). `any` ignores rejections and resolves with the first fulfilled value.

---

### Real-World Cases

- **`then`/`catch`/`finally`** : chaining API calls, error recovery, cleanup.
- **`Promise.all`** : loading multiple resources concurrently with tuple typing.
- **`Promise.allSettled`** : batch operations where partial failure is acceptable.
- **`Promise.race`** : timeouts and first-response-wins scenarios.
- **`Promise.any`** : fallback servers or mirrors, first successful response wins.

---

## 5. Promise Best Practices

### Definitions

**Core Definition.** **Promise best practices** are guidelines for writing asynchronous TypeScript code that is **safe, readable, and type-correct**, avoiding unhandled rejections, swallowed errors, and common async pitfalls.

**Technical Definition.** Best practices center on **always handling rejections**, **preferring `async`/`await` over long `.then()` chains**, **using `Promise.all` for independent concurrent work**, **using `Promise.allSettled` when partial failure is acceptable**, **typing catch variables as `unknown`**, **wrapping callback APIs in promises**, and **using `AbortController` for cancellation**. TypeScript cannot enforce rejection handling, so discipline and lint rules (e.g., `@typescript-eslint/no-floating-promises`) are essential.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep async code reliable. The big rules are: always handle errors; prefer `await` over `.then` chains; use `Promise.all` for concurrency; type catch errors as `unknown`; and do not leave promises floating without handling them.

### Purposes

- **To prevent unhandled rejections** that crash Node.js processes or log errors.
- **To avoid swallowed errors** from empty or missing `catch` blocks.
- **To maximize readability** with `async`/`await` instead of deeply nested `.then()` chains.
- **To improve performance** by running independent async operations concurrently.
- **To handle partial failures gracefully** with `allSettled`.
- **To enable cancellation** with `AbortController` for fetch and other long-running operations.

### Syntax Rules and Structure

**Always handle rejections:**

```typescript
fetchUser(id).catch((error: unknown) => {
  console.error(error);
});
```

**Prefer `async`/`await`:**

```typescript
async function load() {
  try {
    const user = await fetchUser(id);
    const posts = await fetchPosts(user.id);
    return { user, posts };
  } catch (error: unknown) {
    if (error instanceof Error) console.error(error.message);
    throw error;
  }
}
```

**Concurrency with `Promise.all`:**

```typescript
const [user, posts] = await Promise.all([fetchUser(id), fetchPosts(id)]);
```

**Best-practice rules.**

1. **Always handle rejections.** Use `try`/`catch` with `await` or `.catch()` with `.then()`. Enable `@typescript-eslint/no-floating-promises` to catch unhandled promises.
2. **Prefer `async`/`await` over `.then()` chains.** It is more readable, supports `try`/`catch`, and preserves stack traces better.
3. **Use `Promise.all` for independent concurrent work.** Sequential `await` calls slow down independent operations.
4. **Use `Promise.allSettled` when partial failure is acceptable.** `Promise.all` rejects on the first failure, discarding other results.
5. **Type catch variables as `unknown`.** Since TypeScript 4.4, `useUnknownInCatchVariables` (part of `strict`) types catch errors as `unknown`; narrow with `instanceof Error`.
6. **Wrap callback APIs in promises.** Convert Node.js callback APIs with `util.promisify` or a custom wrapper.
7. **Use `AbortController` for cancellation.** Pass `signal` to `fetch` and abort when the operation is no longer needed.
8. **Avoid mixing `async`/`await` and `.then()` unnecessarily.** Choose one style per function for readability.
9. **Do not use `Promise` constructors for already-async functions.** If you already have a promise, return it directly instead of wrapping it in `new Promise`.

**Component breakdown (promise decision tree):**

```
Do you have multiple independent async operations?
  ├── Yes → Should partial failure be tolerated?
  │        ├── Yes → Use `Promise.allSettled`.
  │        └── No → Use `Promise.all` (concurrent, fails fast).
  └── No → Do you need cancellation?
           ├── Yes → Use `AbortController` with `fetch` or a custom signal.
           └── No → Use `async`/`await` with `try`/`catch`.
```

**Constraints and limitations.**

- TypeScript cannot enforce rejection handling; lint rules are required.
- `Promise.all` fails fast and discards other results on rejection.
- `catch` variables are typed `unknown` under `strict`; narrowing is required.

### Annotated Complete Step by Step Code Examples

#### Example 1: `async`/`await` over `.then()` chains

```typescript
// example_best_async_await.ts
async function fetchUser(id: string): Promise<{ id: string; name: string }> {
  return { id, name: "Ada" };
}

async function fetchPosts(userId: string): Promise<string[]> {
  return ["post1", "post2"];
}

// ✅ async/await with try/catch.
async function loadUserData(id: string) {
  try {
    const user = await fetchUser(id);
    const posts = await fetchPosts(user.id);
    return { user, posts };
  } catch (error: unknown) {
    if (error instanceof Error) console.error(error.message);
    throw error;
  }
}

loadUserData("u1").then((data) => console.log(data.user.name)); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** `async`/`await` flattens the promise chain, and `try`/`catch` handles rejections in one place. The `catch` variable is `unknown` and narrowed with `instanceof Error`.

---

#### Example 2: Concurrent execution with `Promise.all`

```typescript
// example_best_concurrent.ts
async function fetchName(): Promise<string> {
  return new Promise((resolve) => setTimeout(() => resolve("Ada"), 10));
}

async function fetchAge(): Promise<number> {
  return new Promise((resolve) => setTimeout(() => resolve(36), 10));
}

async function main() {
  // ✅ Concurrent: both run in parallel.
  const [name, age] = await Promise.all([fetchName(), fetchAge()]);
  console.log(`${name} is ${age}`); // Expected: Ada is 36
}

main();
```

**Expected Output:**
```
Ada is 36
```

**Why this output occurs.** `Promise.all` starts both promises immediately and waits for both. Sequential `await` would run them one after another, doubling the delay.

---

#### Example 3: `Promise.allSettled` for partial failure

```typescript
// example_best_allsettled.ts
async function main() {
  const results = await Promise.allSettled([
    Promise.resolve("ok"),
    Promise.reject(new Error("failed")),
  ]);

  const successes = results
    .filter((r): r is PromiseFulfilledResult<string> => r.status === "fulfilled")
    .map((r) => r.value);

  console.log(successes); // Expected: [ 'ok' ]
}

main();
```

**Expected Output:**
```
[ 'ok' ]
```

**Why this output occurs.** `allSettled` waits for all promises and returns discriminated results. The filter predicate narrows fulfilled results, preserving the value type.

---

#### Example 4: Handling `catch` as `unknown`

```typescript
// example_best_catch_unknown.ts
async function safeFetch(url: string): Promise<string> {
  try {
    const response = await fetch(url);
    return await response.text();
  } catch (error: unknown) {
    if (error instanceof Error) {
      console.error("Fetch failed:", error.message);
    } else {
      console.error("Unknown error:", error);
    }
    return "";
  }
}

safeFetch("https://example.com").then(console.log);
```

**Expected behavior.** The `catch` variable is narrowed with `instanceof Error`, ensuring safe access to `.message`.

**Why this is best practice.** Since TypeScript 4.4, `strict` mode types catch variables as `unknown`. Narrowing prevents unsafe property access on non-`Error` rejection reasons.

---

### Real-World Cases

- **API clients**: using `try`/`catch` with `async`/`await` and `AbortController`.
- **Batch operations**: `Promise.allSettled` for bulk updates where some may fail.
- **Dashboard loading**: `Promise.all` for concurrent independent fetches.
- **Lint enforcement**: `@typescript-eslint/no-floating-promises` to catch unhandled promises.

---

## References

- TypeScript Handbook — Utility Types (`Awaited<T>`) — https://www.typescriptlang.org/docs/handbook/utility-types.html#awaitedtype
- TypeScript 4.5 Release Notes — The `Awaited` Type — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-5.html#the-awaited-type-and-promise-improvements
- TypeScript 4.4 Release Notes — `unknown` in Catch Variables — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-4.html#defaulting-to-the-unknown-type-in-catch-variables
- MDN — Promise — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
- MDN — Promise.all() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all
- MDN — Promise.allSettled() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled
- MDN — Promise.race() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/race
- MDN — Promise.any() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/any
- Total TypeScript — Handling Promises and Async/Await — https://www.totaltypescript.com/books/total-typescript-essentials
- TypeScript ESLint — `no-floating-promises` Rule — https://typescript-eslint.io/rules/no-floating-promises/
- TypeScript ESLint — `no-misused-promises` Rule — https://typescript-eslint.io/rules/no-misused-promises/
- MDN — AbortController — https://developer.mozilla.org/en-US/docs/Web/API/AbortController
- Steve Kinney — Asynchronous TypeScript — https://stevekinney.com/courses/react-typescript/asynchronous-typescript