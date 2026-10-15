# JavaScript Unit Testing — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Unit testing is the practice of verifying the behaviour of individual, isolated units of code — typically functions, modules, or classes — by providing controlled inputs and asserting on outputs and side effects.

**Technical Definition:** A unit test exercises the smallest testable piece of software in isolation from its dependencies. In JavaScript, units are usually functions, classes, or modules (ESM/CJS). Isolation is achieved via test doubles (mocks, stubs, spies, fakes) that replace real dependencies. Unit tests are automated, fast (milliseconds), deterministic, and numerous. They are typically organised with a `describe`/`it` (or `test`) structure and use an assertion library (`expect`, `assert`, `chai`). Test runners (Jest, Vitest, Mocha, Jasmine, `node:test`) provide discovery, execution, reporting, and mocking utilities. Unit tests are not specified by ECMAScript; they are a third-party ecosystem layer built on JavaScript's module, function, and async features.

**Beginner-Friendly Explanation:** Unit testing means testing one small piece of your code at a time — usually a single function, class, or module — to make sure it works as expected. You give it some input, call it, and check that the result matches what you expect. If your function depends on something else (like a database or an API), you replace that dependency with a fake so you're only testing your code. Unit tests are fast, so you can run thousands of them in seconds and catch bugs immediately when you change something.

### Key Characteristics

- **Isolated:** Tests one unit without real dependencies.
- **Fast:** Milliseconds per test; thousands per second.
- **Deterministic:** Same input, same output, every time.
- **Numerous:** Typically the largest layer in the test pyramid.
- **Focused:** Each test verifies one behaviour.
- **Automated:** Run by CI on every commit.
- **Descriptive:** Test names describe expected behaviour.
- **Independent:** Tests do not depend on each other.

### Prerequisites

- Solid JavaScript fundamentals (functions, closures, `this`, async).
- Understanding of modules (ESM `import`/`export`, CommonJS `require`).
- Familiarity with Promises and `async/await`.
- Awareness of error handling (`try`/`catch`, `throw`).
- Basic knowledge of npm and `package.json`.

### Related Programming Areas

- **Integration testing:** Testing units together.
- **E2E testing:** Testing full user flows.
- **Test-driven development:** Writing tests first.
- **Behaviour-driven development:** Behaviour-focused tests.
- **Code coverage:** Measuring test thoroughness.
- **Test doubles:** Mocks, stubs, spies, fakes.

---

## Testing Functions

### Definitions

**Core Definition:** Testing functions is the practice of verifying a function's behaviour with controlled inputs and asserted outputs.

**Technical Definition:** Function unit tests call the function under test (FUT) with specific arguments and assert on the return value, thrown exceptions, or side effects. Pure functions are easiest to test because their output depends only on input. Impure functions require controlling or mocking external state.

**Beginner-Friendly Explanation:** Testing a function means calling it with certain inputs and checking that the output is correct.

### Purposes

- To verify function contracts.
- To catch logic errors.
- To document expected behaviour.
- To enable safe refactoring.

### Syntax Rules and Structure

```js
import { describe, it, expect } from "vitest";
import { functionUnderTest } from "./module.js";

describe("functionUnderTest", () => {
  it("returns the expected result for a given input", () => {
    expect(functionUnderTest(input)).toBe(expected);
  });
});
```

### Annotated Code Examples

#### Example 1: Testing a Pure Function

```js
// math.js
export function add(a, b) { return a + b; }
export function divide(a, b) {
  if (b === 0) throw new Error("Division by zero");
  return a / b;
}
```

```js
// math.test.js
import { describe, it, expect } from "vitest";
import { add, divide } from "./math.js";

describe("add", () => {
  it("adds two positive numbers", () => {
    expect(add(2, 3)).toBe(5);
  });

  it("handles negative numbers", () => {
    expect(add(-2, -3)).toBe(-5);
  });

  it("handles zero", () => {
    expect(add(0, 0)).toBe(0);
  });
});

describe("divide", () => {
  it("divides two numbers", () => {
    expect(divide(10, 2)).toBe(5);
  });

  it("throws on division by zero", () => {
    expect(() => divide(10, 0)).toThrow("Division by zero");
  });
});
```

**Expected Output:**
```
✓ add > adds two positive numbers
✓ add > handles negative numbers
✓ add > handles zero
✓ divide > divides two numbers
✓ divide > throws on division by zero
```

**Why This Output:** Each test verifies one behaviour; edge cases (zero, negatives, errors) are covered.

#### Example 2: Testing Functions with Side Effects

```js
// counter.js
export function createCounter() {
  let count = 0;
  return {
    increment() { return ++count; },
    decrement() { return --count; },
    getCount() { return count; },
  };
}
```

```js
// counter.test.js
import { describe, it, expect, beforeEach } from "vitest";
import { createCounter } from "./counter.js";

describe("createCounter", () => {
  let counter;
  beforeEach(() => { counter = createCounter(); });

  it("starts at 0", () => {
    expect(counter.getCount()).toBe(0);
  });

  it("increments", () => {
    counter.increment();
    counter.increment();
    expect(counter.getCount()).toBe(2);
  });

  it("decrements", () => {
    counter.decrement();
    expect(counter.getCount()).toBe(-1);
  });

  it("returns independent instances", () => {
    const other = createCounter();
    counter.increment();
    expect(other.getCount()).toBe(0);
  });
});
```

**Expected Output:**
```
✓ createCounter > starts at 0
✓ createCounter > increments
✓ createCounter > decrements
✓ createCounter > returns independent instances
```

**Why This Output:** `beforeEach` creates a fresh counter per test, ensuring isolation.

### Real-World Cases

- **Utility functions:** `formatDate`, `parseQuery`.
- **Pure calculations:** `calculateTax`, `computeDiscount`.
- **Validators:** `isValidEmail`, `isStrongPassword`.
- **Reducers:** Redux reducers are pure functions.

---

## Testing Modules

### Definitions

**Core Definition:** Testing modules is the practice of verifying the public API of a module (exported functions, classes, constants) in isolation.

**Technical Definition:** A module unit test imports the module's exports and tests each exported binding. Modules with internal state or side effects (e.g., top-level code) require careful setup (`beforeEach` to reset state) or module mocking (`vi.mock`). ESM modules are cached; you may need `vi.resetModules()` to re-import fresh copies.

**Beginner-Friendly Explanation:** Testing a module means testing everything the module exports — usually a set of functions or classes — as a unit.

### Purposes

- To verify a module's public API.
- To test related functions together.
- To catch export/import errors.
- To validate module contracts.

### Syntax Rules and Structure

```js
import * as myModule from "./myModule.js";

describe("myModule", () => {
  it("exports the expected API", () => {
    expect(typeof myModule.foo).toBe("function");
  });
});
```

### Annotated Code Examples

#### Example 1: Testing a Utility Module

```js
// stringUtils.js
export function capitalize(str) {
  return str.charAt(0).toUpperCase() + str.slice(1);
}
export function reverse(str) {
  return str.split("").reverse().join("");
}
export function truncate(str, length) {
  return str.length <= length ? str : str.slice(0, length) + "...";
}
```

```js
// stringUtils.test.js
import { describe, it, expect } from "vitest";
import * as stringUtils from "./stringUtils.js";

describe("stringUtils", () => {
  describe("capitalize", () => {
    it("capitalizes the first letter", () => {
      expect(stringUtils.capitalize("hello")).toBe("Hello");
    });
    it("handles empty strings", () => {
      expect(stringUtils.capitalize("")).toBe("");
    });
  });

  describe("reverse", () => {
    it("reverses a string", () => {
      expect(stringUtils.reverse("abc")).toBe("cba");
    });
  });

  describe("truncate", () => {
    it("truncates long strings", () => {
      expect(stringUtils.truncate("hello world", 5)).toBe("hello...");
    });
    it("returns short strings unchanged", () => {
      expect(stringUtils.truncate("hi", 5)).toBe("hi");
    });
  });
});
```

**Expected Output:**
```
✓ stringUtils > capitalize > capitalizes the first letter
✓ stringUtils > capitalize > handles empty strings
✓ stringUtils > reverse > reverses a string
✓ stringUtils > truncate > truncates long strings
✓ stringUtils > truncate > returns short strings unchanged
```

#### Example 2: Testing a Module with Internal State

```js
// store.js
const state = { count: 0 };
export function increment() { state.count++; }
export function getCount() { return state.count; }
export function reset() { state.count = 0; }
```

```js
// store.test.js
import { describe, it, expect, beforeEach } from "vitest";
import { increment, getCount, reset } from "./store.js";

describe("store", () => {
  beforeEach(() => { reset(); });

  it("starts at 0", () => {
    expect(getCount()).toBe(0);
  });

  it("increments", () => {
    increment();
    increment();
    expect(getCount()).toBe(2);
  });
});
```

**Why This Helps:** `beforeEach` resets shared module state to ensure isolation.

### Real-World Cases

- **Utility libraries:** String, array, math helpers.
- **API clients:** Testing method signatures.
- **Config modules:** Default exports.

---

## Testing Classes

### Definitions

**Core Definition:** Testing classes is the practice of verifying the behaviour of class instances — their constructors, methods, and state.

**Technical Definition:** Class unit tests instantiate the class (often in `beforeEach`), call methods, and assert on return values, `this`, or public state. Private fields (`#`) cannot be accessed directly; test observable behaviour. Static methods are tested by calling on the class itself.

**Beginner-Friendly Explanation:** Testing a class means creating an instance and checking that its methods do what they should.

### Purposes

- To verify constructor initialisation.
- To test method behaviour.
- To verify state transitions.
- To test inheritance and polymorphism.

### Annotated Code Examples

#### Example 1: Testing a Simple Class

```js
// BankAccount.js
export class BankAccount {
  #balance = 0;

  constructor(initialBalance = 0) {
    if (initialBalance < 0) throw new Error("Initial balance cannot be negative");
    this.#balance = initialBalance;
  }

  deposit(amount) {
    if (amount <= 0) throw new Error("Deposit must be positive");
    this.#balance += amount;
    return this.#balance;
  }

  withdraw(amount) {
    if (amount <= 0) throw new Error("Withdrawal must be positive");
    if (amount > this.#balance) throw new Error("Insufficient funds");
    this.#balance -= amount;
    return this.#balance;
  }

  get balance() { return this.#balance; }
}
```

```js
// BankAccount.test.js
import { describe, it, expect, beforeEach } from "vitest";
import { BankAccount } from "./BankAccount.js";

describe("BankAccount", () => {
  let account;
  beforeEach(() => { account = new BankAccount(100); });

  it("initialises with the given balance", () => {
    expect(account.balance).toBe(100);
  });

  it("throws on negative initial balance", () => {
    expect(() => new BankAccount(-1)).toThrow("Initial balance cannot be negative");
  });

  it("deposits money", () => {
    account.deposit(50);
    expect(account.balance).toBe(150);
  });

  it("throws on non-positive deposit", () => {
    expect(() => account.deposit(0)).toThrow("Deposit must be positive");
  });

  it("withdraws money", () => {
    account.withdraw(30);
    expect(account.balance).toBe(70);
  });

  it("throws on insufficient funds", () => {
    expect(() => account.withdraw(200)).toThrow("Insufficient funds");
  });
});
```

**Expected Output:**
```
✓ BankAccount > initialises with the given balance
✓ BankAccount > throws on negative initial balance
✓ BankAccount > deposits money
✓ BankAccount > throws on non-positive deposit
✓ BankAccount > withdraws money
✓ BankAccount > throws on insufficient funds
```

**Why This Output:** Private fields (`#balance`) are tested through public methods and the `balance` getter, verifying observable behaviour.

#### Example 2: Testing Inheritance

```js
// Animal.js
export class Animal {
  constructor(name) { this.name = name; }
  speak() { return `${this.name} makes a sound`; }
}

export class Dog extends Animal {
  speak() { return `${this.name} barks`; }
}
```

```js
// Animal.test.js
import { describe, it, expect } from "vitest";
import { Animal, Dog } from "./Animal.js";

describe("Animal", () => {
  it("speaks generically", () => {
    expect(new Animal("Rex").speak()).toBe("Rex makes a sound");
  });
});

describe("Dog", () => {
  it("overrides speak", () => {
    expect(new Dog("Rex").speak()).toBe("Rex barks");
  });

  it("inherits name", () => {
    expect(new Dog("Rex").name).toBe("Rex");
  });

  it("is an instance of Animal", () => {
    expect(new Dog("Rex")).toBeInstanceOf(Animal);
  });
});
```

### Real-World Cases

- **Domain models:** User, Order, Product.
- **Services:** UserService, PaymentService.
- **Data structures:** Stack, Queue, LinkedList.

---

## Testing Async Code

### Definitions

**Core Definition:** Testing async code is the practice of verifying behaviour of Promises and `async/await` functions.

**Technical Definition:** Async tests return a Promise (or use `async`/`await`); the test runner waits for resolution before reporting. Rejected Promises must be handled (via `rejects` or `try`/`catch`) to avoid unhandled rejections. Timeouts and fake timers (`vi.useFakeTimers()`) control timing-sensitive code.

**Beginner-Friendly Explanation:** Testing async code means waiting for the Promise to resolve or reject, then checking the result.

### Purposes

- To verify async functions return the right values.
- To test error handling in async code.
- To test timing-dependent behaviour.
- To verify Promise rejection.

### Syntax Rules and Structure

```js
it("resolves with a value", async () => {
  await expect(fetchData()).resolves.toBe(value);
});

it("rejects with an error", async () => {
  await expect(fetchData()).rejects.toThrow("Error");
});

it("handles async with try/catch", async () => {
  try {
    await failingFn();
    expect.fail("Should have thrown");
  } catch (err) {
    expect(err.message).toBe("Expected");
  }
});
```

### Annotated Code Examples

#### Example 1: Testing Resolved Promises

```js
// api.js
export async function fetchUser(id) {
  if (id <= 0) throw new Error("Invalid ID");
  return { id, name: "Alice" };
}
```

```js
// api.test.js
import { describe, it, expect } from "vitest";
import { fetchUser } from "./api.js";

describe("fetchUser", () => {
  it("resolves with a user", async () => {
    const user = await fetchUser(1);
    expect(user).toEqual({ id: 1, name: "Alice" });
  });

  it("rejects with an error for invalid ID", async () => {
    await expect(fetchUser(0)).rejects.toThrow("Invalid ID");
  });

  it("rejects using try/catch", async () => {
    try {
      await fetchUser(-1);
      expect.fail("Should have thrown");
    } catch (err) {
      expect(err.message).toBe("Invalid ID");
    }
  });
});
```

**Expected Output:**
```
✓ fetchUser > resolves with a user
✓ fetchUser > rejects with an error for invalid ID
✓ fetchUser > rejects using try/catch
```

#### Example 2: Testing Timers

```js
// delay.js
export function delay(ms, value) {
  return new Promise((resolve) => setTimeout(() => resolve(value), ms));
}
```

```js
// delay.test.js
import { describe, it, expect, vi } from "vitest";
import { delay } from "./delay.js";

describe("delay", () => {
  it("resolves after the specified time", async () => {
    vi.useFakeTimers();
    const promise = delay(1000, "done");
    vi.advanceTimersByTime(1000);
    await expect(promise).resolves.toBe("done");
    vi.useRealTimers();
  });
});
```

**Why This Helps:** Fake timers make timing tests instant and deterministic.

#### Example 3: Testing Rejected Promises

```js
it("rejects with a specific error", async () => {
  await expect(
    Promise.reject(new TypeError("bad input"))
  ).rejects.toBeInstanceOf(TypeError);
});
```

### Real-World Cases

- **API clients:** Testing fetch wrappers.
- **Database layers:** Testing async repositories.
- **Timers:** Debounce, throttle, retry.
- **Streams:** Testing readable/writable streams.

---

## Mocking Dependencies

### Definitions

**Core Definition:** Mocking dependencies is the practice of replacing real dependencies with controlled test doubles.

**Technical Definition:** Mocking replaces a module, function, or object with a test double that records calls and returns controlled values. In Vitest/Jest, `vi.mock("./module")` mocks a module; `vi.fn()` creates a mock function; `vi.spyOn(obj, "method")` spies on a method. Mocks verify interactions (`toHaveBeenCalledWith`) or provide canned return values (`mockResolvedValue`).

**Beginner-Friendly Explanation:** Mocking means replacing a real dependency (like an API or database) with a fake so you can test your code in isolation.

### Purposes

- To isolate the unit under test.
- To control dependency behaviour.
- To avoid slow or flaky external resources.
- To verify interactions.

### Syntax Rules and Structure

```js
// Mock a module.
vi.mock("./api.js", () => ({
  fetchUser: vi.fn(),
}));

// Mock a function.
const mockFn = vi.fn().mockReturnValue(42);

// Mock a method.
const spy = vi.spyOn(obj, "method").mockReturnValue(42);
```

### Annotated Code Examples

#### Example 1: Mocking a Module

```js
// userService.js
import { fetchUser } from "./api.js";

export async function getUserName(id) {
  const user = await fetchUser(id);
  return user.name;
}
```

```js
// userService.test.js
import { describe, it, expect, vi, beforeEach } from "vitest";
import { getUserName } from "./userService.js";
import { fetchUser } from "./api.js";

vi.mock("./api.js");

describe("getUserName", () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it("returns the user's name", async () => {
    fetchUser.mockResolvedValue({ id: 1, name: "Alice" });
    const name = await getUserName(1);
    expect(name).toBe("Alice");
    expect(fetchUser).toHaveBeenCalledWith(1);
  });

  it("propagates errors", async () => {
    fetchUser.mockRejectedValue(new Error("Not found"));
    await expect(getUserName(999)).rejects.toThrow("Not found");
  });
});
```

**Expected Output:**
```
✓ getUserName > returns the user's name
✓ getUserName > propagates errors
```

**Why This Output:** `vi.mock` replaces `fetchUser` with a mock; tests control its behaviour and verify calls.

#### Example 2: Mocking with Dependency Injection

```js
// userService.js
export function createUserService(repo) {
  return {
    async getUser(id) {
      const user = await repo.findById(id);
      if (!user) throw new Error("User not found");
      return user;
    },
  };
}
```

```js
// userService.test.js
import { describe, it, expect, vi, beforeEach } from "vitest";
import { createUserService } from "./userService.js";

describe("UserService", () => {
  let service;
  let mockRepo;

  beforeEach(() => {
    mockRepo = { findById: vi.fn() };
    service = createUserService(mockRepo);
  });

  it("returns the user from the repo", async () => {
    mockRepo.findById.mockResolvedValue({ id: 1, name: "Alice" });
    const user = await service.getUser(1);
    expect(user.name).toBe("Alice");
    expect(mockRepo.findById).toHaveBeenCalledWith(1);
  });

  it("throws when user is not found", async () => {
    mockRepo.findById.mockResolvedValue(null);
    await expect(service.getUser(1)).rejects.toThrow("User not found");
  });
});
```

**Why This Helps:** Dependency injection makes mocking trivial — no module-level mocking required.

### Real-World Cases

- **API calls:** Mock `fetch`/`axios`.
- **Database:** Mock repositories.
- **Time:** Mock `Date.now` or use fake timers.
- **Random:** Mock `Math.random`.
- **Environment:** Mock `process.env`.

---

## Stubbing APIs

### Definitions

**Core Definition:** Stubbing APIs is the practice of replacing API calls (fetch, HTTP clients) with controlled responses.

**Technical Definition:** Stubbing replaces network calls with canned responses, avoiding real HTTP requests. Tools: `vi.stubGlobal("fetch", mockFn)`, MSW (Mock Service Worker), nock (Node.js), or manual mocking of the client. Stubs return predetermined data and can simulate errors, delays, and edge cases.

**Beginner-Friendly Explanation:** Stubbing an API means faking the network response so your tests don't need a real server.

### Purposes

- To avoid network dependency in tests.
- To test error handling.
- To simulate edge cases.
- To run tests offline and fast.

### Annotated Code Examples

#### Example 1: Stubbing `fetch`

```js
// api.js
export async function getUser(id) {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}
```

```js
// api.test.js
import { describe, it, expect, vi, beforeEach, afterEach } from "vitest";
import { getUser } from "./api.js";

describe("getUser", () => {
  beforeEach(() => {
    vi.stubGlobal("fetch", vi.fn());
  });

  afterEach(() => {
    vi.unstubAllGlobals();
  });

  it("returns user data on success", async () => {
    fetch.mockResolvedValue({
      ok: true,
      json: async () => ({ id: 1, name: "Alice" }),
    });
    const user = await getUser(1);
    expect(user.name).toBe("Alice");
  });

  it("throws on HTTP error", async () => {
    fetch.mockResolvedValue({ ok: false, status: 404 });
    await expect(getUser(1)).rejects.toThrow("HTTP 404");
  });
});
```

**Expected Output:**
```
✓ getUser > returns user data on success
✓ getUser > throws on HTTP error
```

#### Example 2: Mock Service Worker (MSW)

```js
// mocks/handlers.js
import { http, HttpResponse } from "msw";

export const handlers = [
  http.get("/api/users/:id", ({ params }) => {
    return HttpResponse.json({ id: Number(params.id), name: "Alice" });
  }),
  http.get("/api/users/999", () => {
    return new HttpResponse(null, { status: 404 });
  }),
];
```

```js
// test setup
import { setupServer } from "msw/node";
import { handlers } from "./mocks/handlers.js";

const server = setupServer(...handlers);
beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

**Why This Helps:** MSW intercepts real fetch/XHR at the network layer, so tests use actual fetch calls without real HTTP.

### Real-World Cases

- **REST APIs:** Stub fetch/axios.
- **GraphQL:** Stub queries/mutations.
- **Third-party services:** Stripe, SendGrid.
- **Edge cases:** Timeouts, 500 errors, malformed responses.

---

## Spying on Calls

### Definitions

**Core Definition:** Spying is the practice of observing function calls without replacing their behaviour.

**Technical Definition:** A spy wraps a function (or replaces it) and records calls, arguments, return values, and `this`. In Vitest/Jest, `vi.spyOn(obj, "method")` creates a spy. Spies can be inspected with `toHaveBeenCalled`, `toHaveBeenCalledWith`, `toHaveBeenCalledTimes`, `toHaveReturnedWith`.

**Beginner-Friendly Explanation:** Spying means watching a function to see how it was called, without changing what it does.

### Purposes

- To verify interactions.
- To observe arguments and return values.
- To check call counts.
- To test side-effect-free observation.

### Annotated Code Examples

```js
import { describe, it, expect, vi, beforeEach, afterEach } from "vitest";

const obj = {
  greet(name) { return `Hello, ${name}`; },
};

describe("spyOn", () => {
  let spy;
  beforeEach(() => { spy = vi.spyOn(obj, "greet"); });
  afterEach(() => { spy.mockRestore(); });

  it("records calls", () => {
    obj.greet("Alice");
    obj.greet("Bob");

    expect(spy).toHaveBeenCalledTimes(2);
    expect(spy).toHaveBeenCalledWith("Alice");
    expect(spy).toHaveBeenCalledWith("Bob");
  });

  it("records return values", () => {
    obj.greet("Alice");
    expect(spy).toHaveReturnedWith("Hello, Alice");
  });

  it("can be restored", () => {
    spy.mockRestore();
    expect(obj.greet("Alice")).toBe("Hello, Alice");
    expect(spy).not.toHaveBeenCalled();
  });
});
```

**Expected Output:**
```
✓ spyOn > records calls
✓ spyOn > records return values
✓ spyOn > can be restored
```

### Real-World Cases

- **Event handlers:** Verify handler called.
- **Callbacks:** Verify callback invoked with args.
- **Logging:** Verify logger called.
- **Analytics:** Verify tracking events.

---

## Assertions

### Definitions

**Core Definition:** Assertions are statements that verify a condition holds; if not, the test fails.

**Technical Definition:** Assertions are provided by assertion libraries. Jest/Vitest use `expect(value).matcher(expected)`. Node.js provides `assert` with `strictEqual`, `deepStrictEqual`, `throws`, etc. Chai provides `expect`, `should`, and `assert` styles.

**Beginner-Friendly Explanation:** Assertions are the "checks" in your tests — "this should equal that," "this should throw," etc.

### Common Assertions

```js
expect(value).toBe(other);            // Object.is equality
expect(value).toEqual(other);         // Deep equality
expect(value).toStrictEqual(other);   // Deep + strict types
expect(value).toBeTruthy();           // Truthy
expect(value).toBeFalsy();            // Falsy
expect(value).toBeNull();             // null
expect(value).toBeUndefined();        // undefined
expect(value).toBeDefined();          // not undefined
expect(value).toBeNaN();              // NaN
expect(value).toBeGreaterThan(n);     // >
expect(value).toBeLessThan(n);        // <
expect(value).toBeCloseTo(n);         // floating point
expect(str).toMatch(/regex/);         // regex
expect(arr).toContain(item);          // array containment
expect(obj).toHaveProperty("key");    // property
expect(fn).toThrow(Error);            // throws
expect(promise).resolves.toBe(v);     // resolves
expect(promise).rejects.toThrow();    // rejects
```

### Annotated Code Examples

```js
import { describe, it, expect } from "vitest";

describe("assertions", () => {
  it("uses toBe for primitives", () => {
    expect(1 + 1).toBe(2);
    expect("a").toBe("a");
  });

  it("uses toEqual for objects", () => {
    expect({ a: 1, b: 2 }).toEqual({ a: 1, b: 2 });
  });

  it("uses toStrictEqual for strict object equality", () => {
    expect({ a: 1 }).toStrictEqual({ a: 1 });
    expect({ a: undefined }).not.toStrictEqual({});
  });

  it("checks floating point with toBeCloseTo", () => {
    expect(0.1 + 0.2).toBeCloseTo(0.3);
  });

  it("checks errors", () => {
    expect(() => { throw new Error("boom"); }).toThrow("boom");
  });
});
```

**Expected Output:**
```
✓ assertions > uses toBe for primitives
✓ assertions > uses toEqual for objects
✓ assertions > uses toStrictEqual for strict object equality
✓ assertions > checks floating point with toBeCloseTo
✓ assertions > checks errors
```

### Real-World Cases

- **API responses:** Assert status and body.
- **DOM:** Assert element text and attributes.
- **State:** Assert object shape.

---

## Matchers

### Definitions

**Core Definition:** Matchers are the methods invoked on `expect(value)` that perform specific kinds of comparison.

**Technical Definition:** Matchers are provided by the assertion library. Jest/Vitest provide dozens of matchers; custom matchers can be added via `expect.extend`. Matchers handle equality, truthiness, containment, exceptions, and async. Matchers return a result object with `pass` and `message`.

**Beginner-Friendly Explanation:** Matchers are the "verbs" in assertions — `.toBe()`, `.toEqual()`, `.toThrow()`.

### Categories

- **Equality:** `toBe`, `toEqual`, `toStrictEqual`
- **Truthiness:** `toBeTruthy`, `toBeFalsy`, `toBeNull`, `toBeUndefined`, `toBeDefined`, `toBeNaN`
- **Numbers:** `toBeGreaterThan`, `toBeLessThan`, `toBeCloseTo`
- **Strings:** `toMatch`
- **Arrays/Iterables:** `toContain`, `toContainEqual`, `toHaveLength`
- **Objects:** `toHaveProperty`, `toMatchObject`
- **Functions:** `toThrow`, `toHaveBeenCalled`, `toHaveBeenCalledWith`, `toHaveBeenCalledTimes`, `toHaveReturnedWith`
- **Async:** `resolves`, `rejects`
- **Negation:** `.not`
- **Snapshots:** `toMatchSnapshot`, `toMatchInlineSnapshot`

### Annotated Code Examples

```js
test("matchers", () => {
  expect([1, 2, 3]).toContain(2);
  expect([1, 2, 3]).toHaveLength(3);
  expect({ name: "Alice" }).toHaveProperty("name");
  expect({ name: "Alice", age: 30 }).toMatchObject({ name: "Alice" });
  expect("hello world").toMatch(/world/);
  expect(5).toBeGreaterThan(3);
  expect(0.1 + 0.2).toBeCloseTo(0.3);
  expect(() => { throw new TypeError(); }).toThrow(TypeError);
  expect([1, 2]).toEqual(expect.arrayContaining([1]));
  expect({ a: 1, b: 2 }).toEqual(expect.objectContaining({ a: 1 }));
});
```

### Custom Matchers

```js
expect.extend({
  toBeWithinRange(received, floor, ceiling) {
    const pass = received >= floor && received <= ceiling;
    return {
      pass,
      message: () =>
        `expected ${received} to be within ${floor}-${ceiling}`,
    };
  },
});

test("custom matcher", () => {
  expect(100).toBeWithinRange(90, 110);
});
```

### Real-World Cases

- **Domain-specific assertions:** `toBeValidEmail`.
- **API assertions:** `toBeSuccessResponse`.
- **Component assertions:** `toHaveTextContent`.

---

## Test Organization

### Definitions

**Core Definition:** Test organization is the practice of structuring tests for clarity, maintainability, and efficiency.

**Technical Definition:** Tests are organised into files (usually `*.test.js` or `*.spec.js`), grouped with `describe` blocks, and structured with `it`/`test`. Shared setup uses `beforeEach`/`afterEach` (per test) or `beforeAll`/`afterAll` (per suite). Test files mirror source structure (`src/foo.js` → `src/foo.test.js` or `__tests__/foo.test.js`).

**Beginner-Friendly Explanation:** Test organization means arranging your tests so they're easy to find, read, and run.

### Common Patterns

- **Co-located:** `foo.js` and `foo.test.js` in the same folder.
- **`__tests__` folder:** All tests in a dedicated folder.
- **`tests/` folder:** Top-level tests directory.
- **Mirrored structure:** Tests mirror source tree.
- **Fixtures:** Shared test data in `fixtures/`.
- **Helpers:** Shared utilities in `test-utils/`.

### Annotated Code Examples

```
src/
  utils/
    string.js
    string.test.js
  services/
    userService.js
    userService.test.js
  __tests__/
    integration/
      userFlow.test.js
  test-utils/
    fixtures.js
    mocks.js
```

### Real-World Cases

- **Small projects:** Co-located tests.
- **Large projects:** Mirrored structure with `__tests__`.
- **Monorepos:** Per-package test folders.

---

## Test Naming

### Definitions

**Core Definition:** Test naming is the practice of giving tests descriptive names that explain the behaviour being verified.

**Technical Definition:** Test names typically follow the pattern "it [expected behaviour] when [condition]" or "should [do X] when [Y]". Names should be readable as specifications, describing the observable behaviour rather than implementation details.

**Beginner-Friendly Explanation:** Test names should explain what the test checks, in plain language.

### Best Practices

- **Describe behaviour, not implementation.**
- **Use "it" or "should" style.**
- **Include the condition and expected outcome.**
- **Be specific and concise.**
- **Avoid "works" or "test 1".**

### Annotated Code Examples

```js
// GOOD
describe("sum", () => {
  it("returns the sum of two positive numbers", () => {});
  it("returns a negative result when adding a larger negative", () => {});
  it("throws TypeError when given a non-number", () => {});
});

// BAD
describe("sum", () => {
  it("works", () => {});
  it("test 1", () => {});
  it("adds", () => {});
});
```

### Real-World Cases

- **Documentation:** Test names as spec.
- **Failure messages:** Clear names speed debugging.
- **Coverage reports:** Descriptive names aid review.

---

## Test Best Practices

### Definitions

**Core Definition:** Test best practices are the guidelines for writing maintainable, fast, and reliable unit tests.

**Technical Definition:** Best practices include: test one behaviour per test, use descriptive names, keep tests independent, follow Arrange-Act-Assert, avoid logic in tests, prefer pure functions, use test doubles appropriately, test edge cases, avoid snapshot overuse, and keep tests fast. Use `beforeEach` for setup, `afterEach` for cleanup, and avoid shared mutable state.

**Beginner-Friendly Explanation:** These are the habits that make tests reliable and easy to maintain.

### Best Practices

1. **One behaviour per test.**
2. **Descriptive test names.**
3. **Arrange-Act-Assert (AAA) structure.**
   ```js
   it("adds items to the cart", () => {
     // Arrange
     const cart = new Cart();
     // Act
     cart.add({ price: 10 });
     // Assert
     expect(cart.total()).toBe(10);
   });
   ```
4. **Independent tests.**
5. **No logic in tests.**
6. **Test edge cases:** zero, empty, null, negative, large.
7. **Use the right test double.**
8. **Avoid testing implementation details.**
9. **Keep tests fast.**
10. **Clean up after yourself.**
11. **Use `beforeEach` for per-test setup.**
12. **Avoid snapshot overuse.**
13. **Run tests in CI.**
14. **Fix broken tests immediately.**
15. **Aim for meaningful coverage, not 100%.**

### Annotated Code Examples

#### Example 1: AAA Structure

```js
it("withdraws money from the account", () => {
  // Arrange
  const account = new BankAccount(100);

  // Act
  account.withdraw(30);

  // Assert
  expect(account.balance).toBe(70);
});
```

#### Example 2: Testing Edge Cases

```js
describe("divide", () => {
  it("divides positive numbers", () => {
    expect(divide(10, 2)).toBe(5);
  });

  it("divides negative numbers", () => {
    expect(divide(-10, 2)).toBe(-5);
  });

  it("returns Infinity for division by zero", () => {
    expect(divide(10, 0)).toBe(Infinity);
  });

  it("returns NaN for zero divided by zero", () => {
    expect(divide(0, 0)).toBeNaN();
  });
});
```

#### Example 3: Avoiding Implementation Details

```js
// BAD: Tests internal implementation.
it("calls _validate before saving", () => {
  const spy = vi.spyOn(service, "_validate");
  service.save(user);
  expect(spy).toHaveBeenCalled();
});

// GOOD: Tests observable behaviour.
it("throws when saving an invalid user", () => {
  expect(() => service.save({})).toThrow("Invalid user");
});
```

### Real-World Cases

- **CI/CD:** Fast, reliable tests.
- **Code review:** Clear, focused tests.
- **Refactoring:** Tests catch regressions.
- **Onboarding:** Tests document behaviour.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://tc39.es/ecma262/
- MDN Web Docs – Testing — https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Testing
- Jest Documentation — https://jestjs.io/docs/getting-started
- Jest – Expect API — https://jestjs.io/docs/expect
- Jest – Mock Functions — https://jestjs.io/docs/mock-functions
- Jest – Async Testing — https://jestjs.io/docs/asynchronous
- Vitest Documentation — https://vitest.dev/
- Vitest – Expect — https://vitest.dev/api/expect.html
- Vitest – Mocking — https://vitest.dev/guide/mocking.html
- Vitest – Test API — https://vitest.dev/api/
- Mocha Documentation — https://mochajs.org/
- Jasmine Documentation — https://jasmine.github.io/
- Node.js Documentation – Test runner — https://nodejs.org/api/test.html
- Node.js Documentation – assert — https://nodejs.org/api/assert.html
- Chai Documentation — https://www.chaijs.com/
- Sinon.JS Documentation — https://sinonjs.org/
- Testing Library Documentation — https://testing-library.com/
- MSW (Mock Service Worker) — https://mswjs.io/
- nock — https://github.com/nock/nock
- Martin Fowler – Test Double — https://martinfowler.com/bliki/TestDouble.html
- Martin Fowler – Mocks Aren't Stubs — https://martinfowler.com/articles/mocksArentStubs.html
- Kent C. Dodds – Testing Trophy — https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications
- Kent C. Dodds – Testing Implementation Details — https://kentcdodds.com/blog/testing-implementation-details
- Gerard Meszaros – xUnit Test Patterns — http://xunitpatterns.com/
- Arrange-Act-Assert — https://automationpanda.com/2020/07/07/arrange-act-assert-a-pattern-for-writing-good-tests/
- Istanbul (nyc) — https://istanbul.js.org/
- c8 — https://github.com/bcoe/c8