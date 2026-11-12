# Async Iterators in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **Async iterators** are the asynchronous counterpart to synchronous iterators, allowing consumers to `await` each value produced by a sequence. They implement the **async iteration protocol** (`Symbol.asyncIterator`), returning promises from each `.next()` call. TypeScript types async iterators with **`AsyncIterator<T, TReturn, TNext>`** and **`AsyncIterable<T>`**, and consumes them with **`for await...of`**. Async generators (`async function*`) are the primary way to produce async iterables.

**Technical Definition.** The async iteration protocol defines `Symbol.asyncIterator`, which returns an `AsyncIterator<T>` whose `next()`, `return()`, and `throw()` methods return `Promise<IteratorResult<T, TReturn>>`. The `for await...of` statement awaits each iteration result before assigning the value. TypeScript's `AsyncIterable<T>` and `AsyncIterator<T, TReturn, TNext>` interfaces model this protocol, and `AsyncGenerator<T, TReturn, TNext>` combines both. Async iteration is **erased** at compile time; runtime behavior is standard JavaScript.

**Beginner-Friendly Explanation.** An async iterator is **an iterator where each value arrives asynchronously**—like reading rows from a database or pages from an API. Instead of getting values immediately, you **`await`** each one. `for await...of` is the loop that makes this easy: it waits for each value before running the loop body. Async generators (`async function*`) let you **produce** async iterables with `yield`, and `for await...of` lets you **consume** them.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Async iteration protocol** | Uses `Symbol.asyncIterator` and returns promises from `.next()`. |
| **`for await...of`** | The primary consumption syntax; awaits each value before the loop body. |
| **`AsyncIterable<T>`** | The interface for objects that can be iterated asynchronously. |
| **`AsyncIterator<T, TReturn, TNext>`** | The interface for the iterator object itself. |
| **`AsyncGenerator<T, TReturn, TNext>`** | The interface for async generator functions. |
| **Bidirectional communication** | Values can be passed into the iterator via `.next(value)` (typed by `TNext`). |
| **Cancellation** | `.return()` closes the iterator and runs `finally` blocks. |
| **Erasure** | Async iterator types are compile-time only; runtime behavior is standard JavaScript. |

---

### Prerequisites

- **TypeScript basics**: generics, interfaces, and type annotations.
- **Synchronous iterators and generators** (`function*`, `yield`).
- **Promises** and `async`/`await`.
- **The iteration protocol**: `Symbol.iterator` and `for...of`.
- **`try`/`catch`/`finally`** for error handling.
- **A TypeScript environment** with `target: "es2018"` or later (or downleveling for async iteration).

---

### Related Programming Areas

- **Async generators** (`async function*`) — the primary producer of async iterables.
- **`for await...of`** — the primary consumer of async iterables.
- **Promises** — the foundation of async iteration.
- **Streams and pagination** — real-world async iteration use cases.
- **Cancellation** — `.return()` and `AbortSignal` for terminating iteration.
- **Error handling** — `try`/`catch` around `for await...of`.

---

## Core Concepts / Features

---

## 1. Async Iterators

### Definitions

**Core Definition.** An **async iterator** is an object that implements the async iteration protocol, producing values asynchronously via `await`.

**Technical Definition.** An async iterator implements `Symbol.asyncIterator`, returning an `AsyncIterator<T, TReturn, TNext>`. Its `next()` method returns `Promise<IteratorResult<T, TReturn>>`. The `for await...of` statement calls `next()` and awaits each result before executing the loop body. TypeScript models async iterables with `AsyncIterable<T>` and async generators with `AsyncGenerator<T, TReturn, TNext>`.

**Beginner-Friendly Explanation.** An async iterator is **a sequence whose values arrive asynchronously**. Each value is wrapped in a promise, and `for await...of` waits for each promise before continuing.

### Purposes

- **To consume asynchronous sequences** such as API pagination, database cursors, and file streams.
- **To process values as they arrive** without loading the entire sequence into memory.
- **To compose asynchronous data pipelines** with `for await...of`.
- **To model streaming and event-based data** with backpressure.
- **To enable lazy asynchronous computation** with async generators.

### Syntax Rules and Structure

**Consuming with `for await...of`:**

```typescript
for await (const value of asyncIterable) {
  // value is T
}
```

**Async generator:**

```typescript
async function* fetchPages(): AsyncGenerator<Page, void, unknown> {
  yield page;
}
```

**Component breakdown:**

- `for await (const value of iterable)` — awaits each `next()` result.
- `async function*` — declares an async generator.
- `AsyncGenerator<T, TReturn, TNext>` — the type of the generator object.

**Syntax rules.**

- `for await...of` may only be used inside `async` functions or top-level ES modules.
- The iterable must implement `Symbol.asyncIterator` or be a sync iterable (which is wrapped automatically).
- Async generators may use `await` inside their bodies.
- `.return()` and `.throw()` are available on async iterators.

**Constraints and limitations.**

- `for await...of` awaits each value sequentially; it does not parallelize.
- Async iteration requires `target: "es2018"` or later (or downleveling).
- Errors from the iterator are thrown at the `await` point.

### Annotated Complete Step by Step Code Examples

#### Example 1: Consuming an async iterable

```typescript
// example_async_iterator_basic.ts
// Step 1: Define an async iterable.
async function* countTo(n: number): AsyncGenerator<number, void, unknown> {
  for (let i = 1; i <= n; i++) {
    await new Promise((resolve) => setTimeout(resolve, 10));
    yield i;
  }
}

// Step 2: Consume with for await...of.
async function main() {
  for await (const value of countTo(3)) {
    console.log(value); // Expected: 1, 2, 3
  }
}

main();
```

**Setup.** Compile with `npx tsc example_async_iterator_basic.ts --strict --target es2018` and run `node example_async_iterator_basic.js`.

**Expected Output:**
```
1
2
3
```

**Why this output occurs.** The async generator yields each number after a delay. `for await...of` awaits each yielded promise before logging the value.

---

#### Example 2: Manual async iterator

```typescript
// example_async_iterator_manual.ts
const asyncIterable: AsyncIterable<number> = {
  async *[Symbol.asyncIterator]() {
    yield 1;
    yield 2;
    yield 3;
  },
};

async function main() {
  for await (const n of asyncIterable) {
    console.log(n); // Expected: 1, 2, 3
  }
}

main();
```

**Expected Output:**
```
1
2
3
```

**Why this output occurs.** The object implements `Symbol.asyncIterator` with an async generator method, making it consumable by `for await...of`.

---

### Real-World Cases

- **API pagination**: fetching pages until a `nextCursor` is absent.
- **Database cursors**: streaming rows from a query.
- **File streams**: reading chunks asynchronously.
- **Event streams**: processing server-sent events.

---

## 2. `for await...of`

### Definitions

**Core Definition.** **`for await...of`** is a loop statement that asynchronously iterates over an async iterable, awaiting each value before executing the loop body.

**Technical Definition.** `for await (const x of iterable)` calls `iterable[Symbol.asyncIterator]()` and repeatedly `await`s `.next()`. It exits when `done` is `true`. If the iterable is a synchronous iterable, `for await...of` wraps each value in a resolved promise. Errors thrown by the iterator or the loop body are propagated. The loop respects `.return()` for early termination.

**Beginner-Friendly Explanation.** `for await...of` is **a `for` loop that waits for each value**. It handles the async plumbing for you, so you can focus on the loop body.

### Purposes

- **To consume async iterables** with synchronous-looking syntax.
- **To process streamed data** in order.
- **To handle errors** with `try`/`catch` around the loop.
- **To support early termination** with `break` (which calls `.return()`).
- **To iterate over sync iterables asynchronously** (e.g., awaiting each value).

### Syntax Rules and Structure

**General syntax:**

```typescript
for await (const value of asyncIterable) {
  // value is T
}
```

**With error handling:**

```typescript
try {
  for await (const value of asyncIterable) {
    process(value);
  }
} catch (error: unknown) {
  if (error instanceof Error) console.error(error.message);
}
```

**Component breakdown:**

- `for await` — the async loop keyword.
- `const value` — the iteration variable.
- `of asyncIterable` — the async iterable to consume.
- `break` — exits the loop and calls `.return()` on the iterator.

**Syntax rules.**

- `for await...of` may be used inside `async` functions or top-level ES modules.
- `break` triggers the iterator's `.return()` method.
- Errors from the iterator are thrown at the `await` point.
- Sync iterables are supported and wrapped in promises.

**Constraints and limitations.**

- Cannot use `for await...of` in non-async functions.
- Awaits each value sequentially; does not parallelize.

### Annotated Complete Step by Step Code Examples

#### Example 1: Early termination with `break`

```typescript
// example_for_await_break.ts
async function* infinite(): AsyncGenerator<number, void, unknown> {
  let i = 0;
  while (true) {
    yield i++;
  }
}

async function main() {
  for await (const n of infinite()) {
    if (n >= 3) break; // stops at 3, calls return() internally
    console.log(n); // Expected: 0, 1, 2
  }
}

main();
```

**Expected Output:**
```
0
1
2
```

**Why this output occurs.** The `break` statement terminates the loop and calls the iterator's `.return()` method, stopping the infinite generator.

---

#### Example 2: Error handling

```typescript
// example_for_await_error.ts
async function* failing(): AsyncGenerator<number, void, unknown> {
  yield 1;
  throw new Error("Stream failed");
}

async function main() {
  try {
    for await (const n of failing()) {
      console.log(n); // Expected: 1
    }
  } catch (error: unknown) {
    if (error instanceof Error) {
      console.error(error.message); // Expected: Stream failed
    }
  }
}

main();
```

**Expected Output:**
```
1
Stream failed
```

**Why this output occurs.** The iterator yields `1`, then throws. The `try`/`catch` around the loop catches the error and narrows `unknown` to `Error`.

---

### Real-World Cases

- **Pagination**: `for await (const page of fetchPages())`.
- **Stream processing**: `for await (const chunk of stream)`.
- **Sequential async processing**: awaiting each item before the next.
- **Early termination**: breaking out of an infinite async generator.

---

## 3. Async Iterable Types

### Definitions

**Core Definition.** **Async iterable types** are the TypeScript interfaces that model async iteration: **`AsyncIterable<T>`**, **`AsyncIterator<T, TReturn, TNext>`**, and **`AsyncGenerator<T, TReturn, TNext>`**.

**Technical Definition.** `AsyncIterable<T>` declares `[Symbol.asyncIterator](): AsyncIterator<T>`. `AsyncIterator<T, TReturn, TNext>` declares `next()`, `return()`, and `throw()` methods returning `Promise<IteratorResult<T, TReturn>>`. `AsyncGenerator<T, TReturn, TNext>` extends both `AsyncIterator` and `AsyncIterable`, and is the return type of `async function*`. `TNext` types the values passed to `.next(value)`.

**Beginner-Friendly Explanation.** Async iterable types are **the TypeScript types that describe async sequences**. `AsyncIterable<number>` is an object you can `for await` over to get numbers. `AsyncGenerator<number>` is the type of an async generator function.

### Purposes

- **To type async generators** and their return values.
- **To annotate functions that return async iterables**.
- **To model bidirectional communication** with `TNext`.
- **To distinguish async from sync iterables** in APIs.
- **To enable `for await...of`** with correct element types.

### Syntax Rules and Structure

**Async iterable:**

```typescript
function getPages(): AsyncIterable<Page> { ... }
```

**Async iterator:**

```typescript
const iterator: AsyncIterator<number, void, unknown> = ...;
```

**Async generator:**

```typescript
async function* fetchPages(): AsyncGenerator<Page, void, unknown> {
  yield page;
}
```

**Component breakdown:**

- `AsyncIterable<T>` — consumable by `for await...of`.
- `AsyncIterator<T, TReturn, TNext>` — the iterator object.
- `AsyncGenerator<T, TReturn, TNext>` — the generator object.
- `T` — yielded value type.
- `TReturn` — final return value type.
- `TNext` — type of values passed to `.next()`.

**Syntax rules.**

- `AsyncGenerator<T, TReturn, TNext>` is the return type of `async function*`.
- `AsyncIterable<T>` is the minimal interface for `for await...of`.
- `TNext` defaults to `unknown` when omitted.

**Constraints and limitations.**

- `AsyncIterable` does not expose `.return()` or `.throw()` directly; use `AsyncIterator`.
- `AsyncGenerator` is invariant in `TNext`.

### Annotated Complete Step by Step Code Examples

#### Example 1: Typed async generator

```typescript
// example_async_iterable_types.ts
interface Page {
  index: number;
  items: string[];
}

async function* fetchPages(): AsyncGenerator<Page, void, unknown> {
  for (let i = 1; i <= 3; i++) {
    yield { index: i, items: [`item-${i}`] };
  }
}

async function main() {
  for await (const page of fetchPages()) {
    console.log(page.index); // Expected: 1, 2, 3
  }
}

main();
```

**Expected Output:**
```
1
2
3
```

**Why this output occurs.** `AsyncGenerator<Page, void, unknown>` types each yielded value as `Page`, so `page.index` is typed `number`.

---

#### Example 2: Typed async iterable function

```typescript
// example_async_iterable_function.ts
function rangeAsync(start: number, end: number): AsyncIterable<number> {
  return {
    async *[Symbol.asyncIterator]() {
      for (let i = start; i < end; i++) {
        yield i;
      }
    },
  };
}

async function main() {
  for await (const n of rangeAsync(0, 3)) {
    console.log(n); // Expected: 0, 1, 2
  }
}

main();
```

**Expected Output:**
```
0
1
2
```

**Why this output occurs.** The function returns an `AsyncIterable<number>`, which `for await...of` consumes with `number` typed values.

---

### Real-World Cases

- **API clients**: `fetchPages(): AsyncIterable<Page>`.
- **Database cursors**: `queryRows(): AsyncIterable<Row>`.
- **Stream APIs**: `readLines(): AsyncIterable<string>`.

---

## 4. Async Iterator Best Practices

### Definitions

**Core Definition.** **Async iterator best practices** are guidelines for writing and consuming async iterables that are **type-safe, cancelable, and error-resilient**, avoiding unhandled rejections, resource leaks, and unbounded memory use.

**Technical Definition.** Best practices center on **annotating async generator return types**, **handling errors with `try`/`catch`**, **using `.return()` or `break` for early termination**, **respecting `finally` blocks for cleanup**, **avoiding unbounded async iterables without cancellation**, and **using `AbortSignal` for external cancellation**. Async iteration is **backpressure-aware** by design—each `next()` is awaited before the next—so it avoids overwhelming consumers.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep async iteration safe. The big rules are: annotate the types; handle errors; clean up resources with `finally`; support cancellation; and do not create infinite async streams without a way to stop them.

### Purposes

- **To prevent resource leaks** from abandoned iterators.
- **To ensure errors are handled** and not silently swallowed.
- **To support cancellation** for long-running streams.
- **To keep async iteration readable** and type-safe.
- **To respect backpressure** by awaiting each value.

### Syntax Rules and Structure

**Best-practice rules:**

1. **Annotate async generator return types.** Use `AsyncGenerator<T, TReturn, TNext>` or `AsyncIterable<T>` to document the yielded type.
2. **Handle errors with `try`/`catch`.** Wrap `for await...of` in `try`/`catch` and narrow `unknown` errors.
3. **Use `finally` in async generators for cleanup.** Release resources when the generator completes or is closed.
4. **Support early termination.** Use `break` or `.return()` to stop iteration; this triggers `finally` blocks.
5. **Support cancellation with `AbortSignal`.** Check `signal.aborted` or pass the signal to underlying APIs.
6. **Avoid unbounded async iterables without cancellation.** Infinite async generators must have a termination mechanism.
7. **Prefer `for await...of` over manual `.next()` calls.** It handles completion and errors automatically.
8. **Do not parallelize `for await...of`.** If you need concurrency, collect items and use `Promise.all` on batches.

**Component breakdown (async iterator design decision tree):**

```
Do you need to produce an async sequence?
  ├── Yes → Use `async function*` with `AsyncGenerator<T>`.
  │        ├── Does it hold resources? → Use `try`/`finally` for cleanup.
  │        ├── Is it infinite? → Support cancellation with `AbortSignal`.
  │        └── Can it fail? → Let errors propagate; callers use try/catch.
  └── No → Do you need to consume an async sequence?
           ├── Yes → Use `for await...of` with `try`/`catch`.
           └── No → Use promises or sync iteration.
```

**Constraints and limitations.**

- `for await...of` is sequential; it does not parallelize.
- Abandoned async iterators may hold resources until garbage-collected.
- Cancellation requires cooperation from the iterator implementation.

### Annotated Complete Step by Step Code Examples

#### Example 1: Cleanup with `finally`

```typescript
// example_best_cleanup.ts
async function* readLines(lines: string[]): AsyncGenerator<string, void, unknown> {
  try {
    for (const line of lines) {
      yield line;
    }
  } finally {
    console.log("cleanup"); // Expected: cleanup (on completion or break)
  }
}

async function main() {
  for await (const line of readLines(["a", "b", "c"])) {
    console.log(line); // Expected: a, b, c
  }
  // finally runs after normal completion.
}

main();
```

**Expected Output:**
```
a
b
c
cleanup
```

**Why this output occurs.** The `finally` block runs when the generator completes or is closed, releasing resources.

---

#### Example 2: Cancellation with `AbortSignal`

```typescript
// example_best_cancellation.ts
async function* fetchUntilAborted(
  signal: AbortSignal
): AsyncGenerator<number, void, unknown> {
  let i = 0;
  while (!signal.aborted) {
    yield i++;
    await new Promise((resolve) => setTimeout(resolve, 10));
  }
}

async function main() {
  const controller = new AbortController();
  setTimeout(() => controller.abort(), 35);

  for await (const n of fetchUntilAborted(controller.signal)) {
    console.log(n); // Expected: 0, 1, 2 (approximately)
  }
}

main();
```

**Expected Output (approximate):**
```
0
1
2
```

**Why this output occurs.** The generator checks `signal.aborted` before each iteration. When the controller aborts, the generator completes and the loop exits.

---

#### Example 3: Error handling in async iteration

```typescript
// example_best_error.ts
async function* risky(): AsyncGenerator<number, void, unknown> {
  yield 1;
  yield 2;
  throw new Error("Stream failed");
}

async function main() {
  try {
    for await (const n of risky()) {
      console.log(n); // Expected: 1, 2
    }
  } catch (error: unknown) {
    if (error instanceof Error) {
      console.error(error.message); // Expected: Stream failed
    }
  }
}

main();
```

**Expected Output:**
```
1
2
Stream failed
```

**Why this output occurs.** The error thrown by the generator is caught by the surrounding `try`/`catch`, and the `unknown` error is narrowed to `Error`.

---

### Real-World Cases

- **Pagination with cleanup**: closing database cursors in `finally` blocks.
- **Stream processing with cancellation**: aborting file reads when the user navigates away.
- **API clients**: async iterables with `AbortSignal` support.
- **Code review**: rejecting unbounded async generators without cancellation.

---

## References

- TypeScript 2.3 Release Notes — Async Iterators and `for await...of` — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-3.html
- TypeScript 3.6 Release Notes — `AsyncIterator` Type Improvements — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-6.html
- TypeScript Handbook — Iterators and Generators — https://www.typescriptlang.org/docs/handbook/iterators-and-generators.html
- MDN — `for await...of` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for-await...of
- MDN — Async Iteration Protocol — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#the_async_iterator_and_async_iterable_protocols
- MDN — `AsyncGenerator` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/AsyncGenerator
- MDN — `Symbol.asyncIterator` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/asyncIterator
- MDN — `AbortController` — https://developer.mozilla.org/en-US/docs/Web/API/AbortController
- TypeScript ESLint — `no-floating-promises` Rule — https://typescript-eslint.io/rules/no-floating-promises/
- TypeScript ESLint — `await-thenable` Rule — https://typescript-eslint.io/rules/await-thenable/