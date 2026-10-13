# `this` Binding — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** `this` is a special keyword in JavaScript that refers to the **execution context** of the currently executing function. Its value is determined by **how a function is called**, not where it is defined. Per MDN, "JavaScript에서 `this`의 값은 함수가 정의 시점이 아니라 함수가 어떤 방식으로 호출(binding) 되었는지에 따라 달라집니다".

**Technical Definition:** In ECMAScript, `this` is a binding that is set when a function is invoked. The value of `this` depends on the **call-site** — the location in code where the function is called. JavaScript defines four binding rules (default, implicit, explicit, and `new` binding) with a strict order of precedence. Arrow functions are the exception: they do not have their own `this` binding; they inherit `this` from the enclosing lexical scope.

**Beginner-Friendly Explanation:** `this` is a way for a function to know "who called me." When you call a method on an object like `user.sayHi()`, `this` inside `sayHi` refers to `user`. But if you call the same function without an object, like `sayHi()`, `this` becomes the global object (or `undefined` in strict mode). The value of `this` is not fixed — it changes depending on how the function is called. The only exception is arrow functions, which always use the `this` from where they were written.

### Key Characteristics

- **Dynamic binding:** `this` is determined at **call time**, not at definition time.
- **Four binding rules:** Default, Implicit, Explicit, and `new` binding — each with different precedence.
- **Arrow functions are different:** They have no own `this`; they use lexical `this`.
- **Strict mode matters:** In strict mode, default binding is `undefined`; in non-strict mode, it is the global object.
- **`call`, `apply`, `bind`:** These methods allow explicit control over `this`.
- **Precedence:** `new` binding > Explicit binding > Implicit binding > Default binding.

### Prerequisites

- Understanding of JavaScript functions (declarations, expressions, arrow functions).
- Familiarity with objects and methods.
- Basic knowledge of scope and execution contexts.
- Awareness of strict mode (`"use strict"`).

### Related Programming Areas

- **Object-oriented programming:** Methods and constructors rely on `this`.
- **Event-driven programming:** Event handlers have special `this` rules.
- **Functional programming:** Arrow functions and `bind` are used to manage `this` in callbacks.
- **Framework development:** React class components, Vue methods, and Angular services all depend on `this` binding.

### Core Concepts / Features

---

## `this` in Global Scope

### Definitions

**Core Definition:** In the global scope (outside any function), `this` refers to the global object — `window` in browsers and `global` in Node.js.

**Technical Definition:** Per MDN, "在函数或对象上下文之外，`this` 是指 `globalThis` 属性，它是指向大多数 JavaScript 环境中的全局对象的引用". In browsers, `globalThis` is `window`; in Node.js, it is `global`. In strict mode, the global `this` is still the global object (strict mode only affects `this` inside functions).

**Beginner-Friendly Explanation:** When you use `this` outside of any function, it refers to the global object — the "main container" of your JavaScript environment. In a browser, that's `window`. In Node.js, that's `global`. You can think of it as the top-level object that everything else lives in.

### Purposes

- To access global variables and functions from the global scope.
- To provide a reference to the global object for environment detection.
- To serve as the default binding when no other rule applies.

### Syntax Rules and Structure

```js
// Browser:
console.log(this === window); // true

// Node.js:
console.log(this === global); // true

// Universal:
console.log(this === globalThis); // true (ES2020+)
```

**Component Breakdown:**
- `this`: The global object reference.
- `globalThis`: A standardised alias for the global object (ES2020).

**Syntax Rules:**
- In the global scope, `this` is always the global object, regardless of strict mode.
- Inside a function called without a context, `this` depends on strict mode (see Default Binding).

**Constraints and Limitations:**
- In ES modules, `this` at the top level is `undefined`, not the global object.
- Relying on global `this` is generally discouraged in favour of explicit imports and modules.

### Annotated Code Examples

#### Example 1: Global `this` in Browser and Node.js

```js
// Step 1: Check the global object in a browser.
console.log(this === window); // true (in browser)

// Step 2: Check the global object in Node.js.
console.log(this === global); // true (in Node.js)

// Step 3: Use globalThis for cross-environment compatibility.
console.log(this === globalThis); // true (in both)
```

**Expected Output (browser):**
```
true
false
true
```

**Why This Output:** In a browser, `this` is `window`. In Node.js, `this` is `global`. The `globalThis` property is a standardised alias that works in all environments.

### Real-World Cases

- **Environment detection:** `const isBrowser = this === window;`
- **Polyfills:** Accessing `globalThis` for feature detection.
- **Legacy code:** Some older libraries rely on global `this`.

---

## `this` in Functions (Default Binding)

### Definitions

**Core Definition:** When a function is called as a standalone function (not as a method, not with `call`/`apply`/`bind`, and not with `new`), `this` follows the **default binding** rule.

**Technical Definition:** Per web.dev, "在严格模式之外，`this` 还会引用独立函数内的全局对象... 使用严格模式时，`this` 在独立函数内的值为 `undefined`". This is the **lowest priority** rule in the `this` binding precedence chain.

**Beginner-Friendly Explanation:** When you call a function by itself — like `myFunction()` — `this` doesn't have an object to refer to. In non-strict mode, it defaults to the global object (`window` or `global`). In strict mode, it becomes `undefined`. This is the "fallback" rule when nothing else applies.

### Purposes

- To provide a predictable fallback when no context is provided.
- To detect whether a function was called as a method or standalone.
- To enforce stricter behaviour in strict mode.

### Syntax Rules and Structure

```js
function standalone() {
  console.log(this);
}

// Non-strict mode:
standalone(); // Window (or global)

// Strict mode:
function strictStandalone() {
  "use strict";
  console.log(this); // undefined
}
strictStandalone();
```

**Component Breakdown:**
- `standalone()`: A function call without an object context.
- In non-strict mode: `this` → global object.
- In strict mode: `this` → `undefined`.

**Syntax Rules:**
- Default binding applies when no other rule matches.
- Strict mode is enabled per-function or per-script with `"use strict"`.
- The `this` value is **not** coerced to an object in strict mode.

**Constraints and Limitations:**
- Default binding is the most error-prone rule because `this` may unexpectedly be the global object.
- Strict mode makes the behaviour safer by returning `undefined`.

### Annotated Code Examples

#### Example 1: Default Binding in Non-Strict vs Strict Mode

```js
// Step 1: Non-strict function.
function nonStrict() {
  console.log("Non-strict this:", this);
}

// Step 2: Strict function.
function strict() {
  "use strict";
  console.log("Strict this:", this);
}

// Step 3: Call both as standalone functions.
nonStrict(); // Window (or global)
strict();    // undefined
```

**Expected Output:**
```
Non-strict this: [object Window] (or [object global])
Strict this: undefined
```

**Why This Output:** In non-strict mode, default binding sets `this` to the global object. In strict mode, `this` remains `undefined` because the coercion to the global object is removed.

### Real-World Cases

- **Utility functions:** Functions that don't rely on `this` should be written in strict mode to avoid accidental global access.
- **Legacy code:** Non-strict default binding is a common source of bugs in older codebases.

---

## `this` in Methods (Implicit Binding)

### Definitions

**Core Definition:** When a function is called as a method of an object (e.g., `obj.method()`), `this` refers to the object that owns the method. This is **implicit binding**.

**Technical Definition:** Per MDN, "일반 함수가 객체의 메서드로 호출될 때(`obj.method()`), `this`는 해당 객체를 가리킵니다". The binding is "implicit" because you don't explicitly specify `this` — it is determined by the dot notation at the call site.

**Beginner-Friendly Explanation:** When you call a function using dot notation — like `user.sayHello()` — `this` inside `sayHello` refers to `user`. The object "before the dot" becomes `this`. This is the most common and intuitive way `this` works.

### Purposes

- To allow methods to access and modify the properties of their owning object.
- To enable code reuse across different objects with the same methods.
- To provide natural object-oriented behaviour.

### Syntax Rules and Structure

```js
const obj = {
  value: 42,
  getValue() {
    return this.value; // this === obj
  }
};

console.log(obj.getValue()); // 42
```

**Component Breakdown:**
- `obj.getValue()`: The call site with `obj` before the dot.
- `this` inside `getValue` is bound to `obj`.
- The method can access `this.value` (which is `42`).

**Syntax Rules:**
- Implicit binding applies when a function is called as a property of an object.
- The object immediately preceding the dot becomes `this`.
- Only the **last** object in a chain matters: `a.b.c.method()` → `this` is `a.b.c`.

**Constraints and Limitations:**
- Implicit binding is **lost** when a method is extracted from its object (see `this` Pitfalls).
- Nested objects don't "inherit" `this`: `a.b.method()` binds `this` to `b`, not `a`.

### Annotated Code Examples

#### Example 1: Basic Implicit Binding

```js
// Step 1: Create an object with a method.
const person = {
  name: "Alice",
  greet() {
    return `Hello, I'm ${this.name}`;
  },
};

// Step 2: Call the method.
console.log(person.greet()); // "Hello, I'm Alice"
```

**Expected Output:**
```
Hello, I'm Alice
```

**Why This Output:** `person.greet()` uses implicit binding. `this` inside `greet` is bound to `person`, so `this.name` is `"Alice"`.

#### Example 2: Nested Object Binding

```js
// Step 1: Create a nested object.
const company = {
  name: "TechCorp",
  department: {
    name: "Engineering",
    getName() {
      return this.name;
    },
  },
};

// Step 2: Call the nested method.
console.log(company.department.getName()); // "Engineering"
```

**Expected Output:**
```
Engineering
```

**Why This Output:** The call site is `company.department.getName()`. The object immediately before the dot is `department`, so `this` is `department`. `this.name` is `"Engineering"`, not `"TechCorp"`.

### Real-World Cases

- **Class methods:** Methods in ES6 classes use implicit binding when called on instances.
- **API objects:** Objects that encapsulate data and behaviour (e.g., `user.save()`, `cart.addItem()`).
- **DOM elements:** Methods like `element.addEventListener()` bind `this` to the element.

---

## `this` in Arrow Functions

### Definitions

**Core Definition:** Arrow functions do **not** have their own `this` binding. Instead, `this` inside an arrow function is inherited from the **enclosing lexical scope** — the scope where the arrow function was defined.

**Technical Definition:** Per MDN, "화살표 함수는 정의될 때의 상위 스코프로부터 `this`를 상속받습니다. ... 화살표 함수는 자체적인 `this` 바인딩을 갖지 않습니다. 따라서 `bind()`, `apply()`, `call()` 메서드로 `this` 값을 설정할 수 없으며, 객체 메서드 안에서 현재 객체를 가리키지도 않습니다". Arrow functions do not have a `[[ThisMode]]` of `lexical`; they resolve `this` through the scope chain.

**Beginner-Friendly Explanation:** Arrow functions don't get their own `this`. Instead, they use the `this` from wherever they were written. If you write an arrow function inside a method, `this` inside the arrow function is the same as `this` inside the method. This makes arrow functions perfect for callbacks where you want to keep the outer `this`.

### Purposes

- To preserve the outer `this` in callbacks and asynchronous operations.
- To avoid the need for `.bind(this)` or `const self = this`.
- To make `this` behaviour predictable and lexical.
- To simplify event handlers and promise chains.

### Syntax Rules and Structure

```js
const obj = {
  value: 42,
  getValue: function() {
    return () => this.value; // 'this' is obj
  }
};

const fn = obj.getValue();
console.log(fn()); // 42
```

**Component Breakdown:**
- `getValue`: A regular function; `this` inside it is `obj`.
- The arrow function captures `this` from `getValue` (which is `obj`).
- Even when the arrow function is called later, `this` remains `obj`.

**Syntax Rules:**
- Arrow functions have no own `this`, `arguments`, `super`, or `new.target`.
- `call()`, `apply()`, and `bind()` cannot change `this` in arrow functions.
- Arrow functions cannot be used as constructors (no `new`).

**Constraints and Limitations:**
- Arrow functions are unsuitable as object methods if you need `this` to refer to the object.
- Arrow functions cannot be used as constructors.
- `this` in an arrow function at the top level of a module is `undefined`.

### Annotated Code Examples

#### Example 1: Arrow Function in a Callback

```js
// Step 1: Object with a method that uses setTimeout.
const counter = {
  count: 0,
  start() {
    // Arrow function captures 'this' from start().
    setInterval(() => {
      this.count++;
      console.log(this.count);
    }, 1000);
  },
};

counter.start();
// Output: 1, 2, 3, ... (every second)
```

**Expected Output:** (After 1 second: `1`, after 2 seconds: `2`, and so on.)

**Why This Output:** The arrow function inside `setInterval` captures `this` from `start`, which is the `counter` object. It correctly increments `this.count`. A traditional function would have its own `this` (the global object or `undefined`).

#### Example 2: Arrow Function vs Regular Function in Object Method

```js
const obj = {
  value: 100,
  regularMethod() {
    setTimeout(function() {
      console.log("Regular this:", this); // Window or undefined
    }, 0);
  },
  arrowMethod() {
    setTimeout(() => {
      console.log("Arrow this.value:", this.value); // 100
    }, 0);
  },
};

obj.regularMethod();
obj.arrowMethod();
```

**Expected Output:**
```
Regular this: [object Window] (or undefined in strict mode)
Arrow this.value: 100
```

**Why This Output:** The regular function inside `setTimeout` has its own `this` binding (default binding). The arrow function captures `this` from `arrowMethod`, which is `obj`.

### Real-World Cases

- **React class components:** Arrow functions for event handlers to preserve `this`.
- **Promise chains:** `.then(data => this.process(data))`.
- **Array methods inside methods:** `this.items.map(item => item.name)`.
- **Timers:** `setInterval(() => this.tick(), 1000)`.

---

## `this` in Constructors

### Definitions

**Core Definition:** When a function is called with the `new` keyword, `this` refers to the **newly created object** that will be returned by the constructor.

**Technical Definition:** This is **`new` binding**. Per the precedence rules, "`new` binding (overrides all the rest)". When `new Foo()` is called, a new empty object is created, `this` is bound to it, the constructor body runs, and the object is returned (unless the constructor explicitly returns a different object).

**Beginner-Friendly Explanation:** When you use `new` to create an object from a constructor function, `this` inside the constructor refers to the brand-new object being created. This lets the constructor set properties on the new object using `this.property = value`.

### Purposes

- To allow constructors to initialise the newly created object.
- To support object-oriented programming with prototypes.
- To enable inheritance through prototype chains.

### Syntax Rules and Structure

```js
function Person(name, age) {
  this.name = name;  // 'this' is the new object
  this.age = age;
}

const alice = new Person("Alice", 30);
console.log(alice.name); // "Alice"
```

**Component Breakdown:**
- `new Person(...)`: The `new` operator creates a new object.
- `this` inside `Person` is bound to the new object.
- Properties set on `this` become properties of the new object.

**Syntax Rules:**
- `new` binding has the **highest precedence** of all binding rules.
- If the constructor returns an object, that object replaces the `this` object.
- Arrow functions cannot be used with `new` (they have no `[[Construct]]`).

**Constraints and Limitations:**
- Forgetting `new` causes `this` to follow default binding (often the global object), leading to unexpected global pollution.
- Class constructors in ES6+ require `new`; calling without it throws a `TypeError`.

### Annotated Code Examples

#### Example 1: Basic Constructor Binding

```js
// Step 1: Define a constructor function.
function User(name) {
  this.name = name;
  this.greet = function() {
    return `Hello, I'm ${this.name}`;
  };
}

// Step 2: Create an instance with new.
const user = new User("Bob");

// Step 3: Access the instance.
console.log(user.name);     // "Bob"
console.log(user.greet());  // "Hello, I'm Bob"
```

**Expected Output:**
```
Bob
Hello, I'm Bob
```

**Why This Output:** When `new User("Bob")` is called, a new object is created and `this` is bound to it. The constructor sets `this.name = "Bob"`. The `greet` method closes over `this` (the instance), so `this.name` is `"Bob"` when called.

#### Example 2: Constructor Returning an Object

```js
function Weird() {
  this.a = 1;
  return { b: 2 }; // Explicitly returning an object overrides 'this'
}

const obj = new Weird();
console.log(obj.a); // undefined
console.log(obj.b); // 2
```

**Expected Output:**
```
undefined
2
```

**Why This Output:** When a constructor returns an object, that object replaces the `this` object. The returned `{ b: 2 }` becomes the result of `new Weird()`, so `obj.a` is `undefined` and `obj.b` is `2`.

### Real-World Cases

- **Class constructors:** `class User { constructor(name) { this.name = name; } }`.
- **Factory functions:** Functions that create and return configured objects.
- **Prototype-based inheritance:** Setting up `this` for instances.

---

## `call()`, `apply()`, and `bind()`

### Definitions

**Core Definition:** `call()`, `apply()`, and `bind()` are methods on `Function.prototype` that allow **explicit control** over the value of `this` when invoking a function.

**Technical Definition:** Per MDN, "`Function.prototype.bind()` 메서드는 `this` 바인딩이 고정된 함수를 생성할 수 있으며, `Function.prototype.apply()`와 `Function.prototype.call()`로 특정 호출에 대한 `this` 값을 설정할 수 있습니다". Together, these methods form **explicit binding**, which takes precedence over implicit and default binding.

**Beginner-Friendly Explanation:** Sometimes you want to call a function but force `this` to be a specific object. `call()` and `apply()` let you do that immediately — they invoke the function with the `this` you specify. `bind()` doesn't call the function; it creates a new function permanently bound to the `this` you specify. The difference between `call` and `apply` is how they pass arguments: `call` takes them individually, `apply` takes them as an array.

### Purposes

- To explicitly set `this` when calling a function.
- To borrow methods from one object and use them on another.
- To create functions with permanently bound `this` (via `bind`).
- To support function composition and partial application.

### Syntax Rules and Structure

```js
// call: fn.call(thisArg, arg1, arg2, ...)
// apply: fn.apply(thisArg, [arg1, arg2, ...])
// bind: fn.bind(thisArg, arg1, arg2, ...) → returns a new function

function greet(greeting, punctuation) {
  return `${greeting}, ${this.name}${punctuation}`;
}

const person = { name: "Alice" };

// call:
greet.call(person, "Hello", "!"); // "Hello, Alice!"

// apply:
greet.apply(person, ["Hi", "?"]); // "Hi, Alice?"

// bind:
const boundGreet = greet.bind(person, "Hey");
boundGreet("!"); // "Hey, Alice!"
```

**Component Breakdown:**
- `thisArg`: The object to bind `this` to.
- `call`: Arguments passed individually after `thisArg`.
- `apply`: Arguments passed as an array after `thisArg`.
- `bind`: Returns a new function with `this` permanently bound and optional pre-applied arguments.

**Syntax Rules:**
- `call` and `apply` invoke the function immediately.
- `bind` returns a new function without invoking it.
- `bind` can also perform partial application (pre-filling arguments).
- `call` and `apply` cannot be used with `new` (see Pitfalls).

**Constraints and Limitations:**
- `bind` creates a new function each time, which has memory overhead.
- `bind` cannot be overridden by `call` or `apply` (hard binding).
- Arrow functions ignore `call`, `apply`, and `bind` for `this` (they use lexical `this`).

### Annotated Code Examples

#### Example 1: `call` vs `apply` vs `bind`

```js
// Step 1: Define a function that uses 'this'.
function introduce(greeting, punctuation) {
  return `${greeting}, my name is ${this.name}${punctuation}`;
}

// Step 2: Create an object.
const person = { name: "Alice" };

// Step 3: call — arguments individually.
console.log(introduce.call(person, "Hello", "!")); // "Hello, my name is Alice!"

// Step 4: apply — arguments as array.
console.log(introduce.apply(person, ["Hi", "?"])); // "Hi, my name is Alice?"

// Step 5: bind — returns a new function.
const boundIntroduce = introduce.bind(person, "Hey");
console.log(boundIntroduce(".")); // "Hey, my name is Alice."
```

**Expected Output:**
```
Hello, my name is Alice!
Hi, my name is Alice?
Hey, my name is Alice.
```

**Why This Output:** All three methods bind `this` to `person`. `call` and `apply` invoke immediately; `bind` returns a new function that can be called later.

#### Example 2: Method Borrowing with `call`

```js
// Step 1: Define an object with a method.
const calculator = {
  multiply(a, b) {
    return a * b;
  },
};

// Step 2: Borrow the method for another object.
const other = { multiply: calculator.multiply };

// Step 3: Use call to set 'this'.
console.log(other.multiply.call(calculator, 3, 4)); // 12
```

**Expected Output:**
```
12
```

**Why This Output:** The `multiply` method doesn't actually use `this`, but `call` explicitly sets `this` to `calculator`. This demonstrates method borrowing — using a method from one object on another.

### Real-World Cases

- **Method borrowing:** Using `Array.prototype.slice.call(arguments)` to convert array-like objects to arrays.
- **Event handlers:** `handler.bind(this)` to preserve context in callbacks.
- **Function composition:** Using `bind` for partial application.
- **Class inheritance:** Calling parent constructors with `Parent.call(this, ...)`.

---

## Explicit Binding

### Definitions

**Core Definition:** Explicit binding is when you explicitly specify the value of `this` using `call()`, `apply()`, or `bind()`.

**Technical Definition:** Explicit binding **takes precedence over implicit binding** and default binding, but is overridden by `new` binding. When you use `fn.call(obj)`, `this` inside `fn` is explicitly set to `obj`, regardless of how `fn` is called or what object it belongs to.

**Beginner-Friendly Explanation:** Explicit binding is when you tell JavaScript exactly what `this` should be. Instead of letting the call site determine `this`, you override it with `call`, `apply`, or `bind`. This is useful when you want to borrow a method or force a specific context.

### Purposes

- To override the default or implicit `this` binding.
- To borrow methods from one object and use them with another.
- To create functions with a fixed `this` value.
- To support flexible function reuse across different contexts.

### Syntax Rules and Structure

```js
// Explicit binding with call:
fn.call(thisArg, arg1, arg2);

// Explicit binding with apply:
fn.apply(thisArg, [arg1, arg2]);

// Explicit binding with bind:
const boundFn = fn.bind(thisArg, arg1);
```

**Component Breakdown:**
- `thisArg`: The explicit value for `this`.
- `call` and `apply` invoke immediately; `bind` returns a new function.

**Syntax Rules:**
- Explicit binding takes precedence over implicit and default binding.
- `bind` creates a **hard binding** that cannot be overridden by `call` or `apply`.
- `new` binding can override `bind` (see Precedence).

**Constraints and Limitations:**
- Explicit binding is more verbose than implicit binding.
- Overuse of `bind` can lead to memory overhead and confusing code.

### Annotated Code Examples

#### Example 1: Explicit Binding Overrides Implicit Binding

```js
function getName() {
  return this.name;
}

const obj1 = { name: "Alice", getName };
const obj2 = { name: "Bob" };

// Implicit binding:
console.log(obj1.getName()); // "Alice"

// Explicit binding overrides implicit:
console.log(obj1.getName.call(obj2)); // "Bob"
```

**Expected Output:**
```
Alice
Bob
```

**Why This Output:** `obj1.getName()` uses implicit binding (`this` = `obj1`). `obj1.getName.call(obj2)` uses explicit binding to set `this` to `obj2`, even though the method belongs to `obj1`.

#### Example 2: Hard Binding with `bind`

```js
function greet() {
  return `Hello, ${this.name}`;
}

const person = { name: "Alice" };
const boundGreet = greet.bind(person);

// bind creates a permanent binding:
console.log(boundGreet()); // "Hello, Alice"

// call cannot override bind:
console.log(boundGreet.call({ name: "Bob" })); // "Hello, Alice" (not Bob)
```

**Expected Output:**
```
Hello, Alice
Hello, Alice
```

**Why This Output:** `bind` creates a **hard binding** that cannot be overridden by `call` or `apply`. Even when `boundGreet.call({ name: "Bob" })` is used, `this` remains `person`.

### Real-World Cases

- **Event handlers:** `element.addEventListener("click", handler.bind(this))`.
- **Array methods:** `Array.prototype.map.call(arrayLike, callback)`.
- **Partial application:** `const add5 = add.bind(null, 5)`.
- **Inheritance:** `Parent.call(this, ...)` in ES5-style inheritance.

---

## Implicit Binding

### Definitions

**Core Definition:** Implicit binding is when `this` is automatically set to the object that owns the method being called, determined by the call site using dot notation.

**Technical Definition:** Per web.dev, "将函数作为对象的方法调用时，该方法内的 `this` 实例会引用包含该方法的对象". Implicit binding takes precedence over default binding but is overridden by explicit binding and `new` binding.

**Beginner-Friendly Explanation:** Implicit binding is the "natural" way `this` works. When you call `obj.method()`, `this` automatically becomes `obj`. You don't have to do anything special — just use dot notation.

### Purposes

- To provide natural object-oriented behaviour.
- To allow methods to access their object's properties.
- To enable code reuse across objects.

### Syntax Rules and Structure

```js
const obj = {
  name: "Alice",
  greet() {
    return `Hello, ${this.name}`;
  },
};

obj.greet(); // 'this' is obj
```

**Component Breakdown:**
- `obj.greet()`: The call site determines `this`.
- `this` is bound to the object immediately before the dot.

**Syntax Rules:**
- Only the **last** object in a property chain matters: `a.b.c.method()` → `this` is `a.b.c`.
- Implicit binding is lost when the method is extracted from its object.

**Constraints and Limitations:**
- Losing `this` when passing methods as callbacks is a common pitfall.
- Nested functions inside methods lose the implicit binding.

### Annotated Code Examples

#### Example 1: Implicit Binding in a Method

```js
const user = {
  name: "Alice",
  sayHi() {
    return `Hi, I'm ${this.name}`;
  },
};

console.log(user.sayHi()); // "Hi, I'm Alice"
```

**Expected Output:**
```
Hi, I'm Alice
```

**Why This Output:** `user.sayHi()` uses implicit binding. `this` is `user`, so `this.name` is `"Alice"`.

#### Example 2: Losing Implicit Binding

```js
const user = {
  name: "Alice",
  sayHi() {
    return `Hi, I'm ${this.name}`;
  },
};

const fn = user.sayHi; // Extracting the method
console.log(fn()); // "Hi, I'm undefined" (or error in strict mode)
```

**Expected Output (non-strict mode):**
```
Hi, I'm undefined
```

**Why This Output:** When `user.sayHi` is assigned to `fn`, the implicit binding is lost. Calling `fn()` uses default binding, so `this` is the global object (which has no `name` property).

### Real-World Cases

- **Class methods:** `instance.method()` uses implicit binding.
- **Object literals:** Methods defined in object literals use implicit binding when called with dot notation.
- **Chained methods:** `obj.a().b().c()` — each method's `this` is the object returned by the previous call.

---

## Default Binding

### Definitions

**Core Definition:** Default binding is the fallback rule that applies when no other binding rule matches. `this` is set to the global object (non-strict) or `undefined` (strict).

**Technical Definition:** Per web.dev, "在严格模式之外，`this` 还会引用独立函数内的全局对象... 使用严格模式时，`this` 在独立函数内的值为 `undefined`". Default binding has the **lowest precedence** of all binding rules.

**Beginner-Friendly Explanation:** Default binding is what happens when you call a function without any context — just `myFunction()`. In non-strict mode, `this` becomes the global object (`window` or `global`). In strict mode, it becomes `undefined`. This is the "last resort" rule.

### Purposes

- To provide a fallback when no context is available.
- To enable global functions to access the global object (non-strict).
- To enforce stricter behaviour in strict mode.

### Syntax Rules and Structure

```js
function standalone() {
  console.log(this);
}

standalone(); // Non-strict: Window; Strict: undefined
```

**Component Breakdown:**
- `standalone()`: A function call without an object.
- Non-strict: `this` → global object.
- Strict: `this` → `undefined`.

**Syntax Rules:**
- Default binding applies only when no other rule matches.
- Strict mode changes the value from the global object to `undefined`.

**Constraints and Limitations:**
- Default binding is a common source of bugs (accidental global access).
- Always use strict mode to catch these issues early.

### Annotated Code Examples

#### Example 1: Default Binding in Non-Strict Mode

```js
function logThis() {
  console.log(this);
}

logThis(); // Window { ... } (in browser)
```

**Expected Output:**
```
Window { ... }
```

**Why This Output:** In non-strict mode, `logThis()` uses default binding, so `this` is the global object (`window` in browsers).

#### Example 2: Default Binding in Strict Mode

```js
function logThis() {
  "use strict";
  console.log(this);
}

logThis(); // undefined
```

**Expected Output:**
```
undefined
```

**Why This Output:** In strict mode, default binding sets `this` to `undefined` instead of the global object.

### Real-World Cases

- **Utility functions:** Functions that don't use `this` are unaffected by default binding.
- **Strict mode modules:** ES modules are always strict, so default binding is `undefined`.
- **Legacy code:** Non-strict default binding is a common source of global pollution bugs.

---

## `new` Binding

### Definitions

**Core Definition:** `new` binding occurs when a function is called with the `new` keyword. `this` is bound to the newly created object.

**Technical Definition:** `new` binding has the **highest precedence** of all binding rules. When `new Foo()` is called, a new object is created, `this` is bound to it, the constructor runs, and the object is returned.

**Beginner-Friendly Explanation:** When you use `new` to create an object, `this` inside the constructor refers to the new object. This is how constructors set properties on instances.

### Purposes

- To initialise newly created objects.
- To support object-oriented programming.
- To enable prototype-based inheritance.

### Syntax Rules and Structure

```js
function Person(name) {
  this.name = name; // 'this' is the new object
}

const person = new Person("Alice");
console.log(person.name); // "Alice"
```

**Component Breakdown:**
- `new Person("Alice")`: Creates a new object.
- `this` inside `Person` is the new object.
- Properties set on `this` become instance properties.

**Syntax Rules:**
- `new` binding overrides all other binding rules.
- If the constructor returns an object, that object replaces `this`.
- Arrow functions cannot be used with `new`.

**Constraints and Limitations:**
- Forgetting `new` leads to default binding (global pollution).
- `new` cannot be combined with `call` or `apply`.

### Annotated Code Examples

#### Example 1: Basic `new` Binding

```js
function Car(make, model) {
  this.make = make;
  this.model = model;
}

const car = new Car("Toyota", "Camry");
console.log(car.make); // "Toyota"
console.log(car.model); // "Camry"
```

**Expected Output:**
```
Toyota
Camry
```

**Why This Output:** `new Car(...)` creates a new object and binds `this` to it. The constructor sets `this.make` and `this.model`, which become properties of the new object.

#### Example 2: `new` Overrides `bind`

```js
function foo(something) {
  this.a = something;
}

const obj = {};
const bar = foo.bind(obj);
bar(2);
console.log(obj.a); // 2

const baz = new bar(3);
console.log(obj.a); // 2 (not 3)
console.log(baz.a); // 3
```

**Expected Output:**
```
2
2
3
```

**Why This Output:** `bar` is hard-bound to `obj`. But when `new bar(3)` is called, `new` binding overrides the hard binding. `this` becomes the new object (`baz`), so `baz.a` is `3`, while `obj.a` remains `2`.

### Real-World Cases

- **Class constructors:** `new User(...)` in ES6+.
- **Factory functions:** Functions that create objects.
- **Prototype inheritance:** Setting up instance properties.

---

## `this` Precedence

### Definitions

**Core Definition:** The `this` precedence rules determine which binding rule applies when multiple rules could be eligible.

**Technical Definition:** Per the precedence rules, from highest to lowest: **`new` binding** > **Explicit binding** (`call`/`apply`/`bind`) > **Implicit binding** > **Default binding**. Arrow functions do not participate in this precedence — they use lexical `this`.

**Beginner-Friendly Explanation:** When you call a function, JavaScript checks the binding rules in order. First, it asks: "Was it called with `new`?" If yes, `this` is the new object. If not, it asks: "Was it called with `call`, `apply`, or `bind`?" If yes, `this` is what you explicitly specified. If not, it asks: "Was it called as a method?" If yes, `this` is the object. If none of these, `this` falls back to the default rule.

### Purposes

- To provide a deterministic order for `this` resolution.
- To help developers predict `this` in any situation.
- To explain why some bindings override others.

### Syntax Rules and Structure

The precedence order is:

1. **`new` binding:** `new Foo()`
2. **Explicit binding:** `foo.call(obj)`, `foo.apply(obj)`, `foo.bind(obj)`
3. **Implicit binding:** `obj.foo()`
4. **Default binding:** `foo()`

**Component Breakdown:**
- Each rule is checked in order from highest to lowest.
- The first matching rule determines `this`.
- Arrow functions bypass all rules (lexical `this`).

**Syntax Rules:**
- `new` binding cannot be combined with `call`/`apply`.
- `bind` creates a hard binding that cannot be overridden by `call`/`apply`, but **can** be overridden by `new`.
- Implicit binding is lost when methods are extracted.

**Constraints and Limitations:**
- Precedence rules apply only to regular functions; arrow functions are exempt.
- Some combinations are not allowed (e.g., `new foo.call(obj)`).

### Annotated Code Examples

#### Example 1: Precedence Demonstration

```js
function foo() {
  console.log(this.a);
}

const obj1 = { a: 1, foo };
const obj2 = { a: 2, foo };

// Default binding (non-strict):
// foo(); // undefined (or global a)

// Implicit binding:
obj1.foo(); // 1
obj2.foo(); // 2

// Explicit binding:
obj1.foo.call(obj2); // 2

// new binding:
function Bar() {
  this.a = 3;
}
const bar = new Bar();
console.log(bar.a); // 3
```

**Expected Output:**
```
1
2
2
3
```

**Why This Output:** Each binding rule produces a different `this` value. Implicit binding uses the object before the dot. Explicit binding uses the object passed to `call`. `new` binding creates a new object with `a = 3`.

#### Example 2: `new` Overrides `bind`

```js
function foo(something) {
  this.a = something;
}

const obj = {};
const bar = foo.bind(obj);
bar(2);
console.log(obj.a); // 2

const baz = new bar(3);
console.log(obj.a); // 2
console.log(baz.a); // 3
```

**Expected Output:**
```
2
2
3
```

**Why This Output:** `bar` is bound to `obj` via `bind`. But when `new bar(3)` is called, `new` binding takes precedence. `this` becomes the new object (`baz`), so `baz.a` is `3`, and `obj.a` remains `2`.

### Real-World Cases

- **Debugging:** Understanding precedence helps identify why `this` is unexpected.
- **Framework internals:** React and Vue rely on these rules for event handling.
- **Method borrowing:** Explicit binding overrides implicit binding when borrowing methods.

---

## `this` Pitfalls

### Definitions

**Core Definition:** `this` pitfalls are common mistakes and unexpected behaviours that arise from misunderstanding `this` binding rules.

**Technical Definition:** The most common pitfalls include: (1) losing `this` when passing methods as callbacks, (2) nested functions inside methods inheriting the wrong `this`, (3) forgetting `new` with constructors, (4) using arrow functions as object methods, and (5) confusion between lexical and dynamic `this`.

**Beginner-Friendly Explanation:** `this` is one of the most confusing parts of JavaScript. Common mistakes include: passing a method as a callback and losing `this`, using a regular function inside a method and getting the global object instead of the object, and forgetting `new` when calling a constructor. Understanding these pitfalls helps you avoid bugs.

### Purposes

- To recognise and avoid common `this`-related bugs.
- To write more predictable and maintainable code.
- To choose the right function type (regular vs arrow) for the context.

### Syntax Rules and Structure

Common pitfalls and solutions:

```js
// Pitfall 1: Losing this in callbacks
const obj = {
  name: "Alice",
  greet() {
    setTimeout(function() {
      console.log(this.name); // undefined (or global)
    }, 100);
  },
};
// Fix: use arrow function
setTimeout(() => console.log(this.name), 100);

// Pitfall 2: Nested functions
const obj2 = {
  name: "Bob",
  greet() {
    function inner() {
      console.log(this.name); // undefined
    }
    inner();
  },
};
// Fix: use arrow function or bind

// Pitfall 3: Forgetting new
function Person(name) {
  this.name = name;
}
const p = Person("Alice"); // 'this' is global (non-strict)
// Fix: always use new
```

**Component Breakdown:**
- **Callback loss:** Extracting a method from its object loses implicit binding.
- **Nested function loss:** Inner functions don't inherit the outer `this`.
- **Forgotten `new`:** Constructors called without `new` pollute the global object.

**Syntax Rules:**
- Use arrow functions for callbacks that need the outer `this`.
- Use `bind` when you need a permanently bound function.
- Always use `new` with constructors (or use classes).

**Constraints and Limitations:**
- Arrow functions cannot be used as constructors.
- `bind` creates a new function, which has memory overhead.

### Annotated Code Examples

#### Example 1: Losing `this` in a Callback

```js
// Step 1: Object with a method that uses setTimeout.
const user = {
  name: "Alice",
  greet() {
    setTimeout(function() {
      console.log(this.name); // 'this' is global, not user
    }, 100);
  },
};

user.greet(); // Output: undefined (or error in strict mode)
```

**Expected Output (non-strict mode):**
```
undefined
```

**Why This Output:** The anonymous function inside `setTimeout` is called as a standalone function, so `this` follows default binding (global object). The global object has no `name` property, so `this.name` is `undefined`.

**Fix:**
```js
setTimeout(() => {
  console.log(this.name); // 'this' is user (lexical)
}, 100);
```

#### Example 2: Forgetting `new`

```js
function Person(name) {
  this.name = name;
}

const alice = Person("Alice"); // Forgot new!
console.log(alice); // undefined
console.log(globalThis.name); // "Alice" (global pollution)
```

**Expected Output (non-strict mode):**
```
undefined
Alice
```

**Why This Output:** Calling `Person("Alice")` without `new` uses default binding. `this` is the global object, so `this.name = "Alice"` sets a global property. The function returns `undefined`, so `alice` is `undefined`.

**Fix:**
```js
const alice = new Person("Alice"); // Use new
```

### Real-World Cases

- **Event handlers:** `this` inside event handlers is the element, not the component.
- **React class components:** Methods lose `this` when passed as callbacks.
- **Array methods:** `array.map(this.callback)` loses `this` unless bound.

---

## `this` Best Practices

### Definitions

**Core Definition:** `this` best practices are guidelines for using `this` in a predictable, maintainable, and bug-free manner.

**Technical Definition:** Best practices include: preferring arrow functions for callbacks to preserve lexical `this`; using `bind` sparingly and preferring arrow functions instead; avoiding `this` in standalone functions; using classes with explicit `this`; and documenting `this` expectations in function signatures.

**Beginner-Friendly Explanation:** Use arrow functions when you want to keep the outer `this`. Use regular functions when you need dynamic `this` (like object methods). Avoid using `this` in functions that don't need it. Always use `new` with constructors. When in doubt, use arrow functions for callbacks.

### Purposes

- To avoid `this`-related bugs.
- To write code that is easy to understand and maintain.
- To choose the right function type for each situation.
- To make `this` behaviour explicit and predictable.

### Syntax Rules and Structure

Best practices:

```js
// 1. Use arrow functions for callbacks:
setTimeout(() => this.doSomething(), 100);

// 2. Use bind when you need a fixed this:
const handler = this.handleClick.bind(this);

// 3. Avoid this in standalone functions:
function utility() {
  // Don't use this here
}

// 4. Use classes for object-oriented code:
class User {
  constructor(name) {
    this.name = name;
  }
  greet() {
    return `Hello, ${this.name}`;
  }
}

// 5. Document this expectations:
/**
 * @this {User}
 */
function greet() {
  return `Hello, ${this.name}`;
}
```

**Component Breakdown:**
- **Arrow functions:** Preserve lexical `this` for callbacks.
- **`bind`:** Creates a hard binding when needed.
- **Classes:** Provide clear `this` semantics.
- **Documentation:** JSDoc `@this` annotation clarifies expectations.

**Syntax Rules:**
- Prefer arrow functions over `bind` for callbacks.
- Use regular functions for object methods that need dynamic `this`.
- Avoid `const self = this` (use arrow functions instead).
- Use strict mode to catch accidental global `this`.

**Constraints and Limitations:**
- Arrow functions cannot be used as constructors.
- `bind` creates a new function each time (memory overhead).
- Overusing `this` can make code harder to reason about.

### Annotated Code Examples

#### Example 1: Arrow Function vs `bind`

```js
// Step 1: Object with a method that uses a callback.
const timer = {
  seconds: 0,
  start() {
    // Good: arrow function preserves this
    setInterval(() => {
      this.seconds++;
      console.log(this.seconds);
    }, 1000);
  },
  startWithBind() {
    // Also works: bind creates a hard binding
    setInterval(function() {
      this.seconds++;
      console.log(this.seconds);
    }.bind(this), 1000);
  },
};

timer.start();
```

**Expected Output:** (After 1 second: `1`, after 2 seconds: `2`, and so on.)

**Why This Output:** Both approaches work. The arrow function captures `this` lexically, while `bind` explicitly binds `this` to the `timer` object. The arrow function is more concise and preferred by modern style guides.

#### Example 2: Strict Mode for Safety

```js
"use strict";

function standalone() {
  console.log(this); // undefined (not global)
}

standalone();
```

**Expected Output:**
```
undefined
```

**Why This Output:** Strict mode prevents default binding from coercing `this` to the global object. This catches accidental global access early.

### Real-World Cases

- **React:** Use arrow functions for event handlers to avoid `bind` in the constructor.
- **Vue:** Methods are automatically bound to the component instance.
- **Node.js:** Use arrow functions for callbacks in asynchronous code.
- **TypeScript:** `noImplicitThis` compiler option catches `this`-related errors.

---

## References

- MDN Web Docs – this — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this
- MDN Web Docs – Function — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function
- MDN Web Docs – Function.prototype.call() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call
- MDN Web Docs – Function.prototype.apply() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply
- MDN Web Docs – Function.prototype.bind() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind
- MDN Web Docs – Arrow function expressions — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions
- web.dev – The this keyword — https://web.dev/learn/javascript/functions/this
- You Don't Know JS: this & Object Prototypes (Kyle Simpson) — https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/this%20&%20object%20prototypes/ch2.md
- JavaScript.info – Function binding — https://javascript.info/bind
- Google JavaScript Style Guide – this — https://google.github.io/styleguide/jsguide.html
- FreeCodeCamp – How to Use the "this" Keyword in JavaScript — https://www.freecodecamp.org/news/how-to-use-the-this-keyword-in-javascript/
- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://ecma-international.org/wp-content/uploads/ECMA-262_16th_edition_june_2025.pdf
- TypeScript Handbook – this — https://www.typescriptlang.org/docs/handbook/2/functions.html#this