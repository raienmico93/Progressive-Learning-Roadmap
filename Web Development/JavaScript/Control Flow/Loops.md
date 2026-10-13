# JavaScript Loops: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
JavaScript loops are control-flow constructs that repeatedly execute a block of code as long as a specified condition remains true, or until an iteration sequence is exhausted.

**Technical Definition**
Per the ECMAScript Language Specification, iteration statements (`for`, `for...in`, `for...of`, `while`, `do...while`) are productions within `IterationStatement`. Each evaluates an expression, performs `ToBoolean` or iterator-protocol operations, and repeatedly executes a `Statement` according to defined completion semantics. The `for` statement consists of three optional expressions (initialization, condition, afterthought) separated by semicolons. The `for...of` statement invokes the `[Symbol.iterator]()` method of an iterable object and repeatedly calls `next()` until the iterator is done. The `for...in` statement enumerates all enumerable string-keyed properties, including inherited ones.

**Beginner-Friendly Explanation**
Loops are how you tell a computer to do something over and over without writing the same code repeatedly. Instead of writing `console.log(1); console.log(2); … console.log(100);`, you write a loop that counts from 1 to 100. Loops are the backbone of almost every program—they process lists, repeat actions, search for items, and much more.

### Key Characteristics

- **Repetition**: Loops execute a block multiple times until a termination condition is met.
- **Four main variants**: `for` (C-style counter), `for...in` (object keys), `for...of` (iterable values), `while`/`do...while` (condition-driven).
- **Control transfer**: `break` exits early; `continue` skips to the next iteration; labels extend these to outer loops.
- **Coercion in conditions**: `while` and `for` conditions use `ToBoolean`; `for...of` uses the iterator protocol.
- **Performance-sensitive**: Loop choice, length caching, and operation placement measurably affect execution time.
- **Nestable**: Loops can contain other loops for multi-dimensional data processing.

### Prerequisites

- Variables and data types (`number`, `string`, `boolean`, `object`, `array`).
- Comparison operators (`<`, `>`, `<=`, `>=`, `===`).
- Basic understanding of truthy and falsy values.
- Familiarity with code blocks `{ }` and statement execution order.
- For `for...of`: basic understanding of iterables and the iterator protocol.
- For `for...in`: understanding of object properties and enumerability.

### Related Programming Areas

- Conditional statements (`if`, `switch`) for branch logic inside loops.
- Array methods (`map`, `filter`, `reduce`) as declarative alternatives.
- Asynchronous iteration (`for await...of`, async generators).
- Algorithm complexity analysis (O(n), O(n²) for nested loops).
- DOM traversal and event handling.
- Data processing pipelines and streaming.

### Core Concepts / Features

| Concept | Category |
|---|---|
| `for` | Iteration statement |
| `for...in` | Object enumeration |
| `for...of` | Iterable iteration |
| `while` | Condition-driven loop |
| `do...while` | Post-test loop |
| `break` | Control transfer |
| `continue` | Control transfer |
| Labels | Control transfer |
| Infinite loops | Anti-pattern / intentional |
| Loop performance | Optimization |
| Nested loops | Composition |
| Loop optimization | Optimization |

---

## Core Concepts / Features

---

## 1. `for` Statement

### Definitions

**Core Definition**
The `for` statement creates a loop with three optional expressions—initialization, condition, and afterthought—separated by semicolons, followed by a statement to execute repeatedly.

**Technical Definition**
The production `ForStatement : for ( ExpressionNoInopt ; Expressionopt ; Expressionopt ) Statement` evaluates the first expression once before the loop, evaluates the second as a test before each iteration, and evaluates the third after each iteration. Variables declared with `let` are scoped to the loop body; variables declared with `var` share the enclosing function scope. If the condition is omitted, it always evaluates to `true`, creating a potential infinite loop.

**Beginner-Friendly Explanation**
The `for` loop is the "classic" loop with a built-in counter. You say: "Start at 0, keep going as long as `i` is less than 10, and add 1 to `i` after each round." It is the most common loop in JavaScript because it gives you full control over the counter.

### Purposes

- To iterate a known number of times with a counter variable.
- To access array elements by index in sequential order.
- To repeat an operation with precise control over initialization, condition, and increment.
- To implement countdown, count-up, and step-based iteration patterns.

### Syntax Rules and Structure

**Complete General Syntax**
```js
for (initialization; condition; afterthought)
  statement
```

With a block statement:
```js
for (initialization; condition; afterthought) {
  statements
}
```

**Component Breakdown**

| Component | Description |
|---|---|
| `initialization` (optional) | Expression or variable declaration evaluated once before the loop. Typically initializes a counter. `var` is function-scoped; `let` is block-scoped to the loop. |
| `condition` (optional) | Expression evaluated before each iteration. If truthy, the body executes. If falsy, the loop exits. Omitted condition always evaluates to `true`. |
| `afterthought` (optional) | Expression evaluated at the end of each iteration, before the next condition check. Typically increments or decrements the counter. |
| `statement` | The statement or block executed as long as the condition is truthy. |

**Syntax Rules**

- All three expressions are optional; `for (;;)` creates an infinite loop.
- The initialization may declare variables with `var` or `let`. With `let`, a new binding is created for each iteration (important for closures).
- The condition is evaluated before every iteration, including the first.
- Multiple expressions can be separated by commas in the initialization and afterthought.
- The statement body can be a single statement or a block `{ }`.

**Constraints and Limitations**

- Using `var` in the initialization makes the variable leak into the enclosing scope.
- Modifying the counter inside the body can lead to unexpected iteration counts.
- `for` cannot directly iterate over object properties or iterable values without manual index management.

### Annotated Code Examples

**Example 1: Basic count-up**

```js
// Step 1: Initialize i to 0 before the loop starts.
// Step 2: Before each iteration, check i < 5.
// Step 3: After each iteration, increment i by 1.
for (let i = 0; i < 5; i++) {
  // Step 4: Execute the body while the condition is true.
  console.log(i);
}

// Expected Output:
// 0
// 1
// 2
// 3
// 4
```

**Breakdown**: `i` starts at 0. The condition `0 < 5` is true, so `0` is logged. `i` becomes 1. This repeats until `i` is 5; `5 < 5` is false, and the loop exits. `i` is 5 after the loop, but `let` scopes it to the loop, so it is not accessible outside.

**Example 2: Countdown with step**

```js
// Step 1: Start at 10, go down to 0, step by 2.
for (let i = 10; i >= 0; i -= 2) {
  console.log(i);
}

// Expected Output:
// 10
// 8
// 6
// 4
// 2
// 0
```

**Breakdown**: The afterthought `i -= 2` subtracts 2 after each iteration. The condition `i >= 0` allows the loop to continue until `i` reaches 0.

**Example 3: `let` vs `var` scoping**

```js
// Step 1: Using let — i is block-scoped.
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(`let: ${i}`), 0);
}

// Step 2: Using var — i is function-scoped.
for (var j = 0; j < 3; j++) {
  setTimeout(() => console.log(`var: ${j}`), 0);
}

// Expected Output:
// let: 0
// let: 1
// let: 2
// var: 3
// var: 3
// var: 3
```

**Breakdown**: With `let`, each iteration creates a new binding, so each `setTimeout` closure captures a different `i`. With `var`, there is a single function-scoped `j`, and all closures see the final value `3`.

### Real-World Cases

- **Array traversal**: `for (let i = 0; i < arr.length; i++) { … }`
- **Matrix operations**: `for (let r = 0; r < rows; r++) { for (let c = 0; c < cols; c++) { … } }`
- **Countdown timers**: `for (let t = 10; t >= 0; t--) { … }`
- **Pagination**: Iterating pages with a step-based counter.

### References

- MDN Web Docs — `for` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for
- ECMAScript Language Specification — The `for` Statement — https://tc39.es/ecma262/#sec-for-statement
- MDN Web Docs — Loops and iteration — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Loops_and_iteration

---

## 2. `for...in` Statement

### Definitions

**Core Definition**
The `for...in` statement iterates over all enumerable string-keyed properties of an object, including inherited enumerable properties.

**Technical Definition**
The production `ForInOfStatement : for ( LeftHandSideExpression in Expression ) Statement` and its `ForDeclaration` variants evaluate the expression, apply `ToObject`, and enumerate all enumerable string-keyed properties in a spec-defined order: integer-like keys in ascending numeric order, then string keys in insertion order, then symbol keys (which are ignored). The loop variable receives each property name as a string.

**Beginner-Friendly Explanation**
`for...in` walks through the property names of an object. If you have `{ name: "Alice", age: 30 }`, it gives you `"name"` and `"age"`. It is designed for objects, not arrays, and it also picks up properties that the object inherited from its prototype.

### Purposes

- To enumerate the property names of a plain object.
- To inspect an object's keys for debugging or reflection.
- To iterate over inherited enumerable properties when intentional.
- To implement generic object iteration when the key set is not known in advance.

### Syntax Rules and Structure

**Complete General Syntax**
```js
for (variable in object)
  statement
```

**Component Breakdown**

| Component | Description |
|---|---|
| `variable` | Receives a string property name on each iteration. May be declared with `const`, `let`, `var`, or be an existing assignment target. `var` is function-scoped; `const`/`let` are block-scoped. |
| `object` | The object whose enumerable string-keyed properties are iterated. |
| `statement` | Executed for each property name. |

**Syntax Rules**

- Iterates enumerable, non-symbol string-keyed properties.
- Includes inherited enumerable properties, not just own properties.
- Property enumeration order: integer-like keys ascending, then string keys in insertion order.
- The loop variable is assigned the property name, not the value.
- `break` and `continue` can be used inside the body.

**Constraints and Limitations**

- **Not recommended for arrays**: It iterates indices as strings and includes non-index properties. Use `for...of` or a classic `for` loop instead.
- Inherited properties can cause unexpected iterations; use `Object.hasOwn()` or `Object.keys()` to filter.
- The order of enumeration is specified but can be surprising for mixed key types.
- Does not iterate symbol-keyed properties.

### Annotated Code Examples

**Example 1: Basic object iteration**

```js
// Step 1: Define an object with three properties.
const person = { name: "Alice", age: 30, city: "Paris" };

// Step 2: for...in yields the property name (key) on each iteration.
for (const key in person) {
  // Step 3: Access the value using bracket notation.
  console.log(`${key}: ${person[key]}`);
}

// Expected Output:
// name: Alice
// age: 30
// city: Paris
```

**Breakdown**: The loop variable `key` receives `"name"`, `"age"`, and `"city"` in insertion order. `person[key]` retrieves the corresponding value.

**Example 2: Inherited properties and `Object.hasOwn()`**

```js
// Step 1: Create a prototype object.
const proto = { inherited: true };

// Step 2: Create an object that inherits from proto.
const obj = Object.create(proto);
obj.own = "value";

// Step 3: for...in includes inherited properties.
for (const key in obj) {
  console.log(`for...in: ${key}`);
}

// Step 4: Filter to own properties only.
for (const key in obj) {
  if (Object.hasOwn(obj, key)) {
    console.log(`own: ${key}`);
  }
}

// Expected Output:
// for...in: own
// for...in: inherited
// own: own
```

**Breakdown**: `for...in` iterates both `own` and `inherited`. `Object.hasOwn()` filters out the inherited property, leaving only `"own"`.

### Real-World Cases

- **Debugging**: Logging all properties of an object including inherited ones.
- **Configuration merging**: Iterating over option objects.
- **Serialisation**: Converting plain objects to query strings.
- **Legacy code**: Older codebases often use `for...in` for object iteration.
- **Avoid for arrays**: Use `for...of` or index-based `for` for arrays.

### References

- MDN Web Docs — `for...in` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in
- ECMAScript Language Specification — The `for-in` and `for-of` Statements — https://tc39.es/ecma262/#sec-for-in-and-for-of-statements
- MDN Web Docs — Enumerability and ownership of properties — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Enumerability_and_ownership_of_properties

---

## 3. `for...of` Statement

### Definitions

**Core Definition**
The `for...of` statement iterates over the values of an iterable object, such as arrays, strings, maps, sets, and generators.

**Technical Definition**
The production `ForInOfStatement : for ( LeftHandSideExpression of AssignmentExpression ) Statement` and its `ForDeclaration` variants evaluate the right-hand expression, obtain the `[Symbol.iterator]()` method, call it to get an iterator, and repeatedly call `next()`. If `done` is `false`, the loop variable is assigned `value` and the body executes. If `done` is `true`, the loop exits. `for...of` works on both sync iterables and, in async contexts, `for await...of` works on async iterables.

**Beginner-Friendly Explanation**
`for...of` gives you the values of an array directly, without needing an index. Instead of `arr[i]`, you just get each item. It works on arrays, strings, maps, sets, and anything that can be "iterated."

### Purposes

- To iterate array elements directly without index management.
- To process the values of Maps and Sets in a clean, readable way.
- To consume generator output value by value.
- To iterate strings by Unicode code point.
- To replace index-based `for` loops when the index is not needed.

### Syntax Rules and Structure

**Complete General Syntax**
```js
for (variable of iterable)
  statement
```

**Component Breakdown**

| Component | Description |
|---|---|
| `variable` | Receives a value from the iterable on each iteration. May be declared with `const`, `let`, `var`, or be an assignment target. Supports destructuring patterns. |
| `iterable` | An object that implements the `[Symbol.iterator]()` method (e.g., Array, String, Map, Set, generator, NodeList). |
| `statement` | Executed for each value. |

**Syntax Rules**

- The iterable must implement the iterable protocol (`[Symbol.iterator]()`).
- Iterates values, not indices or keys.
- Supports destructuring: `for (const [key, value] of map) { … }`.
- Can be used with `break` and `continue`.
- Available in all modern browsers since July 2015.

**Constraints and Limitations**

- **Performance**: `for...of` is generally slower than a classic `for` loop for large arrays because it invokes the iterator protocol on each iteration. A WebKit bug report measured `for...of` as noticeably slower than `forEach` and classic `for` loops in Safari, though the difference was smaller in Chrome and Firefox.
- Does not work on plain objects (which are not iterable by default).
- Cannot modify the iteration variable to change the underlying collection (though you can mutate object properties).

### Annotated Code Examples

**Example 1: Array iteration**

```js
// Step 1: Define an array of fruits.
const fruits = ["apple", "banana", "cherry"];

// Step 2: for...of yields each value directly.
for (const fruit of fruits) {
  console.log(fruit);
}

// Expected Output:
// apple
// banana
// cherry
```

**Breakdown**: The iterator protocol returns `"apple"`, then `"banana"`, then `"cherry"`. Each is assigned to `fruit` and logged.

**Example 2: Map iteration with destructuring**

```js
// Step 1: Create a Map with key-value pairs.
const map = new Map([["a", 1], ["b", 2], ["c", 3]]);

// Step 2: Destructure each entry into key and value.
for (const [key, value] of map) {
  console.log(`${key} -> ${value}`);
}

// Expected Output:
// a -> 1
// b -> 2
// c -> 3
```

**Breakdown**: A Map's iterator yields `[key, value]` pairs. Destructuring extracts both into separate variables.

**Example 3: String iteration by code point**

```js
// Step 1: Define a string with a multi-byte character.
const str = "a🌟b";

// Step 2: for...of iterates by Unicode code point.
for (const char of str) {
  console.log(char);
}

// Expected Output:
// a
// 🌟
// b
```

**Breakdown**: Unlike index-based iteration, `for...of` correctly handles surrogate pairs and iterates by code point.

### Real-World Cases

- **Array processing**: Summing values, filtering, transforming.
- **Map/Set traversal**: Configuration maps, deduplicated collections.
- **Generator consumption**: Reading paginated API results.
- **DOM collections**: Iterating `NodeList` from `querySelectorAll`.
- **String processing**: Character-by-character analysis.

### References

- MDN Web Docs — `for...of` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of
- ECMAScript Language Specification — The `for-in` and `for-of` Statements — https://tc39.es/ecma262/#sec-for-in-and-for-of-statements
- MDN Web Docs — Iteration protocols — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols

---

## 4. `while` Statement

### Definitions

**Core Definition**
The `while` statement creates a loop that executes a specified statement as long as a test condition evaluates to true, with the condition evaluated before each execution.

**Technical Definition**
The production `IterationStatement : while ( Expression ) Statement` evaluates the `Expression`, applies `ToBoolean` to the result, and if truthy, evaluates the `Statement` and repeats. If falsy, the loop exits and execution continues with the next statement. The condition is evaluated before the first iteration, so a falsy condition means the body never executes.

**Beginner-Friendly Explanation**
`while` means "keep doing this as long as the condition is true." Unlike `for`, there is no built-in counter. You manage the loop variable yourself. If you forget to change the variable, the loop never ends.

### Purposes

- To repeat code while a condition remains true, without a known iteration count.
- To poll for a state change (e.g., waiting for a resource to become available).
- To process data streams or queues until exhausted.
- To implement loops with complex or dynamic termination conditions.

### Syntax Rules and Structure

**Complete General Syntax**
```js
while (condition)
  statement
```

**Component Breakdown**

| Component | Description |
|---|---|
| `condition` | Evaluated before each iteration. If truthy, the body executes; if falsy, the loop exits. |
| `statement` | The statement or block executed as long as the condition is truthy. |

**Syntax Rules**

- Parentheses around the condition are mandatory.
- The condition is evaluated before the first iteration; if initially falsy, the body never runs.
- `break` and `continue` can be used inside the body.
- The condition can be an assignment expression (e.g., `while (node = iterator.nextNode())`), though this can reduce readability.

**Constraints and Limitations**

- If the condition never becomes falsy, the loop becomes infinite.
- Unlike `do...while`, the body may execute zero times.
- Forgetting to update the loop variable is a common bug.

### Annotated Code Examples

**Example 1: Basic `while` loop**

```js
// Step 1: Initialize a counter outside the loop.
let i = 0;

// Step 2: Check the condition before each iteration.
while (i < 5) {
  console.log(i);
  // Step 3: Update the counter to eventually make the condition false.
  i++;
}

// Expected Output:
// 0
// 1
// 2
// 3
// 4
```

**Breakdown**: The condition `i < 5` is checked before each iteration. When `i` reaches 5, the condition is false and the loop exits.

**Example 2: `while` with assignment as condition**

```js
// Step 1: Simulate a queue of items.
const queue = ["task1", "task2", "task3"];

// Step 2: Process items until the queue is empty.
while (queue.length > 0) {
  const task = queue.shift();
  console.log(`Processing: ${task}`);
}

// Expected Output:
// Processing: task1
// Processing: task2
// Processing: task3
```

**Breakdown**: `queue.shift()` removes and returns the first element. The condition `queue.length > 0` becomes false when the queue is empty.

### Real-World Cases

- **Game loops**: `while (gameRunning) { update(); render(); }`
- **Retry logic**: `while (retries < maxRetries && !success) { … }`
- **Reading input**: `while ((line = readLine()) !== null) { … }`
- **Polling**: `while (!dataReady) { await sleep(100); }`

### References

- MDN Web Docs — `while` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/while
- ECMAScript Language Specification — The `while` Statement — https://tc39.es/ecma262/#sec-while-statement
- MDN Web Docs — Loops and iteration — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Loops_and_iteration

---

## 5. `do...while` Statement

### Definitions

**Core Definition**
The `do...while` statement creates a loop that executes a specified statement at least once, then repeats as long as the test condition evaluates to true.

**Technical Definition**
The production `IterationStatement : do Statement while ( Expression ) ;` evaluates the `Statement` first, then evaluates `Expression` and applies `ToBoolean`. If truthy, the loop repeats. If falsy, the loop exits. The condition is evaluated after the body, guaranteeing at least one execution.

**Beginner-Friendly Explanation**
`do...while` is like `while`, but the condition is checked after the body runs. This means the body always runs at least once, even if the condition is false from the start. It is useful for "do this, then check if we should do it again."

### Purposes

- To guarantee at least one execution of a loop body.
- To implement retry logic where the first attempt always happens.
- To create interactive prompts that must display at least once.
- To process data where the first item must be handled unconditionally.

### Syntax Rules and Structure

**Complete General Syntax**
```js
do
  statement
while (condition);
```

**Component Breakdown**

| Component | Description |
|---|---|
| `do` | Keyword introducing the loop body. |
| `statement` | Executed at least once, then repeatedly as long as the condition is truthy. |
| `while` | Keyword introducing the condition check. |
| `condition` | Evaluated after each execution of the body. If truthy, the loop repeats. |
| `;` | Semicolon terminates the statement (required, though ASI may insert it). |

**Syntax Rules**

- The body executes before the condition is first evaluated.
- A semicolon after `while (condition)` is required.
- `break` and `continue` can be used inside the body.
- `continue` in a `do...while` loop jumps to the condition evaluation.

**Constraints and Limitations**

- Less commonly used than `while`; easy to forget the trailing semicolon.
- The body always executes at least once, which may be undesirable if the condition is initially false.
- No built-in counter; you manage the loop variable yourself.

### Annotated Code Examples

**Example 1: Basic `do...while`**

```js
// Step 1: Initialize a counter.
let i = 0;

// Step 2: Execute the body first.
do {
  console.log(i);
  i++;
// Step 3: Check the condition after each iteration.
} while (i < 5);

// Expected Output:
// 0
// 1
// 2
// 3
// 4
```

**Breakdown**: The body runs, printing `0` and incrementing `i`. The condition `i < 5` is checked; if true, the loop repeats. When `i` reaches 5, the loop exits.

**Example 2: Body executes even when condition is initially false**

```js
// Step 1: Set a condition that is false from the start.
let count = 10;

// Step 2: The body still runs once.
do {
  console.log(`Executed with count = ${count}`);
  count++;
} while (count < 5);

// Expected Output:
// Executed with count = 10
```

**Breakdown**: `count` starts at 10. The body runs and prints `10`. The condition `10 < 5` is false, so the loop exits after one iteration.

### Real-World Cases

- **User prompts**: `do { input = prompt("Enter a number"); } while (isNaN(input));`
- **Retry logic**: `do { result = attempt(); } while (!result.success && ++retries < 3);`
- **Menu systems**: `do { showMenu(); choice = getUserChoice(); } while (choice !== "exit");`
- **Tokenisation**: Reading tokens until an end-of-file marker is consumed.

### References

- MDN Web Docs — `do...while` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/do...while
- ECMAScript Language Specification — The `do-while` Statement — https://tc39.es/ecma262/#sec-do-while-statement
- MDN Web Docs — Loops and iteration — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Loops_and_iteration

---

## 6. `break` Statement (Loop Control)

### Definitions

**Core Definition**
The `break` statement terminates the innermost enclosing loop or `switch` statement and transfers control to the statement following the terminated construct.

**Technical Definition**
When `break;` is encountered, the program exits the innermost enclosing `switch`, `while`, `do...while`, `for`, `for...in`, or `for...of` statement and continues with the next statement after it. With a label, `break label;` exits the labelled statement, which may be an outer loop.

**Beginner-Friendly Explanation**
`break` is the "stop the loop now" instruction. It exits the loop immediately, even if the condition is still true.

### Purposes

- To exit a loop early when a desired result is found.
- To stop infinite loops when a termination condition is met.
- To break out of nested loops using a label.
- To terminate a `switch` case and prevent fall-through.

### Syntax Rules and Structure

**Complete General Syntax**
```js
break;
break label;
```

**Component Breakdown**

| Component | Description |
|---|---|
| `break` | Keyword that terminates the enclosing loop or switch. |
| `label` | Optional identifier of a labelled statement to exit. |

**Syntax Rules**

- `break;` must be inside a loop or `switch` unless it has a label.
- `break label;` requires the label to be in scope (i.e., the `break` must be nested inside the labelled statement).
- Cannot be used at the top level of a script, module, function, or static block.
- Labels cannot be reserved words.

**Constraints and Limitations**

- Only exits the innermost applicable statement (or the labelled one).
- Overuse of labels can reduce readability.

### Annotated Code Examples

**Example 1: `break` in a loop**

```js
// Step 1: Loop from 0 to 4.
for (let i = 0; i < 5; i++) {
  // Step 2: Stop when i equals 3.
  if (i === 3) {
    break; // Exits the for loop.
  }
  console.log(i);
}

// Expected Output:
// 0
// 1
// 2
```

**Breakdown**: The loop prints 0, 1, 2. When `i === 3`, `break` terminates the loop before printing 3.

**Example 2: Early exit from a search loop**

```js
// Step 1: Define an array and a target.
const numbers = [1, 2, 3, 4, 5];
const target = 3;

// Step 2: Search for the target.
for (const num of numbers) {
  if (num === target) {
    console.log(`Found ${target}!`);
    break; // Step 3: Exit the loop immediately.
  }
  console.log(`Checking ${num}...`);
}

// Expected Output:
// Checking 1...
// Checking 2...
// Found 3!
```

**Breakdown**: The loop checks 1 and 2, logging each. When 3 matches, it logs the success and `break` exits the loop, skipping 4 and 5.

**Example 3: Labelled `break` in nested loops**

```js
// Step 1: Define a 3x3 grid.
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (i === 1 && j === 1) {
      console.log("Breaking out of both loops");
      break outer; // Step 2: Exit the outer loop entirely.
    }
    console.log(`i=${i}, j=${j}`);
  }
}

// Expected Output:
// i=0, j=0
// i=0, j=1
// i=0, j=2
// i=1, j=0
// Breaking out of both loops
```

**Breakdown**: Without the label, `break` would only exit the inner loop. With `break outer`, both loops terminate.

### Real-World Cases

- **Search operations**: Stop scanning once the target is found.
- **Validation**: Break out of a validation loop on the first error.
- **Game logic**: Exit the update loop when the game ends.
- **Nested matrix operations**: Exit both loops when a condition is met.

### References

- MDN Web Docs — `break` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/break
- ECMAScript Language Specification — The `break` Statement — https://tc39.es/ecma262/#sec-break-statement
- MDN Web Docs — Labeled statement — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/label

---

## 7. `continue` Statement (Loop Control)

### Definitions

**Core Definition**
The `continue` statement terminates the current iteration of the innermost enclosing loop and proceeds to the next iteration.

**Technical Definition**
When `continue;` is encountered, the current iteration's remaining statements are skipped. In a `while` or `do...while` loop, execution jumps to the condition evaluation. In a `for` loop, execution jumps to the afterthought expression. In `for...in`, `for...of`, and `for await...of`, execution jumps to the next iteration. With a label, `continue label;` jumps to the next iteration of the labelled loop.

**Beginner-Friendly Explanation**
`continue` means "skip the rest of this round and go to the next one." It does not stop the loop—it just skips ahead.

### Purposes

- To skip processing for certain values while continuing the loop.
- To avoid deeply nested `if` blocks inside loops.
- To implement filtering logic within a loop.
- To skip invalid or irrelevant iterations without exiting the loop.

### Syntax Rules and Structure

**Complete General Syntax**
```js
continue;
continue label;
```

**Component Breakdown**

| Component | Description |
|---|---|
| `continue` | Keyword that skips to the next iteration. |
| `label` | Optional identifier of a labelled loop to continue. |

**Syntax Rules**

- `continue;` must be inside a loop.
- `continue label;` requires the label to be in scope.
- Cannot be used at the top level of a script, module, function, or static block.
- In a `for` loop, `continue` jumps to the afterthought, not the condition.

**Constraints and Limitations**

- Overuse can make control flow harder to follow.
- Cannot be used to skip iterations of an outer loop without a label.
- `continue` in a `do...while` loop jumps to the condition check, not the top of the body.

### Annotated Code Examples

**Example 1: Skipping even numbers**

```js
// Step 1: Loop from 1 to 6.
for (let i = 1; i <= 6; i++) {
  // Step 2: Skip even numbers.
  if (i % 2 === 0) {
    continue; // Skips the rest of this iteration.
  }
  console.log(i);
}

// Expected Output:
// 1
// 3
// 5
```

**Breakdown**: When `i` is even, `continue` skips the `console.log` and proceeds to the next iteration. Odd numbers are logged.

**Example 2: Labelled `continue`**

```js
// Step 1: Outer loop labelled "outer".
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (j === 1) {
      continue outer; // Step 2: Skip to the next outer iteration.
    }
    console.log(`i=${i}, j=${j}`);
  }
}

// Expected Output:
// i=0, j=0
// i=1, j=0
// i=2, j=0
```

**Breakdown**: When `j === 1`, `continue outer` skips the rest of the inner loop and moves to the next iteration of the outer loop. The inner loop never reaches `j = 2`.

### Real-World Cases

- **Data filtering**: Skip invalid records while processing a batch.
- **Grid rendering**: Skip empty cells in a matrix.
- **Validation**: Skip optional fields that are not present.
- **Game loops**: Skip entities that are inactive.

### References

- MDN Web Docs — `continue` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/continue
- ECMAScript Language Specification — The `continue` Statement — https://tc39.es/ecma262/#sec-continue-statement
- MDN Web Docs — Labeled statement — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/label

---

## 8. Labels

### Definitions

**Core Definition**
A label is an identifier prefix on a statement that allows `break` or `continue` to target that specific statement from nested code.

**Technical Definition**
The production `LabelledStatement : LabelIdentifier : LabelledItem` creates a labelled statement. The identifier is available for `break label;` (which exits the labelled statement) and `continue label;` (which continues the labelled loop). JavaScript has no `goto`; labels are only usable with `break` and `continue`. A statement may have multiple labels, all functionally equivalent.

**Beginner-Friendly Explanation**
Labels let you name a loop so that `break` or `continue` can refer to it, even from inside another loop. Without labels, `break` and `continue` only affect the innermost loop. Labels solve the "how do I exit both loops at once?" problem.

### Purposes

- To exit an outer loop from within an inner loop using `break label`.
- To skip to the next iteration of an outer loop using `continue label`.
- To make nested loop control flow explicit and intentional.
- To avoid flag variables or complex condition checks in nested loops.

### Syntax Rules and Structure

**Complete General Syntax**
```js
label: statement
```

With a loop:
```js
label: for (…) { … }
label: while (…) { … }
label: do { … } while (…);
```

**Component Breakdown**

| Component | Description |
|---|---|
| `label` | Any JavaScript identifier that is not a reserved word. |
| `statement` | The statement to label. Can be a loop, block, or any statement. |

**Syntax Rules**

- A label can prefix any statement, but `continue label` only works if the labelled statement is a loop.
- `break label` works with any labelled statement.
- Labels are scoped to the statement they label.
- Multiple labels on the same statement are functionally equivalent.
- The `break` or `continue` referencing the label must be nested inside the labelled statement.

**Constraints and Limitations**

- Overuse reduces readability; many style guides discourage labels.
- JavaScript has no `goto`; labels are only for `break` and `continue`.
- Labels cannot be used to jump into a block from outside.

### Annotated Code Examples

**Example 1: Labelled `break` to exit nested loops**

```js
// Step 1: Label the outer loop as "search".
search: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (i === 1 && j === 1) {
      console.log("Found at (1,1)");
      break search; // Step 2: Exit both loops entirely.
    }
    console.log(`Checking (${i},${j})`);
  }
}

// Expected Output:
// Checking (0,0)
// Checking (0,1)
// Checking (0,2)
// Checking (1,0)
// Found at (1,1)
```

**Breakdown**: Without the label, `break` would only exit the inner `j` loop. `break search` exits the outer `i` loop as well.

**Example 2: Labelled `continue` to skip outer iterations**

```js
// Step 1: Label the outer loop as "rows".
rows: for (let row = 0; row < 3; row++) {
  for (let col = 0; col < 3; col++) {
    if (col === 1) {
      continue rows; // Step 2: Skip to the next row.
    }
    console.log(`row=${row}, col=${col}`);
  }
}

// Expected Output:
// row=0, col=0
// row=1, col=0
// row=2, col=0
```

**Breakdown**: When `col === 1`, `continue rows` skips the rest of the inner loop and moves to the next iteration of the outer `row` loop. The inner loop never reaches `col = 2`.

### Real-World Cases

- **Matrix search**: Find a value in a 2D array and exit both loops.
- **Game boards**: Skip entire rows when a condition is met.
- **Nested validation**: Skip an entire record if a critical field is invalid.
- **Pathfinding**: Abort exploration of a branch and move to the next.

### References

- MDN Web Docs — Labeled statement — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/label
- ECMAScript Language Specification — Labelled Statements — https://tc39.es/ecma262/#sec-labelled-statements
- MDN Web Docs — `break` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/break

---

## 9. Infinite Loops

### Definitions

**Core Definition**
An infinite loop is a loop whose termination condition never becomes false, causing it to execute indefinitely.

**Technical Definition**
An infinite loop occurs when the loop condition always evaluates to a truthy value. Common causes include omitting the condition in a `for` loop (`for (;;)`), using a condition that never changes (e.g., `while (true)` without a `break`), or failing to update the loop variable. Intentional infinite loops are sometimes used with `break` as the exit mechanism.

**Beginner-Friendly Explanation**
An infinite loop is a loop that never stops. It happens when the condition for stopping is never met—either because you forgot to update a variable or because the condition is always true. This can freeze your program or browser tab.

### Purposes

- To create a loop that runs until an explicit `break` condition is met (e.g., game loops).
- To implement event loops and servers that run indefinitely.
- To act as a deliberate anti-pattern that developers must recognise and avoid.
- To serve as a debugging challenge that teaches careful condition management.

### Syntax Rules and Structure

**Common Infinite Loop Patterns**
```js
// Pattern 1: Missing condition
for (;;) { … }

// Pattern 2: Always-true condition
while (true) { … }

// Pattern 3: Condition never changes
let i = 0;
while (i < 10) { … }  // i is never incremented
```

**Component Breakdown**

| Pattern | Description |
|---|---|
| `for (;;)` | Omitting the condition makes it always true. |
| `while (true)` | Explicit always-true condition. |
| Unchanged variable | The loop variable is never updated, so the condition remains true. |

**Syntax Rules**

- `for (;;)` is valid and creates an infinite loop.
- `while (true)` is valid and creates an infinite loop.
- An infinite loop can be exited with `break`, `return`, or by throwing an exception.
- In a browser, an infinite loop blocks the main thread and freezes the UI.

**Constraints and Limitations**

- Unintentional infinite loops are a common bug and can crash the browser tab or Node.js process.
- Intentional infinite loops require a reliable exit condition.
- Infinite loops in asynchronous code must not block the event loop.

### Annotated Code Examples

**Example 1: Unintentional infinite loop**

```js
// Step 1: Initialize a counter.
let i = 0;

// Step 2: The condition is always true because i is never incremented.
while (i < 5) {
  console.log(i);
  // Missing i++ → infinite loop!
}
```

**Breakdown**: `i` remains 0 forever, so `i < 5` is always true. The loop never exits. To fix, add `i++` inside the body.

**Example 2: Intentional infinite loop with `break`**

```js
// Step 1: Loop forever.
let count = 0;
while (true) {
  count++;
  console.log(`Iteration ${count}`);
  // Step 2: Exit when count reaches 3.
  if (count >= 3) {
    break;
  }
}

// Expected Output:
// Iteration 1
// Iteration 2
// Iteration 3
```

**Breakdown**: `while (true)` creates an infinite loop, but `break` exits it when `count` reaches 3.

### Real-World Cases

- **Game loops**: `while (true) { update(); render(); if (quit) break; }`
- **Server event loops**: Node.js event loop runs continuously.
- **Animation frames**: `requestAnimationFrame` recursively schedules itself.
- **Anti-pattern examples**: Teaching beginners to identify and fix infinite loops.

### References

- MDN Web Docs — Loops and iteration — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Loops_and_iteration
- MDN Web Docs — `for` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for
- MDN Web Docs — `while` — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/while

---

## 10. Loop Performance

### Definitions

**Core Definition**
Loop performance refers to the execution speed and resource efficiency of different loop constructs and iteration strategies.

**Technical Definition**
Loop performance is influenced by the number of iterations, the complexity of operations inside the loop, the loop construct used, property access patterns, and the JavaScript engine's optimisation capabilities (e.g., V8's Maglev and Turboshaft compilers). Modern engines perform inline caching, loop peeling, and JIT compilation, but these optimisations are sensitive to code structure.

**Beginner-Friendly Explanation**
Some loops are faster than others. A classic `for` loop is usually the fastest for simple operations. `for...of` is cleaner but slightly slower. `for...in` is the slowest for arrays. Caching the array length and avoiding work inside the loop can make a big difference when processing thousands of items.

### Purposes

- To process large datasets efficiently without degrading user experience.
- To reduce execution time in performance-critical code paths.
- To avoid blocking the main thread with slow iterations.
- To make informed choices between loop constructs based on measurable trade-offs.

### Syntax Rules and Structure

**Performance Hierarchy (General Guidance)**

| Loop Type | Relative Speed | Best For |
|---|---|---|
| Classic `for` | Fastest | Performance-critical array processing |
| Reverse `while` | Very fast | When order doesn't matter |
| `forEach` | Moderate | Readability with acceptable performance |
| `for...of` | Moderate | Clean syntax with good balance |
| `for...in` | Slowest | Objects only (not arrays) |

**Component Breakdown**

| Factor | Impact |
|---|---|
| Array length caching | Eliminates property lookup on every iteration. |
| Loop construct choice | Classic `for` is fastest; `for...of` is slower due to iterator protocol. |
| Operations inside loop | Function calls, DOM access, and object creation are expensive. |
| Loop nest depth | Each additional level multiplies iterations (O(n²), O(n³)). |
| Variable declarations | `let` is preferred over `var` for block scoping. |

**Syntax Rules**

- Cache `array.length` in a variable before the loop.
- Avoid function calls inside loops when the result is constant.
- Use `break` to exit early when the result is found.
- Prefer `for` over `for...in` for arrays.
- Use `Set` or `Map` for lookups instead of `Array.includes` inside loops.

**Constraints and Limitations**

- Performance differences are engine-dependent and often negligible for small datasets.
- Premature optimisation can reduce readability without meaningful gains.
- V8 optimises loops that run many times; loops that run only once may not be JIT-compiled.

### Annotated Code Examples

**Example 1: Caching array length**

```js
// ❌ Slow: array.length is accessed on every iteration.
const array = Array.from({ length: 100000 }, (_, i) => i);

for (let i = 0; i < array.length; i++) {
  // process array[i]
}

// ✅ Fast: length is cached before the loop.
const length = array.length;
for (let i = 0; i < length; i++) {
  // process array[i]
}
```

**Breakdown**: Accessing `array.length` requires a property lookup on each iteration. Caching it eliminates that lookup. For arrays with thousands of elements, this provides measurable performance gains.

**Example 2: Reverse `while` loop**

```js
// ✅ Fast: comparing to zero is slightly faster than comparing to a variable.
let i = array.length;
while (i--) {
  process(array[i]);
}
```

**Breakdown**: The `while (i--)` pattern compares `i` to zero (which is fast) and decrements in the same expression. This is the most optimised version for order-independent processing.

**Example 3: Moving function calls out of the loop**

```js
// ❌ Slow: expensiveCalculation() is called every iteration.
for (let i = 0; i < array.length; i++) {
  array[i] = array[i] * expensiveCalculation();
}

// ✅ Fast: called once before the loop.
const multiplier = expensiveCalculation();
for (let i = 0; i < array.length; i++) {
  array[i] = array[i] * multiplier;
}
```

**Breakdown**: If a function returns the same value on every iteration, call it once outside the loop. This eliminates redundant computation.

### Real-World Cases

- **Data processing**: Processing millions of records in Node.js.
- **DOM manipulation**: Batch reads and writes to avoid layout thrashing.
- **Image processing**: Iterating over pixel arrays in canvas applications.
- **Game engines**: Updating thousands of entities per frame.
- **Financial calculations**: Processing large transaction histories.

### References

- CoreUI — How to optimize loop performance in JavaScript — https://coreui.io/answers/how-to-optimize-loop-performance-in-javascript/
- WebKit Bug 283118 — `for-of` loops noticeably slower than alternatives — https://wiki.webkit.org/show_bug.cgi?id=283118
- MDN Web Docs — Loops and iteration — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Loops_and_iteration
- V8 Blog — https://v8.dev/blog

---

## 11. Nested Loops

### Definitions

**Core Definition**
A nested loop is a loop placed inside the body of another loop, allowing multi-dimensional iteration.

**Technical Definition**
Since any statement can appear in the body of a loop, a loop can contain another loop. The inner loop completes all its iterations for each single iteration of the outer loop. Total iterations equal the product of the outer and inner iteration counts. `break` and `continue` without labels affect only the innermost loop; labels are required to control outer loops.

**Beginner-Friendly Explanation**
Nested loops are loops inside loops. They are used for working with grids, tables, and multi-dimensional arrays. For every single step of the outer loop, the inner loop runs completely.

### Purposes

- To iterate over multi-dimensional data structures (matrices, grids, tables).
- To generate combinations or permutations.
- To compare every element of one collection with every element of another.
- To render rows and columns in UI layouts.

### Syntax Rules and Structure

**Complete General Syntax**
```js
for (outer initialization; outer condition; outer afterthought) {
  for (inner initialization; inner condition; inner afterthought) {
    statements
  }
}
```

**Component Breakdown**

| Component | Description |
|---|---|
| Outer loop | Controls the outer dimension (e.g., rows). |
| Inner loop | Runs to completion for each outer iteration (e.g., columns). |
| Total iterations | Outer count × Inner count. |

**Syntax Rules**

- The inner loop is reinitialized for each outer iteration.
- `break` and `continue` without labels apply to the innermost loop.
- Labels can target outer loops for `break` and `continue`.
- Use different variable names for each loop level (e.g., `i`, `j`, `k`).

**Constraints and Limitations**

- **O(n²) complexity**: Nested loops over the same collection produce quadratic time complexity.
- Deep nesting (>3 levels) reduces readability and is prone to logic errors.
- Consider alternative algorithms (sorting, hashing, divide-and-conquer) for large datasets.

### Annotated Code Examples

**Example 1: Multiplication table**

```js
// Step 1: Outer loop controls rows (1 to 3).
for (let i = 1; i <= 3; i++) {
  let row = "";
  // Step 2: Inner loop controls columns (1 to 3).
  for (let j = 1; j <= 3; j++) {
    row += `${i * j}\t`;
  }
  // Step 3: Print the completed row.
  console.log(row);
}

// Expected Output:
// 1	2	3
// 2	4	6
// 3	6	9
```

**Breakdown**: For each `i` from 1 to 3, the inner loop runs `j` from 1 to 3, computing `i * j`. The row string accumulates the products, then is logged.

**Example 2: 2D array traversal**

```js
// Step 1: Define a 3x3 matrix.
const matrix = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
];

// Step 2: Outer loop iterates rows.
for (let row = 0; row < matrix.length; row++) {
  // Step 3: Inner loop iterates columns.
  for (let col = 0; col < matrix[row].length; col++) {
    console.log(`matrix[${row}][${col}] = ${matrix[row][col]}`);
  }
}

// Expected Output:
// matrix[0][0] = 1
// matrix[0][1] = 2
// matrix[0][2] = 3
// matrix[1][0] = 4
// ...
// matrix[2][2] = 9
```

**Breakdown**: The outer loop iterates over rows, and the inner loop iterates over columns within each row. This is the standard pattern for 2D array traversal.

### Real-World Cases

- **Matrix operations**: Addition, multiplication, transposition.
- **Chess/checkers boards**: Iterating squares by rank and file.
- **Image processing**: Iterating pixels by x and y coordinates.
- **Combination generation**: Creating all pairs from two lists.
- **Calendar rendering**: Rows for weeks, columns for days.

### References

- MDN Web Docs — Loops and iteration — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Loops_and_iteration
- Educative — Understanding JavaScript Loops for Program Flow Control — https://www.educative.io/courses/learn-html-css-javascript-from-scratch/lta/program-flow-loops
- MDN Web Docs — Labeled statement — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/label

---

## 12. Loop Optimization

### Definitions

**Core Definition**
Loop optimization is the practice of restructuring loop code to reduce execution time, memory usage, or both.

**Technical Definition**
Loop optimisation encompasses algorithmic improvements (reducing complexity), micro-optimisations (caching lengths, avoiding property lookups), and engine-aware patterns (choosing loop constructs that JIT compilers optimise well). V8's Maglev and Turboshaft compilers perform loop peeling for trivial loops (no calls, no forward jumps, under 120 bytecodes), and inline caching optimises monomorphic property access in hot loops.

**Beginner-Friendly Explanation**
Loop optimisation is about making your loops run faster. The biggest wins come from doing less work inside the loop, using the right loop type, and avoiding unnecessary calculations. The best optimisation is often to use a better algorithm instead of a faster loop.

### Purposes

- To reduce execution time for large datasets.
- To minimise memory allocations and garbage collection pressure.
- To avoid blocking the main thread in browser applications.
- To improve battery life and responsiveness on mobile devices.
- To prepare code for scaling to larger inputs.

### Syntax Rules and Structure

**Optimisation Techniques**

| Technique | Description |
|---|---|
| Cache array length | `const len = arr.length;` before the loop. |
| Reverse iteration | `for (let i = arr.length - 1; i >= 0; i--)` |
| Move invariants out | Compute constant values once before the loop. |
| Avoid property lookups | Store `obj.prop` in a local variable. |
| Use `break` early | Exit as soon as the result is found. |
| Prefer `for` over `for...in` | `for...in` is slow for arrays. |
| Use `Set`/`Map` for lookups | Replace `Array.includes` inside loops with `Set.has`. |
| Combine iterations | Merge multiple loops over the same data into one. |
| Avoid creating objects | Reuse objects or use primitives where possible. |
| Use typed arrays | For numeric data, `Float64Array` etc. reduce overhead. |

**Component Breakdown**

| Factor | Optimization |
|---|---|
| Length property | Cache in a variable. |
| Loop variable type | `let` is preferred for block scoping; engines optimise it well. |
| Function calls | Hoist constant-result calls outside the loop. |
| DOM access | Batch reads and writes; never read/write alternately. |
| Nested loops | Consider algorithmic alternatives (sorting + two-pointer). |

**Syntax Rules**

- Always measure before optimising; use `performance.now()` or the Performance API.
- Modern JavaScript engines optimise well-written code; premature micro-optimisation can be counterproductive.
- Caching `array.length` matters most for arrays with 10,000+ elements and tight loops.

**Constraints and Limitations**

- Performance is engine-dependent (V8, SpiderMonkey, JavaScriptCore).
- Optimisations that help in one engine may not help in another.
- Readability should not be sacrificed without measurable gains.

### Annotated Code Examples

**Example 1: Replacing `Array.includes` with `Set`**

```js
// ❌ Slow: O(n²) — includes scans the array for each item.
const items = [/* large array */];
const lookup = [/* large array */];
for (const item of items) {
  if (lookup.includes(item)) { // O(n) per item
    // process
  }
}

// ✅ Fast: O(n) — Set.has is O(1).
const lookupSet = new Set(lookup);
for (const item of items) {
  if (lookupSet.has(item)) { // O(1) per item
    // process
  }
}
```

**Breakdown**: `Array.includes` performs a linear scan for each item, resulting in O(n²) complexity. Converting the lookup array to a `Set` reduces each check to O(1), making the overall loop O(n).

**Example 2: Combining iterations**

```js
// ❌ Slow: Two passes over the same data.
const doubled = numbers.map(n => n * 2);
const evens = doubled.filter(n => n % 2 === 0);

// ✅ Fast: One pass with a single loop.
const evens = [];
for (const n of numbers) {
  const doubled = n * 2;
  if (doubled % 2 === 0) {
    evens.push(doubled);
  }
}
```

**Breakdown**: The first version iterates twice (map then filter), creating an intermediate array. The second version does both operations in a single pass, reducing memory allocation and iteration count.

**Example 3: Hoisting DOM access**

```js
// ❌ Slow: DOM read/write alternation causes layout thrashing.
for (let i = 0; i < items.length; i++) {
  const height = items[i].offsetHeight; // read
  items[i].style.height = height + 10 + "px"; // write
}

// ✅ Fast: Batch reads, then batch writes.
const heights = [];
for (let i = 0; i < items.length; i++) {
  heights.push(items[i].offsetHeight); // read
}
for (let i = 0; i < items.length; i++) {
  items[i].style.height = heights[i] + 10 + "px"; // write
}
```

**Breakdown**: Alternating DOM reads and writes forces the browser to recalculate layout repeatedly. Separating reads from writes eliminates layout thrashing.

### Real-World Cases

- **Large data processing**: ETL pipelines in Node.js.
- **Frontend rendering**: Virtual DOM diffing, list virtualisation.
- **Game engines**: Entity-component-system updates.
- **Financial modelling**: Monte Carlo simulations.
- **Machine learning**: Matrix operations in TensorFlow.js.

### References

- CoreUI — How to optimize loop performance in JavaScript — https://coreui.io/answers/how-to-optimize-loop-performance-in-javascript/
- PatternsDev — JavaScript Performance Patterns — https://github.com/PatternsDev/skills/blob/main/skills/javascript/js-performance-patterns/SKILL.md
- V8 Blog — https://v8.dev/blog
- WebKit Bug 283118 — `for-of` loops noticeably slower than alternatives — https://wiki.webkit.org/show_bug.cgi?id=283118

---

## Summary Table of References

| Resource | URL |
|---|---|
| MDN — `for` | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for |
| MDN — `for...in` | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in |
| MDN — `for...of` | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of |
| MDN — `while` | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/while |
| MDN — `do...while` | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/do...while |
| MDN — `break` | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/break |
| MDN — `continue` | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/continue |
| MDN — Labeled statement | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/label |
| MDN — Loops and iteration | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Loops_and_iteration |
| MDN — Iteration protocols | https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols |
| ECMAScript — The `for` Statement | https://tc39.es/ecma262/#sec-for-statement |
| ECMAScript — `for-in`/`for-of` | https://tc39.es/ecma262/#sec-for-in-and-for-of-statements |
| ECMAScript — `while` Statement | https://tc39.es/ecma262/#sec-while-statement |
| ECMAScript — `do-while` Statement | https://tc39.es/ecma262/#sec-do-while-statement |
| ECMAScript — `break` Statement | https://tc39.es/ecma262/#sec-break-statement |
| ECMAScript — `continue` Statement | https://tc39.es/ecma262/#sec-continue-statement |
| CoreUI — Loop Performance | https://coreui.io/answers/how-to-optimize-loop-performance-in-javascript/ |
| WebKit Bug 283118 — `for-of` Performance | https://wiki.webkit.org/show_bug.cgi?id=283118 |
| V8 Blog | https://v8.dev/blog |
| PatternsDev — JS Performance Patterns | https://github.com/PatternsDev/skills/blob/main/skills/javascript/js-performance-patterns/SKILL.md |
| Educative — Understanding JavaScript Loops | https://www.educative.io/courses/learn-html-css-javascript-from-scratch/lta/program-flow-loops |