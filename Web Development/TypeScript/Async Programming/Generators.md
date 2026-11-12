# Generators in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **generator** is a special kind of function that can be **paused and resumed** during execution, producing a sequence of values lazily over time. Generators are declared with `function*` and use the **`yield` operator** to emit values and suspend execution. TypeScript types generators through the **`Generator<T, TReturn, TNext>`** interface, providing static guarantees about the types of yielded values, the return value, and the values passed back into the generator.

**Technical Definition.** A generator function returns an **iterator object** that conforms to the **`Generator<T, TReturn, TNext>`** interface, which extends both **`Iterator<T, TReturn, TNext>`** and **`IterableIterator<T>`**. The `T` parameter is the type of values produced by `yield`; `TReturn` is the type of the final `return` value; `TNext` is the type of values passed to `.next(value)`. The generator's body executes **lazily**—only when `.next()` is called—and the `yield` expression evaluates to the value passed to the subsequent `.next()` call. Generators are **erased** to standard JavaScript iterator state machines.

**Beginner-Friendly Explanation.** A generator is **a function that can pause itself and continue later**. Instead of computing all values at once, it produces them **one at a time** using `yield`. Each time you ask for the next value (with `.next()`), the function resumes from where it paused, does some work, and yields the next value. TypeScript's `Generator<T>` type tells you **what type of values the generator produces**, making iteration type-safe. Generators are ideal for **lazy sequences**, **infinite streams**, and **custom iteration**.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **`function*` declaration** | Generator functions use an asterisk after `function`. |
| **`yield` pauses execution** | The `yield` operator emits a value and suspends the function. |
| **Lazy evaluation** | The body runs only when `.next()` is called, not when the generator is created. |
| **`Generator<T, TReturn, TNext>`** | TypeScript's generic type for generator objects. |
| **Iterable protocol** | Generators are iterable, so they work with `for...of`, spread, and destructuring. |
| **Bidirectional communication** | Values can be passed **into** the generator via `.next(value)`, typed by `TNext`. |
| **`return` and `throw`** | The generator can be terminated with `.return()` or have errors injected with `.throw()`. |
| **Erasure** | Generator types are compile-time only; runtime behavior is standard JavaScript. |

---

### Prerequisites

- **TypeScript basics**: functions, generics, and interfaces.
- **Iteration protocol**: `Symbol.iterator`, `for...of`, and iterables.
- **Generics**: type parameters and constraints.
- **Union types** and **type narrowing**.
- **The `yield` and `yield*` operators**.
- **A TypeScript environment** with `target: "es2015"` or later (or downleveling).

---

### Related Programming Areas

- **Iterators and iterables** — the protocol generators implement.
- **`for...of` loops** — the primary consumer of generators.
- **Async generators** (`async function*`) — asynchronous lazy sequences.
- **Lazy evaluation** — infinite sequences and on-demand computation.
- **Custom iterables** — implementing `Symbol.iterator` with generators.
- **Coroutines and state machines** — generators as pausable computations.

---

## Core Concepts / Features

---

## 1. Generators

### Definitions

**Core Definition.** A **generator** is a function that produces a sequence of values lazily, pausing at each `yield` and resuming when the consumer requests the next value.

**Technical Definition.** A generator function is declared with `function*` and returns a `Generator<T, TReturn, TNext>` object. The generator body executes incrementally: each `.next()` call runs until the next `yield`, `return`, or `throw`. The `yield` expression produces a value of type `T` and evaluates to the value passed to the next `.next(value)` call, typed as `TNext`. The generator completes when the body returns (`TReturn`) or throws.

**Beginner-Friendly Explanation.** A generator is **a function that produces values on demand**. You call it once to create an iterator, then call `.next()` to get each value. The function remembers where it paused.

### Purposes

- **To produce lazy sequences** without allocating the entire sequence in memory.
- **To model infinite sequences** that are computed on demand.
- **To implement custom iteration** with `for...of`.
- **To simplify state machines** and coroutine-style logic.
- **To generate values incrementally** in algorithms and data processing.

### Syntax Rules and Structure

**General syntax:**

```typescript
function* generatorName(): Generator<T, TReturn, TNext> {
  yield value; // T
  return result; // TReturn
}
```

**Component breakdown:**

- `function*` — the generator function declaration.
- `yield value` — emits `value` of type `T` and pauses.
- `return result` — completes the generator with `result` of type `TReturn`.
- The returned object is a `Generator<T, TReturn, TNext>`.

**Syntax rules.**

- The asterisk may be attached to `function` or the name (`function* f` or `function *f`).
- `yield` may appear only inside generator functions.
- The generator body does not run until `.next()` is called.
- The `Generator` type is `Generator<T, TReturn, TNext>`; defaults are `unknown` for `TReturn` and `TNext`.
- `for...of` consumes the yielded values but ignores the return value.

**Constraints and limitations.**

- Generators cannot be constructors.
- The return value of `.next()` is a discriminated union `IteratorResult<T, TReturn>`.
- Generator types are erased; they provide no runtime validation.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic generator

```typescript
// example_generator_basic.ts
// Step 1: Declare a generator function.
function* countTo(n: number): Generator<number, void, unknown> {
  for (let i = 1; i <= n; i++) {
    yield i; // emits number
  }
}

// Step 2: Consume with for...of.
for (const value of countTo(3)) {
  console.log(value); // Expected: 1, 2, 3
}
```

**Setup.** Compile with `npx tsc example_generator_basic.ts --strict --target es2015` and run `node example_generator_basic.js`.

**Expected Output:**
```
1
2
3
```

**Why this output occurs.** The generator yields each number lazily. The `for...of` loop calls `.next()` repeatedly until the generator completes.

---

#### Example 2: Return value and `.next()` result

```typescript
// example_generator_return.ts
function* generate(): Generator<string, number, unknown> {
  yield "a";
  yield "b";
  return 42;
}

const gen = generate();
console.log(gen.next()); // { value: 'a', done: false }
console.log(gen.next()); // { value: 'b', done: false }
console.log(gen.next()); // { value: 42, done: true }
```

**Expected Output:**
```
{ value: 'a', done: false }
{ value: 'b', done: false }
{ value: 42, done: true }
```

**Why this output occurs.** Each `.next()` returns an `IteratorResult<string, number>`. The final result has `done: true` and the return value `42`.

---

### Real-World Cases

- **Lazy sequences**: generating IDs, pagination cursors, or database rows.
- **Infinite sequences**: Fibonacci numbers, random walks, or timers.
- **Tree traversal**: yielding nodes depth-first without recursion depth limits.
- **Data pipelines**: streaming transformations with `for...of`.

---

## 2. `function*`

### Definitions

**Core Definition.** The **`function*` syntax** declares a **generator function** that returns a generator object.

**Technical Definition.** The asterisk marks the function as a generator. The function returns a `Generator<T, TReturn, TNext>` object rather than a plain value. The body may contain `yield` and `yield*` expressions. Generator methods (`next`, `return`, `throw`) control execution.

**Beginner-Friendly Explanation.** `function*` is **the keyword combination that creates a generator**. The `*` tells TypeScript and JavaScript that this function can pause and resume.

### Purposes

- **To declare generator functions** with clear syntax.
- **To signal that a function produces values lazily**.
- **To enable `yield` and `yield*`** inside the function body.
- **To distinguish generators from regular functions** in the type system.

### Syntax Rules and Structure

**Function declaration:**

```typescript
function* f(): Generator<number> { yield 1; }
```

**Generator method:**

```typescript
class C {
  *[Symbol.iterator](): Generator<number> { yield 1; }
}
```

**Arrow functions:**

```typescript
// ❌ Arrow functions cannot be generators.
// ✅ Use function expressions.
const g = function* (): Generator<number> { yield 1; };
```

**Component breakdown:**

- `function*` — the generator declaration.
- `: Generator<T, TReturn, TNext>` — the return type.
- The body may contain `yield`.

**Syntax rules.**

- The asterisk may be placed anywhere between `function` and the name.
- Generator methods are declared with `*` before the method name.
- Arrow functions **cannot** be generators.
- `yield` and `yield*` are only valid inside generator bodies.

**Constraints and limitations.**

- Generators cannot be constructors (`new` is not allowed).
- Generator functions cannot be used with `async` unless declared as `async function*` (async generators).

### Annotated Complete Step by Step Code Examples

#### Example 1: Generator method

```typescript
// example_generator_method.ts
class Countdown {
  constructor(private from: number) {}

  *[Symbol.iterator](): Generator<number, void, unknown> {
    for (let i = this.from; i > 0; i--) {
      yield i;
    }
  }
}

const countdown = new Countdown(3);
for (const n of countdown) {
  console.log(n); // Expected: 3, 2, 1
}
```

**Expected Output:**
```
3
2
1
```

**Why this output occurs.** The `*[Symbol.iterator]` method makes the class iterable, and `for...of` calls the generator until completion.

---

#### Example 2: `yield*` delegation

```typescript
// example_generator_yield_star.ts
function* inner(): Generator<number, void, unknown> {
  yield 1;
  yield 2;
}

function* outer(): Generator<number, void, unknown> {
  yield 0;
  yield* inner(); // delegates to inner
  yield 3;
}

console.log([...outer()]); // Expected: [ 0, 1, 2, 3 ]
```

**Expected Output:**
```
[ 0, 1, 2, 3 ]
```

**Why this output occurs.** `yield*` delegates to another iterable, yielding all of its values in sequence.

---

### Real-World Cases

- **Custom iterables**: implementing `Symbol.iterator` with generators.
- **Delegation**: composing generators with `yield*`.
- **State machines**: pausing and resuming computation.

---

## 3. `yield`

### Definitions

**Core Definition.** The **`yield` operator** emits a value from a generator and **pauses execution** until the next `.next()` call.

**Technical Definition.** `yield value` produces `value` as the next result of the generator and suspends the function. When `.next(input)` is called, the `yield` expression **evaluates to `input`**, allowing bidirectional communication. `yield*` delegates to another iterable. The type of `yield value` is `TNext`, and the type of `value` is `T`.

**Beginner-Friendly Explanation.** `yield` means **"produce this value and pause"**. When the consumer asks for the next value, the generator resumes and the `yield` expression returns whatever was passed to `.next()`.

### Purposes

- **To emit values lazily** from a generator.
- **To receive input** from the consumer via `.next(value)`.
- **To implement coroutines** with bidirectional communication.
- **To delegate iteration** with `yield*`.

### Syntax Rules and Structure

**Basic yield:**

```typescript
function* f(): Generator<number, void, string> {
  const input: string = yield 1; // input is TNext
}
```

**Delegating yield:**

```typescript
function* g(): Generator<number> {
  yield* [1, 2, 3];
}
```

**Component breakdown:**

- `yield value` — emits `value` and pauses.
- The `yield` expression evaluates to the argument of the next `.next()` call.
- `yield* iterable` — delegates to another iterable.

**Syntax rules.**

- `yield` may appear only inside generator functions.
- `yield` has low precedence; wrap in parentheses when used in expressions.
- `yield*` accepts any iterable.

**Constraints and limitations.**

- `yield` cannot be used in arrow functions or non-generator functions.
- The first `.next()` call starts the generator; its argument is ignored.

### Annotated Complete Step by Step Code Examples

#### Example 1: Bidirectional communication

```typescript
// example_yield_communication.ts
function* accumulator(): Generator<number, number, number> {
  let total = 0;
  while (true) {
    const input: number = yield total;
    if (input === undefined) break;
    total += input;
  }
  return total;
}

const acc = accumulator();
acc.next(); // start
console.log(acc.next(10).value); // Expected: 10
console.log(acc.next(5).value);  // Expected: 15
```

**Expected Output:**
```
10
15
```

**Why this output occurs.** Each `.next(value)` passes `value` into the generator as the result of the `yield` expression. The generator accumulates the values and yields the running total.

---

#### Example 2: `yield*` with arrays

```typescript
// example_yield_star_array.ts
function* flatten<T>(arrays: T[][]): Generator<T, void, unknown> {
  for (const arr of arrays) {
    yield* arr;
  }
}

console.log([...flatten([[1, 2], [3], [4, 5]])]); // Expected: [ 1, 2, 3, 4, 5 ]
```

**Expected Output:**
```
[ 1, 2, 3, 4, 5 ]
```

**Why this output occurs.** `yield* arr` delegates to each inner array, yielding its elements in sequence.

---

### Real-World Cases

- **Coroutines**: passing values into generators for stateful computations.
- **Flattening**: `yield*` for nested iterables.
- **Pagination**: yielding pages and receiving next-page tokens.

---

## 4. Generator Types

### Definitions

**Core Definition.** **Generator types** are the TypeScript types that describe generator objects: **`Generator<T, TReturn, TNext>`**, **`IterableIterator<T>`**, and **`AsyncGenerator<T, TReturn, TNext>`**.

**Technical Definition.** `Generator<T, TReturn, TNext>` extends `Iterator<T, TReturn, TNext>` and `IterableIterator<T>`. `T` is the yielded value type, `TReturn` is the return value type, and `TNext` is the input type passed to `.next()`. `IterableIterator<T>` is the simpler interface for generators that do not use input values or return values. `AsyncGenerator` is the async counterpart.

**Beginner-Friendly Explanation.** Generator types are **the TypeScript types for generators**. `Generator<number>` means a generator that yields numbers. The additional type parameters describe the return value and the values passed back in.

### Purposes

- **To type generator return values** precisely.
- **To enable `for...of` iteration** with correct element types.
- **To model bidirectional communication** with `TNext`.
- **To distinguish sync and async generators**.
- **To compose generators** in typed pipelines.

### Syntax Rules and Structure

**Basic generator type:**

```typescript
function* f(): Generator<number, void, unknown> { yield 1; }
```

**IterableIterator:**

```typescript
function* g(): IterableIterator<string> { yield "a"; }
```

**Async generator:**

```typescript
async function* h(): AsyncGenerator<number, void, unknown> { yield 1; }
```

**Component breakdown:**

- `Generator<T, TReturn, TNext>` — full generator type.
- `IterableIterator<T>` — simpler type for `for...of`-only generators.
- `AsyncGenerator<T, TReturn, TNext>` — async generator type.

**Syntax rules.**

- `Generator` defaults to `Generator<T, unknown, unknown>` when parameters are omitted.
- `IterableIterator<T>` is assignable from `Generator<T, any, any>`.
- `AsyncGenerator` is used with `async function*`.

**Constraints and limitations.**

- `Generator` is **invariant** in `TNext`; mismatches cause type errors.
- `IterableIterator` does not expose `.return()` or `.throw()` in older TypeScript versions.

### Annotated Complete Step by Step Code Examples

#### Example 1: `Generator` vs. `IterableIterator`

```typescript
// example_generator_types.ts
function* numbers(): Generator<number, void, unknown> {
  yield 1;
  yield 2;
}

const gen: IterableIterator<number> = numbers(); // assignable
for (const n of gen) {
  console.log(n); // Expected: 1, 2
}
```

**Expected Output:**
```
1
2
```

**Why this output occurs.** `Generator<number, void, unknown>` is assignable to `IterableIterator<number>` because it satisfies the iteration protocol.

---

#### Example 2: Typed bidirectional generator

```typescript
// example_generator_types_bidirectional.ts
function* echo(): Generator<string, void, string> {
  while (true) {
    const input: string = yield "ready";
    console.log(`Received: ${input}`);
  }
}

const e = echo();
e.next();
e.next("hello"); // Expected: Received: hello
```

**Expected Output:**
```
Received: hello
```

**Why this output occurs.** The `TNext` type parameter is `string`, so `.next("hello")` is type-checked, and the `yield` expression is typed as `string`.

---

### Real-World Cases

- **Typed iteration**: `function* range(start: number, end: number): Generator<number>`.
- **Async streaming**: `async function* fetchPages(): AsyncGenerator<Page>`.
- **Coroutines**: bidirectional generators for stateful protocols.

---

## 5. Generator Best Practices

### Definitions

**Core Definition.** **Generator best practices** are guidelines for writing generators that are **type-safe, readable, and performant**, avoiding common pitfalls such as untyped yields, unbounded infinite loops, and iterator misuse.

**Technical Definition.** Best practices center on **annotating generator return types explicitly**, **preferring `IterableIterator<T>` for simple generators**, **using `Generator<T, TReturn, TNext>` for bidirectional communication**, **avoiding unbounded infinite generators unless consumed lazily**, **closing generators with `.return()` when abandoning iteration**, and **using `for...of` for consumption**. Async generators should handle rejections and cancellation properly.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep generators safe and readable. The big rules are: annotate the return type; use the simplest generator type that fits; do not create unbounded infinite generators unless you consume them lazily; and close generators when you are done with them.

### Purposes

- **To preserve type safety** with explicit generator annotations.
- **To avoid memory leaks** from abandoned generators.
- **To keep generators readable** with the simplest appropriate type.
- **To handle async generators safely** with rejection and cancellation.
- **To ensure lazy consumption** of infinite generators.

### Syntax Rules and Structure

**Best-practice rules:**

1. **Annotate generator return types explicitly.** Use `Generator<T, TReturn, TNext>` or `IterableIterator<T>` to document the yielded type.
2. **Prefer `IterableIterator<T>` for simple generators.** Use `Generator<T, TReturn, TNext>` only when you need bidirectional communication or the return value.
3. **Avoid unbounded infinite generators unless consumed lazily.** Infinite generators are safe with `for...of` and `take`-style helpers but dangerous with `[...gen]`.
4. **Close generators with `.return()` when abandoning iteration.** This runs `finally` blocks and releases resources.
5. **Use `for...of` for consumption.** It handles `.next()` and completion automatically.
6. **Handle async generator rejections.** Wrap `for await...of` in `try`/`catch` and narrow `unknown` errors.
7. **Use `yield*` for delegation** rather than manual loops.
8. **Avoid yielding `undefined` unintentionally.** Ensure all code paths yield the declared type.

**Component breakdown (generator design decision tree):**

```
Do you need bidirectional communication or a return value?
  ├── Yes → Use `Generator<T, TReturn, TNext>`.
  └── No → Use `IterableIterator<T>` for simple iteration.
           ├── Is the generator infinite?
           │   ├── Yes → Consume lazily (for...of, take).
           │   └── No → Safe to spread into arrays.
           └── Is the generator abandoned early?
               ├── Yes → Call `.return()` in a finally block.
               └── No → Use `for...of`.
```

**Constraints and limitations.**

- Infinite generators must be consumed lazily; spreading them hangs the program.
- Generators hold references to their state; abandoned generators may leak resources until garbage-collected.
- Async generators require `for await...of` and rejections must be handled.

### Annotated Complete Step by Step Code Examples

#### Example 1: Annotated finite generator

```typescript
// example_best_annotated.ts
function* range(start: number, end: number): IterableIterator<number> {
  for (let i = start; i < end; i++) {
    yield i;
  }
}

const values = [...range(0, 5)];
console.log(values); // Expected: [ 0, 1, 2, 3, 4 ]
```

**Expected Output:**
```
[ 0, 1, 2, 3, 4 ]
```

**Why this output occurs.** The generator is finite and annotated with `IterableIterator<number>`, making it safe to spread into an array.

---

#### Example 2: Closing a generator with `.return()`

```typescript
// example_best_close.ts
function* resource(): Generator<string, void, unknown> {
  try {
    yield "resource acquired";
    yield "resource in use";
  } finally {
    console.log("resource released"); // Expected: resource released
  }
}

const gen = resource();
console.log(gen.next().value); // Expected: resource acquired
gen.return(); // closes the generator, runs finally
```

**Expected Output:**
```
resource acquired
resource released
```

**Why this output occurs.** Calling `.return()` terminates the generator and runs the `finally` block, releasing resources even though the generator did not complete normally.

---

#### Example 3: Async generator with error handling

```typescript
// example_best_async_generator.ts
async function* fetchPages(): AsyncGenerator<number, void, unknown> {
  for (let page = 1; page <= 3; page++) {
    yield page;
  }
}

async function main() {
  try {
    for await (const page of fetchPages()) {
      console.log(`Page ${page}`); // Expected: Page 1, Page 2, Page 3
    }
  } catch (error: unknown) {
    if (error instanceof Error) console.error(error.message);
  }
}

main();
```

**Expected Output:**
```
Page 1
Page 2
Page 3
```

**Why this output occurs.** `for await...of` consumes the async generator, awaiting each yielded value. The `try`/`catch` handles any rejection, narrowing `unknown` to `Error`.

---

### Real-World Cases

- **Pagination**: async generators yielding pages until exhausted.
- **Stream processing**: generators yielding chunks of data lazily.
- **Resource management**: `finally` blocks in generators for cleanup.
- **Code review**: rejecting unannotated generators and unbounded infinite generators without lazy consumption.

---

## References

- TypeScript 2.3 Release Notes — Generators and Iteration — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-3.html
- TypeScript 2.5 Release Notes — `Generator` Type Improvements — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-5.html
- TypeScript Handbook — Iterators and Generators — https://www.typescriptlang.org/docs/handbook/iterators-and-generators.html
- TypeScript Handbook — `Symbol.iterator` and `for...of` — https://www.typescriptlang.org/docs/handbook/2/iterators-and-generators.html
- MDN — `function*` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*
- MDN — `yield` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/yield
- MDN — `yield*` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/yield*
- MDN — `Generator` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator
- MDN — `AsyncGenerator` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/AsyncGenerator
- MDN — Iteration Protocols — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols
- TypeScript ESLint — `no-unnecessary-generics` Rule — https://typescript-eslint.io/rules/no-unnecessary-type-arguments/