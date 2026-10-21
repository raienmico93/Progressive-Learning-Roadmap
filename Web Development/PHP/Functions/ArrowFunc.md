Here is a comprehensive research cheat sheet on PHP Arrow Functions, structured for both academic clarity and beginner accessibility.

---

## Topic Overview: PHP Arrow Functions

### Definitions

**Core Definition**
Arrow functions are a concise syntax for anonymous functions in PHP, introduced in PHP 7.4. They are declared with the `fn` keyword and consist of a parameter list, the `=>` operator, and a single expression that is implicitly returned.

**Technical Definition**
Arrow functions (also called “short closures”) are implemented using the `Closure` class, just like traditional anonymous functions. The basic form is `fn (argument_list) => expr`. The key distinction is that variables used in the expression are automatically captured **by value** from the parent scope, without requiring an explicit `use` clause. Arrow functions support the same signatures as anonymous functions—including parameter types, return types, default values, variadics, and by-reference passing/returning—with two notable exceptions: the body must be a single expression, and they cannot declare a `void` return type.

**Beginner-Friendly Explanation**
An arrow function is a shorthand way to write a small anonymous function. If your function just needs to return a single value—like “multiply this number by 2” or “get the name of this user”—you can write `fn($x) => $x * 2` instead of the longer `function ($x) { return $x * 2; }`. They automatically grab any outside variables they need, but they can only grab copies, not the originals. Think of them as sticky notes with a single instruction, ready to be used wherever a callback is needed.

---

### Key Characteristics

- **Concise Syntax**: `fn` keyword, `=>` operator, and a single expression replace the verbose `function` syntax and `return` statement.
- **Automatic By-Value Capture**: Variables from the parent scope are implicitly imported by value; no `use` clause is needed or permitted.
- **Single Expression Only**: The body must be an expression, not a block of statements; `return` is implicit and forbidden.
- **`$this` Auto-Binding**: When defined inside a class method, `$this` is automatically bound unless the arrow function is declared `static`.
- **No `void` Return Type**: Unlike anonymous functions, arrow functions cannot declare `: void` because they must return the value of the expression.
- **Lowest Precedence**: The `=>` operator has the lowest precedence, so the expression on the right is consumed as fully as possible.

---

### Prerequisites

- **PHP 7.4+**: Arrow functions were introduced in PHP 7.4.0.
- **Basic Anonymous Functions**: Understanding of closures and the `use` keyword helps appreciate what arrow functions simplify.
- **`declare(strict_types=1)` Awareness**: Strict types affect how parameter and return types are enforced, just as with anonymous functions.

---

### Related Programming Areas

- **Callbacks and Higher-Order Functions**: Arrow functions are the idiomatic way to pass short transforms to `array_map`, `array_filter`, `usort`, etc..
- **Functional Programming**: Enable concise expression-level transformations without side effects.
- **Object-Oriented Programming**: `$this` binding and `static` arrow functions interact with class scope.
- **Coding Standards (PSR-12)**: Specific whitespace rules apply to `fn` and `=>`.

---

## Core Concepts / Features

---

### 1. Arrow Functions (`fn() =>`)

#### Definitions

**Core Definition**
An arrow function is a short-form anonymous function declared with `fn`, designed for single-expression callbacks.

**Technical Definition**
The syntax is `fn (parameter_list) => expr`. The expression `expr` is evaluated and its value is implicitly returned. Arrow functions are `Closure` objects and support the same parameter and return type declarations as anonymous functions, except that the body is restricted to a single expression and a `void` return type is forbidden.

**Beginner-Friendly Explanation**
An arrow function is a one-line function. You write `fn` instead of `function`, use `=>` instead of curly braces and `return`, and keep everything on a single line (or a single expression).

#### Purposes

- To reduce boilerplate when passing simple callbacks to functions like `array_map` and `array_filter`.
- To improve readability by making the transformation expression the visual focus rather than the surrounding syntax.
- To encourage side-effect-free, expression-oriented code for common collection operations.

#### Syntax Rules and Structure

**General Syntax**
```php
$fn = fn (parameter_list) => expr;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `fn` | The keyword; must not be followed by a space before the parameter list. |
| `(parameter_list)` | Parameters, with optional types, defaults, and variadics. |
| `=>` | The arrow operator; must be surrounded by spaces per PSR-12. |
| `expr` | A single expression whose value is returned. |

**Syntax Rules**

- The body is exactly one expression; there is no `{ ... }` block and no explicit `return`.
- `fn` and `(` are written together: `fn($x)`, not `fn ($x)` (spacing rule).
- The `=>` must have a space before and after it: `fn($x) => $x * 2`.
- Type declarations are allowed on parameters and return: `fn(int $x): int => $x * 2`.
- By-reference parameter passing and returning are supported: `fn(&$x) => $x++` (parameter by reference) and `fn&($x) => $x` (return by reference).

**Constraints and Limitations**

- **Single expression only**: You cannot use multiple statements, `if` blocks, loops, or explicit `return` statements.
- **No `void` return type**: Declaring `: void` causes a fatal error because the arrow function always returns the expression’s value.
- **By-value capture only**: You cannot modify parent-scope variables directly; use a traditional closure with `use (&$var)` if mutation is required.

#### Annotated Code Examples

**Example 1: Basic Arrow Function as a Callback**

```php
<?php
$numbers = [1, 2, 3, 4];
$factor = 10;

// Arrow function captures $factor automatically by value
$scaled = array_map(fn($n) => $n * $factor, $numbers);

print_r($scaled);
// Outputs: Array ( [0] => 10 [1] => 20 [2] => 30 [3] => 40 )
?>
```

**Expected Output:**
```
Array
(
    [0] => 10
    [1] => 20
    [2] => 30
    [3] => 40
)
```

**Why This Output:**  
The arrow function `fn($n) => $n * $factor` captures `$factor` (value `10`) automatically. `array_map` applies it to each element, producing the scaled values.

**Step-by-Step Setup:**
1. Save the code as `arrow_map.php`.
2. Run `php arrow_map.php`.
3. The output shows each number multiplied by 10.

---

**Example 2: Arrow Function with Type Declarations**

```php
<?php
declare(strict_types=1);

$users = [
    ['name' => 'Alice', 'active' => true],
    ['name' => 'Bob', 'active' => false],
    ['name' => 'Carol', 'active' => true],
];

// Extract names of active users using an arrow function with types
$activeNames = array_map(
    fn(array $u): string => $u['name'],
    array_filter($users, fn(array $u): bool => $u['active'])
);

print_r($activeNames);
// Outputs: Array ( [0] => Alice [1] => Carol )
?>
```

**Expected Output:**
```
Array
(
    [0] => Alice
    [1] => Carol
)
```

**Why This Output:**  
The inner arrow function filters for `active === true`; the outer arrow function extracts the `name` as a string. The return types `: bool` and `: string` are enforced under `strict_types=1`, ensuring the expressions return the declared types.

---

**Example 3: By-Reference Parameter (Call-Site, Not Capture)**

```php
<?php
$x = 1;

// Arrow function with a by-reference parameter
$fn = fn(&$val) => $val++;

$fn($x);           // Passes $x by reference at call time
var_export($x);    // Outputs: 2
?>
```

**Expected Output:**
```
2
```

**Why This Output:**  
The `&$val` in the parameter list makes the parameter receive the argument by reference. Passing `$x` at the call site allows the arrow function to increment the original variable. This is distinct from auto-capture: the by-value auto-capture rule applies only to variables *used in the expression from the parent scope*, not to parameters explicitly declared by reference.

---

#### Real-World Cases

- **Collection Transformation**: `array_map(fn($u) => $u->email, $users)` extracts a field from every object.
- **Filtering**: `array_filter($numbers, fn($n) => $n % 2 === 0)` keeps only even numbers.
- **Sorting**: `usort($products, fn($a, $b) => $a->price <=> $b->price)` sorts by price in one line.
- **Factory Functions**: `function multiplierOf(int $factor): Closure { return fn($n) => $n * $factor; }` returns a configured arrow function.

---

### 2. Auto-Capture (`fn() =>` Capture Semantics)

#### Definitions

**Core Definition**
Auto-capture is the mechanism by which arrow functions implicitly import variables from the parent scope **by value**, without an explicit `use` clause.

**Technical Definition**
When a variable used in the arrow function’s expression is defined in the parent scope, PHP automatically binds that variable by value at the time the arrow function is created. This is roughly equivalent to writing `use ($var)` for every such variable. Because the binding is by value, the arrow function receives a copy; modifications inside the arrow function do not affect the outer variable. Nested arrow functions also capture by value through each layer.

**Beginner-Friendly Explanation**
Auto-capture means the arrow function automatically “takes a snapshot” of any outside variables it needs. It does not need a permission slip (`use`), but it also cannot change the originals—it only gets copies.

#### Purposes

- To eliminate the boilerplate of manually listing every needed variable in a `use` clause.
- To make simple callbacks visually cleaner and easier to read.
- To enforce a side-effect-free style by preventing accidental mutation of parent-scope variables.

#### Syntax Rules and Structure

**General Syntax**
```php
$outer = 10;
$fn = fn($x) => $x + $outer;   // $outer is auto-captured by value
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Used variable | Any variable referenced in the expression that exists in the parent scope is auto-captured. |
| Capture mode | By value only; no ampersand is permitted for auto-capture. |
| Nested capture | Inner arrow functions capture from their immediate parent, propagating values outward. |

**Syntax Rules**

- Auto-capture applies only to variables that are **literally used** in the expression. Variable variables (`$$name`) are not auto-captured.
- The capture happens at **definition time**, not call time; the value is fixed when the arrow function is created.
- If the variable is undefined at definition time, an undefined-variable notice is generated when the arrow function is called (not at definition time).

**Constraints and Limitations**

- **No by-reference auto-capture**: You cannot write `fn() => $x++` and expect `$x` to change; the arrow function operates on a copy.
- **No modification of outer scope**: To mutate an outer variable, use a traditional anonymous function with `use (&$var)`.
- **Literal use required**: Dynamic variable access (variable variables) is not captured automatically.

#### Annotated Code Examples

**Example 1: By-Value Snapshot at Definition Time**

```php
<?php
$message = 'hello';

$fn = fn() => $message;   // captures 'hello' at definition time

$message = 'world';        // change the outer variable

echo $fn() . "\n";        // Outputs: hello (the captured copy)
?>
```

**Expected Output:**
```
hello
```

**Why This Output:**  
The arrow function captured the value `'hello'` when it was defined. Later reassigning `$message` to `'world'` does not affect the already-captured copy inside the arrow function.

---

**Example 2: Modification Has No Effect on Outer Scope**

```php
<?php
$x = 1;
$fn = fn() => $x++;   // increments the captured copy
$fn();
var_export($x);        // Outputs: 1
?>
```

**Expected Output:**
```
1
```

**Why This Output:**  
The arrow function captured `$x` by value. The expression `$x++` increments the local copy, not the outer `$x`. The outer variable remains `1`.

---

**Example 3: Nested Arrow Functions Capture Through Layers**

```php
<?php
$z = 1;

$fn = fn($x) => fn($y) => $x * $y + $z;

echo $fn(5)(10) . "\n";   // Outputs: 51
?>
```

**Expected Output:**
```
51
```

**Why This Output:**  
The outer arrow function captures `$z = 1`. It returns an inner arrow function that captures `$x = 5` (from the outer call) and also has access to `$z` through the outer capture chain. The inner call with `10` computes `5 * 10 + 1 = 51`.

---

#### Real-World Cases

- **Configuration-Driven Transforms**: `$taxRate = 0.2; $addTax = fn($price) => $price * (1 + $taxRate);` — the tax rate is snapshotted; later changes to `$taxRate` do not retroactively affect existing arrow functions.
- **Template Rendering**: `$prefix = 'user_'; $formatKey = fn($id) => $prefix . $id;` — the prefix is captured once per arrow function definition.
- **Higher-Order Factories**: `function multiplierOf(int $factor): Closure { return fn($n) => $n * $factor; }` — each returned arrow function captures its own `$factor` by value.

---

### 3. Arrow Function Best Practices

#### Definitions

**Core Definition**
Arrow function best practices are guidelines for using arrow functions appropriately—where they improve clarity and where another construct is safer or more readable.

**Technical Definition**
Best practices recommend using arrow functions **only for tiny, single-expression transforms with no side effects**. Logic with branching, multiple statements, I/O, or mutation should use a named function, a method, or a traditional anonymous function. Static arrow functions should be used when `$this` is not needed to avoid unnecessary object binding and potential memory-cycle issues. PSR-12 spacing rules (`fn(` with no space, ` => ` with spaces) should be followed for consistency.

**Beginner-Friendly Explanation**
Use arrow functions for quick, simple jobs—like “multiply by 2” or “get the name.” Do not try to cram complex logic into them. If you need multiple steps, an `if` statement, or to change an outside variable, use a regular function instead.

#### Purposes

- To keep code readable by using the right tool for the job.
- To avoid unreadable one-liners that hide complex logic inside an arrow function.
- To prevent subtle bugs caused by assuming auto-capture allows mutation.
- To align with coding standards and static analysis expectations.

#### Guidelines

| Practice | Description |
|----------|-------------|
| **Single expression only** | Arrow functions are for expression-level transforms; avoid forcing multi-step logic into them. |
| **No side effects** | Do not perform I/O, logging, or mutation inside an arrow function; those belong in a named method or regular closure. |
| **Use `static fn` when `$this` is unused** | Prevents `$this` binding, which can create object cycles and make garbage collection less predictable. |
| **Do not rely on auto-capture for mutation** | If you need to change an outer variable, use a traditional closure with `use (&$var)`; arrow functions capture by value only. |
| **Follow PSR-12 spacing** | Write `fn(` with no space, and ` => ` with spaces on both sides. |
| **Extract complex logic to a method** | If an arrow function becomes a nested ternary or requires early returns, move the logic to a private method and call that method from the arrow function. |

**Constraints and Limitations**

- **No `void` return type**: Never attempt to declare `: void` on an arrow function; it is a fatal error.
- **No statements**: You cannot use `if`, `foreach`, `try`, or explicit `return` inside an arrow function.
- **`$this` binding**: Unless declared `static`, an arrow function defined in a class method keeps a reference to `$this`, which may prolong the object’s lifetime.

#### Annotated Code Examples

**Example 1: Good vs. Bad Use of Arrow Functions**

```php
<?php
declare(strict_types=1);

// GOOD: simple, focused transform
$multiplier = 3;
$numbers = [1, 2, 3];
$tripled = array_map(fn(int $n): int => $n * $multiplier, $numbers);

// BAD: complex logic crammed into an arrow function
$processed = array_map(
    fn(Order $o): string => $o->isPaid()
        ? ($o->isShipped() ? 'delivered' : 'processing')
        : ($o->isCancelled() ? 'cancelled' : 'pending'),
    $orders
);

// BETTER: extract the logic to a method
$processed = array_map(fn(Order $o): string => $o->statusLabel(), $orders);
?>
```

**Expected Output:**
No output (definitions only). The “bad” example is difficult to read and violates the single-responsibility principle; the “better” example delegates complex branching to a method.

**Why This Guidance:**  
Arrow functions are designed to make simple transforms readable. When the expression becomes a multi-level ternary, the arrow function syntax hides the complexity rather than clarifying it.

---

**Example 2: Static Arrow Function to Avoid `$this` Binding**

```php
<?php
class Service {
    private string $data = 'large payload';

    public function getCallback(): Closure {
        // BAD: Binds $this, keeping the Service object alive
        // return fn() => 'ok';

        // GOOD: Static arrow function does not bind $this
        return static fn() => 'ok';
    }
}

$service = new Service();
$callback = $service->getCallback();
unset($service);   // Service can be garbage-collected
echo $callback() . "\n"; // Outputs: ok
?>
```

**Expected Output:**
```
ok
```

**Why This Output:**  
The `static` keyword prevents the arrow function from binding `$this`. After `unset($service)`, the `Service` object has no remaining references from the arrow function and can be freed.

---

**Example 3: PSR-12 Spacing**

```php
<?php
// CORRECT per PSR-12
$fn = fn(int $x, int $y): int => $x + $y;

// INCORRECT: space between fn and (
// $fn = fn (int $x) => $x;

// INCORRECT: no spaces around =>
// $fn = fn($x)=>$x;
?>
```

**Expected Output:**
No output (definitions only).

**Why This Guidance:**  
PSR-12 and common PHP coding standards require `fn(` without a space and ` => ` with spaces. Following these rules ensures consistency across codebases and avoids style-checker failures.

---

#### Real-World Cases

- **Laravel Collections**: The framework encourages arrow functions for simple `map`/`filter` callbacks but recommends named methods for complex transformations.
- **Symfony Codebase**: Symfony’s coding standards enforce static closures and arrow functions where `$this` is unused, reducing memory leaks in long-running processes.
- **Static Analysis (PHPStan/Psalm)**: Tools flag arrow functions that attempt side effects or contain unreachable code, reinforcing the single-expression, side-effect-free rule.

---

## References

**PHP Manual – Arrow Functions** - https://static.php.net/manual/en/functions.arrow.php

**PHP RFC: Arrow Functions 2.0** - https://wiki.php.net/rfc/arrow_functions_v2

**PHP RFC: Arrow Functions (Original)** - https://wiki.php.net/rfc/arrow_functions

**PHP RFC: Short Closures 2.0 (Auto-capture context)** - https://wiki.php.net/rfc/short_closures_v2

**PHP Best Practices – Modern Arrow Functions (Agent Skills)** - https://github.com/voku/agent-skills/blob/main/skills/php-best-practices/rules/modern-arrow-functions.md

**PSR-12 Extended Coding Style (Spacing for Short Closures)** - https://www.php-fig.org/psr/psr-12/

**PHP Bug #79517 – Cannot specify “void” as return type of arrow function** - https://bugs.php.net/bug.php?id=79517

**PHP: Static Keyword (Context for static arrow functions)** - https://static.php.net/manual/en/language.oop5.static.php