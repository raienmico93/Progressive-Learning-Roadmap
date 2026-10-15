# JavaScript Integration Testing — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Integration testing is the practice of verifying that two or more units, modules, or components work together correctly — testing the interactions and boundaries between them rather than their isolated behaviour.

**Technical Definition:** Integration tests exercise the seams between components: module-to-module calls, service-to-database queries, frontend-to-API requests, and component-to-DOM interactions. Unlike unit tests, integration tests use real (or realistic) dependencies — a test database, a mock HTTP server, a real DOM — to catch interface mismatches, data-format errors, and configuration problems that unit tests miss. In the test pyramid, integration tests sit between unit tests (many, fast, isolated) and end-to-end tests (few, slow, full-stack). Frameworks: Vitest, Jest, Mocha, `node:test` for Node.js; Testing Library, Playwright Component Testing, Cypress Component Testing for UI. Real dependencies: `pg`, `mongodb`, `ioredis`, `supertest`, MSW.

**Beginner-Friendly Explanation:** Unit tests check one piece of code at a time. Integration tests check that pieces work together — like a function that talks to a database, a component that renders in a browser, or a form that submits to an API. Integration tests catch problems that only appear when parts are combined: wrong data shapes, missing configuration, timing issues, or mismatched contracts. They're slower than unit tests but give much higher confidence that your app actually works.

### Key Characteristics

- **Multiple units:** Tests exercise interactions between components.
- **Realistic dependencies:** Uses real DBs, servers, or DOM where practical.
- **Slower than unit tests:** Typically seconds per test, not milliseconds.
- **Fewer than unit tests:** Middle layer of the test pyramid.
- **Higher confidence:** Catches interface and integration issues.
- **Stateful:** Often requires setup/teardown of databases or servers.
- **Environment-sensitive:** May depend on network, filesystem, or browser.
- **Complements unit and E2E:** Not a replacement for either.

### Prerequisites

- Solid understanding of unit testing.
- Familiarity with test runners (Vitest, Jest, Mocha).
- Knowledge of modules, async/await, Promises.
- Experience with the DOM and browser APIs.
- Basic knowledge of databases and HTTP.

### Related Programming Areas

- **Unit testing:** The layer below integration.
- **End-to-end testing:** The layer above integration.
- **Contract testing:** Verifying service agreements.
- **Component testing:** UI integration in isolation.
- **Test databases:** Real or in-memory databases for tests.
- **Mock servers:** MSW, nock, json-server.

---

## Testing Components

### Definitions

**Core Definition:** Component testing verifies that a UI component renders correctly, responds to user interactions, and integrates with its dependencies.

**Technical Definition:** Component tests mount a component in a realistic environment (real DOM or jsdom/happy-dom), render it with props or state, simulate user events, and assert on the rendered output. Tools: React Testing Library, Vue Test Utils, Svelte Testing Library, Angular TestBed, Playwright Component Testing. Component tests are a form of integration testing because they exercise the component's template, logic, styles, and child components together.

**Beginner-Friendly Explanation:** Component testing means rendering a UI component in a browser-like environment and checking that it displays correctly and responds to clicks, typing, and other interactions.

### Purposes

- To verify rendering with props.
- To test user interactions (clicks, input, submit).
- To verify integration with child components.
- To test state changes and re-renders.
- To catch accessibility issues.

### Syntax Rules and Structure

```jsx
import { render, screen, fireEvent } from "@testing-library/react";
import { Button } from "./Button";

test("renders and handles click", () => {
  const handleClick = vi.fn();
  render(<Button label="Click" onClick={handleClick} />);
  fireEvent.click(screen.getByText("Click"));
  expect(handleClick).toHaveBeenCalled();
});
```

### Annotated Code Examples

#### Example 1: React Component Test

```jsx
// Counter.jsx
import { useState } from "react";

export function Counter({ initial = 0 }) {
  const [count, setCount] = useState(initial);
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

```jsx
// Counter.test.jsx
import { describe, it, expect } from "vitest";
import { render, screen, fireEvent } from "@testing-library/react";
import { Counter } from "./Counter";

describe("Counter", () => {
  it("renders the initial count", () => {
    render(<Counter initial={5} />);
    expect(screen.getByText("Count: 5")).toBeInTheDocument();
  });

  it("increments on click", () => {
    render(<Counter initial={0} />);
    fireEvent.click(screen.getByText("Increment"));
    expect(screen.getByText("Count: 1")).toBeInTheDocument();
  });

  it("increments multiple times", () => {
    render(<Counter initial={0} />);
    const btn = screen.getByText("Increment");
    fireEvent.click(btn);
    fireEvent.click(btn);
    fireEvent.click(btn);
    expect(screen.getByText("Count: 3")).toBeInTheDocument();
  });
});
```

**Expected Output:**
```
✓ Counter > renders the initial count
✓ Counter > increments on click
✓ Counter > increments multiple times
```

**Why This Output:** The component is rendered in a jsdom environment; `fireEvent` simulates real clicks; assertions check the DOM.

#### Example 2: Vue Component Test

```js
// Counter.vue
<template>
  <div>
    <p>Count: {{ count }}</p>
    <button @click="count++">Increment</button>
  </div>
</template>

<script setup>
import { ref } from "vue";
const count = ref(0);
</script>
```

```js
// Counter.test.js
import { describe, it, expect } from "vitest";
import { mount } from "@vue/test-utils";
import Counter from "./Counter.vue";

describe("Counter", () => {
  it("increments on click", async () => {
    const wrapper = mount(Counter);
    expect(wrapper.text()).toContain("Count: 0");
    await wrapper.find("button").trigger("click");
    expect(wrapper.text()).toContain("Count: 1");
  });
});
```

#### Example 3: Testing with Providers and Context

```jsx
// ThemeProvider.test.jsx
import { render, screen } from "@testing-library/react";
import { ThemeProvider } from "./ThemeContext";
import { ThemedButton } from "./ThemedButton";

it("renders with theme from provider", () => {
  render(
    <ThemeProvider value="dark">
      <ThemedButton />
    </ThemeProvider>
  );
  expect(screen.getByRole("button")).toHaveClass("dark");
});
```

### Real-World Cases

- **Forms:** Input validation, submit handling.
- **Lists:** Rendering, filtering, sorting.
- **Modals:** Open/close, focus management.
- **Routing:** Link navigation.

---

## Testing Modules Together

### Definitions

**Core Definition:** Testing modules together verifies that multiple modules integrate correctly through their public APIs.

**Technical Definition:** Multi-module integration tests import several modules and exercise the interactions between them: a service module calling a repository module, a controller calling a service, a utility module used by a parser. Unlike unit tests, dependencies are real (not mocked), or only external dependencies (database, HTTP) are mocked.

**Beginner-Friendly Explanation:** Testing modules together means combining two or more modules and verifying that they cooperate correctly.

### Purposes

- To verify module interfaces.
- To catch integration bugs.
- To test data flow across layers.
- To validate contracts between modules.

### Annotated Code Examples

#### Example 1: Service + Repository Integration

```js
// repository.js
export class InMemoryUserRepository {
  #users = new Map();
  async save(user) { this.#users.set(user.id, user); return user; }
  async findById(id) { return this.#users.get(id) ?? null; }
  async findAll() { return [...this.#users.values()]; }
}

// userService.js
export class UserService {
  constructor(repo) { this.repo = repo; }

  async createUser({ name, email }) {
    if (!name || !email) throw new Error("Name and email are required");
    const user = { id: Date.now(), name, email, createdAt: new Date() };
    return this.repo.save(user);
  }

  async getUser(id) {
    const user = await this.repo.findById(id);
    if (!user) throw new Error("User not found");
    return user;
  }
}
```

```js
// integration.test.js
import { describe, it, expect, beforeEach } from "vitest";
import { InMemoryUserRepository } from "./repository.js";
import { UserService } from "./userService.js";

describe("UserService + Repository integration", () => {
  let service;
  let repo;

  beforeEach(() => {
    repo = new InMemoryUserRepository();
    service = new UserService(repo);
  });

  it("creates and retrieves a user", async () => {
    const created = await service.createUser({ name: "Alice", email: "alice@example.com" });
    const fetched = await service.getUser(created.id);
    expect(fetched.name).toBe("Alice");
    expect(fetched.email).toBe("alice@example.com");
  });

  it("throws when creating an invalid user", async () => {
    await expect(service.createUser({ name: "" })).rejects.toThrow("Name and email are required");
  });

  it("throws when user not found", async () => {
    await expect(service.getUser(999)).rejects.toThrow("User not found");
  });
});
```

**Expected Output:**
```
✓ UserService + Repository integration > creates and retrieves a user
✓ UserService + Repository integration > throws when creating an invalid user
✓ UserService + Repository integration > throws when user not found
```

**Why This Output:** The test uses a real repository (an in-memory fake) with the real service, verifying data flows through both correctly.

#### Example 2: Parser + Validator + Formatter

```js
// parser.js
export function parseQuery(str) {
  return Object.fromEntries(new URLSearchParams(str));
}

// validator.js
export function validateQuery(query, required = []) {
  for (const key of required) {
    if (!query[key]) throw new Error(`Missing required query param: ${key}`);
  }
  return query;
}

// formatter.js
export function formatQuery(query) {
  return Object.entries(query).map(([k, v]) => `${k}=${v}`).join("&");
}
```

```js
// pipeline.test.js
import { describe, it, expect } from "vitest";
import { parseQuery } from "./parser.js";
import { validateQuery } from "./validator.js";
import { formatQuery } from "./formatter.js";

describe("query pipeline", () => {
  it("parses, validates, and formats", () => {
    const parsed = parseQuery("name=Alice&age=30");
    const validated = validateQuery(parsed, ["name"]);
    const formatted = formatQuery(validated);
    expect(formatted).toBe("name=Alice&age=30");
  });

  it("throws when required param missing", () => {
    const parsed = parseQuery("age=30");
    expect(() => validateQuery(parsed, ["name"])).toThrow("Missing required query param: name");
  });
});
```

### Real-World Cases

- **API layers:** Controller + service + repository.
- **Pipelines:** Parse → validate → transform.
- **Event systems:** Publisher + subscriber.
- **State management:** Action → reducer → store.

---

## Testing with Real Dependencies

### Definitions

**Core Definition:** Testing with real dependencies uses actual databases, HTTP servers, or file systems rather than test doubles.

**Technical Definition:** Real-dependency integration tests exercise the actual implementation of dependencies (a real PostgreSQL database, a real HTTP server), typically in a controlled test environment. This catches issues that mocks cannot: query syntax errors, serialisation bugs, network behaviour, and configuration problems. Trade-offs: slower, requires setup/teardown, may need Docker or CI services.

**Beginner-Friendly Explanation:** Testing with real dependencies means using the actual database, server, or file system instead of a fake, so you catch problems that only appear with the real thing.

### Purposes

- To verify real queries work.
- To catch serialisation issues.
- To test configuration.
- To validate migrations.
- To ensure production parity.

### Annotated Code Examples

#### Example 1: Testing with a Real Database (Node.js + pg)

```js
// db.js
import pg from "pg";
export const pool = new pg.Pool({
  connectionString: process.env.DATABASE_URL,
});

// userRepo.js
export async function createUser(name, email) {
  const result = await pool.query(
    "INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *",
    [name, email]
  );
  return result.rows[0];
}

export async function findUser(id) {
  const result = await pool.query("SELECT * FROM users WHERE id = $1", [id]);
  return result.rows[0] ?? null;
}
```

```js
// userRepo.integration.test.js
import { describe, it, expect, beforeAll, afterAll, beforeEach } from "vitest";
import { pool } from "./db.js";
import { createUser, findUser } from "./userRepo.js";

describe("UserRepo (real DB)", () => {
  beforeAll(async () => {
    await pool.query(`
      CREATE TABLE IF NOT EXISTS users (
        id SERIAL PRIMARY KEY,
        name TEXT NOT NULL,
        email TEXT UNIQUE NOT NULL
      )
    `);
  });

  beforeEach(async () => {
    await pool.query("TRUNCATE users RESTART IDENTITY CASCADE");
  });

  afterAll(async () => {
    await pool.query("DROP TABLE IF EXISTS users");
    await pool.end();
  });

  it("creates and retrieves a user", async () => {
    const created = await createUser("Alice", "alice@example.com");
    expect(created.id).toBeDefined();
    const fetched = await findUser(created.id);
    expect(fetched.name).toBe("Alice");
  });
});
```

**Why This Helps:** Uses the real PostgreSQL driver and real SQL, catching query and schema issues that mocks cannot.

#### Example 2: Testing with a Real File System

```js
// fileStore.js
import { readFile, writeFile, mkdir } from "node:fs/promises";
import { join } from "node:path";

export class FileStore {
  constructor(dir) { this.dir = dir; }

  async init() { await mkdir(this.dir, { recursive: true }); }

  async write(key, value) {
    await writeFile(join(this.dir, key), JSON.stringify(value));
  }

  async read(key) {
    const data = await readFile(join(this.dir, key), "utf8");
    return JSON.parse(data);
  }
}
```

```js
// fileStore.test.js
import { describe, it, expect, beforeEach, afterEach } from "vitest";
import { mkdtemp, rm } from "node:fs/promises";
import { tmpdir } from "node:os";
import { join } from "node:path";
import { FileStore } from "./fileStore.js";

describe("FileStore", () => {
  let dir, store;

  beforeEach(async () => {
    dir = await mkdtemp(join(tmpdir(), "filestore-"));
    store = new FileStore(dir);
    await store.init();
  });

  afterEach(async () => {
    await rm(dir, { recursive: true, force: true });
  });

  it("writes and reads a value", async () => {
    await store.write("user.json", { name: "Alice" });
    const data = await store.read("user.json");
    expect(data.name).toBe("Alice");
  });
});
```

### Real-World Cases

- **SQL databases:** PostgreSQL, MySQL, SQLite.
- **NoSQL:** MongoDB, Redis.
- **File systems:** Config, uploads.
- **Queues:** RabbitMQ, SQS.

---

## Testing with Test Databases

### Definitions

**Core Definition:** Testing with test databases uses a dedicated, isolated database instance for integration tests.

**Technical Definition:** Test databases are separate instances (or schemas) that are created, migrated, seeded, and torn down per test run. Common approaches: Docker containers (Testcontainers), in-memory databases (SQLite `:memory:`, `mongodb-memory-server`), or cloud test databases. Best practices: isolate tests with transactions rolled back per test, truncate tables in `beforeEach`, or use unique schemas.

**Beginner-Friendly Explanation:** A test database is a separate database just for tests, so you never touch production data.

### Purposes

- To isolate tests from production data.
- To ensure reproducible test runs.
- To catch schema and query issues.
- To test migrations.

### Annotated Code Examples

#### Example 1: In-Memory SQLite

```js
import { describe, it, expect, beforeEach, afterEach } from "vitest";
import Database from "better-sqlite3";

describe("UserRepo (SQLite)", () => {
  let db;

  beforeEach(() => {
    db = new Database(":memory:");
    db.exec(`
      CREATE TABLE users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        email TEXT UNIQUE NOT NULL
      )
    `);
  });

  afterEach(() => { db.close(); });

  it("inserts and retrieves", () => {
    const insert = db.prepare("INSERT INTO users (name, email) VALUES (?, ?)");
    const info = insert.run("Alice", "alice@example.com");
    const select = db.prepare("SELECT * FROM users WHERE id = ?");
    const user = select.get(info.lastInsertRowid);
    expect(user.name).toBe("Alice");
  });
});
```

#### Example 2: MongoDB Memory Server

```js
import { describe, it, expect, beforeAll, afterAll, beforeEach } from "vitest";
import { MongoMemoryServer } from "mongodb-memory-server";
import { MongoClient } from "mongodb";

describe("UserRepo (MongoDB)", () => {
  let mongod, client, db;

  beforeAll(async () => {
    mongod = await MongoMemoryServer.create();
    client = new MongoClient(mongod.getUri());
    await client.connect();
    db = client.db("test");
  });

  beforeEach(async () => {
    await db.collection("users").deleteMany({});
  });

  afterAll(async () => {
    await client.close();
    await mongod.stop();
  });

  it("inserts and finds", async () => {
    await db.collection("users").insertOne({ name: "Alice" });
    const user = await db.collection("users").findOne({ name: "Alice" });
    expect(user.name).toBe("Alice");
  });
});
```

### Real-World Cases

- **CI pipelines:** Spin up DB containers.
- **Local dev:** In-memory DBs for speed.
- **Migration tests:** Verify schema changes.

---

## Testing API Calls

### Definitions

**Core Definition:** Testing API calls verifies that HTTP requests are constructed, sent, and handled correctly.

**Technical Definition:** API integration tests use real HTTP clients (fetch, axios) against a real server (your app or a mock server like MSW or nock) to verify request construction, response parsing, error handling, and retries. Tools: `supertest` (Node HTTP servers), MSW (browser and Node), `nock` (Node), `msw/node`.

**Beginner-Friendly Explanation:** Testing API calls means checking that your code sends the right request and handles the response correctly.

### Purposes

- To verify request URLs, headers, and bodies.
- To test response parsing.
- To test error handling (4xx, 5xx, network errors).
- To test retries and timeouts.

### Annotated Code Examples

#### Example 1: Testing an Express API with Supertest

```js
// app.js
import express from "express";
const app = express();
app.use(express.json());

const users = new Map();

app.post("/api/users", (req, res) => {
  const { name, email } = req.body;
  if (!name || !email) return res.status(400).json({ error: "Missing fields" });
  const user = { id: Date.now(), name, email };
  users.set(user.id, user);
  res.status(201).json(user);
});

app.get("/api/users/:id", (req, res) => {
  const user = users.get(Number(req.params.id));
  if (!user) return res.status(404).json({ error: "Not found" });
  res.json(user);
});

export default app;
```

```js
// app.test.js
import { describe, it, expect } from "vitest";
import request from "supertest";
import app from "./app.js";

describe("Users API", () => {
  it("creates a user", async () => {
    const res = await request(app)
      .post("/api/users")
      .send({ name: "Alice", email: "alice@example.com" });
    expect(res.status).toBe(201);
    expect(res.body.name).toBe("Alice");
  });

  it("returns 400 for missing fields", async () => {
    const res = await request(app).post("/api/users").send({ name: "Alice" });
    expect(res.status).toBe(400);
    expect(res.body.error).toBe("Missing fields");
  });

  it("returns 404 for unknown user", async () => {
    const res = await request(app).get("/api/users/999");
    expect(res.status).toBe(404);
  });
});
```

**Expected Output:**
```
✓ Users API > creates a user
✓ Users API > returns 400 for missing fields
✓ Users API > returns 404 for unknown user
```

#### Example 2: Stubbing HTTP with MSW

```js
// handlers.js
import { http, HttpResponse } from "msw";

export const handlers = [
  http.get("/api/users/:id", ({ params }) => {
    if (params.id === "999") {
      return new HttpResponse(null, { status: 404 });
    }
    return HttpResponse.json({ id: Number(params.id), name: "Alice" });
  }),
];
```

```js
// apiClient.test.js
import { describe, it, expect, beforeAll, afterAll, afterEach } from "vitest";
import { setupServer } from "msw/node";
import { handlers } from "./handlers.js";
import { getUser } from "./apiClient.js";

const server = setupServer(...handlers);
beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe("getUser", () => {
  it("returns user data", async () => {
    const user = await getUser(1);
    expect(user.name).toBe("Alice");
  });

  it("throws on 404", async () => {
    await expect(getUser(999)).rejects.toThrow();
  });
});
```

### Real-World Cases

- **REST APIs:** Verify endpoints.
- **GraphQL:** Verify queries/mutations.
- **Auth:** Verify headers and tokens.
- **Retries:** Verify exponential backoff.

---

## Testing DOM Interactions

### Definitions

**Core Definition:** Testing DOM interactions verifies that code correctly manipulates the DOM in response to user events and state changes.

**Technical Definition:** DOM integration tests run in a browser-like environment (jsdom, happy-dom, or a real browser via Playwright/Cypress) and exercise interactions: clicks, keyboard input, form submission, focus, drag-and-drop. Tools: Testing Library (React/Vue/Svelte), jsdom, happy-dom, Playwright Component Testing.

**Beginner-Friendly Explanation:** Testing DOM interactions means simulating user actions like clicks and typing, then checking that the DOM updates correctly.

### Purposes

- To verify event handlers.
- To test form validation and submission.
- To test focus management.
- To test accessibility.

### Annotated Code Examples

#### Example 1: Form Submission

```jsx
// LoginForm.jsx
import { useState } from "react";

export function LoginForm({ onSubmit }) {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState("");

  function handleSubmit(e) {
    e.preventDefault();
    if (!email.includes("@")) {
      setError("Invalid email");
      return;
    }
    onSubmit({ email, password });
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        aria-label="Email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      <input
        aria-label="Password"
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      {error && <p role="alert">{error}</p>}
      <button type="submit">Log in</button>
    </form>
  );
}
```

```jsx
// LoginForm.test.jsx
import { describe, it, expect, vi } from "vitest";
import { render, screen, fireEvent } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { LoginForm } from "./LoginForm";

describe("LoginForm", () => {
  it("submits valid credentials", async () => {
    const onSubmit = vi.fn();
    render(<LoginForm onSubmit={onSubmit} />);

    await userEvent.type(screen.getByLabelText("Email"), "alice@example.com");
    await userEvent.type(screen.getByLabelText("Password"), "secret");
    await userEvent.click(screen.getByText("Log in"));

    expect(onSubmit).toHaveBeenCalledWith({
      email: "alice@example.com",
      password: "secret",
    });
  });

  it("shows error for invalid email", async () => {
    render(<LoginForm onSubmit={vi.fn()} />);
    await userEvent.type(screen.getByLabelText("Email"), "invalid");
    await userEvent.click(screen.getByText("Log in"));
    expect(screen.getByRole("alert")).toHaveTextContent("Invalid email");
  });
});
```

**Expected Output:**
```
✓ LoginForm > submits valid credentials
✓ LoginForm > shows error for invalid email
```

#### Example 2: Focus Management

```jsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";

it("focuses the first input on mount", () => {
  render(<Form />);
  expect(screen.getByLabelText("Email")).toHaveFocus();
});
```

### Real-World Cases

- **Forms:** Validation, submission.
- **Modals:** Focus trap, escape key.
- **Autocomplete:** Keyboard navigation.
- **Drag-and-drop:** File uploads.

---

## Testing Browser APIs

### Definitions

**Core Definition:** Testing browser APIs verifies code that uses `localStorage`, `fetch`, `geolocation`, `IntersectionObserver`, `Notification`, and other browser-provided interfaces.

**Technical Definition:** Browser API tests run in jsdom/happy-dom (which implement many APIs) or a real browser (Playwright/Cypress). APIs not implemented in jsdom must be mocked or stubbed. Common techniques: `vi.stubGlobal`, defining properties on `globalThis`, using Playwright's real browser context.

**Beginner-Friendly Explanation:** Testing browser APIs means checking that code using things like `localStorage` or `fetch` works correctly.

### Annotated Code Examples

#### Example 1: Mocking localStorage

```js
// themeStore.js
export function getTheme() {
  return localStorage.getItem("theme") ?? "light";
}

export function setTheme(theme) {
  localStorage.setItem("theme", theme);
}
```

```js
// themeStore.test.js
import { describe, it, expect, beforeEach, vi } from "vitest";
import { getTheme, setTheme } from "./themeStore.js";

describe("themeStore", () => {
  let store;
  beforeEach(() => {
    store = {};
    vi.stubGlobal("localStorage", {
      getItem: (k) => store[k] ?? null,
      setItem: (k, v) => { store[k] = String(v); },
      removeItem: (k) => { delete store[k]; },
    });
  });

  it("returns default when unset", () => {
    expect(getTheme()).toBe("light");
  });

  it("stores and retrieves theme", () => {
    setTheme("dark");
    expect(getTheme()).toBe("dark");
  });
});
```

#### Example 2: Mocking `fetch` and `IntersectionObserver`

```js
import { describe, it, expect, vi, beforeEach, afterEach } from "vitest";
import { fetchData } from "./api.js";

describe("fetchData", () => {
  beforeEach(() => {
    vi.stubGlobal("fetch", vi.fn().mockResolvedValue({
      ok: true,
      json: async () => ({ data: "ok" }),
    }));
  });
  afterEach(() => vi.unstubAllGlobals());

  it("fetches and parses JSON", async () => {
    const result = await fetchData("/api/data");
    expect(result.data).toBe("ok");
    expect(fetch).toHaveBeenCalledWith("/api/data");
  });
});
```

#### Example 3: Real Browser API with Playwright

```js
import { test, expect } from "@playwright/test";

test("stores theme in localStorage", async ({ page }) => {
  await page.goto("https://app.example.com");
  await page.click("text=Dark Mode");
  const theme = await page.evaluate(() => localStorage.getItem("theme"));
  expect(theme).toBe("dark");
});
```

### Real-World Cases

- **Storage:** localStorage, sessionStorage, IndexedDB.
- **Network:** fetch, XMLHttpRequest.
- **Sensors:** geolocation, device orientation.
- **Observers:** IntersectionObserver, ResizeObserver.
- **Notifications:** Notification API.

---

## References

- ECMAScript Language Specification (ECMA-262, 16th Edition, June 2025) — https://tc39.es/ecma262/
- MDN Web Docs – Testing — https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Testing
- Jest Documentation — https://jestjs.io/docs/getting-started
- Vitest Documentation — https://vitest.dev/
- Vitest – Mocking — https://vitest.dev/guide/mocking.html
- Testing Library Documentation — https://testing-library.com/
- Testing Library – React — https://testing-library.com/docs/react-testing-library/intro/
- Testing Library – Vue — https://testing-library.com/docs/vue-testing-library/intro/
- Vue Test Utils — https://test-utils.vuejs.org/
- MSW (Mock Service Worker) — https://mswjs.io/
- nock — https://github.com/nock/nock
- Supertest — https://github.com/ladjs/supertest
- Playwright Documentation — https://playwright.dev/
- Playwright Component Testing — https://playwright.dev/docs/test-components
- Cypress Documentation — https://docs.cypress.io/
- Cypress Component Testing — https://docs.cypress.io/guides/component-testing/overview
- jsdom — https://github.com/jsdom/jsdom
- happy-dom — https://github.com/capricorn86/happy-dom
- Testcontainers — https://testcontainers.com/
- mongodb-memory-server — https://github.com/typegoose/mongodb-memory-server
- better-sqlite3 — https://github.com/WiseLibs/better-sqlite3
- Node.js Documentation – Test runner — https://nodejs.org/api/test.html
- Martin Fowler – Integration Test — https://martinfowler.com/bliki/IntegrationTest.html
- Martin Fowler – Test Pyramid — https://martinfowler.com/bliki/TestPyramid.html
- Kent C. Dodds – Testing Trophy — https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications
- Kent C. Dodds – Testing Implementation Details — https://kentcdodds.com/blog/testing-implementation-details