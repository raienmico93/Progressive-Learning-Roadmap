Here is a comprehensive research cheat sheet on PHP variable scope, structured for both academic clarity and beginner accessibility.

---

## Topic Overview: PHP Variable Scope

### Definitions

**Core Definition**
Variable scope in PHP defines the context—such as a specific function, a class, or the entire script—within which a variable is defined and can be accessed. Every variable in PHP has a scope, which dictates where it can be read or modified.

**Technical Definition**
The PHP language specification states that "the scope of a variable is the context within which it is defined" and that PHP primarily uses a **function scope** and a **global scope**. Variables declared outside any function reside in the global scope and are only directly accessible from that global context. Conversely, a variable created inside a function is confined to a new, local function scope for each invocation; it does not exist in the global scope, and any attempt to access a global variable from within a function without explicit declaration will result in an undefined variable notice (or warning in modern PHP). This contrasts with languages like C, where globals are visible inside functions unless shadowed. PHP also provides mechanisms to bridge these scopes, such as the `global` keyword and the `$GLOBALS` superglobal array, and special scope modifiers like `static` for preserving a local variable's value across function calls.

**Beginner-Friendly Explanation**
Think of variable scope as the "visibility" of a variable. A variable declared at the top of your script (global scope) is like a public announcement—everyone outside of any special enclosed area can see it. But the moment you step inside a function, you are in a private room (local scope). A variable created in that room is invisible to the outside world, and the outside world's variables are invisible to you unless you explicitly open a door (using `global` or `$GLOBALS`). This isolation prevents functions from accidentally overwriting important data and makes code more predictable.

---

### Key Characteristics

- **Function Isolation**: Each user-defined function introduces a new local scope. Variables inside it are inaccessible from the global scope and from other functions unless explicitly passed or imported.
- **Global Scope Dominance**: Variables defined outside functions are globally scoped. They are accessible from other files via `include` or `require` as long as the inclusion occurs in the global scope.
- **Explicit Bridging**: To use a global variable inside a function, you must use the `global` keyword or access it through `$GLOBALS`. PHP does not automatically make globals available in local scope.
- **`static` Persistence**: The `static` keyword creates a local variable that does not lose its value when the function exits, preserving state between calls.
- **Superglobal Ubiquity**: A set of built-in variables (like `$_GET`, `$_POST`, `$_SERVER`) are "superglobals," meaning they are available in every scope without any special declaration.
- **Block Insensitivity**: Unlike Java or C++, PHP variables declared inside control structures (`if`, `for`, `while`) are **not** block-scoped; they remain visible in the enclosing function or global scope.

---

### Prerequisites

- **Basic PHP Syntax**: Familiarity with variables (`$var`), functions (`function name() { ... }`), and inclusion (`include`/`require`).
- **PHP 8.0+ Recommended**: Modern PHP versions provide stricter warnings for undefined variables and improved `static` variable handling (e.g., shared statics in inherited methods since PHP 8.1).
- **Understanding of References**: The `global` and `static` keywords are implemented using PHP references, which can lead to subtle behaviors if not understood.

---

### Related Programming Areas

- **Functions**: Parameters and return values are the primary ways to pass data across scopes without relying on globals.
- **Object-Oriented Programming (OOP)**: Properties and methods have their own visibility rules (`public`, `protected`, `private`) and do not share the same scoping model as procedural variables.
- **Superglobals and Security**: Understanding superglobals is critical for handling user input securely and for avoiding common vulnerabilities.
- **Closures and Anonymous Functions**: Closures have their own scoping rules and can inherit variables from the parent scope explicitly via the `use` language construct.
- **Namespaces**: While not a variable scope, namespaces affect the resolution of class and function names and are a key tool for avoiding global namespace pollution.

---

## Core Concepts / Features

---

### 1. Global Scope

#### Definitions

**Core Definition**
Global scope is the default scope for variables declared outside of any function or class method. Variables in this scope are accessible throughout the main script and in any included files (when the inclusion occurs in the global scope).

**Technical Definition**
In PHP, any variable defined outside a function is limited to the global scope. The global scope spans the entire script, including code brought in via `include` or `require`, with the included file inheriting the variable scope of the line on which the inclusion occurs. Global variables can be accessed from within functions only by using the `global` keyword or the `$GLOBALS` superglobal array; they are not automatically available in local function scopes.

**Beginner-Friendly Explanation**
Global scope is the "main area" of your script. Variables you create here are like public tools that any part of your main script can use. However, if you go into a function (a "private workshop"), those tools are not automatically on your workbench—you have to explicitly ask for them.

#### Purposes

- To share configuration data, database connections, or other application-wide resources across different parts of a script.
- To allow included files to contribute to and read from a common set of variables.
- To provide a fallback for simple scripts where formal dependency injection is unnecessary.

#### Syntax Rules and Structure

**General Syntax**
```php
<?php
$globalVar = "I am global";   // Declared in global scope

function readGlobal() {
    // $globalVar is NOT available here without 'global' or $GLOBALS
}
?>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Variable declaration | Any variable assignment outside a function creates a global variable. |
| Inclusion | `include 'file.php';` in global scope makes global variables visible to the included file. |

**Syntax Rules**

- A variable is global if and only if it is first assigned outside any function or method.
- Included files inherit the scope of the line where `include` or `require` is called.
- To access a global variable inside a function, you must use `global $var;` or `$GLOBALS['var']`.

**Constraints and Limitations**

- **Not automatically visible**: Unlike C, PHP does not make global variables available inside functions by default.
- **Naming collisions**: The more variables you place in the global scope, the higher the risk of conflicts with third-party code.
- **`$GLOBALS` is read-only since PHP 8.1**: Writing to `$GLOBALS` directly is no longer permitted, though modifying array elements may still work in some contexts.

#### Annotated Code Examples

**Example 1: Global Variable Accessed via `global` Keyword**

```php
<?php
$counter = 10;                 // Global scope

function incrementCounter() {
    global $counter;           // Import the global into local scope
    $counter++;                // Modifies the global variable
}

incrementCounter();
echo $counter;                 // Outputs: 11
?>
```

**Expected Output:**
```
11
```

**Why This Output:**  
The `global $counter;` statement creates a local reference to the global `$counter`. Incrementing it inside the function therefore modifies the original global variable.

**Step-by-Step Setup:**
1. Save the code as `global_test.php`.
2. Run `php global_test.php` from the command line.
3. The output `11` confirms that the global variable was successfully accessed and modified.

---

**Example 2: Included File Inheriting Global Scope**

```php
<?php
// main.php
$appName = "MyApp";
include 'header.php';
?>
```

```php
<?php
// header.php
echo "Welcome to $appName\n";   // $appName is visible here
?>
```

**Expected Output (when running `main.php`):**
```
Welcome to MyApp
```

**Why This Output:**  
The `include` statement in `main.php` occurs in the global scope, so `header.php` inherits that scope and can access `$appName`.

#### Real-World Cases

- **Configuration Files**: A `config.php` file defines `$dbHost`, `$dbUser`, etc., in the global scope; other scripts include it and use these variables.
- **Legacy Procedural Scripts**: Older PHP applications often rely on global variables for application state, though this is discouraged in modern practice.

---

### 2. Local Scope (Function Scope)

#### Definitions

**Core Definition**
Local scope, also called function scope, is the isolated context created every time a function is called. Variables declared inside a function exist only within that function's body and are destroyed when the function returns (unless declared `static`).

**Technical Definition**
"Any variable created inside a named function or an anonymous function is limited to the scope of the function body". Each invocation of a function creates a fresh local scope; variables do not persist between calls unless the `static` keyword is used. Local variables are not accessible from the global scope or from other functions. PHP does **not** have block-level scoping: a variable declared inside an `if` or `for` block remains accessible throughout the entire function scope.

**Beginner-Friendly Explanation**
Local scope is the function's private workspace. Anything you create here is temporary and invisible to the outside world. Each time you call the function, you get a clean workspace (unless you use `static` to save something for next time).

#### Purposes

- To prevent functions from accidentally interfering with global data or with each other.
- To allow the same variable name to be reused in different functions without conflict.
- To enable recursion by giving each recursive call its own set of local variables.

#### Syntax Rules and Structure

**General Syntax**
```php
<?php
function myFunction() {
    $localVar = "I am local";   // Local to this function call
    echo $localVar;
}
// $localVar is not defined here
?>
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Function body | The `{ ... }` block introduces a new local scope. |
| Variable assignment | Any variable first assigned inside the function is local to that function. |

**Syntax Rules**

- Local variables are created on function entry (for parameters) or when first assigned.
- They are destroyed when the function returns, unless declared `static`.
- Block statements (`if`, `for`, `while`) do **not** create new scopes; a variable assigned inside a loop is still local to the enclosing function.

**Constraints and Limitations**

- **No block scope**: Variables declared inside loops or conditionals are visible throughout the function, which can lead to subtle bugs if not initialized carefully.
- **No automatic global access**: A local variable with the same name as a global variable **shadows** the global; the global is not modified unless explicitly imported.

#### Annotated Code Examples

**Example 1: Local Variable Isolated from Global Scope**

```php
<?php
$name = "Global Alice";

function greet() {
    $name = "Local Bob";       // Shadows the global $name
    echo "Inside: $name\n";
}

greet();
echo "Outside: $name\n";       // Global $name is unchanged
?>
```

**Expected Output:**
```
Inside: Local Bob
Outside: Global Alice
```

**Why This Output:**  
Inside `greet()`, a new local variable `$name` is created. It shadows the global `$name` but does not modify it. The global remains `"Global Alice"`.

---

**Example 2: No Block Scope in PHP**

```php
<?php
function checkLoopScope() {
    for ($i = 0; $i < 3; $i++) {
        if ($i == 1) {
            $found = true;     // Assigned inside an if-block
        }
    }
    var_dump(isset($found));   // $found is still accessible!
}

checkLoopScope();              // Outputs: bool(true)
?>
```

**Expected Output:**
```
bool(true)
```

**Why This Output:**  
PHP does not create a new scope for the `if` block. `$found` is assigned inside the `if` but remains in the function's local scope, so it is accessible after the loop.

#### Real-World Cases

- **Recursive Functions**: Each recursive call has its own local scope, so variables like `$depth` or `$result` do not collide between calls.
- **Utility Functions**: A function like `calculateTax($amount)` uses local variables for intermediate calculations, keeping the global namespace clean.

---

### 3. The `global` Keyword

#### Definitions

**Core Definition**
The `global` keyword is used inside a function to create a reference to a variable from the global scope, allowing the function to read and modify that global variable.

**Technical Definition**
"The `global` keyword is used to bind a variable from a global scope into a local scope". It can be used with a single variable or a comma-separated list. A local variable is created that references the global variable of the same name. If no global variable with that name exists, one is created in the global scope and assigned `null`. The `global` statement creates a reference, not a copy, meaning modifications inside the function affect the global variable directly. An alternative is to use the `$GLOBALS` array, which is an associative array of all global variables.

**Beginner-Friendly Explanation**
The `global` keyword is like a special key that lets you reach out of your function's private room and grab a tool from the main workshop. You are not copying the tool; you are using the actual tool, so any changes you make will be visible to everyone.

#### Purposes

- To allow a function to modify application state stored in a global variable.
- To provide a simple (though often discouraged) way to share data without passing parameters.
- To serve as a bridge in legacy codebases that rely heavily on global state.

#### Syntax Rules and Structure

**General Syntax**
```php
function myFunction() {
    global $globalVar;          // Single variable
    global $var1, $var2;        // Multiple variables
    // or
    $GLOBALS['globalVar'] = "new"; // Alternative syntax
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `global $var;` | Creates a local reference to the global `$var`. |
| `global $a, $b;` | Multiple globals in one statement. |
| `$GLOBALS['var']` | Array access to the global variable. |

**Syntax Rules**

- `global` must be used inside a function; using it in the global scope has no effect.
- The variable name following `global` is the global variable's name (without a dollar sign in the `$GLOBALS` array syntax).
- `global` creates a reference, so assignments to the local alias affect the global variable.

**Constraints and Limitations**

- **Reference semantics**: Assigning a new reference to a variable that was imported with `global` can break the link to the global variable (see the manual's reference example).
- **`$GLOBALS` read-only (PHP 8.1+)**: Writing directly to `$GLOBALS` is no longer allowed; however, modifying its elements (e.g., `$GLOBALS['x'] = 5`) may still work in some contexts but is discouraged.
- **Discouraged in modern code**: Best practices recommend avoiding `global` in favor of dependency injection or passing parameters.

#### Annotated Code Examples

**Example 1: Modifying a Global with `global`**

```php
<?php
$total = 0;

function addToTotal($amount) {
    global $total;
    $total += $amount;          // Modifies the global $total
}

addToTotal(10);
addToTotal(5);
echo $total;                   // Outputs: 15
?>
```

**Expected Output:**
```
15
```

**Why This Output:**  
Each call to `addToTotal` imports the global `$total` and increments it. Because `global` creates a reference, the global variable accumulates the sum.

---

**Example 2: Using `$GLOBALS` Instead of `global`**

```php
<?php
$a = 1;
$b = 2;

function sumUsingGlobals() {
    $GLOBALS['b'] = $GLOBALS['a'] + $GLOBALS['b'];
}

sumUsingGlobals();
echo $b;                       // Outputs: 3
?>
```

**Expected Output:**
```
3
```

**Why This Output:**  
The `$GLOBALS` array provides direct access to global variables. Assigning to `$GLOBALS['b']` modifies the global `$b` without needing the `global` keyword.

#### Real-World Cases

- **Legacy WordPress Plugins**: Older plugins often use `global $wpdb;` to access the WordPress database object.
- **Simple Counters or Flags**: A global `$debugMode` flag can be toggled in a configuration script and read inside functions using `global $debugMode;`.

---

### 4. The `static` Keyword (Static Variables)

#### Definitions

**Core Definition**
The `static` keyword, when applied to a local variable inside a function, causes that variable to retain its value between successive calls to the function. The variable remains local in scope but is not destroyed when the function exits.

**Technical Definition**
"A static variable exists only in a local function scope, but it does not lose its value when program execution leaves this scope". The initialization (`static $count = 0;`) is executed only on the first call. Subsequent calls skip the initialization and use the preserved value. As of PHP 8.3.0, static variables can be initialized with arbitrary expressions, including function calls. Static variables in methods are shared across all instances of a class unless the method is overridden (since PHP 8.1.0, inherited methods share the parent's static variables).

**Beginner-Friendly Explanation**
A `static` variable is like a notebook you keep in your function's private room. Every time you leave the room and come back, the notebook is still there with whatever you last wrote in it. It is not visible from the outside, but it remembers its value between visits.

#### Purposes

- To implement counters, memoization, or other stateful behavior without polluting the global scope.
- To avoid expensive re-initialization (e.g., caching a database connection inside a function).
- To create unique identifiers or sequence generators that persist across function calls.

#### Syntax Rules and Structure

**General Syntax**
```php
function myFunction() {
    static $count = 0;          // Initialized only once
    $count++;
    echo $count;
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `static $var;` | Declares a static local variable. |
| `static $var = expr;` | Initializes it (only on first call). As of PHP 8.3, `expr` can be any expression. |

**Syntax Rules**

- The static declaration must occur inside a function or method.
- Initialization is performed only on the first call to the function.
- Static variables are shared across all calls to the same function, even if called from different places.
- In class methods, static variables are shared across all instances of the class.

**Constraints and Limitations**

- **Not truly global**: Static variables are still local to the function; they cannot be accessed from outside.
- **Initialization restrictions (pre-8.3)**: Before PHP 8.3.0, static variable initializers had to be constant expressions; function calls were not allowed.
- **Reference pitfalls**: Assigning a reference to a static variable can lead to unexpected behavior because references are not stored statically.

#### Annotated Code Examples

**Example 1: A Simple Counter**

```php
<?php
function counter() {
    static $count = 0;          // Initialized only once
    $count++;
    echo $count . "\n";
}

counter();  // Outputs: 1
counter();  // Outputs: 2
counter();  // Outputs: 3
?>
```

**Expected Output:**
```
1
2
3
```

**Why This Output:**  
The `static $count = 0;` line is executed only on the first call. On subsequent calls, the initialization is skipped, and `$count` retains its previous value.

---

**Example 2: Static Variables in a Class Method**

```php
<?php
class Counter {
    public static function increment() {
        static $count = 0;
        $count++;
        return $count;
    }
}

echo Counter::increment() . "\n";  // Outputs: 1
echo Counter::increment() . "\n";  // Outputs: 2
?>
```

**Expected Output:**
```
1
2
```

**Why This Output:**  
The static variable `$count` is shared across all calls to `Counter::increment()`, regardless of whether they are made from different instances (there are no instances here, but the behavior is the same).

#### Real-World Cases

- **Memoization**: A function that computes a expensive result can cache it in a static variable and return the cached value on subsequent calls.
- **Request-Scoped Singletons**: In procedural code, a `getDatabaseConnection()` function can use a static variable to reuse the same connection within a single request.
- **Debug Counters**: A static variable can count how many times a particular function was called during a script's execution.

---

### 5. Superglobals

#### Definitions

**Core Definition**
Superglobals are built-in PHP variables that are automatically available in **all scopes**—global, function, and method—without any need for the `global` keyword or `$GLOBALS` array.

**Technical Definition**
"Superglobals are built-in variables that are always available in all scopes". They are associative arrays that PHP populates with data from the web server, the execution environment, and user input. The standard superglobals include `$GLOBALS`, `$_SERVER`, `$_GET`, `$_POST`, `$_FILES`, `$_COOKIE`, `$_SESSION`, `$_REQUEST`, and `$_ENV`. They are accessible from any function or method without explicit importation. As of PHP 5.4, you cannot use a superglobal's name as a function parameter (a practice called "shadowing").

**Beginner-Friendly Explanation**
Superglobals are like universal remote controls that are available in every room of the house. No matter where you are—in the main script or deep inside a function—you can pick up `$_GET` or `$_POST` and use it. They are PHP's way of making important environmental data accessible everywhere.

#### Purposes

- To provide immediate access to HTTP request data (`$_GET`, `$_POST`), server information (`$_SERVER`), session state (`$_SESSION`), and cookies (`$_COOKIE`) from anywhere in the code.
- To eliminate the need for passing common environmental data through function parameters.
- To serve as the standard interface between the web server and PHP application code.

#### Syntax Rules and Structure

**General Syntax**
```php
// Accessible anywhere, no declaration needed
$userId = $_GET['id'];
$serverName = $_SERVER['SERVER_NAME'];
```

**Component Breakdown (Common Superglobals)**

| Superglobal | Description |
|-------------|-------------|
| `$GLOBALS` | All variables in the global scope. |
| `$_SERVER` | Server and execution environment information. |
| `$_GET` | Query string parameters (HTTP GET). |
| `$_POST` | Form data from HTTP POST requests. |
| `$_FILES` | Uploaded file information. |
| `$_COOKIE` | HTTP cookies. |
| `$_SESSION` | Session variables. |
| `$_REQUEST` | Combined GET, POST, and COOKIE data (discouraged). |
| `$_ENV` | Environment variables. |

**Syntax Rules**

- Superglobals are available automatically in every scope.
- They are associative arrays; access elements with string keys.
- You cannot use a superglobal's name as a parameter in a function declaration (e.g., `function foo($_GET)` is a fatal error).

**Constraints and Limitations**

- **Security**: Superglobals like `$_GET`, `$_POST`, `$_COOKIE`, `$_SERVER` (partially), and `$_REQUEST` contain **user-supplied data** and must never be trusted without validation and sanitization.
- **`$_REQUEST` is discouraged**: It merges GET, POST, and COOKIE data, making it difficult to know the source and increasing security risks.
- **`$GLOBALS` read-only (PHP 8.1+)**: Direct assignment to `$GLOBALS` is no longer allowed.

#### Annotated Code Examples

**Example 1: Accessing `$_GET` Inside a Function**

```php
<?php
function getSearchTerm() {
    // $_GET is available here without 'global'
    return $_GET['q'] ?? 'No search term';
}

// Simulate a query string: ?q=PHP
$_GET['q'] = 'PHP';
echo getSearchTerm();   // Outputs: PHP
?>
```

**Expected Output:**
```
PHP
```

**Why This Output:**  
`$_GET` is a superglobal and is therefore automatically available inside `getSearchTerm()` without any special declaration. The null coalescing operator (`??`) provides a fallback.

---

**Example 2: Using `$_SERVER` to Get the Request Method**

```php
<?php
function isPostRequest(): bool {
    return $_SERVER['REQUEST_METHOD'] === 'POST';
}

var_dump(isPostRequest());   // Outputs: bool(false) in a GET request
?>
```

**Expected Output (in a GET context):**
```
bool(false)
```

**Why This Output:**  
`$_SERVER['REQUEST_METHOD']` contains the HTTP method used to access the script. In a typical browser GET request, it is `"GET"`, so the function returns `false`.

#### Real-World Cases

- **Form Handling**: `$_POST['username']` and `$_POST['password']` are used to retrieve submitted login credentials (after sanitization).
- **Routing**: `$_SERVER['REQUEST_URI']` is used by frameworks to determine which route the user is requesting.
- **Session Management**: `$_SESSION['user_id']` stores the logged-in user's ID across requests.

---

### 6. Scope Best Practices

#### Definitions

**Core Definition**
Scope best practices are guidelines for managing variable visibility in a way that minimizes bugs, avoids naming conflicts, and produces code that is easy to reason about and test.

**Technical Definition**
Best practices include limiting the use of global variables, avoiding the `global` keyword in favor of explicit parameter passing or dependency injection, using `static` variables judiciously for state that genuinely needs to persist across calls, and treating superglobals as untrusted input that must be sanitized at the boundary of the application. Namespacing is recommended to prevent collisions in the global namespace.

**Beginner-Friendly Explanation**
Best practices are like good housekeeping rules for your variables: keep your global workspace tidy, don't leave tools lying around where they can be accidentally knocked over, and always check the mail (superglobals) for dangerous items before bringing it inside.

#### Purposes

- To reduce the risk of naming collisions and accidental overwrites.
- To make functions easier to test in isolation by avoiding hidden dependencies on global state.
- To improve security by centralizing the validation and sanitization of user input from superglobals.
- To make code more predictable and maintainable over time.

#### Guidelines

| Practice | Description |
|----------|-------------|
| **Minimize global variables** | Limit the number of variables added to the global scope to prevent naming conflicts with third-party code. |
| **Avoid `global`; pass parameters instead** | Dependencies should be passed explicitly through constructors or function parameters rather than imported with `global`. |
| **Use `static` sparingly** | Static variables create hidden state that persists across calls; use them only when persistence is genuinely required (e.g., memoization, connection reuse). |
| **Namespace your code** | Place functions and classes in dedicated namespaces to avoid polluting the global namespace. |
| **Sanitize superglobals at the boundary** | Read superglobals once, validate and sanitize the data, and convert it into typed value objects before passing it deeper into the application. |
| **Never trust `$_REQUEST`** | Avoid `$_REQUEST` because it merges GET, POST, and COOKIE data, obscuring the source and increasing security risk. |
| **Initialize local variables** | Because PHP has no block scope, explicitly initialize variables before loops or conditionals to avoid using stale values from previous iterations. |

**Constraints and Limitations**

- **Global state hampers testing**: Functions that rely on globals are harder to unit test because they require setting up global state.
- **Static variables obscure dependencies**: Hidden state in static variables can make debugging difficult, especially in long-running processes.
- **Superglobals are user input**: Data from `$_GET`, `$_POST`, `$_COOKIE`, and `$_SERVER` (like `HTTP_USER_AGENT`) can be manipulated by the client and must be validated.

#### Annotated Code Examples

**Example 1: Poor vs. Good Scope Practice**

```php
<?php
// POOR: Uses global state, hidden dependency
$taxRate = 0.20;

function calculateTotalPoor($amount) {
    global $taxRate;
    return $amount + ($amount * $taxRate);
}

// GOOD: Explicit dependency, no global
function calculateTotalGood(float $amount, float $taxRate): float {
    return $amount + ($amount * $taxRate);
}

echo calculateTotalGood(100, 0.20) . "\n"; // Outputs: 120
?>
```

**Expected Output:**
```
120
```

**Why This Output:**  
The "good" version makes the dependency (`$taxRate`) explicit as a parameter, making the function pure, testable, and independent of global state.

---

**Example 2: Sanitizing a Superglobal at the Boundary**

```php
<?php
function getUserIdFromRequest(): ?int {
    $rawId = $_GET['id'] ?? null;
    if ($rawId === null || !ctype_digit($rawId)) {
        return null;   // Invalid or missing
    }
    return (int) $rawId;
}

// Simulate: ?id=42
$_GET['id'] = '42';
var_dump(getUserIdFromRequest());   // Outputs: int(42)

// Simulate: ?id=abc
$_GET['id'] = 'abc';
var_dump(getUserIdFromRequest());   // Outputs: NULL
?>
```

**Expected Output:**
```
int(42)
NULL
```

**Why This Output:**  
The function reads the superglobal once, validates that it contains only digits, and returns a typed integer (or `null` for invalid input). This isolates the rest of the application from raw user input.

#### Real-World Cases

- **Framework Request Objects**: Modern frameworks like Laravel and Symfony wrap superglobals in request objects, sanitize input, and inject the objects into controllers—avoiding direct superglobal access in application code.
- **Dependency Injection Containers**: Instead of using `global $db`, a DI container passes a database connection object to the classes that need it.
- **Static Memoization in Pure Functions**: A function like `expensiveCalculation(int $n): int` can use a static array as a cache keyed by `$n`, but the cache is an implementation detail that does not affect the function's external contract.

---

## References

**PHP Manual – Variable scope** - https://static.php.net/manual/en/language.variables.scope.php

**PHP Manual – Superglobals** - https://www.php.net/manual/en/language.variables.superglobals.php

**PHP Manual – Predefined Variables** - https://static.php.net/manual/en/reserved.variables.php

**PHP Manual – Static Keyword** - https://static.php.net/manual/it/language.oop5.static.php

**PHP Manual – Tips (Namespacing and Global Scope)** - https://static.php.net/manual/en/userlandnaming.tips.php

**PHP Best Practices – Avoid Globals (Agent Skills)** - https://github.com/voku/agent-skills/blob/main/skills/php-best-practices/rules/perf-avoid-globals.md

**OWASP PHP Security Cheat Sheet (Superglobal Trust)** - https://wiki.owasp.org/index.php/PHP_Security_Cheat_Sheet

**PHP Manual – References with global and static variables** - https://static.php.net/manual/en/language.variables.scope.php#language.variables.scope.references