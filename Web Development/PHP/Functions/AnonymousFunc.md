Here is a comprehensive research cheat sheet on PHP anonymous functions, closures, arrow functions, and related concepts, structured for both academic clarity and beginner accessibility.

---

## Topic Overview: PHP Anonymous Functions and Closures

### Definitions

**Core Definition**
An anonymous function is a function without a specified name, created inline and typically used as a value (e.g., a callback) or assigned to a variable. In PHP, anonymous functions are implemented as objects of the built-in `Closure` class. The term "closure" is often used interchangeably with "anonymous function," though technically a closure is the object that encapsulates the function and its bound scope.

**Technical Definition**
PHP anonymous functions, introduced in PHP 5.3, are created using the `function` keyword without a name. They are automatically converted into instances of the `Closure` class, which provides methods for further control (binding, calling, etc.). By default, an anonymous function does not automatically inherit variables from its parent scope; variables must be explicitly imported using the `use` language construct. When declared inside a class context, the current class is automatically bound to the closure, making `$this` available unless the closure is declared `static`. Arrow functions (`fn() =>`), introduced in PHP 7.4, are a more concise syntax for anonymous functions that automatically capture variables from the parent scope by value.

**Beginner-Friendly Explanation**
An anonymous function is like a sticky note with instructions that you can create on the spot without giving it a permanent name. You can hand it to someone else (pass it as a callback), stick it to a variable (store it), or use it immediately. A **closure** is the "envelope" that holds the sticky note along with any extra information (variables) you explicitly put inside. Arrow functions are an even shorter way to write these sticky notes when the instructions are just a single expression.

---

### Key Characteristics

- **No Name Required**: Anonymous functions do not have a name; they are defined inline.
- **`Closure` Objects**: Every anonymous function is an instance of the `Closure` class, which provides methods like `bindTo()`, `bind()`, and `call()`.
- **Explicit Variable Capture (`use`)**: Traditional anonymous functions do not automatically see parent scope variables; they must be imported via `use`. Arrow functions capture parent scope variables automatically by value.
- **Automatic `$this` Binding**: When defined inside a class, closures automatically bind `$this` unless declared `static`.
- **Static Closures**: Prefixing a closure with `static` prevents automatic `$this` binding and object binding, which can help avoid memory leaks.
- **Arrow Functions**: Concise syntax (`fn() => expr`), automatic by-value capture, and limited to a single expression.
- **First-Class Callable Syntax (PHP 8.1+)**: `strlen(...)` creates a closure from a callable without intermediate syntax.

---

### Prerequisites

- **PHP 5.3+**: Anonymous functions and the `Closure` class were introduced in PHP 5.3.0.
- **PHP 7.4+**: Arrow functions require PHP 7.4.0.
- **PHP 8.1+**: First-class callable syntax requires PHP 8.1.0.
- **Object-Oriented Basics**: Understanding of `$this`, static methods, and class scope is necessary for closure binding.

---

### Related Programming Areas

- **Callbacks and Higher-Order Functions**: Anonymous functions are the primary way to pass behavior as an argument (e.g., `array_map`, `usort`).
- **Event Handling and Routing**: Closures are used extensively in modern frameworks for route definitions and event listeners.
- **Functional Programming**: Closures enable partial application, currying, and composition.
- **Dependency Injection**: Closures can act as lazy factories.
- **Memory Management**: Static closures and explicit `unset()` of captured references help control object lifetimes.

---

## Core Concepts / Features

---

### 1. Anonymous Functions (`function() {}`)

#### Definitions

**Core Definition**
An anonymous function is a function created without a name, typically assigned to a variable or passed directly as an argument.

**Technical Definition**
An anonymous function is created using the `function` keyword followed by an optional parameter list and a body, but no function name. It is most commonly used as the value of a `callable` parameter. PHP automatically converts the expression into an instance of the `Closure` class. The syntax is `function (parameter_list) use (variables) { body }`.

**Beginner-Friendly Explanation**
An anonymous function is a function you write on the fly without giving it a permanent name. It is like writing a quick recipe on a scrap of paper and handing it to someone, rather than writing a whole cookbook.

#### Purposes

- To provide inline callback logic without the overhead of defining a named function.
- To encapsulate small, one-off operations that are not needed elsewhere.
- To enable functional programming patterns like `array_map`, `array_filter`, and `usort`.

#### Syntax Rules and Structure

**General Syntax**
```php
$closure = function ($param1, $param2) use ($outerVar) {
    // body
    return $result;
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `function` | The keyword indicating a function definition. |
| `($param1, $param2)` | The parameter list; may be empty. |
| `use ($outerVar)` | Optional clause to import variables from the parent scope. |
| `{ ... }` | The function body. |

**Syntax Rules**

- The semicolon after the closing brace is required when assigning to a variable.
- A return type declaration, if used, must appear after the `use` clause: `function () use ($x): int { ... }`.
- As of PHP 8.0.0, a trailing comma is allowed in the `use` list.

**Constraints and Limitations**

- **Explicit capture**: Variables from the parent scope are not visible unless imported via `use`.
- **`$this` auto-binding**: Inside a class, `$this` is automatically bound unless the closure is declared `static`.
- **Cannot capture superglobals**: As of PHP 7.1, `use` cannot import superglobals, `$this`, or variables with the same name as a parameter.

#### Annotated Code Examples

**Example 1: Basic Anonymous Function as a Callback**

```php
<?php
$numbers = [1, 2, 3, 4];

// Anonymous function used as a callback in array_map
$squared = array_map(function ($n) {
    return $n * $n;
}, $numbers);

print_r($squared);
// Outputs: Array ( [0] => 1 [1] => 4 [2] => 9 [3] => 16 )
?>
```

**Expected Output:**
```
Array
(
    [0] => 1
    [1] => 4
    [2] => 9
    [3] => 16
)
```

**Why This Output:**  
The anonymous function `function ($n) { return $n * $n; }` is passed to `array_map`, which applies it to each element of `$numbers`. The result is a new array containing the squares.

**Step-by-Step Setup:**
1. Save the code as `anonymous_callback.php`.
2. Run `php anonymous_callback.php`.
3. The output shows the squared values.

---

**Example 2: Assigning an Anonymous Function to a Variable**

```php
<?php
$greet = function ($name) {
    return "Hello, $name!";
};

echo $greet("World") . "\n"; // Outputs: Hello, World!
?>
```

**Expected Output:**
```
Hello, World!
```

**Why This Output:**  
The anonymous function is assigned to `$greet`. Calling `$greet("World")` invokes the closure with the argument `"World"`, which returns the greeting string.

#### Real-World Cases

- **Routing**: `$router->get('/home', function () { return view('home'); });`
- **Sorting**: `usort($users, function ($a, $b) { return $a->age <=> $b->age; });`
- **Filtering**: `array_filter($numbers, function ($n) { return $n % 2 === 0; });`

---

### 2. Closures (The `Closure` Class)

#### Definitions

**Core Definition**
A closure is the object that encapsulates an anonymous function along with its bound scope (variables and `$this`). In PHP, every anonymous function is an instance of the `Closure` class.

**Technical Definition**
The `Closure` class (PHP 5.3+) represents anonymous functions. It provides methods for further control after creation: `bindTo()` and `bind()` (for rebinding `$this` and class scope), `call()` (for binding and immediately invoking), and `fromCallable()` (for converting a callable into a closure). The class also implements `__invoke()`, allowing closure objects to be called like functions.

**Beginner-Friendly Explanation**
The closure is the "envelope" that holds your anonymous function. It also carries any variables you stuffed into it with `use`, and it knows which object (`$this`) it belongs to. The `Closure` class gives you tools to open the envelope, change its contents, or hand it to someone else.

#### Purposes

- To provide an object-oriented representation of anonymous functions.
- To allow rebinding of `$this` and class scope for advanced use cases.
- To enable conversion of any callable into a closure for uniform handling.

#### Syntax Rules and Structure

**General Syntax**
```php
$closure = function () { ... };
// $closure is an instance of Closure

// Checking type
var_dump($closure instanceof Closure); // bool(true)
```

**Component Breakdown**

| Method | Description |
|--------|-------------|
| `bindTo(?object $newThis, object\|string\|null $newScope = "static"): ?Closure` | Duplicates the closure with a new bound object and class scope. |
| `bind(Closure $closure, ?object $newThis, object\|string\|null $newScope = "static"): ?Closure` | Static version of `bindTo()`. |
| `call(object $newThis, mixed ...$args): mixed` | Binds `$this` to `$newThis` and calls the closure. |
| `fromCallable(callable $callback): Closure` | Converts a callable into a closure. |

**Syntax Rules**

- `Closure` objects cannot be instantiated directly with `new`; they are created by anonymous function expressions or `Closure::fromCallable()`.
- `bindTo()` returns a **new** closure; it does not modify the original.

**Constraints and Limitations**

- **Static closures cannot be bound to objects**: A closure declared `static` cannot have `$this` bound to it, though its class scope can still be changed.
- **Internal classes cannot be used as scope**: The `newScope` parameter cannot be an internal class.

#### Annotated Code Examples

**Example 1: Checking the Closure Type**

```php
<?php
$fn = function () { return "Hello"; };

var_dump($fn instanceof Closure); // Outputs: bool(true)

// Closure implements __invoke, so it can be called like a function
echo $fn() . "\n"; // Outputs: Hello
?>
```

**Expected Output:**
```
bool(true)
Hello
```

**Why This Output:**  
The anonymous function expression creates a `Closure` object. The `instanceof` check confirms this, and because `Closure` implements `__invoke()`, the object can be called directly.

#### Real-World Cases

- **Dependency Injection Containers**: Closures are used as lazy service factories; `Closure::fromCallable()` wraps existing callables.
- **Middleware Pipelines**: Closures are passed through a pipeline and invoked with `$next` closures.

---

### 3. The `use` Keyword

#### Definitions

**Core Definition**
The `use` keyword is used in the declaration of an anonymous function to import variables from the parent scope into the closure's scope.

**Technical Definition**
The `use` clause appears after the parameter list and before the return type and body: `function () use ($var1, &$var2) { ... }`. By default, variables are imported **by value**, meaning the closure receives a copy of the variable's value at the time the closure is **defined**. To import by reference, prefix the variable with `&` (`use (&$var)`). As of PHP 7.1, `use` cannot import superglobals, `$this`, or variables whose names collide with parameters. As of PHP 8.0, a trailing comma in the `use` list is allowed.

**Beginner-Friendly Explanation**
The `use` keyword is like a permission slip. Without it, your anonymous function cannot see the variables in the outside world. With `use ($name)`, you are saying, "Please bring a copy of `$name` into my function." If you use `use (&$name)`, you are bringing the actual variable, so changes inside the function affect the outside world.

#### Purposes

- To make parent scope variables available inside a closure without relying on globals.
- To control whether captured variables are copied (by value) or shared (by reference).
- To explicitly document which external variables a closure depends on.

#### Syntax Rules and Structure

**General Syntax**
```php
$closure = function ($param) use ($outerVar, &$refVar) {
    // $outerVar is a copy; $refVar is a reference
};
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `use ($var)` | Imports `$var` by value (a copy at definition time). |
| `use (&$var)` | Imports `$var` by reference (shared with the parent scope). |
| Position | Must appear after the parameter list and before the return type. |

**Syntax Rules**

- By-value capture takes the value at the moment the closure is **defined**, not when it is called.
- By-reference capture reflects changes made to the variable after the closure is defined.
- The `use` clause is not available in arrow functions (they capture automatically).

**Constraints and Limitations**

- **Cannot capture `$this`**: `$this` is automatically available in non-static closures defined in class context; it cannot be listed in `use`.
- **Cannot capture superglobals**: `use ($_GET)` is invalid as of PHP 7.1.
- **Reference pitfalls**: By-reference capture of a variable that is later reassigned can lead to unexpected behavior.

#### Annotated Code Examples

**Example 1: By-Value vs. By-Reference Capture**

```php
<?php
$message = 'hello';

// By value: captures the value at definition time
$byValue = function () use ($message) {
    return $message;
};

// By reference: shares the variable
$byRef = function () use (&$message) {
    return $message;
};

$message = 'world';

echo $byValue() . "\n"; // Outputs: hello (value at definition)
echo $byRef() . "\n";   // Outputs: world (current value)
?>
```

**Expected Output:**
```
hello
world
```

**Why This Output:**  
`$byValue` captured the value `'hello'` when the closure was defined. `$byRef` captured a reference to `$message`, so when `$message` is changed to `'world'`, the closure sees the new value.

---

**Example 2: Modifying a Captured Variable by Reference**

```php
<?php
$count = 0;

$increment = function () use (&$count) {
    $count++;
};

$increment();
$increment();
echo $count . "\n"; // Outputs: 2
?>
```

**Expected Output:**
```
2
```

**Why This Output:**  
The `&$count` capture allows the closure to modify the original `$count` variable. Each call increments it.

#### Real-World Cases

- **Cart Total Calculation**: `function ($quantity, $product) use ($tax, &$total) { ... }` accumulates a total by reference while reading the tax rate by value.
- **Recursive Closures**: A closure captures itself by reference to enable recursion: `$fn = function ($n) use (&$fn) { ... }`.

---

### 4. Arrow Functions (`fn() =>`)

#### Definitions

**Core Definition**
Arrow functions are a concise syntax for anonymous functions, introduced in PHP 7.4. They automatically capture variables from the parent scope by value and are limited to a single expression.

**Technical Definition**
An arrow function is declared with the `fn` keyword, followed by a parameter list, the `=>` operator, and a single expression: `fn (argument_list) => expr`. Arrow functions are implemented using the `Closure` class. Unlike traditional anonymous functions, variables used in the expression are **implicitly captured by value** from the parent scope; no `use` clause is required or permitted. Arrow functions support the same signatures as anonymous functions (parameter types, return types, default values, variadics, by-reference passing/returning), but they cannot have a `void` return type declaration.

**Beginner-Friendly Explanation**
Arrow functions are a shorthand for simple anonymous functions. If your function just returns a single expression, you can write `fn($x) => $x * 2` instead of the longer `function ($x) { return $x * 2; }`. They automatically grab any outside variables they need, but they can only grab copies, not the originals.

#### Purposes

- To reduce boilerplate for simple callback functions.
- To improve readability when the function body is a single expression.
- To enable concise functional-style code with `array_map`, `array_filter`, etc.

#### Syntax Rules and Structure

**General Syntax**
```php
$fn = fn ($param) => $param * 2;
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `fn` | The keyword for arrow functions (no space after `fn`). |
| `($param)` | Parameter list (can be empty: `fn() => 42`). |
| `=>` | The arrow operator (must be surrounded by spaces per PSR-12). |
| `expr` | A single expression whose value is returned. |

**Syntax Rules**

- The body must be a **single expression**, not a block of statements.
- Variables from the parent scope are captured automatically by value; `use` is not allowed.
- Arrow functions cannot have a `void` return type declaration.
- Nested arrow functions also capture automatically.

**Constraints and Limitations**

- **Single expression only**: You cannot write multiple statements or use `return` explicitly.
- **By-value capture only**: You cannot modify parent scope variables (no by-reference capture).
- **No `void` return type**: Unlike anonymous functions, arrow functions cannot declare `: void`.

#### Annotated Code Examples

**Example 1: Basic Arrow Function**

```php
<?php
$y = 1;
$fn = fn($x) => $x + $y;   // $y is automatically captured by value

echo $fn(3) . "\n";         // Outputs: 4
?>
```

**Expected Output:**
```
4
```

**Why This Output:**  
The arrow function captures `$y` (value `1`) automatically. When called with `3`, it computes `3 + 1 = 4`.

---

**Example 2: Arrow Function with `array_map`**

```php
<?php
$factor = 10;
$numbers = [1, 2, 3, 4];

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
The arrow function `fn($n) => $n * $factor` captures `$factor` automatically and multiplies each element by `10`.

---

**Example 3: Arrow Function Cannot Modify Outer Variable**

```php
<?php
$x = 1;
$fn = fn() => $x++;   // Has no effect on $x
$fn();
var_export($x);        // Outputs: 1
?>
```

**Expected Output:**
```
1
```

**Why This Output:**  
Arrow functions capture by value, so `$x` inside the arrow function is a copy. Incrementing the copy does not affect the original `$x`.

#### Real-World Cases

- **Collection Transformations**: `array_map(fn($u) => $u->name, $users)`
- **Filtering**: `array_filter($items, fn($i) => $i->isActive())`
- **Sorting**: `usort($products, fn($a, $b) => $a->price <=> $b->price)`

---

### 5. Closure Binding (`bindTo()` and `bind()`)

#### Definitions

**Core Definition**
Closure binding is the process of creating a new closure with a different bound object (`$this`) and/or a different class scope.

**Technical Definition**
`Closure::bindTo()` is an instance method that duplicates a closure with a new bound object (`$newThis`) and class scope (`$newScope`), returning a new `Closure` object. `Closure::bind()` is the static equivalent, taking the closure as its first argument. The bound object determines the value of `$this` inside the closure; the class scope determines which `private` and `protected` members are accessible. Static closures cannot have an object bound to them (`$newThis` must be `null`), but their class scope can still be changed.

**Beginner-Friendly Explanation**
Closure binding is like giving your anonymous function a new identity. You can tell it, "Pretend you belong to **this** object instead of the one you were created in," or "You now have access to the private members of **this** class." This is useful when you want to reuse a closure in a different context.

#### Purposes

- To reuse a closure with a different `$this` object.
- To grant a closure access to `private` or `protected` members of another class.
- To implement advanced patterns like proxy objects or testing utilities.

#### Syntax Rules and Structure

**General Syntax**
```php
// Using bindTo()
$newClosure = $closure->bindTo($newThis, $newScope);

// Using bind() (static)
$newClosure = Closure::bind($closure, $newThis, $newScope);
```

**Component Breakdown**

| Parameter | Description |
|-----------|-------------|
| `$closure` | The original closure to duplicate (for `bind()`). |
| `$newThis` | The object to bind as `$this`, or `null` to unbind. |
| `$newScope` | The class scope (`object`, class name string, or `'static'` to keep current). |

**Syntax Rules**

- `bindTo()` is called on the closure instance; `bind()` is called statically and takes the closure as the first argument.
- The original closure is not modified; a new closure is returned.
- If `$newThis` is `null` and `$newScope` is `null`, the closure becomes unbound.

**Constraints and Limitations**

- **Static closures**: A closure declared `static` cannot be bound to an object (`$newThis` must be `null`). Attempting to do so triggers a warning (and will be an error in PHP 9).
- **Internal classes**: The `$newScope` cannot be an internal class.
- **Access to private members**: To access `private` members of a class, `$newScope` must be set to that class (or an object of that class).

#### Annotated Code Examples

**Example 1: Rebinding `$this` with `bindTo()`**

```php
<?php
class A {
    private $val;
    public function __construct($val) {
        $this->val = $val;
    }
    public function getClosure() {
        // Closure automatically bound to $this (the A instance)
        return function () {
            return $this->val;
        };
    }
}

$ob1 = new A(1);
$ob2 = new A(2);

$cl = $ob1->getClosure();
echo $cl() . "\n";            // Outputs: 1

$cl = $cl->bindTo($ob2);      // Rebind to $ob2
echo $cl() . "\n";            // Outputs: 2
?>
```

**Expected Output:**
```
1
2
```

**Why This Output:**  
The closure returned by `getClosure()` is bound to `$ob1`, so `$this->val` is `1`. `bindTo($ob2)` creates a new closure bound to `$ob2`, so `$this->val` becomes `2`.

---

**Example 2: `Closure::bind()` (Static Version)**

```php
<?php
class Counter {
    private $count = 0;
}

$closure = function () {
    return ++$this->count;
};

$counter = new Counter();
$bound = Closure::bind($closure, $counter, Counter::class);

echo $bound() . "\n"; // Outputs: 1
echo $bound() . "\n"; // Outputs: 2
?>
```

**Expected Output:**
```
1
2
```

**Why This Output:**  
`Closure::bind` duplicates the closure and binds `$this` to `$counter` with the scope of `Counter`. This allows the closure to access the private `$count` property.

#### Real-World Cases

- **Testing**: Binding a closure to a mock object to access private methods.
- **Proxy Patterns**: Rebinding a closure to a different object to change its behavior dynamically.
- **Framework Internals**: Some frameworks bind closures to container instances to give them access to protected services.

---

### 6. Closure Best Practices

#### Definitions

**Core Definition**
Closure best practices are guidelines for writing closures that are memory-safe, readable, and maintainable.

**Technical Definition**
Best practices include: declaring closures `static` when `$this` is not used; preferring arrow functions for single-expression callbacks; explicitly capturing only the variables that are needed; avoiding by-reference capture unless mutation is required; and using `Closure::fromCallable()` or first-class callable syntax instead of wrapping existing functions.

**Beginner-Friendly Explanation**
Best practices are like good hygiene for closures: don't let them hold onto things they don't need (use `static`), keep them short and focused (use arrow functions for simple cases), and only bring in the variables you actually use.

#### Purposes

- To prevent memory leaks caused by unnecessary `$this` binding.
- To improve code readability and reduce visual clutter.
- To make closures easier to test and reason about.

#### Guidelines

| Practice | Description |
|----------|-------------|
| **Use `static` closures when `$this` is not needed** | Prevents implicit `$this` binding, which can keep objects alive longer than necessary and cause memory leaks. |
| **Prefer arrow functions for simple callbacks** | Arrow functions are more concise and automatically capture by value, reducing boilerplate. |
| **Avoid by-reference capture unless necessary** | `use (&$var)` creates shared mutable state; prefer by-value capture or explicit parameters. |
| **Capture only what you need** | Importing unused variables into a closure wastes memory and obscures dependencies. |
| **Use first-class callable syntax (PHP 8.1+)** | Instead of `function ($x) { return strlen($x); }`, use `strlen(...)`. |
| **Unset references when done** | If a closure holds a reference to a large object, `unset()` the closure when it is no longer needed to allow garbage collection. |

**Constraints and Limitations**

- **Static closures cannot use `$this`**: If your closure needs `$this`, do not declare it `static`.
- **Arrow functions cannot be `void`**: If the callback should return nothing, use a traditional anonymous function instead.

#### Annotated Code Examples

**Example 1: Static Closure to Avoid Memory Leak**

```php
<?php
class Service {
    private $data = 'large data';

    public function getCallback(): Closure {
        // BAD: Implicitly binds $this, keeps Service alive
        // return function () { return 'ok'; };

        // GOOD: Static closure does not bind $this
        return static function () {
            return 'ok';
        };
    }
}

$service = new Service();
$callback = $service->getCallback();
unset($service);   // Service object can be garbage-collected
echo $callback() . "\n"; // Outputs: ok
?>
```

**Expected Output:**
```
ok
```

**Why This Output:**  
The `static` keyword prevents the closure from binding `$this`. After `unset($service)`, the `Service` object can be destroyed because the closure does not hold a reference to it.

---

**Example 2: Arrow Function for Simple Callback**

```php
<?php
$numbers = [1, 2, 3];

// Long form
$doubledLong = array_map(function ($n) {
    return $n * 2;
}, $numbers);

// Arrow function (better)
$doubledShort = array_map(fn($n) => $n * 2, $numbers);

print_r($doubledShort);
// Outputs: Array ( [0] => 2 [1] => 4 [2] => 6 )
?>
```

**Expected Output:**
```
Array
(
    [0] => 2
    [1] => 4
    [2] => 6
)
```

**Why This Output:**  
The arrow function `fn($n) => $n * 2` is more concise and produces the same result as the longer anonymous function.

---

**Example 3: First-Class Callable Syntax (PHP 8.1+)**

```php
<?php
$strlen = strlen(...);   // Creates a Closure from the named function

echo $strlen("hello") . "\n"; // Outputs: 5
?>
```

**Expected Output:**
```
5
```

**Why This Output:**  
The `strlen(...)` syntax creates a closure that wraps the `strlen` function, avoiding the need for `function ($s) { return strlen($s); }`.

#### Real-World Cases

- **WordPress Core**: Commits explicitly add `static` to closures that do not use `$this` to prevent memory leaks.
- **Symfony Components**: Use arrow functions extensively for short callbacks in collection pipelines.
- **Laravel Collections**: Encourage arrow functions for `map`, `filter`, and `reduce` operations.

---

## References

**PHP Manual – Anonymous functions** - https://static.php.net/manual/en/functions.anonymous.php

**PHP Manual – Arrow Functions** - https://static.php.net/manual/en/functions.arrow.php

**PHP Manual – Closure class** - https://static.php.net/manual/en/class.closure.php

**PHP Manual – Closure::bindTo** - https://static.php.net/closure.bindto

**PHP Manual – Closure::bind** - https://static.php.net/closure.bind

**PHP RFC: Arrow Functions** - https://wiki.php.net/rfc/arrow_functions_v2

**PHP RFC: Static return type (context for static closures)** - https://wiki.php.net/rfc/static_return_type

**PHP Manual – First-class callable syntax** - https://static.php.net/manual/en/functions.first_class_callable_syntax.php

**PHP: Why `static` on closures? (Memory leak prevention)** - https://developer.aliyun.com/article/1620289 (Chinese; technical explanation of implicit `$this` binding)

**PHP Closure::bindTo user-contributed notes (scope and binding)** - https://static.php.net/closure.bindto