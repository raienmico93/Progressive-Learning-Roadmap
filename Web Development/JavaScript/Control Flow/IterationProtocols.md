# JavaScript Iteration Protocols: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
JavaScript iteration protocols are a set of conventions—not built-in syntax or objects—that allow any object to define or customize how it is iterated, enabling consistent traversal across `for...of` loops, spread syntax, destructuring, and other language constructs.

**Technical Definition**
Per the ECMAScript Language Specification, the iteration protocols consist of two complementary interfaces: the **iterable protocol** (defined by the `%Symbol.iterator%` well-known symbol, whose method returns an iterator object) and the **iterator protocol** (defined by an object's `next()` method, which returns an `IteratorResult` object with `value` and `done` properties). Together, these protocols form the foundation for all synchronous iteration in JavaScript. An async variant exists via `%Symbol.asyncIterator%`, whose `next()` method returns a `Promise` for an `IteratorResult`.

**Beginner-Friendly Explanation**
Iteration protocols are like a universal remote control for loops. Any object can "opt in" to being loopable by following two simple rules: (1) it must have a special method called `Symbol.iterator` that gives out a "cursor," and (2) that cursor must have a `next()` method that hands out one value at a time and says when it is finished. This is how JavaScript makes arrays, strings, maps, sets, and your own custom objects all work seamlessly with `for...of` loops.

### Key Characteristics

- **Protocol-based, not class-based**: Any object can implement them; no inheritance required.
- **Two complementary protocols**: The iterable protocol (what to iterate) and the iterator protocol (how to iterate).
- **Well-known symbols**: `Symbol.iterator` and `Symbol.asyncIterator` are the entry points.
- **Consistent result shape**: Every `next()` call returns `{ value, done }`.
- **Lazy evaluation**: Values are produced on demand, not pre-computed.
- **Language integration**: `for...of`, spread, destructuring, `Array.from`, `Map`/`Set` constructors, and `Promise.all` all consume iterables.

### Prerequisites

- Variables, data types, and object literals.
- Functions and methods (including arrow functions).
- Basic understanding of `this` and prototype chains.
- Familiarity with `for...of` loops (see the Loops cheat sheet).
- Basic understanding of generators (`function*`, `yield`) is helpful but not required.

### Related Programming Areas

- Loops (`for...of`, `for await...of`) and array methods.
- Generators and async generators.
- Destructuring assignment and spread syntax.
- Map, Set, and other collection types.
- Lazy evaluation and stream processing.
- Custom data structures and domain-specific collections.

### Core Concepts / Features

| Concept | Category |
|---|---|
| Iterable protocol | Protocol |
| Iterator protocol | Protocol |
| `Symbol.iterator` | Well-known symbol |
| `next()` | Iterator method |
| `done` | IteratorResult property |
| `value` | IteratorResult property |
| Built-in iterables | Language feature |
| Custom iterables | Implementation pattern |
| Iterating objects | Use case |
| Iterating maps | Use case |
| Iterating sets | Use case |
| Iterating strings | Use case |
| Iterating generators | Use case |

---

## Core Concepts / Features

---

## 1. Iterable Protocol

### Definitions

**Core Definition**
The iterable protocol allows JavaScript objects to define or customize their iteration behaviour, such as what values are looped over in a `for...of` construct.

**Technical Definition**
The iterable interface includes the `%Symbol.iterator%` property, whose value is a function that returns an object conforming to the iterator interface. Any object (or an object on its prototype chain) with a `[Symbol.iterator]` key is considered iterable. When an object needs to be iterated (e.g., at the start of a `for...of` loop), its `[Symbol.iterator]()` method is called with no arguments, and the returned iterator is used to obtain values.

**Beginner-Friendly Explanation**
An object is "iterable" if it has a special method that tells JavaScript how to loop over it. If you can use `for...of` on it, it is iterable. Arrays, strings, Maps, and Sets come with this built in; plain objects do not.

### Purposes

- To provide a standard, uniform way for any object to define its iteration behaviour.
- To enable `for...of` loops, spread syntax, and destructuring to work with custom data structures.
- To decouple iteration logic from the consumer (the loop or spread operator).
- To allow lazy, on-demand value production instead of pre-computed arrays.

### Syntax Rules and Structure

**Complete General Syntax**
```js
const iterable = {
  [Symbol.iterator]() {
    // return an iterator object
    return {
      next() {
        // return { value, done }
      }
    };
  }
};
```

**Component Breakdown**

| Component | Description |
|---|---|
| `[Symbol.iterator]` | A computed property key using the well-known `Symbol.iterator`. |
| `() => { … }` | A zero-argument function (method) that returns an iterator object. |
| `this` inside the method | Refers to the iterable object; can be used to access its data. |
| Return value | Must be an object conforming to the iterator protocol. |

**Syntax Rules**

- The method is called with no arguments when iteration begins.
- It is invoked as a method on the iterable object, so `this` is the iterable.
- The method can be a regular function or a generator function (which returns an iterator).
- If the method returns a non-object value, the iterable is "non-well-formed" and will cause a `TypeError` when used.

**Constraints and Limitations**

- Objects that do not implement `[Symbol.iterator]()` are not iterable and cannot be used with `for...of`, spread, or destructuring.
- Inherited `[Symbol.iterator]` methods are found via the prototype chain.
- A non-well-formed iterable (returning a non-object) causes runtime exceptions.

### Annotated Code Examples

**Example 1: A minimal iterable using a generator function**

```js
// Step 1: Create an object with a [Symbol.iterator] method.
const myIterable = {
  // Step 2: Use a generator function to define iteration.
  *[Symbol.iterator]() {
    yield 1;
    yield 2;
    yield 3;
  }
};

// Step 3: Spread the iterable into an array.
console.log([...myIterable]);

// Expected Output:
// [1, 2, 3]
```

**Breakdown**: The `*[Symbol.iterator]()` method is a generator function. When `[...myIterable]` runs, JavaScript calls `myIterable[Symbol.iterator]()`, gets a generator object, and calls `next()` repeatedly until `done` is `true`. The yielded values `1`, `2`, `3` are collected into the array.

**Example 2: A manual iterator (without generators)**

```js
// Step 1: Define an object with a custom iterable protocol implementation.
const countdown = {
  data: [3, 2, 1],
  // Step 2: The iterable protocol method.
  [Symbol.iterator]() {
    let index = 0;
    const data = this.data;
    // Step 3: Return an object conforming to the iterator protocol.
    return {
      next() {
        if (index < data.length) {
          return { value: data[index++], done: false };
        }
        return { value: undefined, done: true };
      }
    };
  }
};

// Step 4: Use the iterable in a for...of loop.
for (const num of countdown) {
  console.log(num);
}

// Expected Output:
// 3
// 2
// 1
```

**Breakdown**: `[Symbol.iterator]()` returns an iterator object with a `next()` method. Each call to `next()` returns the next value with `done: false`, and finally `{ value: undefined, done: true }`.

### Real-World Cases

- **Custom collections**: Linked lists, trees, or graphs that need to be looped over.
- **Lazy sequences**: Infinite or expensive-to-compute sequences (e.g., Fibonacci, prime generators).
- **Domain objects**: A `Playlist` object that yields songs, a `Matrix` that yields rows.
- **Data pipelines**: Transformers that yield processed values on demand.

### References

- MDN Web Docs — Iteration protocols — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols
- ECMAScript Language Specification — The Iterable Interface — https://tc39.es/ecma262/#sec-iterable-interface
- MDN Web Docs — `Symbol.iterator` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/iterator

---

## 2. Iterator Protocol

### Definitions

**Core Definition**
The iterator protocol defines a standard way to produce a sequence of values (finite or infinite) through a `next()` method that returns an object with `value` and `done` properties.

**Technical Definition**
An object is an iterator when it implements a `next()` method with the following semantics: `next()` accepts zero or one argument and returns an object conforming to the `IteratorResult` interface. If a non-object value is returned when used by built-in language features (like `for...of`), a `TypeError` is thrown. All iterator protocol methods (`next()`, `return()`, and `throw()`) should return an `IteratorResult` object.

**Beginner-Friendly Explanation**
An iterator is like a bookmark in a book. Each time you call `next()`, it reads the next page (value) and tells you whether you have reached the end (done). You keep calling `next()` until `done` is `true`.

### Purposes

- To provide a uniform, low-level mechanism for producing values one at a time.
- To enable lazy evaluation, where values are computed only when requested.
- To support both finite and infinite sequences.
- To allow early termination of iteration via the optional `return()` method.

### Syntax Rules and Structure

**Complete General Syntax**
```js
const iterator = {
  next() {
    // Must return { value: any, done: boolean }
    return { value: something, done: false };
  },
  // Optional: called when iteration is terminated early
  return(value) {
    return { value, done: true };
  },
  // Optional: called when an error is injected
  throw(exception) {
    throw exception;
  }
};
```

**Component Breakdown**

| Component | Description |
|---|---|
| `next()` | Required. Returns the next `IteratorResult`. |
| `return(value)` | Optional. Called when the consumer stops iteration early (e.g., `break` in `for...of`). Returns an `IteratorResult`. |
| `throw(exception)` | Optional. Called to inject an error into the iterator. |

**Syntax Rules**

- `next()` must return an object (not a primitive).
- The returned object may have `value` and `done` properties; both are optional.
- If `done` is omitted, it is treated as `false`.
- If `value` is omitted, it is treated as `undefined`.
- When `done` is `true`, `value` optionally specifies the iterator's return value.
- The `return()` and `throw()` methods are optional but are called by language features like `for...of` and `yield*` after an existence check.

**Constraints and Limitations**

- Returning a non-object from `next()` throws a `TypeError` in built-in consumers.
- The iterator protocol does not guarantee that `next()` is idempotent after `done` is `true`; most built-in iterators continue returning `{ value: undefined, done: true }`.
- The `return()` method is not enforced; callers should check for its existence.

### Annotated Code Examples

**Example 1: A simple finite iterator**

```js
// Step 1: Create an iterator factory function.
function createIterator(items) {
  let index = 0;
  // Step 2: Return an object with a next() method.
  return {
    next() {
      if (index < items.length) {
        // Step 3: Return the next value with done: false.
        return { value: items[index++], done: false };
      }
      // Step 4: Signal completion with done: true.
      return { value: undefined, done: true };
    }
  };
}

// Step 5: Use the iterator manually.
const it = createIterator(["a", "b"]);
console.log(it.next()); // { value: "a", done: false }
console.log(it.next()); // { value: "b", done: false }
console.log(it.next()); // { value: undefined, done: true }

// Expected Output:
// { value: 'a', done: false }
// { value: 'b', done: false }
// { value: undefined, done: true }
```

**Breakdown**: The `next()` method returns a new `IteratorResult` each time. When the index exceeds the array length, it returns `{ value: undefined, done: true }`.

**Example 2: An infinite iterator with a `return()` method**

```js
// Step 1: Create an infinite iterator that counts upward.
function createInfiniteCounter() {
  let count = 0;
  return {
    next() {
      return { value: count++, done: false };
    },
    // Step 2: Optional return() for early termination.
    return(value) {
      console.log("Iterator closed early.");
      return { value, done: true };
    }
  };
}

// Step 3: Use it in a for...of loop with a break.
const counter = createInfiniteCounter();
for (const num of counter) {
  if (num >= 3) break;
  console.log(num);
}

// Expected Output:
// 0
// 1
// 2
// Iterator closed early.
```

**Breakdown**: The `break` statement causes `for...of` to call the iterator's `return()` method, which logs the message and returns `{ value: undefined, done: true }`.

### Real-World Cases

- **Pagination**: An iterator that fetches the next page of API results.
- **File reading**: A line-by-line iterator over a large file.
- **Infinite sequences**: Prime generators, Fibonacci sequences, ID generators.
- **Custom data structures**: Tree traversals (in-order, pre-order).

### References

- MDN Web Docs — Iteration protocols (Iterator protocol) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#the_iterator_protocol
- ECMAScript Language Specification — The Iterator Interface — https://tc39.es/ecma262/#sec-iterator-interface
- ECMAScript Language Specification — The IteratorResult Interface — https://tc39.es/ecma262/#sec-iteratorresult-interface

---

## 3. `Symbol.iterator`

### Definitions

**Core Definition**
`Symbol.iterator` is a well-known symbol that specifies the default iterator for an object. It is the key used by the iterable protocol to look up the method that returns an iterator.

**Technical Definition**
The `Symbol.iterator` static data property represents the well-known symbol `@@iterator`. The iterable protocol looks up this symbol for the method that returns the iterator for an object. In order for an object to be iterable, it must have an `[Symbol.iterator]` key. The property has the attributes `Writable: no`, `Enumerable: no`, `Configurable: no`.

**Beginner-Friendly Explanation**
`Symbol.iterator` is a special, unique key that JavaScript uses to find the "how to loop over me" method on an object. If you want your object to work with `for...of`, you add a method with this key.

### Purposes

- To provide a unique, collision-free key for the iteration method.
- To enable any object to opt into the iterable protocol without naming conflicts.
- To serve as the standard interface between iterables and language features like `for...of`.
- To allow built-in types to expose their default iteration behaviour.

### Syntax Rules and Structure

**Complete General Syntax**
```js
// On an object literal
const obj = {
  [Symbol.iterator]() { /* return iterator */ }
};

// On a class
class MyClass {
  *[Symbol.iterator]() { /* yield values */ }
}

// On a prototype
MyClass.prototype[Symbol.iterator] = function* () { /* yield values */ };
```

**Component Breakdown**

| Component | Description |
|---|---|
| `Symbol.iterator` | A well-known symbol, unique and non-writable. |
| `[Symbol.iterator]` | Computed property key. |
| Method | A zero-argument function returning an iterator. |

**Syntax Rules**

- The property key must be the computed `[Symbol.iterator]`.
- The method is called with no arguments when iteration begins.
- It can be an ordinary function or a generator function.
- It is invoked as a method on the iterable, so `this` refers to the iterable.

**Constraints and Limitations**

- The symbol itself is not writable, enumerable, or configurable.
- `Symbol.iterator` is not the same as `"Symbol.iterator"` (a string key); it must be the actual symbol.
- Non-well-formed iterables (returning a non-object) cause `TypeError`.

### Annotated Code Examples

**Example 1: Adding `Symbol.iterator` to an object**

```js
// Step 1: Create a plain object.
const myIterable = {};

// Step 2: Add a [Symbol.iterator] method using a generator.
myIterable[Symbol.iterator] = function* () {
  yield 1;
  yield 2;
  yield 3;
};

// Step 3: Spread into an array.
console.log([...myIterable]);

// Expected Output:
// [1, 2, 3]
```

**Breakdown**: The generator function assigned to `[Symbol.iterator]` is called when `[...myIterable]` runs, producing a generator that yields 1, 2, 3.

**Example 2: Using `Symbol.iterator` inside a class**

```js
// Step 1: Define a class with a computed [Symbol.iterator] method.
class Range {
  constructor(start, end) {
    this.start = start;
    this.end = end;
  }
  *[Symbol.iterator]() {
    for (let i = this.start; i <= this.end; i++) {
      yield i;
    }
  }
}

// Step 2: Create an instance and iterate.
const range = new Range(1, 5);
console.log([...range]);

// Expected Output:
// [1, 2, 3, 4, 5]
```

**Breakdown**: The generator method `*[Symbol.iterator]()` uses `this.start` and `this.end` to yield each number in the range. The spread operator collects them into an array.

### Real-World Cases

- **Custom collections**: Adding iteration to a `LinkedList` or `Tree` class.
- **Math sequences**: A `Fibonacci` class that yields numbers lazily.
- **Data wrappers**: An `APIResponse` object that yields records.
- **Compatibility shims**: Polyfilling `Symbol.iterator` for older environments (pre-ES2015).

### References

- MDN Web Docs — `Symbol.iterator` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/iterator
- ECMAScript Language Specification — Well-Known Symbols — https://tc39.es/ecma262/#sec-well-known-symbols
- TypeScript Documentation — Iterators and Generators — https://www.typescriptlang.org/docs/handbook/iterators-and-generators.html

---

## 4. `next()`

### Definitions

**Core Definition**
`next()` is the method on an iterator object that advances the iteration and returns the next `IteratorResult` object.

**Technical Definition**
The `next()` method accepts zero or one argument and returns an object conforming to the `IteratorResult` interface. The argument, if provided, is passed to the iterator and can be used to send values into a generator (via the `yield` expression's return value). In built-in consumers like `for...of`, `next()` is called with no arguments.

**Beginner-Friendly Explanation**
`next()` is the "give me the next value" button on an iterator. Each press returns an object with the next value and a flag telling you whether the sequence is finished.

### Purposes

- To advance the iterator to the next value in the sequence.
- To signal completion by returning `done: true`.
- To allow bidirectional communication with generators (sending values in).
- To provide a uniform interface for all iterators, regardless of the underlying data structure.

### Syntax Rules and Structure

**Complete General Syntax**
```js
const result = iterator.next();
// result: { value: any, done: boolean }
```

With an argument (for generators):
```js
const result = generator.next(sentValue);
```

**Component Breakdown**

| Component | Description |
|---|---|
| `next()` | The method name; required for all iterators. |
| Argument (optional) | A value passed into the iterator. In generators, this becomes the result of the suspended `yield` expression. |
| Return value | An `IteratorResult` object with `value` and `done` properties. |

**Syntax Rules**

- `next()` must return an object; primitives throw a `TypeError`.
- The `done` property defaults to `false` if omitted.
- The `value` property defaults to `undefined` if omitted.
- For generators, the first `next()` call starts execution; subsequent calls resume from the last `yield`.

**Constraints and Limitations**

- Calling `next()` after `done` is `true` typically continues to return `{ value: undefined, done: true }` for built-in iterators, but this is not guaranteed for custom iterators.
- The argument to `next()` is ignored by most built-in iterators; it is primarily useful with generators.
- `next()` is not reentrant; calling it from within a generator's own execution is not allowed.

### Annotated Code Examples

**Example 1: Manual iteration with `next()`**

```js
// Step 1: Create an array iterator.
const arr = [10, 20, 30];
const iter = arr[Symbol.iterator]();

// Step 2: Call next() repeatedly.
console.log(iter.next()); // { value: 10, done: false }
console.log(iter.next()); // { value: 20, done: false }
console.log(iter.next()); // { value: 30, done: false }
console.log(iter.next()); // { value: undefined, done: true }

// Expected Output:
// { value: 10, done: false }
// { value: 20, done: false }
// { value: 30, done: false }
// { value: undefined, done: true }
```

**Breakdown**: Each call to `next()` returns the next array element with `done: false`. When the array is exhausted, it returns `{ value: undefined, done: true }`.

**Example 2: Sending values into a generator**

```js
// Step 1: Define a generator that receives values.
function* echo() {
  const received = yield "First";
  console.log(`Received: ${received}`);
  yield "Second";
}

// Step 2: Create the generator.
const gen = echo();
console.log(gen.next());        // { value: "First", done: false }
console.log(gen.next("hello")); // Logs "Received: hello", returns { value: "Second", done: false }
console.log(gen.next());        // { value: undefined, done: true }

// Expected Output:
// { value: 'First', done: false }
// Received: hello
// { value: 'Second', done: false }
// { value: undefined, done: true }
```

**Breakdown**: The first `next()` starts the generator and yields `"First"`. The second `next("hello")` resumes the generator, and the argument `"hello"` becomes the result of the `yield "First"` expression, which is stored in `received`. The third `next()` completes the generator.

### Real-World Cases

- **Manual iteration**: Consuming an iterator one value at a time with full control.
- **Generator communication**: Sending values into a generator to influence its behaviour.
- **Infinite sequences**: Calling `next()` a limited number of times on an infinite iterator.
- **Testing**: Verifying iterator state after each `next()` call.

### References

- MDN Web Docs — Iteration protocols (The iterator protocol) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#the_iterator_protocol
- MDN Web Docs — `Generator.prototype.next()` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator/next
- ECMAScript Language Specification — The Iterator Interface — https://tc39.es/ecma262/#sec-iterator-interface

---

## 5. `done`

### Definitions

**Core Definition**
`done` is a Boolean property of the `IteratorResult` object that indicates whether the iterator has completed its sequence.

**Technical Definition**
In the `IteratorResult` interface, `done` is a Boolean. If the iterator has completed its sequence, `done` is `true`; otherwise, it is `false`. If `done` is omitted, it is treated as `false`. When `done` is `true`, the `value` property optionally specifies the iterator's return value.

**Beginner-Friendly Explanation**
`done` is the iterator's way of saying "I'm finished." When you see `done: true`, stop calling `next()`—there are no more values.

### Purposes

- To signal the end of a finite sequence.
- To allow consumers like `for...of` to know when to stop iterating.
- To distinguish between "no value yet" and "no more values."
- To enable infinite iterators that never set `done: true`.

### Syntax Rules and Structure

**Complete General Syntax**
```js
// In an iterator's next() method:
return { value: someValue, done: false };  // more values remain
return { value: finalValue, done: true };  // sequence complete
```

**Component Breakdown**

| Component | Description |
|---|---|
| `done: false` | The iterator has produced a value and may produce more. |
| `done: true` | The iterator has finished its sequence. |
| Omitted `done` | Treated as `false`. |

**Syntax Rules**

- `done` is optional in the returned object; omission is equivalent to `false`.
- When `done` is `true`, `value` is the iterator's return value (often `undefined`).
- Built-in iterators continue returning `{ value: undefined, done: true }` after completion.

**Constraints and Limitations**

- Custom iterators may behave differently after `done` is `true`.
- Setting `done: true` does not automatically clean up resources; use `return()` for that.
- `for...of` and other consumers check `done` after each `next()` call.

### Annotated Code Examples

**Example 1: `done` transitions from false to true**

```js
// Step 1: Create an iterator over two values.
function makeIterator() {
  let step = 0;
  return {
    next() {
      step++;
      if (step === 1) return { value: "a", done: false };
      if (step === 2) return { value: "b", done: false };
      return { value: undefined, done: true };
    }
  };
}

// Step 2: Observe the done flag.
const it = makeIterator();
console.log(it.next()); // { value: "a", done: false }
console.log(it.next()); // { value: "b", done: false }
console.log(it.next()); // { value: undefined, done: true }

// Expected Output:
// { value: 'a', done: false }
// { value: 'b', done: false }
// { value: undefined, done: true }
```

**Breakdown**: The `done` flag remains `false` while values are produced. After the second value, `done` becomes `true`.

**Example 2: `done` omitted is treated as false**

```js
// Step 1: Create an iterator that omits done.
const it = {
  next() {
    return { value: 42 }; // done is omitted → treated as false
  }
};

console.log(it.next()); // { value: 42 }
console.log(it.next()); // { value: 42 }

// Expected Output:
// { value: 42 }
// { value: 42 }
```

**Breakdown**: Because `done` is omitted, it defaults to `false`. The iterator never signals completion. This is useful for infinite iterators.

### Real-World Cases

- **Finite sequences**: Arrays, strings, maps, and sets all set `done: true` when exhausted.
- **Infinite generators**: Never set `done: true`; the consumer must break.
- **Pagination**: Set `done: true` when the last page has been consumed.
- **Resource cleanup**: Combine `done: true` with `return()` for proper teardown.

### References

- MDN Web Docs — Iteration protocols (IteratorResult) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#the_iterator_protocol
- ECMAScript Language Specification — The IteratorResult Interface — https://tc39.es/ecma262/#sec-iteratorresult-interface

---

## 6. `value`

### Definitions

**Core Definition**
`value` is the property of the `IteratorResult` object that holds the current value produced by the iterator.

**Technical Definition**
In the `IteratorResult` interface, `value` is any JavaScript value. It is the result of the current `next()` call. When `done` is `true`, `value` optionally specifies the iterator's return value. If omitted, `value` is treated as `undefined`.

**Beginner-Friendly Explanation**
`value` is the actual data the iterator is handing you at each step. In a `for...of` loop, this is what gets assigned to the loop variable.

### Purposes

- To deliver the current item from the sequence to the consumer.
- To allow iterators to produce values of any type (numbers, strings, objects, etc.).
- To carry the return value when `done` is `true` (e.g., from a generator's `return` statement).
- To enable destructuring and assignment in `for...of` loops.

### Syntax Rules and Structure

**Complete General Syntax**
```js
return { value: anyJavaScriptValue, done: false };
```

**Component Breakdown**

| Component | Description |
|---|---|
| `value` | The current value. Can be any type: primitive, object, `undefined`. |
| `done` | Boolean indicating whether the sequence is complete. |

**Syntax Rules**

- `value` is optional; if omitted, it is `undefined`.
- When `done` is `true`, `value` is the iterator's return value.
- The type of `value` is not constrained; it can change between iterations.
- In `for...of`, `value` is assigned to the loop variable.

**Constraints and Limitations**

- `value` is not used by consumers when `done` is `true` (except for return values in generators).
- If `next()` returns a non-object, `value` cannot be accessed and a `TypeError` is thrown.

### Annotated Code Examples

**Example 1: `value` carries different types**

```js
// Step 1: Create an iterator with mixed value types.
const it = {
  values: [42, "hello", { key: "value" }, null],
  index: 0,
  next() {
    if (this.index < this.values.length) {
      return { value: this.values[this.index++], done: false };
    }
    return { value: undefined, done: true };
  }
};

// Step 2: Observe the value property.
console.log(it.next()); // { value: 42, done: false }
console.log(it.next()); // { value: "hello", done: false }
console.log(it.next()); // { value: { key: "value" }, done: false }
console.log(it.next()); // { value: null, done: false }
console.log(it.next()); // { value: undefined, done: true }

// Expected Output:
// { value: 42, done: false }
// { value: 'hello', done: false }
// { value: { key: 'value' }, done: false }
// { value: null, done: false }
// { value: undefined, done: true }
```

**Breakdown**: Each `next()` call returns the next value in the array, regardless of type. `null` is a valid value (not the same as `undefined`).

**Example 2: `value` in a `for...of` loop**

```js
// Step 1: Define an iterable.
const numbers = {
  *[Symbol.iterator]() {
    yield 10;
    yield 20;
    yield 30;
  }
};

// Step 2: The loop variable receives the value.
for (const num of numbers) {
  console.log(num);
}

// Expected Output:
// 10
// 20
// 30
```

**Breakdown**: The `for...of` loop internally calls `next()` and assigns `result.value` to `num` on each iteration.

### Real-World Cases

- **Data streams**: Each `value` is a chunk of data (e.g., a line from a file).
- **Pagination**: Each `value` is a record or page of results.
- **Generators**: `value` is the result of each `yield` expression.
- **Custom collections**: `value` is each element in the collection.

### References

- MDN Web Docs — Iteration protocols (IteratorResult) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#the_iterator_protocol
- ECMAScript Language Specification — The IteratorResult Interface — https://tc39.es/ecma262/#sec-iteratorresult-interface

---

## 7. Built-in Iterables

### Definitions

**Core Definition**
Built-in iterables are JavaScript types that come with a default `[Symbol.iterator]` implementation, making them natively compatible with `for...of`, spread, and destructuring.

**Technical Definition**
The built-in iterable types include `Array`, `TypedArray`, `String`, `Map`, and `Set`, as well as `arguments`, `NodeList`, and `Intl.Segmenter.Segments`. Each of these has a `[Symbol.iterator]()` method on its prototype, implementing the iterable protocol. `Array.prototype[Symbol.iterator]()` yields each element; `String.prototype[Symbol.iterator]()` yields Unicode code points; `Map.prototype[Symbol.iterator]()` yields `[key, value]` pairs; `Set.prototype[Symbol.iterator]()` yields values in insertion order.

**Beginner-Friendly Explanation**
Some JavaScript types are "born iterable"—arrays, strings, Maps, Sets, and a few others. You can loop over them directly with `for...of` without any extra setup. Plain objects are not iterable by default.

### Purposes

- To provide default iteration behaviour for the most common data structures.
- To enable uniform traversal syntax across arrays, strings, maps, and sets.
- To allow built-in types to be consumed by spread, destructuring, and `Array.from`.
- To serve as reference implementations for custom iterables.

### Syntax Rules and Structure

**General Syntax (consuming a built-in iterable)**
```js
// for...of
for (const item of iterable) { … }

// Spread
const array = [...iterable];

// Destructuring
const [first, second] = iterable;

// Array.from
const array2 = Array.from(iterable);
```

**Component Breakdown**

| Type | `[Symbol.iterator]()` yields | Order |
|---|---|---|
| `Array` | Each element | Index order |
| `String` | Each Unicode code point | Code point order |
| `Map` | `[key, value]` pairs | Insertion order |
| `Set` | Each value | Insertion order |
| `TypedArray` | Each numeric element | Index order |
| `arguments` | Each argument | Argument order |

**Syntax Rules**

- Built-in iterables work with all consuming syntaxes (`for...of`, spread, destructuring, `Array.from`, `Promise.all`).
- The iterator is obtained by calling `[Symbol.iterator]()` with no arguments.
- Built-in iterators also implement `[Symbol.iterator]()` returning `this`, so they are both iterable and iterator.

**Constraints and Limitations**

- Plain objects (`{}`) are not iterable by default.
- `Map` and `Set` iteration order is insertion order, not sorted order.
- String iteration by code point splits grapheme clusters (e.g., emoji with skin tone modifiers may be split).

### Annotated Code Examples

**Example 1: Array and string iteration**

```js
// Step 1: Array iteration.
const arr = [1, 2, 3];
for (const num of arr) {
  console.log(num);
}

// Step 2: String iteration (by Unicode code point).
const str = "a🌟b";
for (const char of str) {
  console.log(char);
}

// Expected Output:
// 1
// 2
// 3
// a
// 🌟
// b
```

**Breakdown**: Arrays yield elements in index order. Strings yield Unicode code points as individual strings, correctly handling the surrogate pair for the emoji.

**Example 2: Map and Set iteration**

```js
// Step 1: Map iteration yields [key, value] pairs.
const map = new Map([["a", 1], ["b", 2]]);
for (const [key, value] of map) {
  console.log(`${key} -> ${value}`);
}

// Step 2: Set iteration yields values.
const set = new Set([10, 20, 30]);
for (const value of set) {
  console.log(value);
}

// Expected Output:
// a -> 1
// b -> 2
// 10
// 20
// 30
```

**Breakdown**: The Map iterator yields `[key, value]` arrays that are destructured in the loop header. The Set iterator yields each value in insertion order.

### Real-World Cases

- **Array processing**: Summing, filtering, mapping with `for...of`.
- **String analysis**: Character-by-character processing, Unicode handling.
- **Map/Set traversal**: Configuration maps, deduplicated collections.
- **DOM iteration**: `NodeList` from `querySelectorAll` with `for...of`.

### References

- MDN Web Docs — Iteration protocols (Built-in iterables) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols
- MDN Web Docs — `Array.prototype[Symbol.iterator]()` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Symbol.iterator
- MDN Web Docs — `String.prototype[Symbol.iterator]()` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/Symbol.iterator
- MDN Web Docs — `Map.prototype[Symbol.iterator]()` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/Symbol.iterator
- MDN Web Docs — `Set.prototype[Symbol.iterator]()` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/Symbol.iterator

---

## 8. Custom Iterables

### Definitions

**Core Definition**
A custom iterable is any user-defined object that implements the iterable protocol by providing a `[Symbol.iterator]()` method, enabling it to be used with `for...of`, spread, and other iteration constructs.

**Technical Definition**
To create a custom iterable, an object must have a `[Symbol.iterator]()` method (own or inherited) that returns an object conforming to the iterator protocol. This can be implemented with a plain object returning a `next()` method, or more concisely with a generator function. The method is called with no arguments and `this` refers to the iterable object.

**Beginner-Friendly Explanation**
You can make your own objects loopable by adding a special method that tells JavaScript how to iterate over them. This lets you use `for...of` on your custom data structures, just like arrays.

### Purposes

- To enable `for...of` and spread on custom data structures (linked lists, trees, graphs).
- To encapsulate iteration logic within the object itself.
- To support lazy evaluation for expensive or infinite sequences.
- To provide a standard interface for consumers without exposing internal data.

### Syntax Rules and Structure

**Complete General Syntax (manual iterator)**
```js
const myIterable = {
  data: [...],
  [Symbol.iterator]() {
    let index = 0;
    const data = this.data;
    return {
      next() {
        if (index < data.length) {
          return { value: data[index++], done: false };
        }
        return { value: undefined, done: true };
      }
    };
  }
};
```

**Complete General Syntax (generator-based)**
```js
const myIterable = {
  *[Symbol.iterator]() {
    yield value1;
    yield value2;
  }
};
```

**Component Breakdown**

| Component | Description |
|---|---|
| `[Symbol.iterator]()` | The iterable protocol method. |
| `data` | The internal data to iterate over. |
| `index` | A closure variable tracking the current position. |
| `next()` | Returns the next `{ value, done }` pair. |
| `yield` (generator version) | Produces values lazily. |

**Syntax Rules**

- The `[Symbol.iterator]` method must be a computed property.
- It must return an object with a `next()` method.
- The `next()` method must return an object (not a primitive).
- The method is invoked as a method on the iterable, so `this` is the iterable.

**Constraints and Limitations**

- Plain objects are not iterable by default; you must add `[Symbol.iterator]`.
- The iterator must maintain state (usually via closure variables).
- A new iterator should be returned each time `[Symbol.iterator]()` is called (so that multiple loops do not interfere).

### Annotated Code Examples

**Example 1: A custom iterable with a generator**

```js
// Step 1: Define a custom iterable using a generator.
const fibonacci = {
  limit: 10,
  *[Symbol.iterator]() {
    let a = 0, b = 1;
    for (let i = 0; i < this.limit; i++) {
      yield a;
      [a, b] = [b, a + b];
    }
  }
};

// Step 2: Use it in a for...of loop.
for (const num of fibonacci) {
  console.log(num);
}

// Expected Output:
// 0
// 1
// 1
// 2
// 3
// 5
// 8
// 13
// 21
// 34
```

**Breakdown**: The generator method `*[Symbol.iterator]()` uses `this.limit` to control the number of iterations. Each `yield` produces the next Fibonacci number.

**Example 2: A custom iterable with a manual iterator**

```js
// Step 1: Define a class that is iterable.
class Countdown {
  constructor(start) {
    this.start = start;
  }
  [Symbol.iterator]() {
    let current = this.start;
    return {
      next() {
        if (current >= 0) {
          return { value: current--, done: false };
        }
        return { value: undefined, done: true };
      }
    };
  }
}

// Step 2: Create and iterate.
const countdown = new Countdown(3);
console.log([...countdown]);

// Expected Output:
// [3, 2, 1, 0]
```

**Breakdown**: The `[Symbol.iterator]()` method returns a new iterator each time it is called, with its own `current` state. The spread operator collects all values.

### Real-World Cases

- **Linked lists**: Iterate node by node.
- **Trees**: In-order, pre-order, or post-order traversal.
- **Matrices**: Iterate rows or cells.
- **API pagination**: Yield pages of results lazily.
- **Infinite sequences**: Prime numbers, random walks, ID generators.

### References

- MDN Web Docs — `Symbol.iterator` (User-defined iterables) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/iterator
- MDN Web Docs — Iteration protocols — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols
- CodeSignal — Building A Custom Iterable Class — https://codesignal.com/learn/courses/iterables-and-generators/

---

## 9. Iterating Objects

### Definitions

**Core Definition**
Iterating an object means traversing its properties or key-value pairs. Plain objects are not iterable by default, but they can be made iterable or iterated using helper methods.

**Technical Definition**
Plain objects (`{}`) do not have a `[Symbol.iterator]` method. To iterate over an object's properties, you must either (a) use `Object.keys()`, `Object.values()`, or `Object.entries()` to convert to an array (which is iterable), (b) use a `for...in` loop (which enumerates enumerable string-keyed properties including inherited ones), or (c) add a custom `[Symbol.iterator]` method to the object. `Object.entries()` is particularly useful with `for...of` because it allows elegant iteration over key-value pairs.

**Beginner-Friendly Explanation**
You cannot use `for...of` directly on a plain object like `{ a: 1, b: 2 }`. You first need to convert it to something iterable, such as an array of `[key, value]` pairs, using `Object.entries()`.

### Purposes

- To traverse an object's properties for processing or inspection.
- To convert object data into an iterable format for use with `for...of`.
- To support custom iteration behaviour for domain-specific objects.
- To avoid the pitfalls of `for...in` (inherited properties, string keys).

### Syntax Rules and Structure

**General Syntax (using `Object.entries`)**
```js
for (const [key, value] of Object.entries(object)) {
  // process key and value
}
```

**General Syntax (custom `[Symbol.iterator]`)**
```js
object[Symbol.iterator] = function* () {
  for (const key of Object.keys(this)) {
    yield [key, this[key]];
  }
};
```

**Component Breakdown**

| Approach | Description |
|---|---|
| `Object.keys(obj)` | Returns an array of own enumerable string keys. |
| `Object.values(obj)` | Returns an array of own enumerable values. |
| `Object.entries(obj)` | Returns an array of `[key, value]` pairs. |
| `for...in` | Iterates enumerable string keys including inherited. |
| Custom `[Symbol.iterator]` | Makes the object directly iterable. |

**Syntax Rules**

- `Object.entries()` returns only own enumerable string-keyed properties.
- `for...in` includes inherited enumerable properties; use `Object.hasOwn()` to filter.
- Custom `[Symbol.iterator]` makes the object work with `for...of`, spread, and destructuring.

**Constraints and Limitations**

- Plain objects are not iterable by default; `for...of` on a plain object throws a `TypeError`.
- `Object.entries()` creates an intermediate array, which may be memory-intensive for large objects.
- `for...in` is the slowest iteration method and should be avoided for performance-critical code.

### Annotated Code Examples

**Example 1: Iterating an object with `Object.entries()`**

```js
// Step 1: Define a plain object.
const person = { name: "Alice", age: 30, city: "Paris" };

// Step 2: Convert to entries and iterate with for...of.
for (const [key, value] of Object.entries(person)) {
  console.log(`${key}: ${value}`);
}

// Expected Output:
// name: Alice
// age: 30
// city: Paris
```

**Breakdown**: `Object.entries(person)` returns `[["name", "Alice"], ["age", 30], ["city", "Paris"]]`. The `for...of` loop destructures each pair into `key` and `value`.

**Example 2: Making a plain object iterable**

```js
// Step 1: Define a plain object.
const config = { host: "localhost", port: 8080 };

// Step 2: Add a custom [Symbol.iterator] method.
config[Symbol.iterator] = function* () {
  for (const key of Object.keys(this)) {
    yield [key, this[key]];
  }
};

// Step 3: Iterate with for...of.
for (const [key, value] of config) {
  console.log(`${key} = ${value}`);
}

// Expected Output:
// host = localhost
// port = 8080
```

**Breakdown**: The generator method iterates over the object's own keys and yields `[key, value]` pairs. The `for...of` loop destructures them.

### Real-World Cases

- **Configuration objects**: Iterating over settings.
- **Data serialisation**: Converting objects to query strings or JSON.
- **Debugging**: Logging all properties of an object.
- **Form data**: Iterating over field names and values.

### References

- MDN Web Docs — `Object.entries()` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/entries
- MDN Web Docs — `Object.keys()` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys
- MDN Web Docs — `Object.values()` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/values
- You Don't Know JS — Iterators — https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/es6 & beyond/ch2.md

---

## 10. Iterating Maps

### Definitions

**Core Definition**
Iterating a Map means traversing its key-value pairs in insertion order using the Map's built-in `[Symbol.iterator]()` method.

**Technical Definition**
`Map.prototype[Symbol.iterator]()` is the same function as `Map.prototype.entries()`. It returns a Map iterator that yields `[key, value]` arrays in insertion order. The `keys()` and `values()` methods return iterators for just the keys or just the values, respectively.

**Beginner-Friendly Explanation**
A Map is a collection of key-value pairs, like a dictionary. You can loop over it directly with `for...of`, and each iteration gives you a `[key, value]` pair that you can destructure.

### Purposes

- To traverse a Map's entries in insertion order.
- To process keys and values together in a single loop.
- To convert a Map to an array of entries via spread.
- To filter or transform Map data using iteration.

### Syntax Rules and Structure

**Complete General Syntax**
```js
// Iterate over entries (default)
for (const [key, value] of map) { … }

// Iterate over keys
for (const key of map.keys()) { … }

// Iterate over values
for (const value of map.values()) { … }

// Iterate over entries explicitly
for (const [key, value] of map.entries()) { … }
```

**Component Breakdown**

| Method | Yields | Description |
|---|---|---|
| `map[Symbol.iterator]()` | `[key, value]` | Default iterator (same as `entries()`). |
| `map.keys()` | `key` | Yields keys in insertion order. |
| `map.values()` | `value` | Yields values in insertion order. |
| `map.entries()` | `[key, value]` | Yields key-value pairs. |

**Syntax Rules**

- Map iteration order is insertion order (keys are not sorted).
- The default iterator yields `[key, value]` arrays.
- Destructuring can be used in the `for...of` loop header.
- `keys()`, `values()`, and `entries()` return new iterator objects each time.

**Constraints and Limitations**

- Map keys can be any type (strings, numbers, objects, etc.).
- Modifying the Map during iteration is supported but can lead to unexpected results if not careful.
- The default iterator is the same as `entries()`, not `values()`.

### Annotated Code Examples

**Example 1: Iterating Map entries with destructuring**

```js
// Step 1: Create a Map with key-value pairs.
const horses = new Map([
  [5, "QuickBucks"],
  [8, "Chocolate"],
  [3, "Filippone"]
]);

// Step 2: Iterate entries with destructuring.
for (const [key, value] of horses) {
  console.log(`${key} -> ${value}`);
}

// Expected Output:
// 5 -> QuickBucks
// 8 -> Chocolate
// 3 -> Filippone
```

**Breakdown**: The Map iterator yields `[key, value]` arrays. Destructuring assigns them to `key` and `value`. The order is insertion order.

**Example 2: Iterating keys and values separately**

```js
// Step 1: Create a Map.
const scores = new Map([["Alice", 95], ["Bob", 87], ["Charlie", 92]]);

// Step 2: Iterate keys.
console.log("Keys:");
for (const name of scores.keys()) {
  console.log(name);
}

// Step 3: Iterate values.
console.log("Values:");
for (const score of scores.values()) {
  console.log(score);
}

// Expected Output:
// Keys:
// Alice
// Bob
// Charlie
// Values:
// 95
// 87
// 92
```

**Breakdown**: `keys()` yields just the keys, and `values()` yields just the values. Both are iterable and work with `for...of`.

### Real-World Cases

- **Caching**: Iterating over a cache's key-value pairs.
- **Configuration**: Processing a Map of settings.
- **Graph adjacency**: A Map of node IDs to neighbour lists.
- **Frequency counting**: A Map of words to counts.

### References

- MDN Web Docs — `Map.prototype[Symbol.iterator]()` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/Symbol.iterator
- MDN Web Docs — `Map.prototype.entries()` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/entries
- Educative — Iterables with Sets and Maps — https://github.com/schuebel/educative.io_courses

---

## 11. Iterating Sets

### Definitions

**Core Definition**
Iterating a Set means traversing its unique values in insertion order using the Set's built-in `[Symbol.iterator]()` method.

**Technical Definition**
`Set.prototype[Symbol.iterator]()` is the same function as `Set.prototype.values()`. It returns a Set iterator that yields each value in insertion order. Because Sets store unique values, each value appears exactly once. The `entries()` method yields `[value, value]` pairs (for compatibility with Map iteration).

**Beginner-Friendly Explanation**
A Set is a collection of unique values. You can loop over it directly with `for...of`, and each iteration gives you one value. Duplicate values are automatically removed.

### Purposes

- To traverse the unique values of a Set in insertion order.
- To process each element of a Set without worrying about duplicates.
- To convert a Set to an array via spread or `Array.from`.
- To perform set operations (union, intersection, difference) using iteration.

### Syntax Rules and Structure

**Complete General Syntax**
```js
// Iterate values (default)
for (const value of set) { … }

// Iterate values explicitly
for (const value of set.values()) { … }

// Iterate entries (value, value pairs)
for (const [value, value2] of set.entries()) { … }
```

**Component Breakdown**

| Method | Yields | Description |
|---|---|---|
| `set[Symbol.iterator]()` | `value` | Default iterator (same as `values()`). |
| `set.values()` | `value` | Yields values in insertion order. |
| `set.keys()` | `value` | Alias for `values()` (for Map compatibility). |
| `set.entries()` | `[value, value]` | Yields `[value, value]` pairs. |

**Syntax Rules**

- Set iteration order is insertion order.
- The default iterator yields values, not `[value, value]` pairs.
- `keys()` and `values()` are identical for Sets.
- Duplicate values are not stored; each value appears once.

**Constraints and Limitations**

- Set values are compared using the SameValueZero algorithm (similar to `===` but `NaN` equals `NaN`).
- Modifying a Set during iteration is supported but can lead to unexpected results.
- Object references in a Set are compared by reference, not by value.

### Annotated Code Examples

**Example 1: Basic Set iteration**

```js
// Step 1: Create a Set with duplicate values.
const numbers = new Set([1, 2, 2, 3, 3, 3]);

// Step 2: Iterate over the unique values.
for (const num of numbers) {
  console.log(num);
}

// Expected Output:
// 1
// 2
// 3
```

**Breakdown**: The Set removes duplicates, so only 1, 2, and 3 are iterated. The order is insertion order.

**Example 2: Using `entries()` for Map-like iteration**

```js
// Step 1: Create a Set.
const colors = new Set(["red", "green", "blue"]);

// Step 2: Iterate entries (yields [value, value]).
for (const [value, value2] of colors.entries()) {
  console.log(`${value} = ${value2}`);
}

// Expected Output:
// red = red
// green = green
// blue = blue
```

**Breakdown**: `entries()` yields `[value, value]` pairs. This is designed for compatibility with Map iteration, where the same pattern yields `[key, value]`.

### Real-World Cases

- **Deduplication**: Iterating over unique values from a list.
- **Tag processing**: Iterating over a Set of tags.
- **Graph traversal**: Iterating over visited nodes.
- **Permission checks**: Iterating over a Set of permissions.

### References

- MDN Web Docs — `Set.prototype[Symbol.iterator]()` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/Symbol.iterator
- MDN Web Docs — `Set.prototype.values()` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/values
- Chrome for Developers — Collecting and Iterating (ES6 Way) — https://developer.chrome.com/blog/collecting-and-iterating-the-es6-way

---

## 12. Iterating Strings

### Definitions

**Core Definition**
Iterating a string means traversing its Unicode code points as individual strings using the String's built-in `[Symbol.iterator]()` method.

**Technical Definition**
`String.prototype[Symbol.iterator]()` returns a string iterator that yields the Unicode code points of the string value as individual strings. This correctly handles surrogate pairs (characters outside the Basic Multilingual Plane) but splits grapheme clusters (e.g., emoji with skin tone modifiers).

**Beginner-Friendly Explanation**
You can loop over a string with `for...of`, and each iteration gives you one character. Unlike the old `charAt()` method, `for...of` correctly handles characters like emoji that are made of two code units.

### Purposes

- To process a string character by character, correctly handling Unicode.
- To count code points (not UTF-16 code units).
- To iterate over grapheme clusters (with additional segmentation).
- To convert a string to an array of characters via spread.

### Syntax Rules and Structure

**Complete General Syntax**
```js
for (const char of string) { … }
```

**Component Breakdown**

| Component | Description |
|---|---|
| `string` | Any string primitive or String object. |
| `char` | Each Unicode code point as a single-character string. |

**Syntax Rules**

- The iterator yields code points, not UTF-16 code units.
- Surrogate pairs are preserved (e.g., `𝌆` is yielded as one character).
- Grapheme clusters (e.g., emoji with modifiers) may be split into multiple code points.
- Works with spread (`[...str]`) and `Array.from(str)`.

**Constraints and Limitations**

- Splits grapheme clusters: `"👋🏽"` (waving hand with skin tone) is split into `"👋"` and `"🏽"`.
- For grapheme-aware iteration, use `Intl.Segmenter`.
- Not all characters are single code points; some are sequences.

### Annotated Code Examples

**Example 1: Basic string iteration**

```js
// Step 1: Define a string with a multi-byte character.
const str = "a🌟b";

// Step 2: Iterate by code point.
for (const char of str) {
  console.log(char);
}

// Expected Output:
// a
// 🌟
// b
```

**Breakdown**: The `for...of` loop correctly handles the emoji `🌟` (which is a surrogate pair) as a single character.

**Example 2: Grapheme cluster splitting**

```js
// Step 1: Define a string with a grapheme cluster.
const emoji = "👋🏽"; // waving hand + skin tone modifier

// Step 2: Iterate by code point.
console.log([...emoji]);

// Expected Output:
// ['👋', '🏽']
```

**Breakdown**: The emoji is split into two code points: the waving hand and the skin tone modifier. This is expected behaviour for code-point iteration.

### Real-World Cases

- **Text processing**: Character-by-character analysis.
- **Unicode validation**: Checking for valid code points.
- **String reversal**: Reversing a string by code points.
- **Emoji handling**: Splitting emoji sequences (with additional segmentation for graphemes).

### References

- MDN Web Docs — `String.prototype[Symbol.iterator]()` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/Symbol.iterator
- MDN Web Docs — `Intl.Segmenter` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/Segmenter
- ES6 入门教程 — 字符串的遍历器接口 — https://es6.ruanyifeng.com/

---

## 13. Iterating Generators

### Definitions

**Core Definition**
Iterating a generator means consuming the values produced by a generator object, which is both an iterator and an iterable.

**Technical Definition**
A generator function (`function*`) returns a Generator object that conforms to both the iterator protocol (it has a `next()` method) and the iterable protocol (its `[Symbol.iterator]()` method returns `this`). The `yield` operator pauses the generator and returns a value to the caller; the next `next()` call resumes execution.

**Beginner-Friendly Explanation**
A generator is a function that can pause and resume. It produces values one at a time using `yield`. You can loop over a generator with `for...of`, and it will produce values lazily as you request them.

### Purposes

- To produce values lazily, on demand, without pre-computing them.
- To define infinite sequences that are consumed only as needed.
- To implement custom iteration logic concisely.
- To coordinate asynchronous operations (with async generators).

### Syntax Rules and Structure

**Complete General Syntax**
```js
function* generatorName() {
  yield value1;
  yield value2;
  // ...
}

const gen = generatorName();
for (const value of gen) { … }
```

**Component Breakdown**

| Component | Description |
|---|---|
| `function*` | Declares a generator function. |
| `yield` | Pauses execution and returns a value. |
| `gen.next()` | Resumes execution until the next `yield` or completion. |
| `gen[Symbol.iterator]()` | Returns `this`, making the generator iterable. |

**Syntax Rules**

- `yield` can only be used directly inside a generator function.
- The generator's `next()` returns `{ value, done }`.
- The first `next()` call starts execution; subsequent calls resume from the last `yield`.
- A `return` statement in a generator sets `done: true` and provides the `value`.
- Generators are both iterators and iterables (they have `[Symbol.iterator]()` returning `this`).

**Constraints and Limitations**

- Generators are not reusable; once exhausted, they cannot be reset.
- `yield` cannot be used in nested functions.
- Performance: generators are generally slower than plain loops due to the overhead of the iterator protocol.

### Annotated Code Examples

**Example 1: Basic generator iteration**

```js
// Step 1: Define a generator function.
function* numberGenerator() {
  yield 1;
  yield 2;
  yield 3;
}

// Step 2: Create the generator and iterate.
const gen = numberGenerator();
for (const num of gen) {
  console.log(num);
}

// Expected Output:
// 1
// 2
// 3
```

**Breakdown**: The generator yields 1, 2, and 3. The `for...of` loop calls `next()` until `done` is `true`.

**Example 2: Infinite generator (lazy evaluation)**

```js
// Step 1: Define an infinite generator.
function* infiniteCounter() {
  let i = 0;
  while (true) {
    yield i++;
  }
}

// Step 2: Consume only the first 5 values.
const counter = infiniteCounter();
for (const num of counter) {
  if (num >= 5) break;
  console.log(num);
}

// Expected Output:
// 0
// 1
// 2
// 3
// 4
```

**Breakdown**: The generator never sets `done: true`; it would run forever if not for the `break`. This demonstrates lazy evaluation—values are produced only when requested.

### Real-World Cases

- **Pagination**: Yielding pages of API results lazily.
- **File reading**: Yielding lines from a large file.
- **ID generation**: Yielding unique IDs indefinitely.
- **Async workflows**: Async generators for streaming data.

### References

- MDN Web Docs — `function*` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*
- MDN Web Docs — `yield` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/yield
- MDN Web Docs — Generator — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator
- MDN Web Docs — Iterators and generators — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_generators

---

## 14. Async Iteration Protocols

### Definitions

**Core Definition**
Async iteration protocols extend the synchronous iteration protocols to handle asynchronous data sources, using `Symbol.asyncIterator` and a `next()` method that returns a `Promise` for an `IteratorResult`.

**Technical Definition**
The async iterable protocol uses `%Symbol.asyncIterator%` whose method returns an async iterator. The async iterator protocol requires a `next()` method that returns a `Promise` resolving to an `IteratorResult`. Async generators (`async function*`) implement both. The `for await...of` statement consumes async iterables.

**Beginner-Friendly Explanation**
Async iteration is like regular iteration, but each value arrives asynchronously (e.g., from a network request or file read). You use `for await...of` instead of `for...of`, and each iteration waits for the value to be ready.

### Purposes

- To iterate over data that arrives asynchronously (streams, API pages, file reads).
- To consume async generators and async iterables in a natural, loop-based syntax.
- To coordinate multiple asynchronous data sources sequentially.
- To bridge synchronous and asynchronous iteration (sync iterables are automatically wrapped for `for await...of`).

### Syntax Rules and Structure

**Complete General Syntax**
```js
async function* asyncGenerator() {
  yield await someAsyncOperation();
}

// Consuming with for await...of
for await (const value of asyncIterable) {
  // process value
}
```

**Component Breakdown**

| Component | Description |
|---|---|
| `Symbol.asyncIterator` | The well-known symbol for async iteration. |
| `async next()` | Returns a `Promise` for an `IteratorResult`. |
| `for await...of` | Consumes async iterables; waits for each promise. |
| `async function*` | Declares an async generator. |

**Syntax Rules**

- The async iterator's `next()` must return a `Promise`.
- `for await...of` works on both async iterables and sync iterables (sync values are wrapped in resolved promises).
- Async generators cannot be consumed by `for...of` (only `for await...of`).
- Async iterators do not implement `Symbol.iterator` by default, so they cannot be used with spread or regular `for...of`.

**Constraints and Limitations**

- **Not interchangeable**: Async iterables cannot be used with `for...of`, spread, or destructuring.
- **Performance**: Async iteration has overhead due to promise creation.
- **Error handling**: Rejected promises propagate as exceptions in `for await...of`.
- **Browser support**: Baseline since January 2020.

### Annotated Code Examples

**Example 1: Async generator**

```js
// Step 1: Define an async generator.
async function* asyncNumbers() {
  yield 1;
  yield 2;
  yield 3;
}

// Step 2: Consume with for await...of.
(async () => {
  for await (const num of asyncNumbers()) {
    console.log(num);
  }
})();

// Expected Output:
// 1
// 2
// 3
```

**Breakdown**: The async generator yields values that are wrapped in promises. `for await...of` awaits each promise before assigning the resolved value to `num`.

**Example 2: Custom async iterable**

```js
// Step 1: Create an async iterable object.
const asyncIterable = {
  [Symbol.asyncIterator]() {
    let i = 0;
    return {
      async next() {
        if (i < 3) {
          return { value: i++, done: false };
        }
        return { value: undefined, done: true };
      }
    };
  }
};

// Step 2: Consume with for await...of.
(async () => {
  for await (const value of asyncIterable) {
    console.log(value);
  }
})();

// Expected Output:
// 0
// 1
// 2
```

**Breakdown**: The `[Symbol.asyncIterator]()` method returns an object with an `async next()` method. Each `next()` returns a promise resolving to `{ value, done }`.

### Real-World Cases

- **API pagination**: Fetching pages of results asynchronously.
- **File streaming**: Reading chunks from a file or network stream.
- **Database cursors**: Iterating over query results in batches.
- **Web streams**: `ReadableStream` async iteration.

### References

- MDN Web Docs — `Symbol.asyncIterator` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/asyncIterator
- MDN Web Docs — `for await...of` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for-await...of
- MDN Web Docs — AsyncIterator — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/AsyncIterator
- JavaScript.info — Async iteration and generators — https://javascript.info/async-iterators-generators

---

## Summary Table of References

| Resource | URL |
|---|---|
| MDN — Iteration protocols | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols |
| MDN — `Symbol.iterator` | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/iterator |
| MDN — `Symbol.asyncIterator` | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/asyncIterator |
| MDN — `for...of` | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of |
| MDN — `for await...of` | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for-await...of |
| MDN — `function*` | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function* |
| MDN — `yield` | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/yield |
| MDN — Generator | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator |
| MDN — AsyncIterator | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/AsyncIterator |
| MDN — `Array.prototype[Symbol.iterator]()` | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Symbol.iterator |
| MDN — `String.prototype[Symbol.iterator]()` | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/Symbol.iterator |
| MDN — `Map.prototype[Symbol.iterator]()` | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/Symbol.iterator |
| MDN — `Set.prototype[Symbol.iterator]()` | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/Symbol.iterator |
| ECMAScript — Common Iteration Interfaces | https://tc39.es/ecma262/#sec-common-iteration-interfaces |
| ECMAScript — The Iterable Interface | https://tc39.es/ecma262/#sec-iterable-interface |
| ECMAScript — The Iterator Interface | https://tc39.es/ecma262/#sec-iterator-interface |
| ECMAScript — The IteratorResult Interface | https://tc39.es/ecma262/#sec-iteratorresult-interface |
| TypeScript — Iterators and Generators | https://www.typescriptlang.org/docs/handbook/iterators-and-generators.html |
| WebKit Bug 283118 — `for-of` Performance | https://auto-bugs.webkit.org/show_bug.cgi?id=283118 |
| Chrome for Developers — Collecting and Iterating | https://developer.chrome.com/blog/collecting-and-iterating-the-es6-way |