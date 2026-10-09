# Unit Testing — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Unit testing is the practice of verifying the behaviour of the smallest testable units of an application — individual functions, methods, classes, or modules — in isolation from their dependencies and external systems.

**Technical Definition:** A unit test exercises a single unit of code (a function, a service method, a utility module) by invoking it with controlled inputs and asserting on its outputs, side effects, or interactions with collaborators. Dependencies (database clients, HTTP clients, third-party SDKs) are replaced with test doubles — mocks, stubs, or spies — so that failures in the unit test point to the unit under test and nothing else. In Node.js, Jest is the dominant framework, providing built-in assertions, mocking, spying, snapshot testing, and coverage reporting with zero configuration. Jest runs tests in parallel by default and uses `describe` blocks for grouping and `beforeEach`/`afterEach` for setup and cleanup.

**Beginner-Friendly Explanation:** A unit test is like testing a single component of a car — the fuel pump — on a workbench, without the rest of the car. You feed it fuel at a known pressure, check that it delivers the right amount, and confirm it doesn't leak. If it fails, you know the problem is in the fuel pump, not the engine or the wiring. In code, unit tests isolate one function or one service, replace everything it depends on with fakes, and check that it behaves correctly.

### Key Characteristics

- **Isolation:** The unit under test is exercised without its real dependencies (database, HTTP, file system).
- **Speed:** Unit tests run in milliseconds because they avoid network and disk I/O.
- **Determinism:** Mocked dependencies return fixed values, so tests produce the same result every run.
- **Granularity:** Each test targets one behaviour of one unit, not an entire workflow.
- **Fast feedback:** Failures point to a specific unit, making debugging faster.
- **Coverage measurement:** Tools like Jest's `--coverage` report which lines and branches are exercised.

### Prerequisites

- **Node.js runtime** (v18 or higher).
- **Jest installed:** `npm install --save-dev jest`.
- **Basic understanding of JavaScript modules:** `require`/`module.exports` or ESM imports.
- **Familiarity with async/await and Promises.**
- **A test script in `package.json`:** `"test": "jest"`.

### Related Programming Areas

- **Integration testing:** Tests that exercise multiple units together, often with a real database.
- **End-to-end testing:** Tests that exercise the full HTTP stack, often with Supertest.
- **Mocking libraries:** Jest's built-in mocks, Sinon, Nock (HTTP mocking), `aws-sdk-client-mock`.
- **Test-driven development (TDD):** Writing tests before implementation.
- **Coverage tooling:** Istanbul (nyc), Jest's built-in coverage.

### Core Concepts

1. **Testing Services** — verifying business logic in isolation.
2. **Testing Utilities** — pure functions, formatters, validators.
3. **Mocking Dependencies** — replacing collaborators with controlled fakes.
4. **Spying and Stubbing** — verifying internal method calls and replacing behaviour.
5. **Mocking Global/Third-Party Modules** — Axios, AWS SDK, Stripe.
6. **Testing Custom Error Classes** — verifying error types and instantiation.

---

## Core Concept 1: Testing Services

### Definitions

**Core Definition:** A service is a class or module that encapsulates business logic and orchestrates interactions between data repositories, external APIs, and domain models. Testing a service means verifying that its methods produce the correct outputs, call the correct collaborators, and handle error conditions correctly.

**Technical Definition:** In layered architectures (controllers → services → repositories), services contain the application's business rules. Unit testing a service replaces its injected dependencies (repositories, HTTP clients, SDKs) with mock implementations, then asserts on the service's outputs and on the calls it made to its collaborators. This ensures the business logic is correct independently of infrastructure concerns.

**Beginner-Friendly Explanation:** A service is the "brain" of a feature — it decides what should happen. When you test it, you plug in fake dependencies that behave however you want, then check that the brain makes the right decisions. You never touch a real database or API.

### Purposes

- To verify business logic independently of infrastructure.
- To ensure the service calls its dependencies with the correct arguments.
- To test error handling paths (e.g., "what happens when the repository returns null?").
- To enable refactoring of infrastructure without breaking business logic tests.

### Syntax Rules and Structure

**Service with injected dependency:**
```js
// user.service.js
class UserService {
  constructor(database) {
    this.db = database;  // Injected dependency
  }

  async getUser(id) {
    const user = await this.db.findById(id);
    if (!user) {
      throw new Error('User not found');
    }
    return user;
  }
}

module.exports = UserService;
```

**Test with mocked dependency:**
```js
// user.service.test.js
const UserService = require('./user.service');

describe('UserService', () => {
  let service;
  let mockDatabase;

  beforeEach(() => {
    mockDatabase = { findById: jest.fn() };
    service = new UserService(mockDatabase);
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  it('should return user when found', async () => {
    const mockUser = { id: 1, name: 'John' };
    mockDatabase.findById.mockResolvedValue(mockUser);

    const result = await service.getUser(1);

    expect(result).toEqual(mockUser);
    expect(mockDatabase.findById).toHaveBeenCalledWith(1);
    expect(mockDatabase.findById).toHaveBeenCalledTimes(1);
  });

  it('should throw error when user not found', async () => {
    mockDatabase.findById.mockResolvedValue(null);

    await expect(service.getUser(1)).rejects.toThrow('User not found');
  });
});
```

| Pattern | Purpose |
|---------|---------|
| `beforeEach` | Create fresh mocks and service instance before each test. |
| `afterEach` | Clear mock state to prevent test pollution. |
| `mockResolvedValue` | Define what an async mock returns. |
| `mockRejectedValue` | Define what an async mock rejects with. |
| `toHaveBeenCalledWith` | Assert the mock was called with specific arguments. |

**Rules:**
- Always inject dependencies via the constructor (dependency injection) to make the service testable.
- Reset mocks between tests with `jest.clearAllMocks()` to prevent call-count pollution.
- Test both the happy path and error paths.
- Assert on both the returned value and the calls made to dependencies.
- Use `mockResolvedValue` for async methods and `mockReturnValue` for sync methods.

### Annotated Code Example

```js
// services/orderService.js
class OrderService {
  constructor(orderRepo, paymentGateway, inventoryService) {
    this.orderRepo = orderRepo;
    this.paymentGateway = paymentGateway;
    this.inventoryService = inventoryService;
  }

  async placeOrder(userId, items) {
    // Check inventory
    const available = await this.inventoryService.checkStock(items);
    if (!available) {
      throw new Error('Insufficient stock');
    }

    // Process payment
    const payment = await this.paymentGateway.charge(userId, items.total);
    if (!payment.success) {
      throw new Error('Payment failed');
    }

    // Create order
    const order = await this.orderRepo.create({
      userId,
      items,
      paymentId: payment.id,
      status: 'confirmed'
    });

    return order;
  }
}

module.exports = OrderService;
```

```js
// services/orderService.test.js
const OrderService = require('./orderService');

describe('OrderService.placeOrder', () => {
  let service;
  let mockOrderRepo;
  let mockPaymentGateway;
  let mockInventoryService;

  beforeEach(() => {
    mockOrderRepo = { create: jest.fn() };
    mockPaymentGateway = { charge: jest.fn() };
    mockInventoryService = { checkStock: jest.fn() };
    service = new OrderService(mockOrderRepo, mockPaymentGateway, mockInventoryService);
  });

  afterEach(() => jest.clearAllMocks());

  it('should place order successfully when stock and payment succeed', async () => {
    const items = { total: 5000, products: [{ id: 1, qty: 2 }] };
    const mockPayment = { success: true, id: 'pay_123' };
    const mockOrder = { id: 'order_1', userId: 1, items, status: 'confirmed' };

    mockInventoryService.checkStock.mockResolvedValue(true);
    mockPaymentGateway.charge.mockResolvedValue(mockPayment);
    mockOrderRepo.create.mockResolvedValue(mockOrder);

    const result = await service.placeOrder(1, items);

    expect(result).toEqual(mockOrder);
    expect(mockInventoryService.checkStock).toHaveBeenCalledWith(items);
    expect(mockPaymentGateway.charge).toHaveBeenCalledWith(1, 5000);
    expect(mockOrderRepo.create).toHaveBeenCalledWith({
      userId: 1,
      items,
      paymentId: 'pay_123',
      status: 'confirmed'
    });
  });

  it('should throw error when stock is insufficient', async () => {
    mockInventoryService.checkStock.mockResolvedValue(false);

    await expect(service.placeOrder(1, { total: 5000 }))
      .rejects.toThrow('Insufficient stock');

    // Payment should never be attempted
    expect(mockPaymentGateway.charge).not.toHaveBeenCalled();
  });

  it('should throw error when payment fails', async () => {
    mockInventoryService.checkStock.mockResolvedValue(true);
    mockPaymentGateway.charge.mockResolvedValue({ success: false });

    await expect(service.placeOrder(1, { total: 5000 }))
      .rejects.toThrow('Payment failed');

    expect(mockOrderRepo.create).not.toHaveBeenCalled();
  });
});
```

**Expected Output (Jest console):**
```
PASS  services/orderService.test.js
  OrderService.placeOrder
    ✓ should place order successfully when stock and payment succeed (5 ms)
    ✓ should throw error when stock is insufficient (2 ms)
    ✓ should throw error when payment fails (1 ms)

Tests: 3 passed, 3 total
```

**Why this output:** Each test mocks all three dependencies. The happy path verifies the full orchestration and asserts that each dependency was called with the correct arguments. The error-path tests verify that the service short-circuits correctly — when stock is insufficient, payment is never attempted; when payment fails, the order is never created.

### Real-World Cases

- **E-commerce:** Testing order placement logic without a real payment gateway or database.
- **SaaS:** Testing subscription upgrade logic without hitting Stripe.
- **Fintech:** Testing transaction processing with mocked ledgers and payment rails.

---

## Core Concept 2: Testing Utilities

### Definitions

**Core Definition:** Utility functions are small, pure functions that perform a specific transformation, validation, or formatting task. Because they have no dependencies, they are the simplest units to test.

**Technical Definition:** A pure function always returns the same output for the same input and has no side effects. Utilities include validators (`isValidEmail`), formatters (`formatCurrency`), calculators (`calculateAge`), and string manipulators (`slugify`). Testing these functions requires no mocking — the test calls the function with various inputs and asserts on the outputs.

**Beginner-Friendly Explanation:** Utility functions are the "helpers" of your code — they don't talk to databases or APIs. Testing them is straightforward: give them an input, check the output. No fakes needed.

### Purposes

- To verify pure transformation and validation logic.
- To cover edge cases (empty strings, negative numbers, boundary values).
- To document the expected behaviour of utility functions.
- To provide fast, dependency-free tests that run in microseconds.

### Syntax Rules and Structure

```js
// utils/validators.js
function isValidEmail(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

function slugify(text) {
  return text.toLowerCase().replace(/\s+/g, '-').replace(/[^\w-]+/g, '');
}

module.exports = { isValidEmail, slugify };
```

```js
// utils/validators.test.js
const { isValidEmail, slugify } = require('./validators');

describe('isValidEmail', () => {
  it('should accept valid emails', () => {
    expect(isValidEmail('user@example.com')).toBe(true);
    expect(isValidEmail('first.last@sub.domain.co')).toBe(true);
  });

  it('should reject invalid emails', () => {
    expect(isValidEmail('invalid-email')).toBe(false);
    expect(isValidEmail('')).toBe(false);
    expect(isValidEmail('test@')).toBe(false);
    expect(isValidEmail('@domain.com')).toBe(false);
  });
});

describe('slugify', () => {
  it('should convert spaces to hyphens', () => {
    expect(slugify('Hello World')).toBe('hello-world');
  });

  it('should remove special characters', () => {
    expect(slugify('Hello! World?')).toBe('hello-world');
  });

  it('should handle multiple spaces', () => {
    expect(slugify('Hello   World')).toBe('hello-world');
  });
});
```

**Expected Output:**
```
PASS  utils/validators.test.js
  isValidEmail
    ✓ should accept valid emails
    ✓ should reject invalid emails
  slugify
    ✓ should convert spaces to hyphens
    ✓ should remove special characters
    ✓ should handle multiple spaces
```

**Why this output:** No mocking is required. Each test calls the utility with a specific input and asserts on the exact output. The tests cover both positive and negative cases, ensuring the utility behaves correctly across its input domain.

### Real-World Cases

- **Form validation:** `isValidEmail`, `isStrongPassword`, `isValidPhone`.
- **Data formatting:** `formatCurrency`, `formatDate`, `truncate`.
- **URL handling:** `slugify`, `buildQueryString`, `parseQueryString`.
- **Math utilities:** `calculateTax`, `calculateDiscount`, `roundTo`.

---

## Core Concept 3: Mocking Dependencies

### Definitions

**Core Definition:** Mocking is the practice of replacing a real dependency with a controlled fake object that simulates the dependency's interface but returns predefined values and records how it was called.

**Technical Definition:** Jest provides three levels of mocking: `jest.fn()` creates a standalone mock function; `jest.spyOn(obj, 'method')` wraps an existing method to track calls while optionally replacing its implementation; `jest.mock('module')` replaces an entire module with an auto-mocked version. Mocked functions expose `.mockReturnValue()`, `.mockResolvedValue()`, `.mockRejectedValue()`, and `.mockImplementation()` to define behaviour, and `.mock.calls` to inspect invocations.

**Beginner-Friendly Explanation:** Mocking is like replacing a real actor with a stand-in who reads from a script. The real database would be slow and unpredictable; the mock returns exactly what you tell it to, every time, so your test can focus on your code's logic.

### Purposes

- To isolate the unit under test from external services, databases, and APIs.
- To simulate error conditions that are hard to reproduce with real dependencies.
- To verify that the unit calls its dependencies correctly.
- To make tests fast, deterministic, and independent of network connectivity.

### Syntax Rules and Structure

**`jest.fn()` — Standalone mock function:**
```js
const mockFn = jest.fn();
mockFn.mockReturnValue(42);
mockFn('arg1', 'arg2');
expect(mockFn).toHaveBeenCalledWith('arg1', 'arg2');
```

**`jest.spyOn()` — Spy on an object method:**
```js
const obj = { method: () => 'original' };
const spy = jest.spyOn(obj, 'method').mockReturnValue('mocked');
expect(obj.method()).toBe('mocked');
spy.mockRestore();  // Restore original
```

**`jest.mock('module')` — Mock an entire module:**
```js
jest.mock('./database');
const Database = require('./database');
Database.mockImplementation(() => ({ findUser: jest.fn() }));
```

| Mocking Level | Use Case | Restores Original? |
|--------------|----------|-------------------|
| `jest.fn()` | Standalone function mock. | N/A |
| `jest.spyOn(obj, 'method')` | Track calls on a real object. | Yes (`.mockRestore()`). |
| `jest.mock('module')` | Replace an entire module. | No (unless `jest.unmock`). |

**Rules:**
- Reset mocks between tests with `jest.clearAllMocks()` in `beforeEach` or `afterEach`.
- Use `jest.resetAllMocks()` to reset mock implementations and call history.
- Use `jest.restoreAllMocks()` to restore original implementations for spies.
- Mock at the boundary: mock the external library (e.g., Axios, Prisma) rather than your own internal wrappers.
- Use `jest.mocked(module)` for type-safe access to auto-mocked modules.

### Annotated Code Example

```js
// services/userService.js
const Database = require('./database');

class UserService {
  constructor() {
    this.db = new Database();
  }

  async getUser(id) {
    const user = await this.db.findUser(id);
    if (!user) throw new Error('User not found');
    return user;
  }
}

module.exports = UserService;
```

```js
// services/userService.test.js
const UserService = require('./userService');
const Database = require('./database');

jest.mock('./database');  // Auto-mock the module

describe('UserService', () => {
  let service;
  let mockDb;

  beforeEach(() => {
    mockDb = { findUser: jest.fn() };
    Database.mockImplementation(() => mockDb);
    service = new UserService();
  });

  afterEach(() => jest.clearAllMocks());

  it('returns user when found', async () => {
    const mockUser = { id: 1, name: 'John' };
    mockDb.findUser.mockResolvedValue(mockUser);

    const result = await service.getUser(1);

    expect(result).toEqual(mockUser);
    expect(mockDb.findUser).toHaveBeenCalledWith(1);
  });

  it('throws when user not found', async () => {
    mockDb.findUser.mockResolvedValue(null);

    await expect(service.getUser(1)).rejects.toThrow('User not found');
  });

  it('propagates database errors', async () => {
    mockDb.findUser.mockRejectedValue(new Error('Connection lost'));

    await expect(service.getUser(1)).rejects.toThrow('Connection lost');
  });
});
```

**Expected Output:**
```
PASS  services/userService.test.js
  UserService
    ✓ returns user when found
    ✓ throws when user not found
    ✓ propagates database errors
```

**Why this output:** `jest.mock('./database')` replaces the entire `Database` module with an auto-mock. The `beforeEach` hook provides a `mockDb` with a `findUser` mock and injects it via `Database.mockImplementation()`. Each test defines the mock's behaviour (`mockResolvedValue`, `mockRejectedValue`) and asserts on both the result and the call.

### Real-World Cases

- **Database repositories:** Mock `UserRepository.findById` to return fixtures.
- **Email services:** Mock `sendEmail` to verify the email was sent without actually sending it.
- **Cache clients:** Mock Redis `get`/`set` to simulate cache hits and misses.

---

## Core Concept 4: Spying and Stubbing

### Definitions

**Core Definition:** Spying is the act of wrapping a function to observe its calls without changing its behaviour; stubbing is the act of replacing a function's implementation with a controlled one. In Jest, `jest.spyOn()` creates a spy that can also act as a stub when `.mockReturnValue()` or `.mockImplementation()` is applied.

**Technical Definition:** `jest.spyOn(object, 'methodName')` creates a mock function that wraps the original method. By default, the original implementation still runs, but the spy records every call. Calling `.mockReturnValue()`, `.mockResolvedValue()`, or `.mockImplementation()` replaces the implementation, turning the spy into a stub. The original method is restored with `.mockRestore()`.

**Beginner-Friendly Explanation:** A spy is like a wiretap — it listens in on a phone call without changing the conversation. A stub is like an actor reading from a script — the original person is replaced, and the stand-in says whatever you want. In Jest, you start with a spy (wiretap) and optionally turn it into a stub (scripted actor).

### Purposes

- To verify that an internal method was called (spying).
- To replace a method's behaviour for a specific test (stubbing).
- To assert on the arguments passed to an internal method.
- To test that a method was not called under certain conditions.
- To verify call counts and call order.

### Syntax Rules and Structure

**Spy without changing behaviour:**
```js
const spy = jest.spyOn(service, 'validateInput');
await service.createUser(data);
expect(spy).toHaveBeenCalledWith(data);
```

**Spy with stubbed behaviour:**
```js
const spy = jest.spyOn(service, 'validateInput').mockReturnValue(true);
await service.createUser(data);
expect(spy).toHaveBeenCalledTimes(1);
spy.mockRestore();  // Restore original method
```

| Assertion | Purpose |
|-----------|---------|
| `toHaveBeenCalled()` | Assert the spy was called at least once. |
| `toHaveBeenCalledTimes(n)` | Assert the exact number of calls. |
| `toHaveBeenCalledWith(...args)` | Assert the arguments of the last call. |
| `toHaveBeenLastCalledWith(...args)` | Assert the arguments of the last call. |
| `not.toHaveBeenCalled()` | Assert the spy was never called. |

**Rules:**
- `jest.spyOn()` preserves the original implementation by default.
- Call `.mockRestore()` to restore the original implementation after the test.
- Use `.mockImplementation()` to replace the implementation completely.
- Use `jest.restoreAllMocks()` in `afterEach` to automatically restore all spies.
- Spying on class methods requires the instance, not the class (e.g., `jest.spyOn(instance, 'method')`).

### Annotated Code Example

```js
// services/paymentService.js
class PaymentService {
  constructor(gateway) {
    this.gateway = gateway;
  }

  async processPayment(userId, amount) {
    this.validateAmount(amount);       // Internal method call
    this.logTransaction(userId, amount); // Internal method call
    return await this.gateway.charge(userId, amount);
  }

  validateAmount(amount) {
    if (amount <= 0) throw new Error('Invalid amount');
  }

  logTransaction(userId, amount) {
    console.log(`Charging ${amount} to user ${userId}`);
  }
}

module.exports = PaymentService;
```

```js
// services/paymentService.test.js
const PaymentService = require('./paymentService');

describe('PaymentService.processPayment', () => {
  let service;
  let mockGateway;

  beforeEach(() => {
    mockGateway = { charge: jest.fn().mockResolvedValue({ success: true }) };
    service = new PaymentService(mockGateway);
  });

  afterEach(() => jest.restoreAllMocks());

  it('calls validateAmount and logTransaction', async () => {
    const validateSpy = jest.spyOn(service, 'validateAmount');
    const logSpy = jest.spyOn(service, 'logTransaction');

    await service.processPayment(1, 5000);

    expect(validateSpy).toHaveBeenCalledWith(5000);
    expect(logSpy).toHaveBeenCalledWith(1, 5000);
  });

  it('does not call gateway when amount is invalid', async () => {
    const validateSpy = jest.spyOn(service, 'validateAmount').mockImplementation(() => {
      throw new Error('Invalid amount');
    });

    await expect(service.processPayment(1, -100))
      .rejects.toThrow('Invalid amount');

    expect(mockGateway.charge).not.toHaveBeenCalled();
  });

  it('stubs logTransaction to prevent console output', async () => {
    jest.spyOn(service, 'logTransaction').mockImplementation(() => {});
    const logSpy = jest.spyOn(service, 'logTransaction');

    await service.processPayment(1, 5000);

    expect(logSpy).toHaveBeenCalled();
  });
});
```

**Expected Output:**
```
PASS  services/paymentService.test.js
  PaymentService.processPayment
    ✓ calls validateAmount and logTransaction
    ✓ does not call gateway when amount is invalid
    ✓ stubs logTransaction to prevent console output
```

**Why this output:** The first test spies on both internal methods without changing their behaviour, verifying they were called with the correct arguments. The second test stubs `validateAmount` to throw, verifying the gateway is never called. The third test stubs `logTransaction` to suppress console output during the test.

### Real-World Cases

- **Verifying audit logging:** Spy on `auditLog.record()` to ensure every sensitive operation is logged.
- **Preventing side effects:** Stub `emailService.send()` to avoid sending real emails during tests.
- **Testing conditional logic:** Spy on `cache.get()` to verify it's called only when appropriate.

---

## Core Concept 5: Mocking Global/Third-Party Modules

### Definitions

**Core Definition:** Mocking third-party modules means replacing the real implementation of an external library — such as Axios for HTTP, the AWS SDK for cloud services, or Stripe for payments — with a controlled fake that returns predefined responses.

**Technical Definition:** `jest.mock('module-name')` auto-mocks an entire module, replacing all its exports with Jest mock functions. For third-party SDKs, dedicated mocking libraries exist: `nock` intercepts Node.js HTTP requests at the `http` module level, `aws-sdk-client-mock` provides typed mocks for AWS SDK v3 clients, and Stripe can be mocked via `jest.mock('stripe')` with custom implementations for specific resources.

**Beginner-Friendly Explanation:** When your code calls an external service, you don't want your tests to actually make that call — it would be slow, cost money, and fail if the service is down. Mocking the module means telling Jest: "When the code imports Stripe, give it a fake Stripe instead." Your code runs as normal, but the fake returns whatever you tell it to.

### Purposes

- To test code that calls external APIs without making real network requests.
- To simulate error responses (500 errors, timeouts) that are hard to reproduce.
- To avoid costs associated with real API calls (Stripe charges, AWS usage).
- To make tests fast, offline, and deterministic.
- To verify that the correct API calls were made with the correct parameters.

### Sub-Feature 5.1: Mocking Axios

```js
// services/apiService.js
const axios = require('axios');

async function fetchUser(id) {
  const response = await axios.get(`https://api.example.com/users/${id}`);
  return response.data;
}

module.exports = { fetchUser };
```

```js
// services/apiService.test.js
const axios = require('axios');
const { fetchUser } = require('./apiService');

jest.mock('axios');  // Auto-mock the entire axios module
const mockedAxios = axios;

describe('fetchUser', () => {
  afterEach(() => jest.clearAllMocks());

  it('returns user data on success', async () => {
    const mockUser = { id: 1, name: 'John' };
    mockedAxios.get.mockResolvedValue({ data: mockUser });

    const user = await fetchUser(1);

    expect(user).toEqual(mockUser);
    expect(mockedAxios.get).toHaveBeenCalledWith(
      'https://api.example.com/users/1'
    );
  });

  it('throws on network error', async () => {
    mockedAxios.get.mockRejectedValue(new Error('Network Error'));

    await expect(fetchUser(1)).rejects.toThrow('Network Error');
  });
});
```

**Expected Output:**
```
PASS  services/apiService.test.js
  fetchUser
    ✓ returns user data on success
    ✓ throws on network error
```

**Why this output:** `jest.mock('axios')` replaces the entire Axios module with auto-mocked functions. `mockedAxios.get.mockResolvedValue()` defines what the mock returns. The test asserts on both the returned data and the URL passed to `axios.get`.

### Sub-Feature 5.2: Mocking AWS SDK v3

```js
// services/s3Service.js
const { S3Client, PutObjectCommand } = require('@aws-sdk/client-s3');

const s3 = new S3Client({ region: 'us-east-1' });

async function uploadFile(key, body) {
  await s3.send(new PutObjectCommand({
    Bucket: 'my-bucket',
    Key: key,
    Body: body
  }));
  return { key, bucket: 'my-bucket' };
}

module.exports = { uploadFile };
```

```js
// services/s3Service.test.js
const { mockClient } = require('aws-sdk-client-mock');
const { S3Client, PutObjectCommand } = require('@aws-sdk/client-s3');
const { uploadFile } = require('./s3Service');

const s3Mock = mockClient(S3Client);

describe('uploadFile', () => {
  beforeEach(() => s3Mock.reset());

  it('uploads file successfully', async () => {
    s3Mock.on(PutObjectCommand).resolves({ ETag: '"abc123"' });

    const result = await uploadFile('test.txt', 'content');

    expect(result).toEqual({ key: 'test.txt', bucket: 'my-bucket' });

    const calls = s3Mock.commandCalls(PutObjectCommand);
    expect(calls).toHaveLength(1);
    expect(calls[0].args[0].input).toEqual({
      Bucket: 'my-bucket',
      Key: 'test.txt',
      Body: 'content'
    });
  });

  it('throws on upload failure', async () => {
    s3Mock.on(PutObjectCommand).rejects(new Error('Access Denied'));

    await expect(uploadFile('test.txt', 'content'))
      .rejects.toThrow('Access Denied');
  });
});
```

**Expected Output:**
```
PASS  services/s3Service.test.js
  uploadFile
    ✓ uploads file successfully
    ✓ throws on upload failure
```

**Why this output:** `aws-sdk-client-mock` provides `mockClient(S3Client)` which intercepts all `s3.send()` calls. `s3Mock.on(PutObjectCommand).resolves()` defines the response. `s3Mock.commandCalls()` returns all calls to the command, allowing assertions on the input parameters.

### Sub-Feature 5.3: Mocking Stripe

```js
// services/paymentService.js
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);

async function createCharge(amount, currency, source) {
  const charge = await stripe.charges.create({
    amount,
    currency,
    source
  });
  return charge;
}

module.exports = { createCharge };
```

```js
// services/paymentService.test.js
jest.mock('stripe', () => {
  const mockCharges = {
    create: jest.fn()
  };
  return jest.fn(() => ({
    charges: mockCharges
  }));
});

const stripe = require('stripe');
const { createCharge } = require('./paymentService');

describe('createCharge', () => {
  let mockCharges;

  beforeEach(() => {
    const stripeInstance = stripe();
    mockCharges = stripeInstance.charges;
  });

  afterEach(() => jest.clearAllMocks());

  it('creates a charge successfully', async () => {
    const mockCharge = { id: 'ch_123', amount: 5000, status: 'succeeded' };
    mockCharges.create.mockResolvedValue(mockCharge);

    const result = await createCharge(5000, 'usd', 'tok_visa');

    expect(result).toEqual(mockCharge);
    expect(mockCharges.create).toHaveBeenCalledWith({
      amount: 5000,
      currency: 'usd',
      source: 'tok_visa'
    });
  });

  it('throws on card decline', async () => {
    mockCharges.create.mockRejectedValue(new Error('Card declined'));

    await expect(createCharge(5000, 'usd', 'tok_chargeDeclined'))
      .rejects.toThrow('Card declined');
  });
});
```

**Expected Output:**
```
PASS  services/paymentService.test.js
  createCharge
    ✓ creates a charge successfully
    ✓ throws on card decline
```

**Why this output:** The `jest.mock('stripe')` factory returns a mock Stripe constructor whose instances have a `charges.create` mock. This allows the test to control the charge response and assert on the parameters passed to Stripe.

### Real-World Cases

- **E-commerce:** Mocking Stripe to test checkout logic without real charges.
- **Cloud services:** Mocking AWS S3 to test file upload logic without real buckets.
- **Third-party APIs:** Mocking Axios to test HTTP client wrappers without network calls.
- **CI pipelines:** All third-party mocks run offline, making CI fast and reliable.

---

## Core Concept 6: Testing Custom Error Classes

### Definitions

**Core Definition:** A custom error class is a subclass of JavaScript's `Error` that carries additional context — such as an error code, HTTP status, or metadata — and can be tested for its type, message, and properties.

**Technical Definition:** Custom error classes extend `Error` and must call `super(message)` in the constructor. Due to a TypeScript breaking change (and a historical issue in JavaScript), subclasses of built-in classes like `Error` may lose their prototype chain when transpiled. The fix is to call `Object.setPrototypeOf(this, CustomError.prototype)` immediately after `super()` so that `instanceof` checks work correctly. Jest's `toThrow(CustomError)` and `toBeInstanceOf(CustomError)` can then verify the error type.

**Beginner-Friendly Explanation:** A custom error is a special type of error that carries extra information — like an HTTP status code or a machine-readable error code. When you test it, you want to check not just that an error was thrown, but that it was the right kind of error and that its properties are correct. The tricky part is that JavaScript's `instanceof` can break for custom errors unless you fix the prototype.

### Purposes

- To verify that a function throws the correct error type (not just any error).
- To assert on the error's message, code, and custom properties.
- To ensure error handling logic branches correctly based on error type.
- To test error serialization (e.g., `toJSON()` for API responses).

### Syntax Rules and Structure

**Custom Error Class:**
```js
class UserNotFoundError extends Error {
  constructor(userId) {
    super(`User ${userId} not found`);
    Object.setPrototypeOf(this, UserNotFoundError.prototype);  // Fix prototype
    this.name = 'UserNotFoundError';
    this.code = 'USER_NOT_FOUND';
    this.statusCode = 404;
    this.userId = userId;
  }

  toJSON() {
    return {
      error: this.name,
      message: this.message,
      code: this.code,
      statusCode: this.statusCode
    };
  }
}
```

**Testing the Error:**
```js
it('throws UserNotFoundError with correct properties', async () => {
  await expect(service.getUser(999))
    .rejects.toThrow(UserNotFoundError);

  try {
    await service.getUser(999);
  } catch (err) {
    expect(err).toBeInstanceOf(UserNotFoundError);
    expect(err.code).toBe('USER_NOT_FOUND');
    expect(err.statusCode).toBe(404);
    expect(err.userId).toBe(999);
  }
});
```

| Assertion | Purpose |
|-----------|---------|
| `toThrow(ErrorClass)` | Assert the thrown error is an instance of the class. |
| `toBeInstanceOf(ErrorClass)` | Assert the caught error is an instance. |
| `toThrow('message')` | Assert the error message contains the string. |
| `toHaveProperty('code', value)` | Assert a custom property. |

**Rules:**
- Always call `Object.setPrototypeOf(this, CustomError.prototype)` after `super()` to fix `instanceof`.
- Set `this.name` to the class name for clean serialization.
- Add a `toJSON()` method for API-friendly error responses.
- Use `rejects.toThrow(ErrorClass)` for async functions and `toThrow(ErrorClass)` for sync functions.
- Test both the error type and its custom properties.

### Annotated Code Example

```js
// errors/AppError.js
class AppError extends Error {
  constructor(message, code, statusCode) {
    super(message);
    Object.setPrototypeOf(this, new.target.prototype);
    this.name = this.constructor.name;
    this.code = code;
    this.statusCode = statusCode;
    Error.captureStackTrace(this, this.constructor);
  }

  toJSON() {
    return {
      error: this.name,
      message: this.message,
      code: this.code,
      statusCode: this.statusCode
    };
  }
}

class ValidationError extends AppError {
  constructor(message, fields) {
    super(message, 'VALIDATION_ERROR', 422);
    Object.setPrototypeOf(this, ValidationError.prototype);
    this.fields = fields;
  }
}

class NotFoundError extends AppError {
  constructor(resource, id) {
    super(`${resource} with id ${id} not found`, 'NOT_FOUND', 404);
    Object.setPrototypeOf(this, NotFoundError.prototype);
    this.resource = resource;
    this.resourceId = id;
  }
}

module.exports = { AppError, ValidationError, NotFoundError };
```

```js
// errors/errors.test.js
const { AppError, ValidationError, NotFoundError } = require('./AppError');

describe('Custom Error Classes', () => {
  describe('AppError', () => {
    it('should set message, code, and statusCode', () => {
      const err = new AppError('Something broke', 'INTERNAL', 500);

      expect(err).toBeInstanceOf(Error);
      expect(err).toBeInstanceOf(AppError);
      expect(err.message).toBe('Something broke');
      expect(err.code).toBe('INTERNAL');
      expect(err.statusCode).toBe(500);
      expect(err.name).toBe('AppError');
    });

    it('should serialize to JSON', () => {
      const err = new AppError('Failed', 'ERR', 400);
      expect(err.toJSON()).toEqual({
        error: 'AppError',
        message: 'Failed',
        code: 'ERR',
        statusCode: 400
      });
    });
  });

  describe('ValidationError', () => {
    it('should include field-level details', () => {
      const fields = [
        { field: 'email', message: 'Invalid email' },
        { field: 'password', message: 'Too short' }
      ];
      const err = new ValidationError('Validation failed', fields);

      expect(err).toBeInstanceOf(AppError);
      expect(err).toBeInstanceOf(ValidationError);
      expect(err.code).toBe('VALIDATION_ERROR');
      expect(err.statusCode).toBe(422);
      expect(err.fields).toEqual(fields);
    });
  });

  describe('NotFoundError', () => {
    it('should build a descriptive message', () => {
      const err = new NotFoundError('User', 42);

      expect(err.message).toBe('User with id 42 not found');
      expect(err.code).toBe('NOT_FOUND');
      expect(err.statusCode).toBe(404);
      expect(err.resource).toBe('User');
      expect(err.resourceId).toBe(42);
    });
  });

  describe('Error throwing in services', () => {
    it('should throw NotFoundError when resource is missing', async () => {
      const mockRepo = { findById: jest.fn().mockResolvedValue(null) };
      const service = { getUser: async (id) => {
        const user = await mockRepo.findById(id);
        if (!user) throw new NotFoundError('User', id);
        return user;
      }};

      await expect(service.getUser(99))
        .rejects.toThrow(NotFoundError);

      try {
        await service.getUser(99);
      } catch (err) {
        expect(err).toBeInstanceOf(NotFoundError);
        expect(err.statusCode).toBe(404);
        expect(err.resourceId).toBe(99);
      }
    });
  });
});
```

**Expected Output:**
```
PASS  errors/errors.test.js
  Custom Error Classes
    AppError
      ✓ should set message, code, and statusCode
      ✓ should serialize to JSON
    ValidationError
      ✓ should include field-level details
    NotFoundError
      ✓ should build a descriptive message
    Error throwing in services
      ✓ should throw NotFoundError when resource is missing
```

**Why this output:** Each custom error class is tested for its type (`instanceof`), message, and custom properties. The `Object.setPrototypeOf` call in each constructor ensures that `instanceof` checks work correctly. The service test verifies that the error is thrown and that its properties are accessible in the `catch` block.

### Real-World Cases

- **API error handling:** Custom errors with `statusCode` and `code` drive consistent HTTP error responses.
- **Validation:** `ValidationError` with field-level details powers form validation feedback.
- **Domain errors:** `InsufficientFundsError`, `DuplicateEmailError`, `RateLimitExceededError`.
- **Error serialization:** `toJSON()` produces RFC 7807-compatible error payloads.

---

## References

- Jest Official Documentation — https://jestjs.io/docs/getting-started
- Jest Mock Functions — https://jestjs.io/docs/mock-functions
- Jest `jest.mock()` API — https://jestjs.io/docs/jest-object#jestmockmodulename-factory-options
- Jest `jest.spyOn()` API — https://jestjs.io/docs/jest-object#jestspyonobject-methodname
- BrowserStack: NodeJS Unit Testing with Jest — https://www.browserstack.com/guide/unit-testing-for-nodejs-using-jest
- CoreUI: How to test Node.js apps with Jest — https://coreui.io/answers/how-to-test-nodejs-apps-with-jest/
- CoreUI: How to mock dependencies in Node.js tests — https://coreui.io/answers/how-to-mock-dependencies-in-nodejs-tests/
- GitHub: awesome-copilot Jest Skill — https://github.com/github/awesome-copilot/blob/main/skills/javascript-typescript-jest/SKILL.md
- GitHub: TRAE-Skills Mocking External Services — https://github.com/MarcoNasi/TRAE-Skills/blob/6ce82c6a9c16837a7ac8edf4b7be9b1a1a98ad3d/testing/Mocking_External_Services_Jest.md
- aws-sdk-client-mock GitHub — https://github.com/jonkoops/aws-sdk-client-mock
- aws-sdk-client-mock-jest on npm — https://www.npmjs.com/package/aws-sdk-client-mock-jest
- Nock HTTP Mocking Library — https://github.com/nock/nock
- Stack Overflow: Custom Error with Jest — https://stackoverflow.com/questions/68899615
- TypeScript Breaking Changes: Extending Built-ins — https://github.com/Microsoft/TypeScript/wiki/Breaking-Changes
- GitHub: backend-testing AI Agent Skill — https://askill.sh/skills/gh/chauhaidang/xq-fitness-write/@backend-testing
- GitHub: Digital-Defiance/test-utils — https://github.com/Digital-Defiance/test-utils
- Devin Docs: Add Unit Tests to Your Payments Service — https://docs.devinenterprise.com/use-cases/gallery/refactor-module
- Jest Native Mocking Patterns — https://github.com/bmad-labs/skills/blob/main/skills/typescript-unit-testing/references/mocking/jest-native.md
- Stack Overflow: How to mock Stripe with Jest — https://stackoverflow.com/questions/57492219