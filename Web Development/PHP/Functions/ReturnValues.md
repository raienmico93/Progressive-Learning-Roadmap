Here is a comprehensive research cheat sheet on PHP return values and return types, structured for both academic clarity and beginner accessibility.

---

## Topic Overview: PHP Return Values and Return Types

### Definitions

**Core Definition**
A return value is the data a function sends back to the code that called it. A return type is an optional declaration that specifies what type of value a function will return, enabling PHP to enforce type safety at the boundary of the function.

**Technical Definition**
In PHP, values are returned using the optional `return` statement, which immediately terminates execution of the current function and passes control back to the calling scope. A `return` statement may return any type, including arrays and objects; if it is omitted, the function implicitly returns `null`. Return type declarations, added incrementally across PHP versions, allow developers to annotate the expected type after the parameter list using a colon (`:`), and PHP throws a `TypeError` if the actual returned value does not match the declared type (subject to coercion rules unless `strict_types=1` is enabled). PHP supports a rich set of return types, including scalar types, class types, nullable types, union types, intersection types, and the special return‑only types `void`, `never`, and `static`, as well as the relative class types `self` and `parent`.

**Beginner-Friendly Explanation**
Think of a function as a worker you send on an errand. The **return value** is what the worker brings back to you—the result of the job. The **return type** is like a label on the errand slip that says, “Bring back a number” or “Bring back a receipt.” If the worker brings back something that does not match the label, PHP complains. Some workers never come back (they throw an exception or call `exit`), and PHP has a special label called `never` for that. Others bring back nothing at all, and that is labelled `void`.

---

### Key Characteristics

- **Single value (usually)**: A function can return at most one value via `return`; multiple values can be returned by packaging them in an array or object.
- **Immediate termination**: Executing `return` stops the function immediately; subsequent statements in the function are not executed.
- **Implicit `null`**: Omitting `return` (or using a bare `return;`) returns `null` by default.
- **Type enforcement**: Return type declarations cause a `TypeError` if the returned value does not match, though coercion may occur without `strict_types=1`.
- **Version‑specific types**: `void` and nullable types arrived in PHP 7.1.0; `mixed` and `static` in PHP 8.0.0; `never` in PHP 8.1.0.
- **Covariance**: Return types are covariant, meaning an overriding method may declare a narrower (more specific) return type than the parent method.

---

### Prerequisites

- **PHP 8.1+ Recommended**: `never` requires PHP 8.1.0; `static` and `mixed` require PHP 8.0.0; `void` requires PHP 7.1.0.
- **Basic Function Syntax**: Understanding of function declaration, parameters, and the `return` statement.
- **`declare(strict_types=1)` Awareness**: Strict typing changes whether scalar return values are coerced or whether a `TypeError` is thrown immediately.
- **Object‑Oriented Basics**: `self`, `parent`, and `static` are only meaningful inside class contexts.

---

### Related Programming Areas

- **Function Parameters**: Parameters and return values form the input/output contract of a function.
- **Object‑Oriented Programming (OOP)**: Methods use the same return rules, plus the relative class types (`self`, `parent`, `static`).
- **Type Systems**: Union, intersection, nullable, and DNF types interact with return declarations.
- **Static Analysis**: Tools like PHPStan and Psalm rely heavily on return type declarations to infer program behavior.
- **API Design**: Return types document what callers can expect and enable safer refactoring.

---

## Core Concepts / Features

---

### 1. Return Values and the `return` Statement

#### Definitions

**Core Definition**
A return value is the result a function sends back to its caller. The `return` statement is the language construct that produces this result and terminates the function.

**Technical Definition**
`return` is a language construct (not a function) that immediately ends execution of the current function and returns its argument as the value of the function call. If no argument is supplied, `null` is returned. Any PHP type may be returned, including arrays and objects, and functions may also return by reference using the `&` operator in both the declaration and the assignment.

**Beginner-Friendly Explanation**
`return` is the moment a function says, “Here is your answer—I am done.” It stops the function right there and hands the value back to whoever called it.

#### Purposes

- To pass the result of a computation back to the calling code.
- To terminate a function early when a condition is met.
- To enable function composition, where one function’s output becomes another’s input.
- To signal errors or special conditions by returning sentinel values (though exceptions are often preferable).

#### Syntax Rules and Structure

**General Syntax**
```php
function functionName(...): ReturnType {
    // ...
    return $value;      // returns $value and terminates the function
    // or
    return;             // returns null and terminates the function
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `return $value;` | Terminates the function and passes `$value` to the caller. |
| `return;` | Terminates the function and returns `null`. |
| `: ReturnType` | Optional return type declaration after the parameter list. |
| `function &name()` | Declares that the function returns by reference. |

**Syntax Rules**

- `return` can appear anywhere in the function body; when executed, the function ends immediately.
- A function can return at most one value via `return`; to return multiple values, return an array or object.
- Returning by reference requires `&` in both the function declaration and the assignment at the call site.

**Constraints and Limitations**

- **Single value**: Direct multiple returns are not supported; use arrays or objects.
- **`void` contradiction**: A function declared `: void` may use a bare `return;` to exit early, but it must not return a value; returning by reference from a `void` function is deprecated as of PHP 8.1.0.
- **`never` contradiction**: A `return` statement inside a `never` function is a compile‑time error, even if unreachable.

#### Annotated Code Examples

**Example 1: Basic Return Value**

```php
<?php
function square($num) {
    return $num * $num;   // computes and returns the square
}

echo square(4);           // Outputs: 16
?>
```

**Expected Output:**
```
16
```

**Why This Output:**  
The `return` statement computes `4 * 4 = 16`, terminates `square`, and passes `16` back to `echo`, which prints it.

**Step-by-Step Setup:**
1. Save as `square.php`.
2. Run `php square.php`.
3. The output confirms the returned value is printed.

---

**Example 2: Returning Multiple Values via an Array**

```php
<?php
function small_numbers() {
    return [0, 1, 2];      // returns an array
}

[$zero, $one, $two] = small_numbers();  // array destructuring
var_dump($zero, $one, $two);
?>
```

**Expected Output:**
```
int(0)
int(1)
int(2)
```

**Why This Output:**  
PHP functions cannot return multiple values directly, but returning an array and using array destructuring (or `list()`) achieves the same effect.

---

**Example 3: Returning by Reference**

```php
<?php
function &getReference() {
    static $value = 10;
    return $value;         // returns a reference to $value
}

$ref =& getReference();    // note the & at assignment
$ref = 20;                 // modifies the static $value
echo getReference();       // Outputs: 20
?>
```

**Expected Output:**
```
20
```

**Why This Output:**  
The `&` in both the function declaration and the assignment makes `$ref` an alias for the static `$value`. Changing `$ref` changes `$value`, so the second call returns `20`.

#### Real-World Cases

- **Mathematical helpers**: `square($n)`, `max($a, $b)`, `calculateTax($amount)` return computed results.
- **Data retrieval**: Repository methods `findById(int $id): ?User` return an object or `null`.
- **Validation**: `validateEmail(string $email): bool` returns `true` or `false`; callers branch on the returned boolean.

---

### 2. Return Types (Return Type Declarations)

#### Definitions

**Core Definition**
A return type is an optional annotation in the function signature that declares what type of value the function will return.

**Technical Definition**
Return type declarations were introduced gradually: scalar return types in PHP 7.0, nullable and `void` in PHP 7.1, `object` in PHP 7.2, `iterable` in PHP 7.1, union types in PHP 8.0, `mixed` and `static` in PHP 8.0, intersection types in PHP 8.1, and `never` in PHP 8.1. The declared type is checked at runtime; without `strict_types=1`, scalar return values may be coerced, but with strict types a `TypeError` is thrown for mismatches. Return types are covariant: an overriding method may narrow the return type relative to the parent method.

**Beginner-Friendly Explanation**
A return type is a promise: “This function will give you back a number” or “This function will give you back a `User` object or `null`.” PHP checks the promise and complains if the function breaks it.

#### Purposes

- To make the function’s output contract explicit and self‑documenting.
- To catch type‑related bugs at the return boundary before they propagate.
- To enable static analysis tools to infer types more precisely.
- To support safe refactoring by ensuring overriding methods remain compatible.

#### Syntax Rules and Structure

**General Syntax**
```php
function functionName(...): ReturnType {
    // ...
}
```

**Component Breakdown**

| Return Type Category | Examples | Minimum PHP Version |
|----------------------|----------|---------------------|
| Scalar | `int`, `float`, `string`, `bool` | 7.0 |
| Compound | `array`, `object`, `iterable`, `callable` | 7.0–7.2 |
| Class/Interface | `User`, `Countable` | 5.0 |
| Nullable | `?string` (sugar for `string|null`) | 7.1 |
| `void` | `void` | 7.1 |
| Union | `int|string` | 8.0 |
| `mixed` | `mixed` | 8.0 |
| `static` | `static` | 8.0 |
| Intersection | `A&B` | 8.1 |
| `never` | `never` | 8.1 |
| DNF | `(A&B)|null` | 8.2 |

**Syntax Rules**

- The return type follows a colon after the parameter list: `function foo(): int`.
- `void` and `never` are return‑only types and cannot be used for parameters.
- `mixed` is a top type equivalent to `object|resource|array|string|float|int|bool|null`.
- `self`, `parent`, and `static` are relative class types usable only inside class contexts.

**Constraints and Limitations**

- **`resource` is forbidden** in user‑land return type declarations; use `mixed` or PHPDoc instead.
- **`mixed` or `never` in unions**: Using `mixed` or `never` as part of a union type results in a compile‑time error.
- **`self`/`parent`/`static` in intersections**: Using any of these relative types inside an intersection type is an error.

#### Annotated Code Examples

**Example 1: Scalar Return Type with Strict Types**

```php
<?php
declare(strict_types=1);

function addNumbers(int $a, int $b): int {
    return $a + $b;
}

echo addNumbers(5, 10) . "\n";   // Outputs: 15

try {
    echo addNumbers("5", 10);     // string where int is expected
} catch (TypeError $e) {
    echo "Type error: " . $e->getMessage() . "\n";
}
?>
```

**Expected Output:**
```
15
Type error: addNumbers(): Argument #1 ($a) must be of type int, string given
```

**Why This Output:**  
With `strict_types=1`, the `int` parameter types enforce exact matching. Passing `"5"` throws a `TypeError` at the parameter boundary, before the return type is even evaluated.

---

**Example 2: Class Return Type**

```php
<?php
class User {
    public function __construct(public string $name) {}
}

function createUser(string $name): User {
    return new User($name);
}

$user = createUser("Alice");
echo $user->name . "\n";  // Outputs: Alice
?>
```

**Expected Output:**
```
Alice
```

**Why This Output:**  
The return type `User` guarantees that `createUser` returns a `User` instance (or a subclass). The caller can safely access `$user->name` without checking the type.

---

**Example 3: Nullable Return Type**

```php
<?php
declare(strict_types=1);

function findUser(int $id): ?User {
    return $id === 1 ? new User("Alice") : null;
}

var_dump(findUser(1));  // object(User)
var_dump(findUser(2));  // NULL
?>
```

**Expected Output:**
```
object(User)#1 (1) {
  ["name"]=>
  string(5) "Alice"
}
NULL
```

**Why This Output:**  
The `?User` return type means the function may return a `User` or `null`. The caller must handle the `null` case, which is visible directly in the signature.

#### Real-World Cases

- **Repository methods**: `findById(int $id): ?User` uses a nullable return type to signal that no record may exist.
- **Factory methods**: `createFromArray(array $data): static` (PHP 8.0+) ensures the correct subclass is instantiated.
- **API responses**: `jsonResponse(array $data, int $status = 200): never` can use `never` because the function always calls `exit` after sending the response.

---

### 3. `void` Return Type

#### Definitions

**Core Definition**
`void` is a return type that indicates a function performs an action but does not return a value.

**Technical Definition**
`void` was introduced in PHP 7.1.0 as a return‑only type. A function declared `: void` must not return a value; a bare `return;` is allowed to exit early. Returning by reference from a `void` function is deprecated as of PHP 8.1.0 because it is contradictory. Callers receive `null` when invoking a `void` function.

**Beginner-Friendly Explanation**
`void` means “this function does something but does not give you anything back.” It is like asking someone to turn on the lights—they do the action, but they do not hand you an object.

#### Purposes

- To document clearly that a function is used for its side effects, not for its return value.
- To prevent callers from accidentally relying on a return value that does not exist.
- To make the function’s intent explicit in its signature.

#### Syntax Rules and Structure

**General Syntax**
```php
function functionName(...): void {
    // perform action
    return;          // optional early exit; no value
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `: void` | Declares that the function returns no value. |
| `return;` | Allowed to exit early; must not include a value. |
| `return $value;` | Compile‑time error inside a `void` function. |

**Syntax Rules**

- A `void` function may use `return;` to exit early, but must not return a value.
- Assigning the result of a `void` function to a variable yields `null`.

**Constraints and Limitations**

- **No value returns**: `return $value;` inside a `void` function is a fatal error.
- **No reference returns**: Returning by reference from `void` is deprecated as of PHP 8.1.0.
- **Not usable in unions**: `void` cannot be part of a union type.

#### Annotated Code Examples

**Example 1: Basic `void` Function**

```php
<?php
function logMessage(string $message): void {
    echo "[LOG] $message\n";
    // no return value
}

logMessage("User logged in");
// Outputs: [LOG] User logged in
?>
```

**Expected Output:**
```
[LOG] User logged in
```

**Why This Output:**  
The function performs the side effect of printing. It returns nothing, and the `: void` declaration documents that behavior.

---

**Example 2: Early Exit with `void`**

```php
<?php
function processValue(int $value): void {
    if ($value < 0) {
        echo "Skipping negative value\n";
        return;        // early exit, no value
    }
    echo "Processing $value\n";
}

processValue(-5);   // Outputs: Skipping negative value
processValue(3);    // Outputs: Processing 3
?>
```

**Expected Output:**
```
Skipping negative value
Processing 3
```

**Why This Output:**  
The bare `return;` exits the function early without returning a value, which is allowed in a `void` function.

#### Real-World Cases

- **Logging functions**: `log(string $message): void` performs I/O but has no meaningful return value.
- **Event dispatchers**: `dispatch(Event $event): void` triggers listeners but does not return a result.
- **Setters**: `setName(string $name): void` mutates object state without returning a value.

---

### 4. `never` Return Type (PHP 8.1+)

#### Definitions

**Core Definition**
`never` is a return‑only type indicating that a function never returns normally—it always throws an exception, calls `exit()`, or enters an infinite loop.

**Technical Definition**
`never` was introduced in PHP 8.1.0. It is the bottom type in type theory, meaning it is a subtype of every other type and can replace any other return type during inheritance. A `return` statement inside a `never` function is a compile‑time error, even if unreachable. If control flow reaches the end of a `never` function without exiting, throwing, or looping indefinitely, a `TypeError` is thrown.

**Beginner-Friendly Explanation**
`never` means “this function never comes back.” It either throws an error, stops the whole script with `exit`, or gets stuck in an infinite loop. It is useful for functions whose entire purpose is to terminate the program or redirect the user.

#### Purposes

- To document functions that always terminate the request or throw an exception.
- To help static analysis tools understand that code after a call to a `never` function is unreachable.
- To provide a correct return type for functions like `redirect()` or `abort()` that call `exit` or throw.

#### Syntax Rules and Structure

**General Syntax**
```php
function functionName(...): never {
    // must throw, exit, or loop forever
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `: never` | Declares that the function never returns normally. |
| `throw` | A common way to satisfy `never`. |
| `exit()` / `die()` | Also satisfies `never`. |
| Infinite loop | `while (true) {}` satisfies `never`. |

**Syntax Rules**

- A `return` statement (with or without a value) inside a `never` function is a compile‑time error.
- `never` is a return‑only type and cannot be used for parameters.
- `never` cannot be part of a union type.

**Constraints and Limitations**

- **Must terminate**: If execution reaches the end of a `never` function, a `TypeError` is thrown.
- **No `return`**: Any `return` inside a `never` function is forbidden.
- **Inheritance**: Because `never` is the bottom type, an overriding method may narrow a return type to `never` in a covariant context.

#### Annotated Code Examples

**Example 1: `never` via Exception**

```php
<?php
function abort(int $code, string $message): never {
    throw new RuntimeException("$code: $message");
}

try {
    abort(404, "Not Found");
} catch (RuntimeException $e) {
    echo $e->getMessage() . "\n";
}
// Outputs: 404: Not Found
?>
```

**Expected Output:**
```
404: Not Found
```

**Why This Output:**  
The function always throws a `RuntimeException`, so it never returns normally. The `never` return type correctly documents this behavior.

---

**Example 2: `never` via `exit`**

```php
<?php
function redirect(string $url): never {
    header("Location: $url");
    exit;
}

// redirect("https://example.com"); // script terminates here
?>
```

**Expected Output:**
No output (the script terminates after sending the redirect header).

**Why This Output:**  
`exit` terminates the script, so `redirect` never returns. The `never` return type is satisfied.

#### Real-World Cases

- **HTTP redirects**: `redirect(string $url): never` sends a header and calls `exit`.
- **Error handlers**: `abort(int $code): never` throws an exception or terminates.
- **Assertions**: `assertOrFail(bool $condition): never` throws if the condition is false.

---

### 5. `mixed` Return Type (PHP 8.0+)

#### Definitions

**Core Definition**
`mixed` is a return type that accepts any value: it explicitly states that a function may return a value of any type.

**Technical Definition**
`mixed` was introduced in PHP 8.0.0. It is equivalent to the union type `object|resource|array|string|float|int|bool|null` and is the top type in PHP’s type system, meaning every other type is a subtype of `mixed`. `mixed` cannot be part of a union type; doing so results in an error.

**Beginner-Friendly Explanation**
`mixed` means “anything goes.” If a function can return a number sometimes, a string other times, or even `null`, `mixed` is the honest way to say that. It is better than leaving the return type off, because it explicitly documents that the caller must be prepared for any type.

#### Purposes

- To explicitly document that a function may return values of different types.
- To provide a type‑safe alternative to leaving the return type undeclared.
- To satisfy static analysis tools while acknowledging dynamic behavior.

#### Syntax Rules and Structure

**General Syntax**
```php
function functionName(...): mixed {
    // may return any type
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `: mixed` | Declares that any return type is possible. |
| Equivalent union | `object|resource|array|string|float|int|bool|null` |

**Syntax Rules**

- `mixed` is a return‑only type in the sense that it is a type alias; it can also be used for parameters and properties.
- `mixed` cannot be combined with other types in a union.

**Constraints and Limitations**

- **No union**: `mixed|int` is a compile‑time error because `mixed` already includes `int`.
- **Static analysis**: Tools may require PHPDoc refinements (e.g., `@return int|string`) to narrow the actual types for better inference.

#### Annotated Code Examples

**Example 1: Function Returning `mixed`**

```php
<?php
declare(strict_types=1);

function parseValue(string $input): mixed {
    if (is_numeric($input)) {
        return (int) $input;
    }
    return $input;
}

var_dump(parseValue("42"));    // int(42)
var_dump(parseValue("hello")); // string(5) "hello"
?>
```

**Expected Output:**
```
int(42)
string(5) "hello"
```

**Why This Output:**  
`parseValue` returns an `int` when the input is numeric and a `string` otherwise. The `mixed` return type honestly documents that the caller must handle multiple possible types.

#### Real-World Cases

- **Deserializers**: `json_decode` returns `mixed` because JSON can decode to scalars, arrays, or `null`.
- **Dynamic configuration**: `getConfig(string $key): mixed` may return a string, integer, boolean, array, or `null`.
- **Database drivers**: `fetchColumn()` may return a scalar or `null`, often typed `mixed` in legacy APIs.

---

### 6. `static` Return Type (PHP 8.0+)

#### Definitions

**Core Definition**
`static` is a return‑only type that requires the returned value to be an instance of the class on which the method was actually called (late static binding), not necessarily the class in which the method is declared.

**Technical Definition**
`static` was introduced as a return type in PHP 8.0.0. It is only allowed in return position (and inside complex return type expressions like `?static` or `static|array`) because it is sound only in covariant contexts. For variance purposes, `static` is considered a subtype of `self`. It is commonly used for named constructors, “with‑er” methods on immutable objects, and fluent interfaces.

**Beginner-Friendly Explanation**
`static` means “return an object of the class that was actually used to call this method.” If class `B` extends class `A` and inherits a method declared `: static` from `A`, calling that method on `B` must return a `B`, not an `A`. It is essential for fluent APIs and immutable “wither” methods.

#### Purposes

- To specify that a method returns an instance of the calling class, enabling fluent chaining and immutable updates.
- To support named constructors that always return the correct subclass.
- To express late static binding precisely in the type system.

#### Syntax Rules and Structure

**General Syntax**
```php
class ParentClass {
    public function method(): static {
        return new static();   // returns an instance of the calling class
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `: static` | Return‑only type; value must be an instance of the called class. |
| `new static()` | Instantiates the called class (late static binding). |
| `return $this;` | Often used in fluent methods to return the same object. |

**Syntax Rules**

- `static` is allowed only in return types; it cannot be used for parameters or properties.
- `static` may appear in complex return types such as `?static` or `static|array`.

**Constraints and Limitations**

- **Return‑only**: Not usable for parameters or properties.
- **Subtype of `self`**: An overriding method may narrow `self` to `static`, but not vice versa.
- **Late static binding required**: The method must return an instance of the runtime class, not a hard‑coded parent class.

#### Annotated Code Examples

**Example 1: Fluent Interface with `static`**

```php
<?php
class QueryBuilder {
    private array $parts = [];

    public function select(string $columns): static {
        $this->parts[] = "SELECT $columns";
        return $this;   // returns the same instance (fluent)
    }

    public function from(string $table): static {
        $this->parts[] = "FROM $table";
        return $this;
    }

    public function getQuery(): string {
        return implode(" ", $this->parts);
    }
}

$query = (new QueryBuilder())
    ->select("*")
    ->from("users")
    ->getQuery();

echo $query . "\n";
// Outputs: SELECT * FROM users
?>
```

**Expected Output:**
```
SELECT * FROM users
```

**Why This Output:**  
Each fluent method returns `$this`, and the `: static` return type guarantees that the returned object is an instance of the calling class, allowing chaining.

---

**Example 2: Named Constructor with Late Static Binding**

```php
<?php
class Model {
    public static function create(array $data): static {
        $instance = new static();   // late static binding
        foreach ($data as $key => $value) {
            $instance->$key = $value;
        }
        return $instance;
    }
}

class User extends Model {
    public string $name;
}

$user = User::create(["name" => "Alice"]);
var_dump($user);   // object(User)
?>
```

**Expected Output:**
```
object(User)#1 (1) {
  ["name"]=>
  string(5) "Alice"
}
```

**Why This Output:**  
`User::create` uses `new static()`, which resolves to `User` because the method was called on `User`. The `: static` return type correctly reflects this late static binding.

#### Real-World Cases

- **Immutable value objects**: `withName(string $name): static` clones the object, changes one property, and returns the clone.
- **Named constructors**: `fromArray(array $data): static` creates an instance of the called class.
- **Fluent query builders and HTTP clients**: Methods like `->where()->orderBy()` return `static` to enable chaining.

---

### 7. `self` Return Type

#### Definitions

**Core Definition**
`self` is a relative class type that requires the returned value to be an instance of the **same class in which the type declaration is written**, not the class that was called.

**Technical Definition**
`self` is a relative class type usable only inside classes. It resolves at compile time to the class in which the declaration appears. Unlike `static`, `self` does not perform late static binding; if a method returning `self` is inherited and called on a subclass, it still returns an instance of the declaring class.

**Beginner-Friendly Explanation**
`self` means “return an object of **this exact class**, the one where the code is written.” Even if a subclass calls the method, `self` still refers to the parent class. It is less flexible than `static` but useful when you truly want the declaring class.

#### Purposes

- To guarantee that a method returns an instance of the declaring class specifically.
- To express constraints that should not be widened by inheritance.
- To provide a narrower alternative to `static` when late static binding is not desired.

#### Syntax Rules and Structure

**General Syntax**
```php
class MyClass {
    public function create(): self {
        return new self();   // instance of MyClass
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `: self` | Return type resolving to the declaring class. |
| `new self()` | Instantiates the declaring class. |

**Syntax Rules**

- `self` can be used as a return type inside any class method.
- `self` is resolved at compile time to the class where the declaration appears.
- `self` can also be used in parameter types and property types, unlike `static`.

**Constraints and Limitations**

- **No late static binding**: Calling an inherited `self` method on a subclass still returns the declaring class instance.
- **Covariance**: An overriding method may narrow `self` to `static` (since `static` is a subtype of `self`), but not the reverse.

#### Annotated Code Examples

**Example 1: `self` vs. `static`**

```php
<?php
class A {
    public static function withSelf(): self {
        return new self();
    }
    public static function withStatic(): static {
        return new static();
    }
}

class B extends A {}

var_dump(A::withSelf());   // object(A)
var_dump(B::withSelf());   // object(A) — still A!
var_dump(B::withStatic()); // object(B) — late static binding
?>
```

**Expected Output:**
```
object(A)#1 (0) {}
object(A)#1 (0) {}
object(B)#1 (0) {}
```

**Why This Output:**  
`withSelf` always returns `new self()`, which resolves to `A` regardless of whether it is called as `A::withSelf()` or `B::withSelf()`. `withStatic` uses `new static()`, which resolves to the called class (`B` when called as `B::withStatic()`).

#### Real-World Cases

- **Value objects that must not be subclassed**: `create(): self` ensures a base value object returns its own exact type.
- **Internal factory methods**: When a class internally constructs instances and inheritance should not change the returned type, `self` is appropriate.
- **Avoiding late static binding**: In cases where late static binding could introduce unexpected behavior, `self` locks the type.

---

### 8. `parent` Return Type

#### Definitions

**Core Definition**
`parent` is a relative class type that requires the returned value to be an instance of the **parent class** of the class in which the type declaration is written.

**Technical Definition**
`parent` is a relative class type usable only inside classes that have a parent. It resolves to the immediate parent class of the declaring class. It is commonly used in overriding methods that delegate to the parent implementation.

**Beginner-Friendly Explanation**
`parent` means “return an object of my parent class.” It is useful when a child method overrides a parent method and wants to guarantee that it returns something compatible with the parent’s type.

#### Purposes

- To express that a method returns an instance of the parent class.
- To maintain compatibility with parent method signatures when overriding.
- To document delegation patterns where a child method wraps a parent method’s result.

#### Syntax Rules and Structure

**General Syntax**
```php
class Child extends ParentClass {
    public function method(): parent {
        return parent::method();   // returns a parent-class instance
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `: parent` | Return type resolving to the parent class. |
| `parent::method()` | Calls the parent implementation. |

**Syntax Rules**

- `parent` is available only inside a class that has a parent class.
- `parent` can be used in return types, parameter types, and property types.

**Constraints and Limitations**

- **No parent, no `parent`**: Using `parent` in a class with no parent is a fatal error.
- **Variance**: Because `parent` is a wider type than `self` or `static`, an overriding method cannot narrow a parent return type to `parent` if the parent declared `self` or `static`.

#### Annotated Code Examples

**Example 1: Overriding with `parent` Return Type**

```php
<?php
class Animal {
    public function create(): Animal {
        return new Animal();
    }
}

class Dog extends Animal {
    public function create(): parent {   // parent = Animal
        return parent::create();          // returns Animal
    }
}

$dog = new Dog();
var_dump($dog->create());   // object(Animal)
?>
```

**Expected Output:**
```
object(Animal)#2 (0) {}
```

**Why This Output:**  
`Dog::create` declares `: parent`, which resolves to `Animal`. It calls `parent::create()`, which returns `new Animal()`. The returned object is an `Animal`, satisfying the `parent` type.

#### Real-World Cases

- **Decorator patterns**: A child decorator method returns the parent’s result while adding behavior.
- **Template method patterns**: A subclass method returns a parent‑class object after performing additional steps.
- **Legacy inheritance**: When overriding a method that already declares a parent return type, `parent` can be used to match the signature.

---

### 9. Return Best Practices

#### Definitions

**Core Definition**
Return best practices are guidelines for choosing return types and structuring return logic to produce safe, readable, and maintainable functions.

**Technical Definition**
Best practices include: always declaring a return type when possible; using `void` for side‑effect‑only functions; using `never` for functions that always terminate; preferring specific types over `mixed`; using `?Type` for nullable returns; enabling `strict_types=1`; keeping return types consistent; and following covariance rules in inheritance. Return type declarations should be compatible with parent methods, and PHPDoc may supplement native types for richer static analysis.

**Beginner-Friendly Explanation**
Best practices are like good manners for functions: always tell callers what they will get back (`: int`, `: ?User`, `: void`), do not surprise them with unexpected types, and use `never` when the function never comes back at all.

#### Purposes

- To make function contracts explicit and predictable.
- To catch type errors at the return boundary before they cause subtle bugs.
- To improve static analysis and IDE support.
- To ensure inheritance hierarchies remain type‑safe through covariance.

#### Syntax Rules and Structure (Guidelines)

| Practice | Description |
|----------|-------------|
| **Always declare a return type** | Use native types whenever possible; use PHPDoc for finer constraints. |
| **Use `void` for side effects** | Functions that only log, print, or mutate state should return `void`. |
| **Use `never` for termination** | Functions that always `exit` or throw should return `never`. |
| **Avoid `mixed` when possible** | Prefer union types or specific types; use `mixed` only when truly any type is possible. |
| **Enable `strict_types=1`** | Enforces exact return type matching and prevents silent coercion. |
| **Respect covariance** | Overriding methods may narrow return types (e.g., `self` → `static`), but not widen them. |
| **Keep returns consistent** | If a function returns a value on success, do not return `null` or `false` without a documented reason. |

**Constraints and Limitations**

- **`void` and `never` are return‑only**: They cannot be used for parameters.
- **`mixed` cannot be unioned**: `mixed|int` is invalid.
- **`static` is return‑only**: It cannot be used for parameters or properties.
- **`self` and `parent` are class‑only**: They cannot be used outside class contexts.

#### Annotated Code Examples

**Example 1: Poor vs. Good Return Type Design**

```php
<?php
// POOR: no return type, ambiguous return
function findUser($id) {
    if ($id === 1) {
        return new User("Alice");
    }
    return null;   // implicit nullable
}

// GOOD: explicit nullable return type, strict types
declare(strict_types=1);

function findUserGood(int $id): ?User {
    return $id === 1 ? new User("Alice") : null;
}
?>
```

**Expected Output:**
No output (definitions only).

**Why This Output:**  
The good version uses `?User` to explicitly document that the function may return `null`, and `strict_types=1` ensures the `int` parameter is not silently coerced. The return contract is clear to both humans and static analysis tools.

---

**Example 2: Using `never` for Guaranteed Termination**

```php
<?php
declare(strict_types=1);

function abort(int $code): never {
    http_response_code($code);
    exit;
}
?>
```

**Expected Output:**
No output (script terminates).

**Why This Output:**  
`abort` always calls `exit`, so it never returns normally. The `never` return type correctly documents this and helps static analysis understand that code after `abort()` is unreachable.

#### Real-World Cases

- **Repository interfaces**: `findById(int $id): ?User` uses a nullable return type; `findAll(): array` returns an array.
- **Command handlers**: `handle(Command $command): void` performs side effects without returning a value.
- **HTTP redirects**: `redirect(string $url): never` sends a header and exits.
- **Fluent APIs**: `withName(string $name): static` returns a new instance of the calling class for chaining.

---

## References

**PHP Manual – Returning values** - https://static.php.net/manual/en/functions.returning-values.php

**PHP Manual – Type declarations** - https://static.php.net/manual/en/language.types.declarations.php

**PHP Manual – Never** - https://static.php.net/manual/en/language.types.never.php

**PHP Manual – Mixed** - https://static.php.net/manual/en/language.types.mixed.php

**PHP Manual – Relative class types (self, parent, static)** - https://www.php.net/manual/en/language.types.relative-class-types.php

**PHP RFC: Static return type** - https://wiki.php.net/rfc/static_return_type

**PHP RFC: Never For Parameter Types (context for bottom type)** - https://wiki.php.net/rfc/never_for_parameter_types

**PHP RFC: Union Types 2.0 (covariance of return types)** - https://wiki.php.net/rfc/union_types_v2

**PHP Manual – Type System (mixed alias)** - https://static.php.net/manual/en/language.types.type-system.php