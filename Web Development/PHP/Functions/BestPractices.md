Here is a comprehensive research cheat sheet on **PHP Function Best Practices**, structured for both academic clarity and beginner accessibility.

---

## Topic Overview: PHP Function Best Practices

### Definitions

**Core Definition**
Function best practices are a set of community-established conventions and design principles for writing PHP functions that are correct, readable, testable, and maintainable. They cover naming, size, responsibility, purity, error handling, and type safety.

**Technical Definition**
Best practices for PHP functions are codified in standards such as **PSR-1** (Basic Coding Standard) and **PSR-12** (Extended Coding Style), and are reinforced by static analysis tools like PHPStan and Psalm. The core principles include: **single responsibility** (a function does one thing); **functional cohesion** (all elements contribute to that one task); **purity** (same input → same output, no side effects); **explicit error handling** via typed exceptions and `try`/`catch`/`finally`; **strict typing** via `declare(strict_types=1)` and full type declarations; and **concise, descriptive naming** using verbs and camelCase. These practices are not enforced by the PHP engine but are widely adopted because they reduce defects, simplify maintenance, and enable tooling to reason about the code.

**Beginner-Friendly Explanation**
Function best practices are the “good habits” of writing functions. They are like the rules of good cooking: name your dishes clearly, don't overcrowd the pan (keep functions small), don't secretly add ingredients the recipe didn't mention (avoid side effects), and always have a plan for when something burns (error handling). Following them makes your code easier for you and others to understand, test, and fix.

---

### Key Characteristics

- **Standardization**: PSR-1 and PSR-12 provide a shared baseline for naming, formatting, and structure across the PHP ecosystem.
- **Tooling Support**: Static analyzers (PHPStan, Psalm) and code style fixers (PHP CS Fixer, Pint) enforce many best practices automatically.
- **Testability**: Pure functions are trivially unit-testable; functions with explicit dependencies and typed exceptions are easier to test in isolation.
- **Safety**: Strict types and typed exceptions catch errors early, before they propagate.
- **Version Awareness**: Modern practices assume PHP 7.4+ (arrow functions, typed properties) and PHP 8.0+ (named arguments, union types, `throw` as an expression).

---

### Prerequisites

- **PHP 8.0+ Recommended**: Typed exceptions, union types, and `throw` as an expression are PHP 8.0+ features.
- **Composer and PSR-4 Autoloading**: Standard project structure is assumed by most PSR-oriented guides.
- **Basic Understanding of Exceptions and Type Declarations**: Familiarity with `try`/`catch`, `Throwable`, and parameter/return types is essential.
- **A Static Analysis Tool (Optional but Recommended)**: PHPStan or Psalm will flag many violations automatically.

---

### Related Programming Areas

- **Object-Oriented Design**: Class methods follow the same best practices, plus visibility rules.
- **Functional Programming**: Purity and immutability are core functional concepts that inform PHP best practices.
- **Testing (PHPUnit)**: Testable design is a direct consequence of purity and explicit dependencies.
- **Code Review and CI**: PSR compliance and static analysis are common CI gates.

---

## Core Concepts / Features

---

### 1. Function Naming

#### Definitions

**Core Definition**
Function naming is the practice of choosing identifiers for functions that are valid, descriptive, and consistent with community conventions.

**Technical Definition**
A valid PHP function name must match the regular expression `^[a-zA-Z_\x80-\xff][a-zA-Z0-9_\x80-\xff]*$` — it starts with a letter or underscore, followed by letters, numbers, or underscores. PSR-12 mandates **camelCase** for function and method names. PSR-1 prohibits leading underscores for visibility signaling (e.g., `_privateMethod`). Beyond syntax, effective names are **verb phrases** that describe the action performed (e.g., `getUserDetailsByEmail`). Boolean-returning functions conventionally use prefixes like `is`, `has`, or `can`.

**Beginner-Friendly Explanation**
A function name should tell you exactly what the function does. Use a verb: `calculateTotal`, not `total`. Use camelCase: `getUserName`, not `get_user_name` or `GetUserName`. If it returns `true` or `false`, start with `is`, `has`, or `can` (e.g., `isValidEmail`).

#### Purposes

- To make code self-documenting, so readers understand a function’s purpose without reading its body.
- To ensure compatibility with PSR standards and tooling.
- To reduce the risk of naming collisions and ambiguous identifiers.
- To improve searchability and IDE navigation.

#### Syntax Rules and Structure

**General Syntax (PSR-12)**
```php
function camelCaseVerbPhrase(Type $param): ReturnType
```

**Component Breakdown**

| Rule | Description |
|------|-------------|
| Case | `camelCase` for functions and methods. |
| Leading underscores | Prohibited for visibility signaling. |
| Verb inclusion | Function names should contain a verb (e.g., `get`, `set`, `create`, `process`). |
| Boolean prefixes | Use `is`, `has`, `can` for boolean-returning functions. |

**Syntax Rules**

- Function names are case-insensitive for ASCII, but always call them as declared.
- Avoid vague names like `handle()`, `process()`, or `execute()` without a domain-specific noun.
- Follow the conventions of your ecosystem: most modern frameworks use `camelCase`; some legacy systems use `snake_case`.

**Constraints and Limitations**

- Reserved keywords cannot be used as function names.
- PSR-12 is a recommendation, not a language rule; some projects (e.g., TastyIgniter) use `snake_case` for global functions and `camelCase` for methods.

#### Annotated Code Examples

**Example 1: Poor vs. Good Naming**

```php
<?php
// POOR: vague, noun-only, no verb
function user($email) { /* ... */ }

// POOR: non-standard case
function get_user_data($id) { /* ... */ }

// GOOD: camelCase, verb phrase, descriptive
function getUserDetailsByEmail(string $email): array {
    return ['name' => 'Alice', 'email' => $email];
}

// GOOD: boolean prefix
function isValidEmail(string $email): bool {
    return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
}

echo getUserDetailsByEmail('alice@example.com')['name'] . "\n"; // Outputs: Alice
var_dump(isValidEmail('alice@example.com')); // bool(true)
?>
```

**Expected Output:**
```
Alice
bool(true)
```

**Why This Output:**  
The good names follow PSR-12 (`camelCase`), include verbs (`get`, `is`), and describe the domain concept (`UserDetailsByEmail`, `ValidEmail`). The poor names are vague or use non-standard formatting.

---

#### Real-World Cases

- **Laravel**: Controllers use `camelCase` method names (`getUser`, `storePost`) that describe the action.
- **Symfony**: Service methods follow `camelCase` and verb-based naming (`calculateTotal`, `findActiveUsers`).
- **PSR-12 Adoption**: Projects using PHP CS Fixer with the `psr12` preset automatically enforce `camelCase` and other naming rules.

---

### 2. Function Length

#### Definitions

**Core Definition**
Function length is the number of lines of code in a function body. Best practice recommends keeping functions short and focused.

**Technical Definition**
PSR standards do not explicitly define a maximum function length, but modern PHP best practices strongly encourage functions to be “short, focused, and responsible for a single task”. In practice, many experienced developers aim for **20–30 lines** per function. A function should be “less than a page in length” so that a reader does not have to scroll to understand it. Functions that exceed this length are almost always doing more than one thing and should be split into smaller functions.

**Beginner-Friendly Explanation**
A good function should fit on your screen without scrolling. If it is longer than about 20–30 lines, it is probably doing too much. Break it into smaller functions, each with a clear name. This makes the code easier to read, test, and fix.

#### Purposes

- To improve readability by keeping the entire function visible at once.
- To enforce single responsibility indirectly: short functions tend to do one thing.
- To make functions easier to unit test and debug.
- To reduce cognitive load for reviewers and maintainers.

#### Syntax Rules and Structure (Guidelines)

| Guideline | Description |
|-----------|-------------|
| **Target length** | 20–30 lines is a common practical target. |
| **Hard ceiling** | A function should not exceed one screen/page. |
| **Split triggers** | If a function has multiple levels of nesting, multiple responsibilities, or requires scrolling, extract sub-functions. |

**Constraints and Limitations**

- **Line length vs. function length**: PSR-12 also limits line length (soft 120, hard 150), which encourages breaking long expressions but is separate from function length.
- **Trivial functions**: Extremely short functions (e.g., one-line getters) are acceptable and often good practice.

#### Annotated Code Examples

**Example 1: Long Function Split into Smaller Functions**

```php
<?php
declare(strict_types=1);

// BEFORE: Long function doing multiple things
function processOrderOld(array $order): string {
    // Validate order
    if (empty($order['items'])) {
        return "Error: No items";
    }
    foreach ($order['items'] as $item) {
        if ($item['quantity'] <= 0) {
            return "Error: Invalid quantity";
        }
    }
    // Calculate total
    $total = 0;
    foreach ($order['items'] as $item) {
        $total += $item['price'] * $item['quantity'];
    }
    // Apply discount
    if ($total > 100) {
        $total *= 0.9;
    }
    // Format output
    return "Total: $" . number_format($total, 2);
}

// AFTER: Short, focused functions
function validateOrder(array $order): ?string {
    if (empty($order['items'])) return "No items";
    foreach ($order['items'] as $item) {
        if ($item['quantity'] <= 0) return "Invalid quantity";
    }
    return null;
}

function calculateOrderTotal(array $order): float {
    $total = 0.0;
    foreach ($order['items'] as $item) {
        $total += $item['price'] * $item['quantity'];
    }
    return $total > 100 ? $total * 0.9 : $total;
}

function formatOrderTotal(float $total): string {
    return "Total: $" . number_format($total, 2);
}

function processOrderNew(array $order): string {
    $error = validateOrder($order);
    if ($error !== null) return "Error: $error";
    $total = calculateOrderTotal($order);
    return formatOrderTotal($total);
}

// Example usage
$order = ['items' => [['price' => 50, 'quantity' => 3]]];
echo processOrderNew($order) . "\n"; // Outputs: Total: $135.00
?>
```

**Expected Output:**
```
Total: $135.00
```

**Why This Output:**  
The original function was over 20 lines and mixed validation, calculation, discount logic, and formatting. The refactored version splits these into four small, focused functions, each of which is easy to read and test.

---

#### Real-World Cases

- **Code Review Standards**: Many teams enforce a maximum function length in code review (e.g., 30 lines) and require extraction of sub-functions.
- **Static Analysis**: PHPStan and Psalm can be configured to flag overly complex functions (cyclomatic complexity), which correlates with length.
- **Legacy Refactoring**: Long functions are prime candidates for extraction when adding tests or fixing bugs.

---

### 3. Function Cohesion

#### Definitions

**Core Definition**
Function cohesion is the degree to which all elements inside a function contribute to a single, well-defined purpose. High cohesion means the function does one thing; low cohesion means it does several unrelated things.

**Technical Definition**
Cohesion is a software design metric. The most desirable form is **functional cohesion**, where a routine performs “one and only one operation”. This is a specific form of the **Single Responsibility Principle (SRP)** applied to functions. Lower forms of cohesion include **logical cohesion** (elements perform similar but unrelated operations, e.g., a `handleError` function that does logging and notification), **temporal cohesion** (elements must execute at the same time, e.g., initialization), and **sequential cohesion** (output of one element is input to the next). Functional cohesion is the goal for well-designed functions.

**Beginner-Friendly Explanation**
Cohesion means “how well the parts of a function stick together.” A cohesive function does one job from start to finish. A non-cohesive function is like a Swiss Army knife that also contains a toothbrush and a screwdriver—it does many things, but none of them well. Keep your functions focused on one task.

#### Purposes

- To make functions easier to understand, because their purpose is unambiguous.
- To improve testability, because a single-purpose function has a clear contract.
- To reduce the risk of unintended side effects when modifying code.
- To enable clean separation of concerns across a codebase.

#### Syntax Rules and Structure (Guidelines)

| Cohesion Type | Description | Quality |
|---------------|-------------|---------|
| **Functional** | Performs exactly one operation (e.g., `sin()`, `getCustomerName()`). | **Best** |
| **Sequential** | Elements must run in order, output of one feeds the next. | Acceptable if tightly related |
| **Communicational** | Elements operate on the same data structure. | Weaker |
| **Temporal** | Elements must run at the same time (e.g., initialization). | Weaker |
| **Logical** | Elements perform similar but unrelated operations (e.g., `handleAllErrors`). | **Poor** |
| **Coincidental** | Elements have no meaningful relationship. | **Worst** |

**Syntax Rules**

- A function should have a single, clearly nameable purpose.
- If you struggle to name the function without using “and,” it likely has low cohesion.
- Related operations should be composed from smaller, cohesive functions rather than combined into one.

**Constraints and Limitations**

- **Cohesion vs. DRY**: Sometimes slight duplication is preferable to a low-cohesion “utility” function that serves unrelated callers.
- **Performance concerns**: Splitting a function for cohesion can introduce a small call overhead; this is almost always negligible compared to maintenance benefits.

#### Annotated Code Examples

**Example 1: Low Cohesion vs. High Cohesion**

```php
<?php
declare(strict_types=1);

// POOR: Low cohesion — mixes validation, storage, and notification
function handleUserRegistrationOld(string $email, string $password): string {
    // 1. Validate
    if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        return "Invalid email";
    }
    if (strlen($password) < 8) {
        return "Password too short";
    }
    // 2. Save to database (simulated)
    $saved = true;
    // 3. Send welcome email (simulated)
    $emailSent = true;
    return "Registered: $email";
}

// GOOD: High cohesion — each function does one thing
function validateRegistration(string $email, string $password): ?string {
    if (!filter_var($email, FILTER_VALIDATE_EMAIL)) return "Invalid email";
    if (strlen($password) < 8) return "Password too short";
    return null;
}

function saveUser(string $email): bool {
    return true; // simulated
}

function sendWelcomeEmail(string $email): bool {
    return true; // simulated
}

function handleUserRegistrationNew(string $email, string $password): string {
    $error = validateRegistration($email, $password);
    if ($error !== null) return $error;
    saveUser($email);
    sendWelcomeEmail($email);
    return "Registered: $email";
}

echo handleUserRegistrationNew('alice@example.com', 'password123') . "\n";
// Outputs: Registered: alice@example.com
?>
```

**Expected Output:**
```
Registered: alice@example.com
```

**Why This Output:**  
The old function combined three unrelated responsibilities (validation, persistence, notification), resulting in logical/temporal cohesion. The new version decomposes these into three functionally cohesive functions and one orchestrator that composes them.

---

#### Real-World Cases

- **Service Classes**: A `UserService` should not also send emails; email sending belongs in a `MailService` or `NotificationService`.
- **Controllers**: A controller action should validate input, call a service, and return a response—not implement business logic directly.
- **Utility Functions**: Avoid “God functions” like `processData()` that do parsing, validation, transformation, and saving. Split them.

---

### 4. Pure Functions

#### Definitions

**Core Definition**
A pure function is a function that always returns the same output for the same input and has no side effects.

**Technical Definition**
A function is pure if it satisfies two conditions: (1) **determinism** — given the same arguments, it always returns the same value; and (2) **no side effects** — it does not modify external state, perform I/O, depend on time or randomness, or mutate its arguments. In PHP, purity is a semantic property, not a language-enforced one; tools like PHPStan allow annotating functions with `@phpstan-pure` to document and verify purity. Pure functions are the building blocks of functional programming and are trivially unit-testable.

**Beginner-Friendly Explanation**
A pure function is like a mathematical formula: `add(2, 3)` always returns `5`, and it does not secretly write to a file or change a global variable. You can call it a million times and nothing else in your program changes. This makes pure functions extremely easy to test and reason about.

#### Purposes

- To make functions trivially unit-testable (no mocks, no setup).
- To enable safe caching and memoization (the result depends only on the input).
- To reduce bugs caused by hidden dependencies and shared mutable state.
- To improve composability: pure functions can be combined freely.

#### Syntax Rules and Structure (Guidelines)

| Rule | Description |
|------|-------------|
| **Deterministic** | The return value depends only on arguments, not on time, random numbers, or global state. |
| **No side effects** | No I/O, no database access, no file operations, no `echo`, no mutation of arguments or globals. |
| **No `void`** | A pure function must return a value; a `void` function is inherently impure because its only purpose is a side effect. |
| **Documented purity** | Use `@phpstan-pure` / `@psalm-pure` annotations so static analysis can verify and remember purity. |

**Constraints and Limitations**

- **PHP has no effect system**: Purity is not enforceable by the language; it relies on discipline and tooling.
- **Impure functions are necessary**: I/O, database access, and logging must happen somewhere; the goal is to isolate them, not eliminate them.

#### Annotated Code Examples

**Example 1: Impure vs. Pure Function**

```php
<?php
declare(strict_types=1);

// IMPURE: depends on global state, has side effect
$discountRate = 0.2;

function calculateDiscountImpure(float $price): float {
    global $discountRate;          // hidden dependency
    echo "Calculating...";          // side effect (output)
    return $price * $discountRate;
}

// PURE: deterministic, no side effects
function calculateDiscountPure(float $price, float $discountRate): float {
    return $price * $discountRate;
}

// Both return the same value for the same effective input…
echo calculateDiscountImpure(100) . "\n";              // Outputs: Calculating...20
echo calculateDiscountPure(100, 0.2) . "\n";           // Outputs: 20
?>
```

**Expected Output:**
```
Calculating...20
20
```

**Why This Output:**  
Both compute `100 * 0.2 = 20`, but the impure version prints “Calculating…” (a side effect) and depends on the global `$discountRate`. The pure version takes the rate as a parameter and has no side effects. The pure version is easier to test: `assert(calculateDiscountPure(100, 0.2) === 20.0)` requires no setup or output capture.

---

#### Real-World Cases

- **Pricing Engines**: Tax and discount calculations should be pure: `calculateTax($subtotal, $rate)`, not `calculateTax($subtotal)` that reads a global configuration.
- **Data Transformation**: `formatCurrency(float $amount, string $currency): string` is pure; it does not write to a log or send an email.
- **Validation**: `isValidEmail(string $email): bool` is pure; it depends only on the input string.

---

### 5. Side Effects

#### Definitions

**Core Definition**
A side effect is any observable change that a function causes outside of returning a value. Common side effects include writing to a file, modifying a global variable, printing output, and calling a database.

**Technical Definition**
A side effect is “anything other than taking a value in and returning another value or values”. PHPStan defines side effects as “sleeping, accessing the filesystem, accessing the database, printing something to the output, depending on time, or calling `rand()`”. Functions that return `void` are considered impure because their sole purpose is a side effect. Side effects are not inherently bad—programs must interact with the world—but they should be **isolated, centralized, and intentional**.

**Beginner-Friendly Explanation**
A side effect is anything a function does besides giving you an answer. If a function calculates a total *and* sends an email, sending the email is a side effect. Side effects are necessary, but they should be obvious and kept in special places (like a “mailer” class), not hidden inside a calculation function.

#### Purposes

- To make functions predictable by separating calculation from interaction.
- To simplify testing: pure calculation functions can be tested without mocking the world.
- To improve debuggability by centralizing I/O and state changes.

#### Syntax Rules and Structure (Guidelines)

| Practice | Description |
|----------|-------------|
| **Isolate side effects** | Keep I/O, database, and logging in dedicated functions or services. |
| **Document side effects** | Use PHPDoc to note when a function performs I/O or mutates state. |
| **Prefer returning values over outputting** | A function that returns a string is more reusable than one that `echo`es it. |
| **Avoid mutating arguments** | Unless using `&$param` intentionally, do not modify caller variables. |

**Constraints and Limitations**

- **Some side effects are unavoidable**: Controllers must send responses; repositories must query databases. The goal is containment, not elimination.
- **`void` functions are inherently impure**: If a function returns nothing, it exists for its side effect.

#### Annotated Code Examples

**Example 1: Hidden vs. Isolated Side Effect**

```php
<?php
declare(strict_types=1);

// POOR: Hidden side effect — logging and calculation mixed
function calculateTotalWithLogging(array $items): float {
    $total = array_sum(array_column($items, 'price'));
    error_log("Calculated total: $total");   // hidden side effect
    return $total;
}

// GOOD: Side effect isolated in its own function
function calculateTotal(array $items): float {
    return array_sum(array_column($items, 'price'));
}

function logTotal(float $total): void {
    error_log("Calculated total: $total");
}

// Usage: explicit composition
$items = [['price' => 10.0], ['price' => 20.0]];
$total = calculateTotal($items);
logTotal($total);
echo "Total: $total\n"; // Outputs: Total: 30
?>
```

**Expected Output:**
```
Total: 30
```

**Why This Output:**  
`calculateTotal` is pure and testable. The logging side effect is isolated in `logTotal`, which is explicitly called by the orchestrating code. If logging needs to change (e.g., to a different logger), only `logTotal` is affected.

---

#### Real-World Cases

- **Repository Pattern**: Database queries are isolated in repository methods; business logic functions receive plain data.
- **HTTP Controllers**: Controllers handle request/response side effects; services contain pure business logic.
- **Event Dispatching**: Dispatching events is a side effect; it should be isolated in a dispatcher, not mixed into domain calculations.

---

### 6. Error Handling

#### Definitions

**Core Definition**
Error handling is the practice of anticipating, detecting, and responding to failures in a controlled way, typically using exceptions and `try`/`catch`/`finally` blocks.

**Technical Definition**
Modern PHP distinguishes between **errors** (engine-level faults like `TypeError`) and **exceptions** (application-level events you throw and catch). Since PHP 7, both implement the `Throwable` interface, with `Error` and `Exception` as the two main branches. Best practices include: **throw early, catch late**; **catch specific types first**, then broader types, with `\Throwable` last (or not at all); **never swallow exceptions silently**; use `finally` for cleanup; and **chain exceptions** to preserve the root cause (`throw new RuntimeException('...', 0, $previous)`). PHP 8.0+ allows `throw` as an expression and non-capturing `catch` blocks.

**Beginner-Friendly Explanation**
Error handling means planning for things going wrong. Instead of letting the program crash, you “try” something that might fail, “catch” the failure if it happens, and “finally” clean up regardless. Throw exceptions for unexpected problems, and catch them only where you can actually do something about them.

#### Purposes

- To fail gracefully instead of crashing with a fatal error.
- To separate error-handling code from normal logic.
- To preserve diagnostic information through exception chaining.
- To ensure cleanup (closing files, releasing locks) via `finally`.

#### Syntax Rules and Structure

**General Syntax**
```php
try {
    // risky operation
    $result = riskyOperation();
} catch (SpecificException | AnotherException $e) {
    // handle recoverable error
    error_log($e->getMessage());
} catch (\Throwable $e) {
    // catch-all, re-throw if unrecoverable
    error_log((string) $e);
    throw $e;
} finally {
    // always runs
    cleanup();
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `try` | Contains code that may throw. |
| `catch` | Handles specific exception types; order from narrow to wide. |
| `finally` | Runs regardless of success or failure; used for cleanup. |
| `throw` | Signals an error; PHP 8.0+ allows `throw` as an expression. |
| Exception chaining | Pass the previous exception as the third argument to preserve the root cause. |

**Syntax Rules**

- Catch the most specific exception types first; order `catch` blocks narrow-to-wide.
- Never catch an exception and do nothing; at minimum, log it or re-throw it.
- Use `finally` for cleanup that must happen on both success and failure.
- Prefer typed exceptions (custom exception classes) over generic `\Exception`.

**Constraints and Limitations**

- **PHP 7+ `Throwable`**: In PHP 5, engine errors were not catchable; since PHP 7, `Throwable` unifies errors and exceptions.
- **Performance**: Exceptions are not free; avoid using them for normal control flow.

#### Annotated Code Examples

**Example 1: Typed Exception with Chaining and Finally**

```php
<?php
declare(strict_types=1);

final class InsufficientFundsException extends RuntimeException {}

function withdraw(int $balance, int $amount): int {
    if ($amount <= 0) {
        throw new InvalidArgumentException('Amount must be positive.');
    }
    if ($amount > $balance) {
        throw new InsufficientFundsException(
            sprintf('Requested %d but only %d available.', $amount, $balance)
        );
    }
    return $balance - $amount;
}

try {
    $newBalance = withdraw(5000, 8000);
} catch (InsufficientFundsException $e) {
    echo 'Declined: not enough money.' . PHP_EOL;   // user-safe message
    error_log('Withdraw failed: ' . $e->getMessage()); // log detail
} catch (InvalidArgumentException $e) {
    echo 'Invalid request.' . PHP_EOL;
    error_log('Invalid argument: ' . $e->getMessage());
} finally {
    echo 'Transaction attempt complete.' . PHP_EOL; // always runs
}
?>
```

**Expected Output:**
```
Declined: not enough money.
Transaction attempt complete.
```

**Why This Output:**  
The `withdraw` function throws a typed `InsufficientFundsException` when the amount exceeds the balance. The `catch` block handles this specific type with a user-safe message and logs the detailed message. The `finally` block always runs, confirming that cleanup or logging occurred.

---

**Example 2: Exception Chaining to Preserve Root Cause**

```php
<?php
declare(strict_types=1);

function loadUser(PDO $pdo, int $id): array {
    try {
        $stmt = $pdo->prepare('SELECT id, email FROM users WHERE id = :id');
        $stmt->execute(['id' => $id]);
        $row = $stmt->fetch(PDO::FETCH_ASSOC);
        if ($row === false) {
            throw new RuntimeException("User {$id} not found.");
        }
        return $row;
    } catch (PDOException $e) {
        // Wrap low-level DB error in a domain error, preserving the original
        throw new RuntimeException('Failed to load user.', 0, $e);
    }
}

try {
    loadUser($pdo, 42);
} catch (RuntimeException $e) {
    error_log('Domain error: ' . $e->getMessage());
    if ($e->getPrevious() !== null) {
        error_log('Root cause: ' . $e->getPrevious()->getMessage());
    }
}
?>
```

**Expected Output (in logs):**
```
Domain error: Failed to load user.
Root cause: SQLSTATE[HY000] [2002] Connection refused
```

**Why This Output:**  
The `PDOException` (a low-level database error) is caught and wrapped in a `RuntimeException` (a domain error). The original exception is passed as the `$previous` argument, so callers can inspect `$e->getPrevious()` to see the root cause without exposing SQL details in the user-facing message.

---

#### Real-World Cases

- **HTTP APIs**: Controllers catch domain exceptions and map them to appropriate HTTP status codes (e.g., `InsufficientFundsException` → `402 Payment Required`).
- **Database Access**: Repository methods wrap `PDOException` in domain-specific exceptions to decouple the domain from the database layer.
- **CLI Tools**: `finally` blocks ensure temporary files are deleted and locks are released even when a command fails.

---

### 7. Function Best Practices (Overarching)

#### Definitions

**Core Definition**
Overarching function best practices synthesize the individual practices (naming, length, cohesion, purity, error handling) into a coherent set of guidelines for writing production-grade PHP functions.

**Technical Definition**
The unifying technical standard is **PSR-12**, which specifies formatting, naming, and structural rules. On top of PSR-12, modern PHP best practices add: **`declare(strict_types=1)`** in every file; **full type declarations** on every parameter and return value; **immutable data flow** where possible; **explicit dependencies** via parameters (not globals); and **automated enforcement** through tools like PHP CS Fixer, PHP_CodeSniffer, PHPStan, and Psalm.

**Beginner-Friendly Explanation**
Putting it all together: write small, well-named functions that do one thing; use types everywhere; avoid hidden side effects; handle errors with exceptions; and let tools check your work. These habits make your code predictable, testable, and easy for your team to work with.

#### Purposes

- To provide a single reference for writing consistent, high-quality PHP functions.
- To enable automation (CI checks, static analysis, code style fixing).
- To reduce onboarding time for new team members.
- To minimize technical debt and maintenance costs over time.

#### Syntax Rules and Structure (Guidelines)

| Practice | Description |
|----------|-------------|
| **`declare(strict_types=1)`** | First statement in every file; enforces exact type matching. |
| **Full type declarations** | Every parameter and return value should have a native type. |
| **PSR-12 formatting** | 4-space indentation, opening braces on new lines for functions, camelCase naming. |
| **Single responsibility** | Each function does one thing and does it well. |
| **Pure by default** | Prefer pure functions; isolate side effects in dedicated functions. |
| **Explicit error handling** | Throw typed exceptions; catch specific types; use `finally` for cleanup. |
| **Avoid globals** | Pass dependencies as parameters; do not use `global` inside functions. |

**Constraints and Limitations**

- **Strict types are per-file**: A strict file can call a function defined in a non-strict file, and coercion happens at the call site of the non-strict file.
- **Tooling is not a substitute for design**: Passing PHPStan at max level does not guarantee good design; it catches many bugs but not all design flaws.

#### Annotated Code Examples

**Example 1: A Function That Follows All Best Practices**

```php
<?php
declare(strict_types=1);

namespace App\Services;

use InvalidArgumentException;
use RuntimeException;

final class PriceCalculator
{
    /**
     * Calculate the total price for a list of items, applying a tax rate.
     *
     * @param array<array{price: float, quantity: int}> $items
     * @param float $taxRate  Tax rate as a decimal (e.g., 0.2 for 20%).
     * @return float
     *
     * @throws InvalidArgumentException If the tax rate is negative.
     */
    public function calculateTotal(array $items, float $taxRate): float
    {
        if ($taxRate < 0) {
            throw new InvalidArgumentException('Tax rate cannot be negative.');
        }

        $subtotal = array_sum(
            array_map(
                static fn(array $item): float => $item['price'] * $item['quantity'],
                $items
            )
        );

        return $subtotal * (1 + $taxRate);
    }
}

// Usage
$calculator = new PriceCalculator();
$items = [
    ['price' => 10.0, 'quantity' => 2],
    ['price' => 5.0, 'quantity' => 3],
];

echo $calculator->calculateTotal($items, 0.2) . "\n"; // Outputs: 42
?>
```

**Expected Output:**
```
42
```

**Why This Output:**  
This function exemplifies best practices: it has `strict_types=1`; it uses full type declarations (array, float); it is named as a verb phrase (`calculateTotal`); it is short (under 20 lines); it has high cohesion (it calculates a total and does nothing else); it is pure (no side effects, no external state); it uses an arrow function with `static` to avoid `$this` binding; and it throws a typed `InvalidArgumentException` for invalid input. The calculation is `(10*2 + 5*3) * 1.2 = 35 * 1.2 = 42`.

---

#### Real-World Cases

- **Laravel/Symfony Projects**: Both frameworks enforce strict types, PSR-12, and dependency injection in their coding standards.
- **Static Analysis in CI**: Projects run PHPStan at level 6+ and PHP CS Fixer in CI to automatically enforce type safety and code style.
- **Open-Source Libraries**: Well-maintained packages (e.g., `symfony/console`, `guzzlehttp/guzzle`) follow these practices consistently, making them reliable and easy to integrate.

---

## References

**PSR-1: Basic Coding Standard** - https://www.php-fig.org/psr/psr-1/

**PSR-12: Extended Coding Style** - https://www.php-fig.org/psr/psr-12/

**PHP Manual – User-defined functions** - https://static.php.net/manual/en/functions.user-defined.php

**PHP Manual – Errors and Exceptions** - https://static.php.net/manual/en/language.exceptions.php

**PHPStan – Function Purity** - https://phpstan.org/blog/remembering-and-forgetting-returned-values

**PHP Best Practices – Strict Types (Agent Skills)** - https://github.com/voku/agent-skills/blob/main/skills/php-best-practices/rules/type-strict-mode.md

**Secure PHP Development – Error Handling** - https://github.com/armourinfosec/Secure-PHP-Development/blob/main/Modern-PHP/Error-Handling.md

**Purpose of a Function in PHP (Egghead.io)** - https://egghead.io/lessons/php-purpose-of-a-function-in-php

**PHP Function Cohesion (Code Complete reference)** - https://stackoverflow.com/questions/5117792/does-exitstatus-violate-the-single-responsibility-principle

**Wikia PHP Coding Conventions – Function Length** - https://github.com/Wikia/guidelines/blob/master/PHP/CodingConventions.md

**INFINITUM FORM – PSR Standards and Function Length** - https://infinitumform.com/development-standards/psr-standards-and-the-foundation-of-high-quality-code