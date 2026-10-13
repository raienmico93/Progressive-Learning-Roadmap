# Async Generators — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** An async generator is a function that combines the features of async functions and generator functions. It is declared with `async function*` and can use both `await` and `yield` within its body, producing a sequence of values asynchronously.

**Technical Definition:** Per MDN, "The `async function*` declaration creates a binding of a new async generator function to a given name. An `async function*` declaration creates an `AsyncGeneratorFunction` object. Each time when an async generator function is called, it returns a new `AsyncGenerator` object, which conforms to the async iterator protocol. Every call to `next()` returns a Promise that resolves to the iterator result object". Async generators were introduced with ECMAScript 2018 and are part of the async iteration proposal. They combine the features of async functions and generator functions: you can use both the `await` and `yield` keywords within the function body.

**Beginner-Friendly Explanation:** An async generator is like a regular generator (which produces values one at a time) but with the added ability to wait for asynchronous operations between values. It's a function that can pause to wait for a network request or a file read, produce a value, pause again, and so on. You consume it with `for await...of`, which waits for each value to be ready before processing it.

### Key Characteristics

- **Combines async and generators:** Uses both `await` (to wait for Promises) and `yield` (to produce values).
- **AsyncIterator protocol:** Returns an `AsyncGenerator` object conforming to the async iterator protocol.
- **Promise-returning `next()`:** Each call to `next()` returns a Promise that resolves to `{ value, done }`.
- **Async iteration:** Consumed with `for await...of` or manual `next()` calls.
- **Lazy evaluation:** Values are produced on demand, one at a time.
- **Streaming-friendly:** Ideal for processing streams of asynchronous data.
- **No arrow function equivalent:** Async generator functions cannot be written as arrow functions.
- **Error propagation:** Errors can be caught with `try...catch` inside the generator.

### Prerequisites

- Understanding of generators (`function*`, `yield`, `next()`, `return()`, `throw()`).
- Familiarity with async functions and `await`.
- Knowledge of Promises and asynchronous programming.
- Awareness of the async iteration protocol (`Symbol.asyncIterator`).

### Related Programming Areas

- **Streaming data:** Processing event streams, file reads, and network responses incrementally.
- **Pagination:** Fetching paginated API results lazily.
- **Real-time data:** Consuming WebSocket messages or Server-Sent Events.
- **Node.js streams:** Async generators can be converted to/from Node.js streams.
- **Array.fromAsync:** Collecting async generator values into an array (ES2025).

### Core Concepts / Features

---

## Async Generator Functions

### Definitions

**Core Definition:** An async generator function is a function declared with `async function*` that returns an `AsyncGenerator` object and can use both `await` and `yield`.

**Technical Definition:** Per MDN, an `async function*` declaration creates an `AsyncGeneratorFunction` object. Each call to the async generator function returns a new `AsyncGenerator` object that conforms to the async iterator protocol. Every call to `next()` returns a Promise that resolves to an iterator result object.

**Beginner-Friendly Explanation:** An async generator function is a function you declare with `async function*`. Inside it, you can `await` asynchronous operations and `yield` values. When you call the function, it returns an async generator object that you can iterate over with `for await...of`.

### Purposes

- To produce a sequence of values asynchronously, waiting for each value to be ready.
- To combine the power of `await` (for asynchronous operations) with `yield` (for producing values).
- To create lazy, asynchronous data sources.
- To process streams of asynchronous data incrementally.
- To implement paginated API fetching with clean, readable code.

### Syntax Rules and Structure

```js
async function* name(param0) {
  statements
}
async function* name(param0, param1, /* …, */ paramN) {
  statements
}
```

**Component Breakdown:**
- `async`: Marks the function as asynchronous (must not have a line terminator between `async` and `function`).
- `function*`: The generator marker.
- `name`: The function name.
- `param0, param1, ...`: The parameter list.
- `statements`: The function body, which may contain both `await` and `yield`.

**Syntax Rules:**
- `async` and `function` must not be separated by a line terminator; otherwise, Automatic Semicolon Insertion makes `async` an identifier.
- `function` and `*` are separate tokens and can be separated by whitespace.
- Async generator functions have no arrow function equivalent.
- The function body can use both `await` and `yield`.

**Constraints and Limitations:**
- `yield` and `await` cannot be used outside their valid contexts.
- Async generators cannot be used as constructors.
- The `AsyncGeneratorFunction` is not a global object; it is created via `async function*` declarations.

### Annotated Code Examples

#### Example 1: Basic Async Generator

```js
// Step 1: Define an async generator function.
async function* generateSequence() {
  yield await Promise.resolve(1);
  yield await Promise.resolve(2);
  yield await Promise.resolve(3);
}

// Step 2: Consume the generator with for await...of.
async function consume() {
  for await (const value of generateSequence()) {
    console.log(value); // 1, 2, 3
  }
}

consume();
```

**Expected Output:**
```
1
2
3
```

**Why This Output:** The async generator yields values one at a time, awaiting each Promise. The `for await...of` loop waits for each value to be ready before processing it.

#### Example 2: Async Generator with Delays

```js
async function* generateWithDelays() {
  yield await new Promise((r) => setTimeout(() => r("First"), 1000));
  yield await new Promise((r) => setTimeout(() => r("Second"), 500));
  yield await new Promise((r) => setTimeout(() => r("Third"), 300));
}

async function main() {
  for await (const value of generateWithDelays()) {
    console.log(value);
  }
}

main();
```

**Expected Output (with delays between each value):**
```
First
Second
Third
```

**Why This Output:** Each value is produced after the specified delay. The `for await...of` loop waits for each value to be available.

### Real-World Cases

- **Paginated API fetching:** Fetching pages of data from an API with `yield await fetchPage(page)`.
- **File reading:** Reading a file line by line in Node.js.
- **WebSocket messages:** Yielding incoming messages as they arrive.

---

## `async function*`

### Definitions

**Core Definition:** `async function*` is the declaration syntax used to create an async generator function.

**Technical Definition:** Per MDN, "The `async function*` declaration creates a binding of a new async generator function to a given name. You can also define async generator functions using the `async function*` expression". It creates an `AsyncGeneratorFunction` object.

**Beginner-Friendly Explanation:** `async function*` is the way you tell JavaScript "this is an async generator function." The `async` part means it can `await`, and the `*` means it's a generator.

### Purposes

- To declare an async generator function.
- To combine `await` and `yield` in a single function.
- To create an async iterable sequence.

### Syntax Rules and Structure

```js
// Declaration:
async function* name(param0) {
  statements
}

// Expression:
const gen = async function* (param0) {
  statements
};
```

**Component Breakdown:**
- `async function*`: The declaration keywords.
- `name`: The function name.
- `statements`: The function body containing `await` and `yield`.

**Syntax Rules:**
- The expression form can be used as an IIFE (Immediately Invoked Function Expression) to create temporary async iterables.
- Async generator functions can be used as methods in classes or object literals.

**Constraints and Limitations:**
- Cannot be used with `new`.
- The `*` must be present; omitting it creates a regular async function.

### Annotated Code Examples

#### Example 1: Declaration and Expression Forms

```js
// Declaration:
async function* declaredGen() {
  yield 1;
  yield 2;
}

// Expression:
const expressedGen = async function* () {
  yield 3;
  yield 4;
};

async function main() {
  for await (const v of declaredGen()) console.log(v); // 1, 2
  for await (const v of expressedGen()) console.log(v); // 3, 4
}
main();
```

**Expected Output:**
```
1
2
3
4
```

**Why This Output:** Both declaration and expression forms produce async generators that can be consumed with `for await...of`.

#### Example 2: IIFE Async Generator

```js
const results = [];

(async function* () {
  yield await Promise.resolve("a");
  yield await Promise.resolve("b");
})()
  .next()
  .then((r) => results.push(r.value));

console.log(results); // ["a"] after microtask
```

**Expected Output:**
```
[ 'a' ]
```

**Why This Output:** The IIFE creates an async generator, and calling `next()` returns a Promise that resolves to the first value.

### Real-World Cases

- **Class methods:** `async *fetchPages()` in a class for paginated API access.
- **Utility functions:** Reusable async generators for common streaming patterns.

---

## `for await...of`

### Definitions

**Core Definition:** `for await...of` is a loop statement that iterates over async iterable objects as well as sync iterables, awaiting each value.

**Technical Definition:** Per MDN, "The `for await...of` statement creates a loop iterating over async iterable objects as well as sync iterables. This statement can only be used in contexts where `await` can be used, which includes inside an async function body and in a module". It works on both sync and async iterables, while `for...of` only works on sync iterables.

**Beginner-Friendly Explanation:** `for await...of` is like a regular `for...of` loop, but it waits for each value to be ready before processing it. It's designed for async generators and other async iterables.

### Purposes

- To consume async generator values in a loop.
- To iterate over async iterables (streams, paginated data, etc.).
- To simplify code that would otherwise require manual `next()` calls.
- To combine iteration with `await` in a readable way.

### Syntax Rules and Structure

```js
for await (const variable of iterable) {
  statement
}
```

**Component Breakdown:**
- `variable`: Receives each value from the sequence. May be a declaration (`const`, `let`, `var`) or an assignment target.
- `iterable`: An async iterable or sync iterable.
- `statement`: The body executed on each iteration.

**Syntax Rules:**
- Only valid inside async functions or at the top level of modules.
- Works on both sync and async iterables.
- Variables declared with `var` are not local to the loop.

**Constraints and Limitations:**
- Sequential: each iteration waits for the previous one.
- Breaking out of the loop calls `return()` on the async iterator, triggering cleanup.

### Annotated Code Examples

#### Example 1: Basic `for await...of`

```js
async function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

async function main() {
  for await (const value of gen()) {
    console.log(value); // 1, 2, 3
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

**Why This Output:** `for await...of` calls `next()` on the async generator, awaiting each result.

#### Example 2: `for await...of` with Sync Iterable

```js
async function main() {
  for await (const value of [1, 2, 3]) {
    console.log(value); // 1, 2, 3
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

**Why This Output:** `for await...of` also works on sync iterables, awaiting each value (even though sync values are immediately available).

### Real-World Cases

- **Stream processing:** `for await (const chunk of readableStream) { ... }`.
- **Paginated APIs:** Iterating through pages of results.
- **WebSocket messages:** Processing incoming messages.

---

## Async Iteration

### Definitions

**Core Definition:** Async iteration is the process of iterating over asynchronous data sources using the async iterator protocol and `for await...of`.

**Technical Definition:** Per MDN, the async iterator protocol is implemented by objects that provide a `next()` method returning a Promise for an iterator result object. An async iterable object must have a `[Symbol.asyncIterator]()` method that returns an async iterator. The `AsyncIterator` object is the base for all async iterators, and it provides a `[Symbol.asyncIterator]()` method that returns the iterator itself, making it also async iterable.

**Beginner-Friendly Explanation:** Async iteration is how JavaScript loops over data that arrives asynchronously — like chunks of a file or pages from an API. The async iterator protocol defines how to get values one at a time, waiting for each.

### Purposes

- To provide a standard protocol for asynchronous data sources.
- To enable `for await...of` loops over async data.
- To allow custom objects to be async iterable.
- To integrate async generators with other async APIs.

### Syntax Rules and Structure

```js
// Async iterable protocol:
const asyncIterable = {
  [Symbol.asyncIterator]() {
    return {
      next() {
        return Promise.resolve({ value: ..., done: ... });
      },
    };
  },
};
```

**Component Breakdown:**
- `[Symbol.asyncIterator]()`: The method that returns an async iterator.
- `next()`: Returns a Promise resolving to `{ value, done }`.

**Syntax Rules:**
- The async iterator protocol requires `next()` to return a Promise.
- The async iterable protocol requires `[Symbol.asyncIterator]()`.
- Async generators automatically implement both protocols.

**Constraints and Limitations:**
- `AsyncIterator.prototype` does not implement `[Symbol.iterator]()`, so async iterators are not sync iterable by default.

### Annotated Code Examples

#### Example 1: Custom Async Iterable

```js
const asyncIterable = {
  [Symbol.asyncIterator]() {
    let i = 0;
    return {
      async next() {
        if (i < 3) {
          return { value: i++, done: false };
        }
        return { value: undefined, done: true };
      },
    };
  },
};

async function main() {
  for await (const value of asyncIterable) {
    console.log(value); // 0, 1, 2
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

**Why This Output:** The custom async iterable implements the protocol, and `for await...of` consumes it.

### Real-World Cases

- **ReadableStream:** Web streams are async iterable.
- **Node.js streams:** Async iterators for reading data.
- **Custom data sources:** Databases, message queues.

---

## Streaming Data

### Definitions

**Core Definition:** Streaming data with async generators means processing data incrementally as it arrives, rather than waiting for the entire dataset.

**Technical Definition:** Per the reference, "The primary use case for async generators is in long-running tasks like consuming event streams or fetching large paginated datasets". Async generators are "effectively a first-class language-level stream construct" and "a Node.js readable stream can be created from an asynchronous generator using the `Readable.from()` utility method".

**Beginner-Friendly Explanation:** Streaming data means processing data in chunks as it arrives. Async generators are perfect for this: they can produce chunks one at a time, and consumers can process each chunk without waiting for the entire dataset.

### Purposes

- To process large datasets without loading everything into memory.
- To consume event streams, file reads, and network responses incrementally.
- To implement backpressure-friendly data processing.
- To create streaming responses in web frameworks.

### Syntax Rules and Structure

```js
async function* streamData() {
  while (hasMoreData()) {
    const chunk = await fetchNextChunk();
    yield chunk;
  }
}

for await (const chunk of streamData()) {
  processChunk(chunk);
}
```

**Component Breakdown:**
- `fetchNextChunk()`: An asynchronous operation that returns the next chunk.
- `yield chunk`: Produces the chunk for the consumer.

**Syntax Rules:**
- Async generators naturally produce values lazily.
- Consumers can process each value before requesting the next.

**Constraints and Limitations:**
- Backpressure is handled by the consumer's processing speed.
- Manually closing the generator is recommended to avoid resource leaks.

### Annotated Code Examples

#### Example 1: Paginated API Streaming

```js
async function* fetchAllPages(baseUrl) {
  let page = 1;
  while (true) {
    const response = await fetch(`${baseUrl}?page=${page}`);
    const data = await response.json();
    if (data.length === 0) break;
    yield data;
    page++;
  }
}

async function processAllPages() {
  for await (const page of fetchAllPages("/api/items")) {
    console.log(`Received ${page.length} items`);
  }
}
processAllPages();
```

**Expected Output:** Each page is logged as it is fetched.

**Why This Output:** The async generator fetches pages one at a time and yields each page. The consumer processes each page before the next is fetched.

#### Example 2: Node.js Stream from Async Generator

```js
import { Readable } from "node:stream";

async function* generateData() {
  for (let i = 0; i < 5; i++) {
    yield `Chunk ${i}\n`;
  }
}

const stream = Readable.from(generateData());
stream.pipe(process.stdout);
```

**Expected Output:**
```
Chunk 0
Chunk 1
Chunk 2
Chunk 3
Chunk 4
```

**Why This Output:** `Readable.from()` converts the async generator into a Node.js readable stream, which pipes to stdout.

### Real-World Cases

- **Server-Sent Events (SSE):** Streaming events from a server.
- **File processing:** Reading large files line by line.
- **Data migration:** Processing records in batches.
- **Real-time dashboards:** Streaming metrics from a server.

---

## Async Generator Use Cases

### Definitions

**Core Definition:** Async generator use cases are the practical scenarios where async generators provide unique value, including streaming, pagination, event processing, and cooperative multitasking.

**Technical Definition:** Per the reference, "Async generators and the `for await...of` syntax extend these ideas to asynchronous iteration—useful for streaming and staged operations". They are particularly suited for long-running tasks, event streams, and paginated datasets.

**Beginner-Friendly Explanation:** Async generators are useful whenever you need to produce a sequence of values asynchronously — like fetching data from an API page by page, processing a stream of events, or reading a large file in chunks.

### Purposes

- To implement paginated API fetching with lazy evaluation.
- To process event streams (WebSockets, SSE, message queues).
- To read and process large files incrementally.
- To create streaming HTTP responses.
- To coordinate multiple asynchronous data sources.

### Syntax Rules and Structure

Async generators are used in various patterns. Common ones include:

```js
// 1. Pagination:
async function* paginate(url) {
  let next = url;
  while (next) {
    const res = await fetch(next);
    const data = await res.json();
    yield data.items;
    next = data.nextPage;
  }
}

// 2. Event stream:
async function* eventStream(emitter) {
  while (true) {
    yield await new Promise((r) => emitter.once("data", r));
  }
}

// 3. File reading:
async function* readLines(file) {
  for await (const chunk of file) {
    for (const line of chunk.split("\n")) {
      yield line;
    }
  }
}
```

**Component Breakdown:**
- **Pagination:** Yields each page of data.
- **Event stream:** Yields events as they occur.
- **File reading:** Yields lines or chunks.

**Syntax Rules:**
- Async generators are ideal when the data source is asynchronous and potentially unbounded.
- They can be composed with other generators using `yield*`.

**Constraints and Limitations:**
- Async generators are single-use; once exhausted, they cannot be restarted.
- Manual cleanup may be required to release resources.

### Annotated Code Examples

#### Example 1: Paginated API Fetching

```js
async function* fetchCommits(repo) {
  let url = `https://api.github.com/repos/${repo}/commits`;
  while (url) {
    const response = await fetch(url);
    const commits = await response.json();
    yield commits;
    const linkHeader = response.headers.get("Link");
    url = linkHeader?.match(/<([^>]+)>;\s*rel="next"/)?.[1];
  }
}

async function main() {
  for await (const commits of fetchCommits("user/repo")) {
    console.log(`Got ${commits.length} commits`);
  }
}
main();
```

**Expected Output:** Each page of commits is logged as it is fetched.

**Why This Output:** The async generator fetches pages lazily, following the `next` link until there are no more pages.

#### Example 2: Event Stream Processing

```js
async function* eventStream(emitter) {
  const queue = [];
  emitter.on("data", (data) => queue.push(data));
  while (true) {
    if (queue.length > 0) {
      yield queue.shift();
    } else {
      await new Promise((r) => setTimeout(r, 100));
    }
  }
}

async function main() {
  for await (const event of eventStream(myEmitter)) {
    console.log("Event:", event);
  }
}
```

**Expected Output:** Events are logged as they arrive.

**Why This Output:** The async generator polls for new events and yields them as they become available.

### Real-World Cases

- **GitHub API:** Fetching commits with pagination.
- **Database cursors:** Iterating over query results in batches.
- **Log processing:** Streaming log files line by line.
- **Real-time notifications:** Streaming push notifications.

---

## Async Generators vs Other Patterns

### Definitions

**Core Definition:** Async generators differ from Promises, async functions, and regular generators in their ability to produce multiple values asynchronously over time.

**Technical Definition:** Per the comparison, "While generators where considered for the implementation of async/await, all JS engines use promises to implement it. Generators on their own do not make code run asynchronously". Async generators return a generator object that vends Promises, while async functions always return a Promise. Promises emit a single value, while generators (and async generators) can emit many values over time.

**Beginner-Friendly Explanation:** A Promise represents a single future value. An async function returns a Promise. An async generator represents a stream of future values — it can produce many values over time, each awaited asynchronously.

### Purposes

- To understand when to use async generators vs Promises or async functions.
- To recognise that async generators are for streams, not single values.
- To choose the right tool for the task.

### Syntax Rules and Structure

```js
// Promise: single value
const promise = fetchData();
promise.then((value) => console.log(value));

// Async function: single value (Promise)
async function fetchOne() {
  return await fetchData();
}

// Async generator: many values
async function* fetchMany() {
  yield await fetchData();
  yield await fetchData();
  yield await fetchData();
}
```

**Component Breakdown:**
- **Promise:** Represents one eventual value.
- **Async function:** Returns one Promise (one value).
- **Async generator:** Returns an AsyncGenerator (many values over time).

**Syntax Rules:**
- Use async generators when you need to produce multiple values asynchronously.
- Use async functions for single asynchronous operations.
- Use Promises for representing a single future value.

**Constraints and Limitations:**
- Async generators cannot be used where a Promise is expected.
- Async generators require `for await...of` or manual `next()` calls.

### Annotated Code Examples

#### Example 1: Async Function vs Async Generator

```js
// Async function: returns one value.
async function fetchUser() {
  return await fetch("/api/user").then((r) => r.json());
}

// Async generator: returns many values.
async function* fetchAllUsers() {
  let page = 1;
  while (true) {
    const users = await fetch(`/api/users?page=${page}`).then((r) => r.json());
    if (users.length === 0) break;
    yield users;
    page++;
  }
}
```

**Expected Output:** The async function returns one user. The async generator returns pages of users.

**Why This Output:** The async function is for a single value; the async generator is for a stream of values.

### Real-World Cases

- **Async function:** Loading a single configuration file.
- **Async generator:** Streaming all configuration changes over time.
- **Promise:** A single network request.
- **Async generator:** A continuous stream of network events.

---

## Async Generator Pitfalls

### Definitions

**Core Definition:** Async generator pitfalls are common mistakes and anti-patterns that lead to bugs, resource leaks, or unmaintainable code.

**Technical Definition:** Common pitfalls include: (1) yielding inside a context manager in an async generator, which is unsafe because cleanup may be delayed until the generator is closed; (2) not manually closing the generator, relying on garbage collection instead; (3) iterating and closing the same generator concurrently; (4) creating a generator when a loop is not already running; (5) missing cleanup under backpressure or runaway memory when the producer outruns the consumer.

**Beginner-Friendly Explanation:** Async generators are powerful but easy to misuse. Common mistakes include forgetting to close them (causing resource leaks), yielding inside resource-managing blocks (which can delay cleanup), and letting the producer generate values faster than the consumer can process them.

### Purposes

- To recognise and avoid common async generator mistakes.
- To write more reliable and resource-safe asynchronous code.
- To prevent memory leaks and resource exhaustion.
- To handle backpressure correctly.

### Syntax Rules and Structure

Common pitfalls and their solutions:

```js
// PITFALL 1: Not closing the generator
async function* gen() {
  const resource = await openResource();
  try {
    yield await resource.read();
  } finally {
    await resource.close(); // Only runs if generator is closed
  }
}

// FIX: Manually close when done
const g = gen();
await g.next();
await g.return(); // Triggers finally block

// PITFALL 2: Producer outrunning consumer
async function* fastProducer() {
  while (true) yield await fetchData(); // Produces faster than consumed
}

// FIX: Use bounded concurrency or backpressure
```

**Component Breakdown:**
- **Not closing:** Resources are not released until garbage collection.
- **Backpressure:** The producer outpaces the consumer, causing memory growth.

**Syntax Rules:**
- Always close async generators when done.
- Use `try...finally` inside generators for cleanup.
- Consider backpressure when producing data faster than consumption.

**Constraints and Limitations:**
- Garbage collection is not deterministic; relying on it for cleanup is unreliable.
- Concurrent iteration and closure of the same generator is undefined behaviour.

### Annotated Code Examples

#### Example 1: Cleanup with `return()`

```js
async function* resourceGenerator() {
  const resource = await openResource();
  try {
    while (true) {
      yield await resource.read();
    }
  } finally {
    await resource.close(); // Cleanup
  }
}

const g = resourceGenerator();
await g.next();
await g.next();
await g.return(); // Triggers cleanup
console.log("Resource closed");
```

**Expected Output:**
```
Resource closed
```

**Why This Output:** Calling `return()` on the async generator triggers the `finally` block, closing the resource.

#### Example 2: Backpressure with Async Generators

```js
async function* boundedProducer(maxBuffer = 10) {
  for (let i = 0; i < 1000; i++) {
    yield i;
    if (i % maxBuffer === 0) {
      await new Promise((r) => setTimeout(r, 10)); // Yield control
    }
  }
}

for await (const value of boundedProducer(10)) {
  await processSlowly(value); // Consumer is slower than producer
}
```

**Expected Output:** Values are produced with periodic pauses, preventing memory buildup.

**Why This Output:** The producer yields control periodically, allowing the consumer to keep up.

### Real-World Cases

- **Database connections:** Ensuring connections are closed when generators are done.
- **File handles:** Closing files after reading.
- **WebSocket connections:** Closing connections when no longer needed.

---

## Async Generator Best Practices

### Definitions

**Core Definition:** Async generator best practices are guidelines for writing clean, maintainable, and resource-safe async generator code.

**Technical Definition:** Per the best practices reference, "Recommend to manually close the generator instead of relying on the gc. Avoid iterating and closing the same generator concurrently. Only create a generator when a loop is already running". Additionally, "Using `contextlib.aclosing` around all call sites of an async generator is a valid way to guarantee timely cleanup".

**Beginner-Friendly Explanation:** Always close your async generators. Don't iterate and close the same generator at the same time. Handle cleanup with `try...finally`. And be mindful of backpressure.

### Purposes

- To prevent resource leaks from unclosed generators.
- To ensure timely cleanup of resources.
- To handle backpressure correctly.
- To write maintainable and predictable async generator code.

### Syntax Rules and Structure

Best practices in practice:

```js
// GOOD: try...finally for cleanup
async function* safeGenerator() {
  const resource = await acquire();
  try {
    yield await resource.read();
  } finally {
    await resource.release(); // Always runs
  }
}

// GOOD: Explicit closure
const gen = safeGenerator();
try {
  for await (const value of gen) {
    process(value);
  }
} finally {
  await gen.return(); // Guarantee cleanup
}

// GOOD: Bounded production
async function* bounded() {
  let count = 0;
  while (count < 100) {
    yield await fetchNext();
    count++;
  }
}
```

**Component Breakdown:**
- **`try...finally`:** Ensures cleanup runs even on early termination.
- **Explicit `return()`:** Manually closes the generator.
- **Bounded production:** Limits the number of yielded values.

**Syntax Rules:**
- Always use `try...finally` for resource cleanup.
- Call `return()` on generators when done.
- Consider backpressure in producers.

**Constraints and Limitations:**
- `await` inside `finally` is allowed in async generators but may be affected by cancellation.
- Some runtimes may not fully support async generator cleanup.

### Annotated Code Examples

#### Example 1: Safe Resource Management

```js
async function* readFileLines(filePath) {
  const file = await fs.promises.open(filePath);
  try {
    const stream = file.createReadStream();
    for await (const chunk of stream) {
      yield chunk.toString();
    }
  } finally {
    await file.close();
  }
}

async function processFile() {
  const gen = readFileLines("large-file.txt");
  try {
    for await (const line of gen) {
      console.log(line);
    }
  } finally {
    await gen.return(); // Guarantee cleanup
  }
}
processFile();
```

**Expected Output:** Each line is logged, and the file is closed when done.

**Why This Output:** The `try...finally` inside the generator ensures the file is closed. The outer `finally` ensures `return()` is called.

### Real-World Cases

- **Node.js file system:** Reading files line by line with proper cleanup.
- **Database cursors:** Closing cursors when iteration is complete.
- **Network streams:** Closing connections when streaming is done.

---

## References

- MDN Web Docs – async function* — https://mdn.org.cn/en-US/docs/Web/JavaScript/Reference/Statements/async_function*
- MDN Web Docs – for await...of — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for-await...of
- MDN Web Docs – AsyncIterator — https://mdn.org.cn/en-US/docs/Web/JavaScript/Reference/Global_Objects/AsyncIterator
- MDN Web Docs – Symbol.asyncIterator — https://mdn.org.cn/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/asyncIterator
- MDN Web Docs – AsyncGenerator — https://mdn.org.cn/en-US/docs/Web/JavaScript/Reference/Global_Objects/AsyncGenerator
- ECMAScript 2026 Language Specification – Async Iteration — https://tc39.es/ecma262/
- TC39 – Asynchronous Iterators for JavaScript (Proposal) — https://www.proposals.es/proposals/Asynchronous%20Iterators
- Node.js Documentation – Streams and Async Generators — https://beta.docs.nodejs.org
- JavaScript AsyncGenerator: Advanced Asynchronous Iteration — https://raw.githubusercontent.com/serpuniversity/learn-docs/refs/heads/main/javascript/JavaScript%20AsyncGenerator.md
- CoreUI – How to create a generator function in JavaScript — https://coreui.io/answers/how-to-create-a-generator-function-in-javascript/