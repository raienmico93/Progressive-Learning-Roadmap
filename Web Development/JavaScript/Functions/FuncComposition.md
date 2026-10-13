# Function Composition — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Function composition is the process of combining two or more functions to produce a new function, where the output of one function becomes the input of the next. It is a fundamental concept in functional programming and mathematics.

**Technical Definition:** In mathematics, function composition is defined as `(f ∘ g)(x) = f(g(x))`, meaning `g` is applied to `x` first, and then `f` is applied to the result. Per the Educative tutorial, "Function composition is an approach where the result of one function is passed on to the next function, which is passed to another until the final function is executed for the final result". In JavaScript, composition is implemented through higher-order functions — functions that take other functions as arguments and return a new function. The two primary utilities are `compose` (right-to-left) and `pipe` (left-to-right).

**Beginner-Friendly Explanation:** Function composition is like an assembly line. You start with raw material (data), pass it through one machine (function) that transforms it, then pass the result to the next machine, and so on, until you get the finished product. For example, you could compose a `trim` function and a `capitalize` function to create a `formatName` function that does both. Composition lets you build complex operations from simple, reusable pieces.

### Key Characteristics

- **Right-to-left or left-to-right:** `compose` applies functions right-to-left; `pipe` applies them left-to-right.
- **Higher-order function:** Composition utilities take functions as arguments and return a new function.
- **Unary constraint:** Except for the first (or last) function, all composed functions should be unary (accept one argument).
- **Point-free friendly:** Composition naturally supports point-free style, where function parameters are omitted.
- **Composability:** Small, pure functions combine cleanly to build complex behaviour.
- **Mathematical foundation:** Based on the mathematical concept of function composition `(f ∘ g)(x) = f(g(x))`.

### Prerequisites

- Understanding of functions (declarations, expressions, arrow functions).
- Familiarity with higher-order functions and callbacks.
- Basic knowledge of pure functions and side effects.
- Awareness of currying and partial application (helpful for point-free style).

### Related Programming Areas

- **Functional programming:** Composition is a core tenet of FP.
- **Data transformation pipelines:** Processing data through a series of steps.
- **Middleware:** Express.js middleware chains are a form of composition.
- **Redux:** Reducers and middleware use composition.
- **Ramda and Lodash/fp:** Libraries that provide `compose` and `pipe` utilities.

### Core Concepts / Features

---

## Composition

### Definitions

**Core Definition:** Composition is the act of combining two or more functions so that the output of one function becomes the input of the next, creating a new function.

**Technical Definition:** In functional programming, "function composition is the process of combining multiple functions to produce a new function". The composed function applies the functions in sequence, passing the result of each function to the next. In JavaScript, composition is achieved through higher-order functions that reduce over an array of functions, applying them one after another. The mathematical notation `(f ∘ g)(x) = f(g(x))` describes the relationship: `g` is applied first, then `f`.

**Beginner-Friendly Explanation:** Composition means chaining functions together. If you have a function that doubles a number and another that adds 5, you can compose them into a new function that doubles and then adds 5. Instead of writing `add5(double(x))`, you can create a composed function and call it like `doubleThenAdd5(x)`.

### Purposes

- To build complex operations from simple, reusable functions.
- To improve code readability by eliminating deeply nested function calls.
- To enable declarative data transformation pipelines.
- To facilitate point-free programming style.
- To support modular and testable code design.

### Syntax Rules and Structure

Composition is not a built-in JavaScript operator; it is implemented via higher-order functions:

```js
// Manual composition:
const composed = (x) => f(g(x));

// With compose utility:
const composed = compose(f, g);
composed(x); // f(g(x))
```

**Component Breakdown:**
- `f`, `g`: The functions to compose.
- `compose`: A higher-order function that returns a new function.
- The rightmost function (`g`) is applied first; its result is passed to `f`.

**Syntax Rules:**
- All functions except the first (rightmost in `compose`) must be unary.
- The composed function accepts the same arguments as the first function in the chain.
- Composition works best with pure functions (no side effects).

**Constraints and Limitations:**
- Functions with multiple arguments cannot be composed directly without currying or partial application.
- Debugging composed functions can be harder because stack traces are less direct.
- Over-composition can lead to overly abstract code.

### Annotated Code Examples

#### Example 1: Manual Composition

```js
// Step 1: Define two simple functions.
const double = (x) => x * 2;
const addFive = (x) => x + 5;

// Step 2: Compose manually.
const doubleThenAddFive = (x) => addFive(double(x));

// Step 3: Use the composed function.
console.log(doubleThenAddFive(3)); // 11 (3 * 2 + 5)
```

**Expected Output:**
```
11
```

**Why This Output:** `double(3)` returns `6`, then `addFive(6)` returns `11`. The composed function chains the two operations together.

#### Example 2: Composition with a Utility

```js
// Step 1: Define a compose utility.
const compose = (...fns) => (x) =>
  fns.reduceRight((acc, fn) => fn(acc), x);

// Step 2: Define functions.
const trim = (str) => str.trim();
const toLowerCase = (str) => str.toLowerCase();
const capitalize = (str) => str.charAt(0).toUpperCase() + str.slice(1);

// Step 3: Compose them.
const formatName = compose(capitalize, toLowerCase, trim);

// Step 4: Use the composed function.
console.log(formatName("  ALICE  ")); // "Alice"
```

**Expected Output:**
```
Alice
```

**Why This Output:** `compose` applies functions right-to-left: `trim` first, then `toLowerCase`, then `capitalize`. The result is a clean, formatted name.

### Real-World Cases

- **Data validation:** Composing `trim`, `checkLength`, and `checkFormat` into a single validator.
- **API response processing:** Chaining `parseJSON`, `extractData`, and `formatOutput`.
- **Redux middleware:** `applyMiddleware(logger, thunk, promise)` composes middleware functions.

---

## `compose`

### Definitions

**Core Definition:** `compose` is a higher-order function that combines multiple functions from right to left, returning a new function that applies them in sequence.

**Technical Definition:** Per the 30-seconds-of-code reference, "Performs right-to-left function composition. Use `Array.prototype.reduce()` to perform right-to-left function composition. The last (rightmost) function can accept one or more arguments; the remaining functions must be unary". The implementation uses `reduceRight` to apply functions in reverse order:

```js
const compose = (...fns) => fns.reduce((f, g) => (...args) => f(g(...args)));
```

**Beginner-Friendly Explanation:** `compose` is a utility that takes several functions and combines them into one. It applies the **rightmost** function first, then passes the result to the next function to its left, and so on. For example, `compose(add5, multiply)` creates a function that multiplies first, then adds 5.

### Purposes

- To create a single function from a chain of right-to-left operations.
- To support mathematical composition notation `(f ∘ g)(x)`.
- To enable point-free style by eliminating intermediate variables.
- To build reusable transformation pipelines.

### Syntax Rules and Structure

```js
const compose = (...fns) => fns.reduce((f, g) => (...args) => f(g(...args)));

// Usage:
const composed = compose(fn1, fn2, fn3);
composed(value); // fn1(fn2(fn3(value)))
```

**Component Breakdown:**
- `...fns`: A rest parameter collecting all functions to compose.
- `reduce`: Combines functions into a single function.
- `f(g(...args))`: The composition: `g` is applied first, then `f`.

**Syntax Rules:**
- Functions are applied from right to left.
- The rightmost function can accept multiple arguments.
- All other functions must be unary.

**Constraints and Limitations:**
- The order of application is reversed compared to `pipe`.
- Debugging can be confusing because the execution order is opposite to the reading order.

### Annotated Code Examples

#### Example 1: Basic `compose`

```js
// Step 1: Define compose.
const compose = (...fns) => fns.reduce((f, g) => (...args) => f(g(...args)));

// Step 2: Define functions.
const add5 = (x) => x + 5;
const multiply = (x, y) => x * y;

// Step 3: Compose.
const multiplyAndAdd5 = compose(add5, multiply);

// Step 4: Use.
console.log(multiplyAndAdd5(5, 2)); // 15 (5 * 2 + 5)
```

**Expected Output:**
```
15
```

**Why This Output:** `compose(add5, multiply)` applies `multiply` first (rightmost), then `add5`. `multiply(5, 2)` is `10`, then `add5(10)` is `15`.

#### Example 2: `compose` with String Functions

```js
const compose = (...fns) => fns.reduce((f, g) => (...args) => f(g(...args)));

const trim = (s) => s.trim();
const toLowerCase = (s) => s.toLowerCase();
const split = (s) => s.split(" ");

const process = compose(split, toLowerCase, trim);
console.log(process("  Hello World  ")); // ["hello", "world"]
```

**Expected Output:**
```
[ 'hello', 'world' ]
```

**Why This Output:** `trim` removes spaces, `toLowerCase` converts to lowercase, `split` divides into an array. The functions are applied right-to-left.

### Real-World Cases

- **Ramda:** `R.compose` is a core utility in the Ramda library.
- **Redux:** `compose` is used to combine store enhancers.
- **Functional utilities:** Lodash/fp provides `_.compose`.

---

## `pipe`

### Definitions

**Core Definition:** `pipe` is a higher-order function that combines multiple functions from left to right, returning a new function that applies them in sequence.

**Technical Definition:** Per the Educative tutorial, "we can reverse the order of the function invocation by using the `pipe` function". The implementation is identical to `compose` but uses `reduce` instead of `reduceRight`:

```js
const pipe = (...fns) => (x) => fns.reduce((acc, fn) => fn(acc), x);
```

Per the functional pipeline reference, "pipe is a function with arity n that the first argument is a value and the rest are functions that will be applied to the value in sequence".

**Beginner-Friendly Explanation:** `pipe` is like `compose`, but it applies functions from **left to right** — the same direction you read. If you write `pipe(trim, toLowerCase, capitalize)`, it trims first, then lowercases, then capitalises. This is more intuitive for many developers because the execution order matches the reading order.

### Purposes

- To create a left-to-right function pipeline that matches reading order.
- To improve readability by making execution order explicit.
- To support data transformation pipelines in a natural order.
- To enable point-free style with intuitive ordering.

### Syntax Rules and Structure

```js
const pipe = (...fns) => (x) => fns.reduce((acc, fn) => fn(acc), x);

// Usage:
const piped = pipe(fn1, fn2, fn3);
piped(value); // fn3(fn2(fn1(value)))
```

**Component Breakdown:**
- `...fns`: The functions to pipe.
- `reduce`: Applies functions left-to-right.
- `acc`: The accumulator carrying the result.

**Syntax Rules:**
- Functions are applied left to right.
- The leftmost function can accept multiple arguments.
- All other functions must be unary.

**Constraints and Limitations:**
- `pipe` is not a built-in JavaScript method; it must be implemented or imported.
- A pipeline operator (`|>`) is proposed for JavaScript but not yet standard.

### Annotated Code Examples

#### Example 1: Basic `pipe`

```js
// Step 1: Define pipe.
const pipe = (...fns) => (x) => fns.reduce((acc, fn) => fn(acc), x);

// Step 2: Define functions.
const double = (x) => x * 2;
const square = (x) => x * x;

// Step 3: Pipe.
const doubleThenSquare = pipe(double, square);

// Step 4: Use.
console.log(doubleThenSquare(3)); // 36 (3 * 2 = 6, 6 * 6 = 36)
```

**Expected Output:**
```
36
```

**Why This Output:** `pipe(double, square)` applies `double` first (leftmost), then `square`. `double(3)` is `6`, then `square(6)` is `36`.

#### Example 2: `pipe` vs `compose`

```js
const compose = (...fns) => (x) => fns.reduceRight((acc, fn) => fn(acc), x);
const pipe = (...fns) => (x) => fns.reduce((acc, fn) => fn(acc), x);

const double = (x) => x * 2;
const square = (x) => x * x;

console.log(compose(square, double)(3)); // 36 (double first, then square)
console.log(pipe(double, square)(3));    // 36 (double first, then square)
```

**Expected Output:**
```
36
36
```

**Why This Output:** Both produce the same result, but the argument order is reversed. `compose(square, double)` reads as "square of double", while `pipe(double, square)` reads as "double then square".

### Real-World Cases

- **Data processing:** Piping data through `filter`, `map`, and `reduce`.
- **RxJS:** The `pipe` method chains operators in a readable order.
- **Node.js streams:** `.pipe()` connects readable and writable streams.

---

## Point-Free Style

### Definitions

**Core Definition:** Point-free style (also called tacit programming) is a programming style in which function definitions do not explicitly mention their arguments. Instead, functions are defined through composition, currying, and other higher-order techniques.

**Technical Definition:** Per Functional-Light JavaScript, "The *point* of point-free is to use existing utilities and well-known patterns to reduce the clutter of our programs. We do this by reducing/eliminating places where a function parameter ('point') passes through to an inner function call". In point-free style, the focus is on **what** transformation is applied, not on the data being transformed. This is enabled by currying and higher-order functions, and it is closely associated with function composition.

**Beginner-Friendly Explanation:** Point-free style means you don't name the arguments of your functions. Instead of writing `const double = (x) => x * 2` and then `const process = (x) => double(x)`, you just write `const process = double`. The "point" (the argument) is eliminated. This makes code shorter and more declarative, but it can be harder to read for developers unfamiliar with the style.

### Purposes

- To reduce visual clutter by eliminating unnecessary parameter names.
- To make code more declarative and focused on transformation.
- To enable cleaner function composition.
- To support equational reasoning and readability in functional code.

### Syntax Rules and Structure

```js
// Pointed (explicit arguments):
const double = (x) => x * 2;
const process = (x) => double(x); // The point (x) is redundant

// Point-free:
const double = (x) => x * 2;
const process = double; // No explicit point
```

**Component Breakdown:**
- **Pointed style:** Arguments are named and passed explicitly.
- **Point-free style:** Arguments are omitted; functions are composed directly.

**Syntax Rules:**
- Point-free style requires functions to be curried or unary.
- It works best with composition utilities like `compose` and `pipe`.
- The style is not mandatory; it should be used when it improves clarity.

**Constraints and Limitations:**
- Point-free code can be harder to debug because stack traces may not show argument values.
- Overuse can make code cryptic, especially for developers new to functional programming.
- Not all functions can be written point-free.

### Annotated Code Examples

#### Example 1: Pointed vs Point-Free

```js
// Pointed style:
const users = [{ name: "Alice" }, { name: "Bob" }];
const namesPointed = users.map((user) => user.name);

// Point-free style:
const prop = (key) => (obj) => obj[key];
const namesPointFree = users.map(prop("name"));

console.log(namesPointed);   // ["Alice", "Bob"]
console.log(namesPointFree); // ["Alice", "Bob"]
```

**Expected Output:**
```
[ 'Alice', 'Bob' ]
[ 'Alice', 'Bob' ]
```

**Why This Output:** The pointed version explicitly names `user`. The point-free version uses `prop("name")`, a curried function that extracts the `name` property, eliminating the need to name the parameter.

#### Example 2: Point-Free Composition

```js
const compose = (...fns) => (x) => fns.reduceRight((acc, fn) => fn(acc), x);

const trim = (s) => s.trim();
const toLowerCase = (s) => s.toLowerCase();

// Pointed:
const formatPointed = (str) => toLowerCase(trim(str));

// Point-free:
const formatPointFree = compose(toLowerCase, trim);

console.log(formatPointed("  HELLO  "));   // "hello"
console.log(formatPointFree("  HELLO  ")); // "hello"
```

**Expected Output:**
```
hello
hello
```

**Why This Output:** The point-free version eliminates the `str` parameter by composing the functions directly.

### Real-World Cases

- **Ramda:** Ramda is designed for point-free style with auto-curried, data-last functions.
- **Lodash/fp:** Provides a functional programming variant of Lodash that supports point-free style.
- **Pipeline processing:** Data transformation pipelines often use point-free composition.

---

## Function Pipelines

### Definitions

**Core Definition:** A function pipeline is a series of functions connected in sequence, where the output of each function becomes the input of the next, similar to a Unix pipe (`|`).

**Technical Definition:** Per the function-pipeline package description, "Similar to the Unix `|` operator; returns a function that invokes the given series of functions whose output is subsequently passed to the next function in the series". In JavaScript, pipelines are implemented using `pipe` or `compose` utilities, or through the proposed pipeline operator (`|>`). Pipelines are a practical application of function composition, often used for data transformation and middleware chains.

**Beginner-Friendly Explanation:** A function pipeline is like an assembly line: data enters at one end, passes through a series of stations (functions), each doing one small job, and emerges at the other end transformed. Pipelines make complex transformations readable by breaking them into simple steps.

### Purposes

- To process data through a series of well-defined steps.
- To make data transformation logic explicit and readable.
- To support middleware architectures (Express, Redux).
- To enable declarative, composable data flows.

### Syntax Rules and Structure

```js
// With pipe:
const pipeline = pipe(step1, step2, step3);
pipeline(data); // step3(step2(step1(data)))

// With Unix-style pipe (conceptual):
// data | step1 | step2 | step3
```

**Component Breakdown:**
- Each function in the pipeline is a **stage** or **step**.
- The output of each stage is the input to the next.
- The pipeline itself is a new function that encapsulates the entire sequence.

**Syntax Rules:**
- Pipelines are built with `pipe` (left-to-right) or `compose` (right-to-left).
- Each stage should be unary (accept one argument).
- Stages should be pure functions for predictable behaviour.

**Constraints and Limitations:**
- Pipelines are not a native JavaScript feature (the pipeline operator is still a proposal).
- Error handling in pipelines requires additional mechanisms.
- Debugging pipelines can be difficult because intermediate values are hidden.

### Annotated Code Examples

#### Example 1: Data Transformation Pipeline

```js
// Step 1: Define pipeline stages.
const parse = (str) => str.split(",");
const trimAll = (arr) => arr.map((s) => s.trim());
const filterEmpty = (arr) => arr.filter((s) => s.length > 0);
const toNumbers = (arr) => arr.map(Number);

// Step 2: Create the pipeline.
const pipe = (...fns) => (x) => fns.reduce((acc, fn) => fn(acc), x);
const processCSV = pipe(parse, trimAll, filterEmpty, toNumbers);

// Step 3: Use the pipeline.
console.log(processCSV("1, 2, , 3, 4 "));
// [1, 2, 3, 4]
```

**Expected Output:**
```
[ 1, 2, 3, 4 ]
```

**Why This Output:** The pipeline parses the string into an array, trims each element, removes empty strings, and converts the remaining strings to numbers.

#### Example 2: Middleware Pipeline

```js
// Step 1: Define middleware functions.
const logger = (ctx, next) => {
  console.log(`Request: ${ctx.path}`);
  next();
  console.log(`Response: ${ctx.status}`);
};

const auth = (ctx, next) => {
  if (!ctx.user) {
    ctx.status = 401;
    return;
  }
  next();
};

const handler = (ctx) => {
  ctx.status = 200;
  ctx.body = "Hello, World!";
};

// Step 2: Create a pipeline (simplified).
const pipeline = (ctx) => {
  logger(ctx, () => auth(ctx, () => handler(ctx)));
};

// Step 3: Run.
const ctx = { path: "/api", user: "Alice" };
pipeline(ctx);
console.log(ctx.body); // "Hello, World!"
```

**Expected Output:**
```
Request: /api
Response: 200
Hello, World!
```

**Why This Output:** The middleware functions are composed into a pipeline. Each middleware can modify the context and decide whether to call the next function.

### Real-World Cases

- **Express.js middleware:** Request processing goes through a pipeline of middleware functions.
- **RxJS operators:** The `pipe` method chains observable operators.
- **Data ETL:** Extract, transform, load pipelines in data processing.
- **Redux middleware:** Actions pass through a middleware pipeline before reaching reducers.

---

## Composition vs Inheritance

### Definitions

**Core Definition:** Composition and inheritance are two fundamental approaches to code reuse and object design. Inheritance builds objects by extending a parent class (a "is-a" relationship), while composition builds objects by combining smaller, independent components (a "has-a" relationship).

**Technical Definition:** Per the Mastering Modular JavaScript reference, "Composition is an alternative to inheritance. Rather than building objects by vertically stacking functionality, composition relies on stringing together orthogonal aspects of functionality". Per Stack Overflow, "Composition is simply when a class is composed of other classes; or to say it another way, an instance of an object has references to instances of other objects. Inheritance is when a class inherits methods and properties from another class".

**Beginner-Friendly Explanation:** Inheritance is like a family tree: a `Car` inherits from `Vehicle`, so it gets all of `Vehicle`'s features and can add its own. Composition is like building with Lego blocks: instead of inheriting from a `Vehicle`, a `Car` is built by combining an `Engine`, `Wheels`, and `Chassis`. Composition is often more flexible because you can mix and match components freely.

### Purposes

- To choose the right approach for code reuse in a given context.
- To avoid the fragile base class problem associated with deep inheritance hierarchies.
- To enable flexible, modular design through composition.
- To understand when inheritance is appropriate (clear "is-a" relationships).
- To leverage functional composition for data transformation.

### Syntax Rules and Structure

```js
// Inheritance (ES6 class):
class Animal {
  constructor(name) { this.name = name; }
  speak() { return `${this.name} makes a sound.`; }
}
class Dog extends Animal {
  speak() { return `${this.name} barks.`; }
}

// Composition (function-based):
const canSpeak = (state) => ({
  speak: () => `${state.name} makes a sound.`,
});
const createDog = (name) => {
  const state = { name };
  return { ...canSpeak(state), bark: () => `${name} barks.` };
};
```

**Component Breakdown:**
- **Inheritance:** `extends` keyword; child class inherits methods and properties.
- **Composition:** Objects are built by mixing in behaviours from smaller modules.

**Syntax Rules:**
- Inheritance uses `class ... extends ...` or prototype chains.
- Composition uses `Object.assign`, spread syntax, or factory functions to combine behaviours.
- Functional composition (as described in this cheat sheet) is a form of composition for functions.

**Constraints and Limitations:**
- Inheritance creates tight coupling between parent and child; changes to the parent can break children.
- Composition is more flexible but can lead to more verbose code.
- JavaScript supports single inheritance (via prototypes); composition has no such limit.

### Annotated Code Examples

#### Example 1: Inheritance Approach

```js
// Step 1: Define a base class.
class Vehicle {
  constructor(make, model) {
    this.make = make;
    this.model = model;
  }
  start() {
    return `${this.make} ${this.model} is starting.`;
  }
}

// Step 2: Extend the base class.
class Car extends Vehicle {
  constructor(make, model, doors) {
    super(make, model);
    this.doors = doors;
  }
  honk() {
    return "Beep!";
  }
}

// Step 3: Use.
const car = new Car("Toyota", "Camry", 4);
console.log(car.start()); // "Toyota Camry is starting."
console.log(car.honk());  // "Beep!"
```

**Expected Output:**
```
Toyota Camry is starting.
Beep!
```

**Why This Output:** `Car` inherits `start()` from `Vehicle` and adds its own `honk()` method. The "is-a" relationship: a `Car` is a `Vehicle`.

#### Example 2: Composition Approach

```js
// Step 1: Define behaviour modules.
const canStart = (state) => ({
  start: () => `${state.make} ${state.model} is starting.`,
});

const canHonk = () => ({
  honk: () => "Beep!",
});

// Step 2: Compose a car.
const createCar = (make, model, doors) => {
  const state = { make, model, doors };
  return {
    ...canStart(state),
    ...canHonk(),
    doors,
  };
};

// Step 3: Use.
const car = createCar("Toyota", "Camry", 4);
console.log(car.start()); // "Toyota Camry is starting."
console.log(car.honk());  // "Beep!"
```

**Expected Output:**
```
Toyota Camry is starting.
Beep!
```

**Why This Output:** The `createCar` function composes behaviour from `canStart` and `canHonk`. The "has-a" relationship: a `Car` has a `start` behaviour and a `honk` behaviour.

### Real-World Cases

- **React:** Recommends composition over inheritance for component reuse.
- **Redux:** Uses function composition for middleware and reducers.
- **Game development:** Entity-component systems favour composition over deep inheritance.
- **Functional programming:** Composition is the primary mechanism for building complex behaviour.

---

## References

- MDN Web Docs – Function composition (30-seconds-of-code) — https://github.com/ConardLi/30-seconds-of-code/blob/master/snippets/compose.md
- Wikipedia – Function composition (computer science) — https://en.wikipedia.org/wiki/Function_composition_(computer_science)
- Educative – Function composition in JavaScript — https://www.educative.io/answers/function-composition-in-javascript
- Functional-Light JavaScript (Kyle Simpson) – Chapter 4: Composing Functions — https://github.com/getify/Functional-Light-JS
- Ramda Documentation – compose — https://ramdajs.com/docs/#compose
- Ramda Documentation – pipe — https://ramdajs.com/docs/#pipe
- Mastering Modular JavaScript – Composition vs Inheritance — https://github.com/mjavascript/mastering-modular-javascript
- MDN Web Docs – Function — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function
- TC39 – Pipeline Operator Proposal — https://github.com/tc39/proposal-pipeline-operator
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf