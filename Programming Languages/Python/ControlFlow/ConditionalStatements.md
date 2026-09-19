# Python Conditional Statements: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Python conditional statements are control-flow constructs that allow a program to execute different blocks of code depending on whether specified conditions evaluate to true or false.

**Technical Definition**
Conditional statements in Python are compound statements that evaluate one or more expressions in sequence until a true condition is found, then execute the associated suite (block of indented statements). The `if` statement implements the primary conditional construct, optionally followed by zero or more `elif` clauses and an optional `else` clause. Python evaluates conditions using its truthiness model, where any object can be tested for truth value.

**Beginner-Friendly Explanation**
Conditional statements let your program make decisions. Think of them like a choose-your-own-adventure book: "If it's raining, take an umbrella; otherwise, wear sunglasses." In Python, you write similar instructions using `if`, `elif`, and `else`, and the program checks each condition in order until it finds one that is true, then runs the matching block of code.

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Indentation-based Blocks** | Python uses indentation (typically 4 spaces) to define which statements belong to each condition |
| **Sequential Evaluation** | Conditions are checked in order; the first true condition wins |
| **Truthiness Testing** | Any object can be used as a condition; Python implicitly evaluates its truth value |
| **Short-Circuit Evaluation** | `and` and `or` stop evaluating as soon as the result is determined |
| **No Switch/Case** | Python uses `if-elif-else` chains instead of `switch`/`case` statements found in other languages |
| **Expression-Based Ternary** | A conditional expression (ternary operator) provides a one-line value selection |

### Prerequisites

- **Basic Python syntax**: variables, expressions, and statements
- **Comparison operators**: `==`, `!=`, `<`, `>`, `<=`, `>=`
- **Boolean concepts**: understanding of `True` and `False`
- **Indentation awareness**: Python uses whitespace to delimit code blocks

### Related Programming Areas

- **Loops**: `while` and `for` loops also use conditions for iteration
- **Boolean Algebra**: logical operators (`and`, `or`, `not`) and truth tables
- **Exception Handling**: `try`/`except` provides an alternative branching mechanism
- **Pattern Matching**: Python 3.10+ `match` statement for structural pattern matching
- **Comprehensions**: conditional expressions inside list/dict comprehensions

### Core Concepts / Features

1. **`if` Statement** (The fundamental conditional)
2. **`elif` Clause** (Chaining multiple conditions)
3. **`else` Clause** (The fallback branch)
4. **Nested Conditions** (Conditions inside conditions)
5. **Compound Conditions** (Combining conditions with logical operators)
6. **Conditional Expressions / Ternary Operator** (One-line value selection)
7. **Truthiness and Falsiness** (Implicit boolean evaluation)
8. **Short-Circuit Evaluation** (`and` / `or` early termination)

---

## Core Concept 1: The `if` Statement

### Definitions

**Core Definition**
The `if` statement is Python's primary conditional construct that executes a block of code only when a specified condition evaluates to true.

**Technical Definition**
The `if` statement evaluates an expression and, if that expression's truth value is `True`, executes the associated suite (a block of indented statements). Its grammar is `if_stmt: "if" assignment_expression ":" suite ("elif" assignment_expression ":" suite)* ["else" ":" suite]`. The expression is evaluated using Python's truthiness protocol, which is equivalent to calling `bool()` on the object.

**Beginner-Friendly Explanation**
The `if` statement is the most basic decision-maker in Python. You give it a condition (like "is the temperature above 30?"), and if that condition is true, Python runs the indented code underneath. If not, Python skips that block entirely.

### Purposes

- **To execute code conditionally** based on whether a specific condition holds
- **To create branching logic** in programs so different inputs produce different behaviours
- **To validate data** before performing operations (e.g., checking if a number is positive before taking its square root)
- **To control program flow** by directing execution down different paths
- **To implement business rules** that depend on runtime values

### Syntax Rules and Structure

**Complete General Syntax**

```
if condition:
    # suite (block of statements)
    statement1
    statement2
```

**Breakdown:**
- `if`: The keyword that begins the conditional statement.
- `condition`: Any expression that Python can evaluate for truthiness (comparison, variable, function call, etc.).
- `:`: A colon terminates the header and signals the start of the associated suite.
- Indented block: One or more statements that execute only when the condition is true.

**Syntax Rules**

| Rule | Description |
|---|---|
| Colon required | The header must end with `:` |
| Indentation required | The suite must be indented (PEP 8 recommends 4 spaces) |
| Non-empty suite | The block cannot be empty; use `pass` if no action is needed |
| Expression any type | The condition can be any expression, not just a boolean |
| Single-line allowed | A simple `if condition: statement` is permitted for very short bodies |

**Constraints and Limitations**

- **No assignment in condition**: Python does not allow assignment expressions (`=`) inside `if` conditions (use `:=` walrus operator in Python 3.8+ if needed).
- **Indentation consistency**: Mixing tabs and spaces causes `TabError`.
- **Empty body error**: An `if` with no indented body raises `IndentationError`.
- **Truthiness required**: The condition must be evaluable; objects without `__bool__` or `__len__` are always truthy.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic `if` Statement**

```python
# Step 1: Define a variable representing temperature in Celsius
temperature = 35

# Step 2: Check if the temperature exceeds 30 degrees
if temperature > 30:
    # This block runs ONLY if the condition is True
    print("It's a hot day!")
    print("Stay hydrated.")

# Step 3: This statement is outside the if block (not indented)
print("Weather check complete.")
# This line ALWAYS runs, regardless of the condition
```

**Expected Output:**
```
It's a hot day!
Stay hydrated.
Weather check complete.
```

**Breakdown:** The condition `temperature > 30` evaluates to `True` (35 > 30). Therefore, the two indented statements execute. The final `print` is outside the `if` block, so it always runs.

**Example 2: `if` with a Falsy Condition**

```python
# Step 1: Define an empty list
items = []

# Step 2: Check if the list has elements
if items:
    # This block does NOT run because an empty list is falsy
    print(f"You have {len(items)} items.")

# Step 3: This always runs
print("Inventory check done.")
```

**Expected Output:**
```
Inventory check done.
```

**Breakdown:** The empty list `[]` is falsy in Python. The condition `if items:` implicitly calls `bool([])`, which returns `False`, so the indented block is skipped.

**Example 3: `if` with Multiple Statements and a `pass`**

```python
# Step 1: Define a score
score = 85

# Step 2: Check if the score is a passing grade
if score >= 70:
    # The pass statement is a no-op placeholder
    pass  # In a real program, you might log the result or update a database

# Step 3: Always runs
print("Grade processing finished.")
```

**Expected Output:**
```
Grade processing finished.
```

**Breakdown:** The `pass` statement does nothing but satisfies Python's requirement for a non-empty suite. Without it, the empty indented block would raise `IndentationError`.

### Real-World Cases with Explanation

**Case 1: Input Validation**

```python
user_input = input("Enter your age: ")
age = int(user_input)

if age < 0:
    print("Age cannot be negative. Please try again.")
    # In a real application, you might re-prompt the user

print(f"Your age is {age}.")
```

**Why it matters:** Conditional statements are essential for validating user input before processing it. This prevents nonsensical data (like a negative age) from entering the system.

**Case 2: Feature Flags**

```python
FEATURE_ENABLED = True

if FEATURE_ENABLED:
    # New feature code path
    result = new_algorithm(data)
else:
    # Legacy code path
    result = old_algorithm(data)
```

**Why it matters:** Conditionals allow developers to toggle features on and off without changing the code structure, supporting gradual rollouts and A/B testing.

---

## Core Concept 2: The `elif` Clause

### Definitions

**Core Definition**
The `elif` clause (short for "else if") allows chaining multiple mutually exclusive conditions in a single conditional statement.

**Technical Definition**
`elif` introduces an additional condition that is evaluated only if all preceding conditions in the chain evaluated to false. Multiple `elif` clauses may follow an `if` statement, and they are evaluated sequentially from top to bottom until one is found true or the optional `else` clause is reached.

**Beginner-Friendly Explanation**
When you have more than two possible outcomes, `elif` lets you check several conditions in order. Python tests each one from top to bottom. As soon as it finds a true condition, it runs that block and skips the rest. If none are true, the `else` block (if present) runs.

### Purposes

- **To handle multiple mutually exclusive conditions** without deeply nested `if` statements
- **To implement classification logic** (e.g., grade assignment based on score ranges)
- **To replace `switch`/`case` statements** found in other languages
- **To create ordered decision trees** where conditions are checked sequentially
- **To improve readability** by avoiding excessive indentation

### Syntax Rules and Structure

**Complete General Syntax**

```
if condition1:
    # suite for condition1
elif condition2:
    # suite for condition2
elif condition3:
    # suite for condition3
else:
    # suite if all conditions are false
```

**Breakdown:**
- `elif`: Keyword that introduces an alternative condition.
- `condition2`, `condition3`: Expressions evaluated only if previous conditions were false.
- `:`: Each header ends with a colon.
- Indented suites: Each clause has its own indented block.

**Syntax Rules**

| Rule | Description |
|---|---|
| Position | `elif` must follow an `if` or another `elif` |
| Multiple allowed | Zero or more `elif` clauses may appear |
| Sequential evaluation | Conditions checked in order; first true wins |
| Colon and indentation | Same rules as `if` |
| No `else` required | `else` is optional even with `elif` |

**Constraints and Limitations**

- **Order matters**: Changing the order of conditions can change behaviour (e.g., checking `score >= 90` before `score >= 80`).
- **Mutually exclusive**: Only one block in an `if-elif-else` chain ever executes.
- **Exhaustive checking**: If no condition matches and no `else` is present, nothing happens.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Grade Assignment**

```python
# Step 1: Define a numeric score
score = 87

# Step 2: Classify the score using an if-elif chain
if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"      # This branch runs because 87 >= 80
elif score >= 70:
    grade = "C"
elif score >= 60:
    grade = "D"
else:
    grade = "F"

# Step 3: Print the result
print(f"Score: {score}, Grade: {grade}")
```

**Expected Output:**
```
Score: 87, Grade: B
```

**Breakdown:** Python checks `score >= 90` first (false), then `score >= 80` (true). The `grade = "B"` assignment executes, and all remaining `elif`/`else` clauses are skipped.

**Example 2: Traffic Light Simulation**

```python
# Step 1: Define the current light colour
light = "yellow"

# Step 2: Determine the action based on the light
if light == "green":
    action = "Go"
elif light == "yellow":
    action = "Caution"    # This branch runs
elif light == "red":
    action = "Stop"
else:
    action = "Unknown light"

# Step 3: Display the action
print(f"The light is {light}. Action: {action}")
```

**Expected Output:**
```
The light is yellow. Action: Caution
```

**Breakdown:** The first condition (`light == "green"`) is false, but the second (`light == "yellow"`) is true, so `action` is set to `"Caution"`.

**Example 3: `if-elif` Without `else`**

```python
# Step 1: Define a value
value = 15

# Step 2: Check conditions without a fallback
if value > 100:
    print("Very large")
elif value > 10:
    print("Medium")    # This branch runs
elif value > 0:
    print("Small")

# Step 3: This always runs
print("Check complete.")
```

**Expected Output:**
```
Medium
Check complete.
```

**Breakdown:** `value > 100` is false, but `value > 10` is true. The third `elif` is skipped. No `else` is needed because the program simply continues after the chain.

### Real-World Cases with Explanation

**Case 1: HTTP Status Code Handling**

```python
def handle_response(status_code):
    if status_code == 200:
        return "Success"
    elif status_code == 301:
        return "Moved Permanently"
    elif status_code == 404:
        return "Not Found"
    elif status_code == 500:
        return "Server Error"
    else:
        return "Unknown Status"
```

**Why it matters:** Web servers and API clients frequently need to branch based on status codes. The `elif` chain provides a clean, readable mapping.

**Case 2: Tax Bracket Calculation**

```python
income = 75000

if income <= 10000:
    tax_rate = 0.0
elif income <= 40000:
    tax_rate = 0.12
elif income <= 85000:
    tax_rate = 0.22    # This branch runs
else:
    tax_rate = 0.24

print(f"Tax rate: {tax_rate * 100}%")
# Output: Tax rate: 22.0%
```

**Why it matters:** Financial applications use `elif` chains to implement progressive tax brackets, where the applicable rate depends on which range the income falls into.

---

## Core Concept 3: The `else` Clause

### Definitions

**Core Definition**
The `else` clause provides a default block of code that executes when none of the preceding `if` or `elif` conditions are true.

**Technical Definition**
In a conditional statement, the `else` clause is optional and, if present, its suite executes only when all preceding conditions (the `if` condition and any `elif` conditions) evaluate to false. It serves as the catch-all or fallback branch.

**Beginner-Friendly Explanation**
The `else` clause is the "otherwise" part of a decision. If none of the conditions you checked were true, the code under `else` runs. It's like saying, "If it's not any of these things, then do this.".

### Purposes

- **To provide a default action** when no condition matches
- **To handle unexpected or unknown cases** explicitly
- **To ensure exhaustive branching** so every possible input has a defined outcome
- **To improve code clarity** by making the fallback path explicit
- **To implement binary decisions** (if X, do A; otherwise, do B)

### Syntax Rules and Structure

**Complete General Syntax**

```
if condition:
    # suite for true condition
else:
    # suite for false condition
```

**Breakdown:**
- `else`: Keyword that introduces the fallback block.
- `:`: Colon terminates the header.
- Indented suite: Statements that execute when the `if` condition is false.

**Syntax Rules**

| Rule | Description |
|---|---|
| Position | `else` must be the last clause in an `if-elif-else` chain |
| At most one | Only one `else` clause is allowed |
| No condition | `else` does not take a condition |
| Colon and indentation | Same rules as `if` and `elif` |
| Optional | The `else` clause may be omitted entirely |

**Constraints and Limitations**

- **Must follow `if` or `elif`**: `else` cannot appear alone.
- **No condition**: You cannot write `else condition:`.
- **Only one**: You cannot have two `else` clauses in the same conditional chain.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic `if-else`**

```python
# Step 1: Define a variable
is_raining = False

# Step 2: Choose an action based on the condition
if is_raining:
    print("Take an umbrella.")
else:
    print("Wear sunglasses.")  # This branch runs

# Step 3: Always runs
print("Have a nice day!")
```

**Expected Output:**
```
Wear sunglasses.
Have a nice day!
```

**Breakdown:** `is_raining` is `False`, so the `if` condition is false. Python executes the `else` block instead.

**Example 2: `if-elif-else` Chain**

```python
# Step 1: Define a number
number = 0

# Step 2: Classify the number
if number > 0:
    print("Positive")
elif number < 0:
    print("Negative")
else:
    print("Zero")  # This branch runs

# Step 3: Always runs
print("Classification complete.")
```

**Expected Output:**
```
Zero
Classification complete.
```

**Breakdown:** Neither `number > 0` nor `number < 0` is true (number is 0), so the `else` block executes.

**Example 3: `else` with a Function**

```python
def get_discount(is_member, purchase_amount):
    """Return discount percentage based on membership and purchase amount."""
    if is_member and purchase_amount > 100:
        return 0.20
    elif is_member:
        return 0.10
    elif purchase_amount > 200:
        return 0.05
    else:
        return 0.0  # Default: no discount

# Test cases
print(get_discount(True, 150))   # Output: 0.2
print(get_discount(True, 50))    # Output: 0.1
print(get_discount(False, 250))  # Output: 0.05
print(get_discount(False, 50))   # Output: 0.0
```

**Expected Output:**
```
0.2
0.1
0.05
0.0
```

**Breakdown:** Each call exercises a different branch. The final call (non-member, low purchase) matches none of the `if`/`elif` conditions, so the `else` returns `0.0`.

### Real-World Cases with Explanation

**Case 1: Login Validation**

```python
def authenticate(username, password):
    if not username:
        return "Username required"
    elif not password:
        return "Password required"
    elif check_credentials(username, password):
        return "Login successful"
    else:
        return "Invalid credentials"  # Fallback for wrong username/password
```

**Why it matters:** The `else` clause handles the case where the user provided input but the credentials don't match, ensuring no scenario falls through without feedback.

**Case 2: Configuration Defaults**

```python
def get_setting(config, key):
    if key in config:
        return config[key]
    elif key in DEFAULT_CONFIG:
        return DEFAULT_CONFIG[key]
    else:
        return None  # Explicitly return nothing for unknown keys
```

**Why it matters:** The `else` makes it clear that unknown keys yield `None`, preventing implicit `None` returns and improving code readability.

---

## Core Concept 4: Nested Conditions

### Definitions

**Core Definition**
Nested conditions occur when an `if` statement is placed inside the suite of another `if`, `elif`, or `else` block.

**Technical Definition**
A nested conditional is a compound statement whose suite contains another `if` statement (or full `if-elif-else` chain). Python determines the nesting level through indentation: the inner `if` is indented relative to the outer `if`'s header.

**Beginner-Friendly Explanation**
Sometimes one decision depends on the outcome of another. For example: "If you have a ticket, then check if you're over 18; if so, you can enter." The second check is nested inside the first. Python uses indentation to show which statements belong to which condition.

### Purposes

- **To express dependent conditions** where a second check only makes sense after a first passes
- **To implement multi-level decision logic** (e.g., category → subcategory)
- **To handle complex business rules** that require sequential validation
- **To structure hierarchical decisions** clearly
- **To avoid unnecessarily complex compound conditions** when nesting is more readable

### Syntax Rules and Structure

**Complete General Syntax**

```
if outer_condition:
    # outer suite
    if inner_condition:
        # inner suite (nested)
    else:
        # inner else (nested)
else:
    # outer else
```

**Breakdown:**
- The inner `if` is indented one level deeper than the outer `if`.
- Each nesting level adds 4 spaces (PEP 8 recommendation).
- The inner block executes only if both the outer and inner conditions are true.

**Syntax Rules**

| Rule | Description |
|---|---|
| Indentation | Inner `if` must be indented under the outer block |
| Consistent indent | Each level uses the same indentation unit (4 spaces per PEP 8) |
| Unlimited depth | Nesting can theoretically go arbitrarily deep (practically, avoid >3 levels) |
| Alternative | Compound conditions (`and`/`or`) can often replace simple nesting |

**Constraints and Limitations**

- **Readability degrades**: Deeply nested code becomes hard to follow (the "arrow anti-pattern").
- **Indentation errors**: Incorrect indentation is a common source of `IndentationError`.
- **Maintenance burden**: Adding or removing conditions requires re-indenting large blocks.
- **Prefer `and` when possible**: `if a and b:` is usually clearer than nested `if a: if b:`.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Ticket and Age Check**

```python
# Step 1: Define variables
has_ticket = True
age = 25

# Step 2: Outer check — does the person have a ticket?
if has_ticket:
    print("Ticket verified.")
    # Step 3: Inner check — is the person old enough?
    if age >= 18:
        print("Entry granted. Enjoy the show!")
    else:
        print("Sorry, you must be 18 or older.")
else:
    print("Please purchase a ticket first.")

# Step 4: Always runs
print("Next in line, please.")
```

**Expected Output:**
```
Ticket verified.
Entry granted. Enjoy the show!
Next in line, please.
```

**Breakdown:** The outer condition `has_ticket` is true, so the outer suite runs. Inside that suite, the inner condition `age >= 18` is also true, so the innermost block executes.

**Example 2: Grading with Multiple Nesting Levels**

```python
# Step 1: Define a score
score = 85

# Step 2: Nested classification
if score >= 60:
    # Passed the threshold
    print("Result: Pass")
    if score >= 90:
        print("Distinction")
    else:
        if score >= 80:
            print("Merit")      # This runs
        else:
            print("Standard Pass")
else:
    # Did not meet the threshold
    print("Result: Fail")
    if score >= 40:
        print("Near miss — consider a retake.")
    else:
        print("Significant improvement needed.")

print("Grading complete.")
```

**Expected Output:**
```
Result: Pass
Merit
Grading complete.
```

**Breakdown:** The outer `score >= 60` is true, entering the pass branch. Inside, `score >= 90` is false, so the nested `else` runs, which contains another `if` checking `score >= 80` — this is true, printing "Merit".

**Example 3: Nested Conditions vs. Compound Conditions**

```python
# Nested approach
x = 10
y = 20

if x > 5:
    if y > 15:
        print("Both conditions met (nested).")  # This runs

# Equivalent compound approach (preferred for readability)
if x > 5 and y > 15:
    print("Both conditions met (compound).")    # This runs
```

**Expected Output:**
```
Both conditions met (nested).
Both conditions met (compound).
```

**Breakdown:** Both versions produce the same result, but the compound version is flatter and easier to read when the conditions are simple.

### Real-World Cases with Explanation

**Case 1: Loan Approval**

```python
def evaluate_loan(credit_score, income, existing_debt):
    if credit_score >= 650:
        if income >= 50000:
            if existing_debt < income * 0.4:
                return "Approved"
            else:
                return "Denied: debt-to-income ratio too high"
        else:
            return "Denied: income too low"
    else:
        return "Denied: credit score too low"
```

**Why it matters:** Loan approval involves multiple dependent criteria. Nested conditions clearly express the sequential evaluation of credit score, income, and debt ratio.

**Case 2: Game State Validation**

```python
def can_cast_spell(mana, spell_cost, is_silenced, is_dead):
    if not is_dead:
        if not is_silenced:
            if mana >= spell_cost:
                return "Spell cast!"
            else:
                return "Not enough mana"
        else:
            return "You are silenced"
    else:
        return "You are dead"
```

**Why it matters:** Game logic often requires checking several prerequisites in a specific order. Nesting makes the dependency chain explicit.

---

## Core Concept 5: Compound Conditions

### Definitions

**Core Definition**
Compound conditions combine multiple simple conditions into a single expression using the logical operators `and`, `or`, and `not`.

**Technical Definition**
Python's logical operators `and`, `or`, and `not` operate on expressions and return one of the operands (not necessarily a boolean). `and` returns the first falsy operand or the last truthy operand; `or` returns the first truthy operand or the last falsy operand; `not` returns a boolean by inverting truthiness. Operator precedence is `not` > `and` > `or`.

**Beginner-Friendly Explanation**
Compound conditions let you ask several questions at once. "If the temperature is above 20 AND it's not raining, we'll go to the park." Python's `and` requires both sides to be true; `or` requires at least one side; `not` flips a value. You can combine them freely, but use parentheses to keep things readable.

### Purposes

- **To combine multiple criteria** into a single condition
- **To express complex logical requirements** concisely
- **To avoid deep nesting** by flattening conditions
- **To implement business rules** with multiple constraints (e.g., "age between 18 and 65")
- **To control program flow** based on multiple simultaneous factors

### Syntax Rules and Structure

**Complete General Syntax**

```
if condition1 and condition2:
    # both must be true
if condition1 or condition2:
    # at least one must be true
if not condition:
    # condition must be false
if (cond1 or cond2) and not cond3:
    # compound expression with parentheses
```

**Breakdown:**
- `and`: True only when both operands are truthy.
- `or`: True when at least one operand is truthy.
- `not`: Inverts the truth value of its operand.
- Parentheses: Group sub-expressions for clarity and to override precedence.

**Syntax Rules**

| Rule | Description |
|---|---|
| Precedence | `not` > `and` > `or` |
| Associativity | Left-to-right for `and` and `or` |
| Short-circuit | `and` stops at first falsy; `or` stops at first truthy |
| Return value | `and`/`or` return the actual operand value, not necessarily a boolean |
| Parentheses | Recommended for complex expressions to improve readability |

**Constraints and Limitations**

- **Non-boolean operands**: `and`/`or` can return non-boolean values, which may surprise beginners.
- **Common mistake**: `if score == 90 or 100:` is always true because `100` is truthy; use `score == 90 or score == 100`.
- **Precedence confusion**: `a or b and c` evaluates as `a or (b and c)` because `and` has higher precedence.
- **Readability**: Overly long compound conditions should be broken into multiple lines or nested.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: `and` Operator**

```python
# Step 1: Define variables
age = 25
has_license = True

# Step 2: Both conditions must be true to rent a car
if age >= 21 and has_license:
    print("You can rent a car.")  # This runs
else:
    print("You cannot rent a car.")

# Step 3: Test with a failing condition
age = 18
if age >= 21 and has_license:
    print("You can rent a car.")
else:
    print("You cannot rent a car.")  # This runs
```

**Expected Output:**
```
You can rent a car.
You cannot rent a car.
```

**Breakdown:** In the first check, both `age >= 21` (25) and `has_license` are true, so the `if` block runs. In the second, `age >= 21` is false (18), so the `and` expression is false.

**Example 2: `or` Operator**

```python
# Step 1: Define variables
is_weekend = False
is_holiday = True

# Step 2: Office is closed if it's a weekend OR a holiday
if is_weekend or is_holiday:
    print("The office is closed.")  # This runs
else:
    print("The office is open.")
```

**Expected Output:**
```
The office is closed.
```

**Breakdown:** `is_weekend` is false, but `is_holiday` is true, so the `or` expression is true. Python evaluates the left operand first; since it's false, it evaluates the right operand.

**Example 3: `not` Operator**

```python
# Step 1: Define a variable
is_logged_in = False

# Step 2: Prompt login if not logged in
if not is_logged_in:
    print("Please log in to continue.")  # This runs
else:
    print("Welcome back!")
```

**Expected Output:**
```
Please log in to continue.
```

**Breakdown:** `not is_logged_in` inverts `False` to `True`, so the `if` block executes.

**Example 4: Combining Operators with Parentheses**

```python
# Step 1: Define variables
score = 85
has_extra_credit = True
is_past_due = False

# Step 2: Complex condition: (high score OR extra credit) AND not late
if (score >= 90 or has_extra_credit) and not is_past_due:
    print("Requirement met.")  # This runs
else:
    print("Requirement not met.")
```

**Expected Output:**
```
Requirement met.
```

**Breakdown:** The parenthesised expression `(score >= 90 or has_extra_credit)` is true because `has_extra_credit` is true. `not is_past_due` is also true. Therefore, the entire `and` expression is true.

### Real-World Cases with Explanation

**Case 1: Form Validation**

```python
username = "alice"
password = "secure123"
email = "alice@example.com"

if len(username) >= 3 and len(password) >= 8 and "@" in email:
    print("Registration successful.")
else:
    print("Please check your input.")
```

**Why it matters:** Compound conditions efficiently validate multiple form fields in a single expression, reducing code complexity compared to nested `if` statements.

**Case 2: Access Control**

```python
user_role = "admin"
is_active = True
has_permission = False

if (user_role == "admin" or user_role == "moderator") and is_active and not has_permission:
    print("Access denied: permission required.")
elif user_role == "admin" or user_role == "moderator":
    print("Access granted.")
else:
    print("Access denied: insufficient role.")
```

**Why it matters:** Access control systems combine role checks, account status, and permission flags using compound conditions to enforce security policies.

---

## Core Concept 6: Conditional Expressions (Ternary Operator)

### Definitions

**Core Definition**
A conditional expression (often called the ternary operator) is a one-line expression that returns one of two values depending on a condition.

**Technical Definition**
The conditional expression `x if C else y` first evaluates the condition `C`; if `C` is true, `x` is evaluated and returned; otherwise, `y` is evaluated and returned. It has the lowest priority of all Python operations and is defined by the grammar `conditional_expression: or_test ["if" or_test "else" expression]`.

**Beginner-Friendly Explanation**
The ternary operator is a compact way to write a simple `if-else` that produces a value. Instead of writing four lines, you write one: `result = "adult" if age >= 18 else "minor"`. It reads almost like English and is useful for simple value assignments.

### Purposes

- **To assign a value conditionally** in a single expression
- **To reduce code verbosity** for simple `if-else` value selection
- **To use inside expressions** (function arguments, list comprehensions, return statements)
- **To improve readability** when the logic is simple
- **To avoid multi-line `if-else` blocks** for trivial value choices

### Syntax Rules and Structure

**Complete General Syntax**

```
value_if_true if condition else value_if_false
```

**Breakdown:**
- `value_if_true`: Expression evaluated when the condition is true.
- `condition`: Any expression testable for truthiness.
- `value_if_false`: Expression evaluated when the condition is false.

**Syntax Rules**

| Rule | Description |
|---|---|
| Order | `if` comes after the true-value expression, `else` after the condition |
| Both branches required | You cannot omit the `else` part |
| Expression context | Can be used anywhere an expression is valid |
| Lowest precedence | Parentheses often needed for clarity when mixed with other operators |
| Lazy evaluation | Only the selected branch is evaluated |

**Constraints and Limitations**

- **Cannot contain statements**: Only expressions are allowed, not assignments or `print()`.
- **Readability**: Nested ternaries become hard to read; use `if-elif-else` instead.
- **No `elif`**: The ternary operator supports only two branches.
- **Version note**: Introduced in Python 2.5 (PEP 308); available in all modern versions.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Ternary**

```python
# Step 1: Define age
age = 20

# Step 2: Determine status using a conditional expression
status = "adult" if age >= 18 else "minor"
# Reads as: "adult if age >= 18, else minor"

# Step 3: Print result
print(f"Status: {status}")
```

**Expected Output:**
```
Status: adult
```

**Breakdown:** The condition `age >= 18` is true, so the expression returns `"adult"`. If age were 15, it would return `"minor"`.

**Example 2: Ternary Inside a Function**

```python
def get_access_level(is_admin):
    """Return access level based on admin status."""
    return "full" if is_admin else "limited"

# Test cases
print(get_access_level(True))   # full
print(get_access_level(False))  # limited
```

**Expected Output:**
```
full
limited
```

**Breakdown:** The ternary expression is evaluated each time the function is called, returning different values based on the `is_admin` argument.

**Example 3: Ternary in a List Comprehension**

```python
# Step 1: Define a list of numbers
numbers = [1, 2, 3, 4, 5, 6]

# Step 2: Use ternary to label each number as even or odd
labels = ["even" if n % 2 == 0 else "odd" for n in numbers]

# Step 3: Print the result
print(labels)
```

**Expected Output:**
```
['odd', 'even', 'odd', 'even', 'odd', 'even']
```

**Breakdown:** For each `n`, the ternary checks `n % 2 == 0`. If true, `"even"` is selected; otherwise, `"odd"`. The result is a list of labels.

**Example 4: Nested Ternary (Use with Caution)**

```python
# Step 1: Define a score
score = 85

# Step 2: Nested ternary for multiple grades
grade = "A" if score >= 90 else "B" if score >= 80 else "C" if score >= 70 else "F"

# Step 3: Print the grade
print(f"Grade: {grade}")
```

**Expected Output:**
```
Grade: B
```

**Breakdown:** The expression is evaluated left to right. Since `score >= 90` is false, it checks `score >= 80`, which is true, returning `"B"`. The remaining branches are not evaluated. Note: nested ternaries like this are compact but can reduce readability.

### Real-World Cases with Explanation

**Case 1: Default Configuration**

```python
def get_timeout(user_config):
    return user_config.get("timeout", 30) if user_config else 30
```

**Why it matters:** The ternary operator provides a concise way to supply a default value when a configuration is missing or empty.

**Case 2: Display Formatting**

```python
items = ["apple", "banana", "cherry"]
count = len(items)
message = f"{count} item{'s' if count != 1 else ''}"
print(message)
# Output: 3 items
```

**Why it matters:** The ternary handles pluralisation cleanly within an f-string, avoiding an explicit `if-else` block.

---

## Core Concept 7: Truthiness and Falsiness (Implicit Boolean Evaluation)

### Definitions

**Core Definition**
Truthiness and falsiness describe how Python evaluates non-boolean objects as true or false in a boolean context.

**Technical Definition**
In any boolean context (e.g., `if` condition, `while` condition, `and`/`or` operands), Python implicitly calls `bool()` on the object. Objects with a `__bool__()` method that returns `False`, or a `__len__()` method that returns `0`, are falsy; all other objects are truthy. The built-in falsy values include `None`, `False`, zero of any numeric type, and empty containers or strings.

**Beginner-Friendly Explanation**
Python doesn't require you to write `if x == True:`. Instead, any object can be used directly as a condition. Empty things (empty lists, empty strings, the number 0, `None`) are treated as false. Everything else is true. This makes code shorter and more natural.

### Purposes

- **To write concise conditional checks** without explicit boolean comparisons
- **To check for emptiness** of containers or strings elegantly (e.g., `if not my_list:`)
- **To test for `None`** using `if x is None:` or `if not x:` depending on intent
- **To enable Pythonic idioms** like `if items:` instead of `if len(items) > 0:`
- **To provide consistent behaviour** across all built-in types

### Syntax Rules and Structure

**Complete General Syntax**

```
if object:
    # executes if bool(object) is True
if not object:
    # executes if bool(object) is False
```

**Breakdown:**
- `object`: Any Python object.
- Python implicitly calls `bool(object)` to determine truth value.

**Falsy Values (Complete List)**

| Category | Examples |
|---|---|
| Constants | `None`, `False` |
| Zero of any numeric type | `0`, `0.0`, `0j`, `Decimal(0)`, `Fraction(0, 1)` |
| Empty sequences | `""`, `b""`, `[]`, `()`, `range(0)` |
| Empty collections | `{}`, `set()`, `frozenset()` |
| Custom objects | Objects whose `__bool__()` returns `False` or `__len__()` returns `0` |

**Truthy Values**

Everything not listed above is truthy, including:
- Non-zero numbers (including negative numbers)
- Non-empty strings, lists, tuples, dicts, sets
- Arbitrary objects (by default)

**Syntax Rules**

| Rule | Description |
|---|---|
| Implicit `bool()` | Python calls `bool()` automatically in boolean contexts |
| `__bool__` takes priority | If defined, `__bool__()` determines truth value |
| `__len__` fallback | If `__bool__` is not defined, `__len__()` is used |
| Default truthy | Objects with neither method are always truthy |

**Constraints and Limitations**

- **`0` vs `False`**: `0 == False` is `True`, but `0 is False` is `False`.
- **`None` is not `False`**: `None == False` is `False`.
- **Custom objects**: Misimplementing `__bool__` or `__len__` can cause subtle bugs.
- **Explicit comparison sometimes better**: `if x is not None:` is clearer than `if x:` when `None` is the only invalid value.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Checking Empty Containers**

```python
# Step 1: Define an empty and a non-empty list
empty_list = []
non_empty_list = [1, 2, 3]

# Step 2: Use truthiness directly
if empty_list:
    print("Empty list is truthy")   # Does NOT run
else:
    print("Empty list is falsy")    # Runs

if non_empty_list:
    print("Non-empty list is truthy")  # Runs
else:
    print("Non-empty list is falsy")   # Does NOT run
```

**Expected Output:**
```
Empty list is falsy
Non-empty list is truthy
```

**Breakdown:** An empty list is falsy because `bool([])` is `False`. A non-empty list is truthy because `bool([1,2,3])` is `True`.

**Example 2: Numeric Truthiness**

```python
# Step 1: Test various numbers
for value in [0, 1, -1, 0.0, 0.1, -0.1]:
    if value:
        print(f"{value} is truthy")
    else:
        print(f"{value} is falsy")
```

**Expected Output:**
```
0 is falsy
1 is truthy
-1 is truthy
0.0 is falsy
0.1 is truthy
-0.1 is truthy
```

**Breakdown:** Only numeric values equal to zero are falsy. Negative numbers are truthy because they are non-zero.

**Example 3: `None` and Custom Objects**

```python
# Step 1: Test None
x = None
if x:
    print("None is truthy")
else:
    print("None is falsy")  # Runs

# Step 2: Custom object with __bool__
class AlwaysFalse:
    def __bool__(self):
        return False

obj = AlwaysFalse()
if obj:
    print("Custom object is truthy")
else:
    print("Custom object is falsy")  # Runs
```

**Expected Output:**
```
None is falsy
Custom object is falsy
```

**Breakdown:** `None` is always falsy. The custom class defines `__bool__()` returning `False`, so its instances are also falsy.

### Real-World Cases with Explanation

**Case 1: Database Query Results**

```python
def process_results(results):
    if results:
        for row in results:
            print(row)
    else:
        print("No results found.")
```

**Why it matters:** Database libraries often return empty lists or `None` when no records match. Checking `if results:` handles both cases elegantly.

**Case 2: Configuration Loading**

```python
config = load_config()  # Returns a dict, possibly empty

if config:
    apply_settings(config)
else:
    use_defaults()
```

**Why it matters:** An empty configuration dictionary is falsy, so the program falls back to defaults without needing `if len(config) > 0:`.

---

## Core Concept 8: Short-Circuit Evaluation (`and` / `or`)

### Definitions

**Core Definition**
Short-circuit evaluation is the behaviour where `and` and `or` stop evaluating operands as soon as the final result is determined.

**Technical Definition**
In `A and B`, if `A` is falsy, `B` is not evaluated and `A` is returned; otherwise `B` is evaluated and returned. In `A or B`, if `A` is truthy, `B` is not evaluated and `A` is returned; otherwise `B` is evaluated and returned. This is guaranteed by the language specification and can be used to control side effects.

**Beginner-Friendly Explanation**
Short-circuit evaluation means Python is lazy in a good way. For `and`, if the first thing is false, the whole thing must be false, so Python doesn't bother checking the second thing. For `or`, if the first thing is true, the whole thing must be true, so it skips the second. This can prevent errors (like dividing by zero) and improve performance.

### Purposes

- **To prevent errors** by guarding against invalid operations (e.g., `if x != 0 and y / x > 1:`)
- **To improve performance** by skipping unnecessary evaluations
- **To provide default values** using `value or default`
- **To chain conditional checks** safely without nested `if` statements
- **To control side effects** by ensuring certain expressions are only evaluated when needed

### Syntax Rules and Structure

**Complete General Syntax**

```
A and B   # B is evaluated only if A is truthy
A or B    # B is evaluated only if A is falsy
```

**Breakdown:**
- `A`: First operand (evaluated always).
- `B`: Second operand (evaluated conditionally).
- The expression returns the actual operand value, not necessarily a boolean.

**Truth Table for `and` / `or`**

| `A` | `B` | `A and B` | `A or B` |
|---|---|---|---|
| Truthy | Truthy | B | A |
| Truthy | Falsy | B | A |
| Falsy | Truthy | A | B |
| Falsy | Falsy | A | B |

**Syntax Rules**

| Rule | Description |
|---|---|
| Left-to-right | Operands evaluated left to right |
| Precedence | `and` binds tighter than `or` |
| Return value | Returns an operand, not necessarily `True`/`False` |
| Side-effect control | Expressions after the short-circuit point are not evaluated |
| Parentheses | Use to control evaluation order and readability |

**Constraints and Limitations**

- **Non-boolean return**: `x = a or b` assigns `a` if truthy, otherwise `b`. This is useful but can be surprising.
- **No guarantee of boolean**: `5 and 3` returns `3`, not `True`.
- **Precedence pitfalls**: `a or b and c` is `a or (b and c)`, not `(a or b) and c`.
- **Not a replacement for `if`**: Using `and`/`or` for control flow can reduce readability.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: `and` Short-Circuit Preventing Division by Zero**

```python
# Step 1: Define variables
x = 0
y = 10

# Step 2: Safe division using short-circuit
if x != 0 and y / x > 1:
    print("Ratio is greater than 1.")
else:
    print("Cannot compute ratio (x is zero).")  # This runs
```

**Expected Output:**
```
Cannot compute ratio (x is zero).
```

**Breakdown:** `x != 0` is false (x is 0). Because it's the left operand of `and`, Python short-circuits and does **not** evaluate `y / x`, avoiding a `ZeroDivisionError`.

**Example 2: `or` for Default Values**

```python
# Step 1: Simulate a missing configuration value
user_setting = None

# Step 2: Use 'or' to provide a default
timeout = user_setting or 30
print(f"Timeout: {timeout}")  # Output: Timeout: 30

# Step 3: Test with a provided value
user_setting = 60
timeout = user_setting or 30
print(f"Timeout: {timeout}")  # Output: Timeout: 60
```

**Expected Output:**
```
Timeout: 30
Timeout: 60
```

**Breakdown:** When `user_setting` is `None` (falsy), `or` returns the right operand (`30`). When it's `60` (truthy), `or` returns the left operand (`60`). This is a common Pythonic idiom for defaults.

**Example 3: Demonstrating Short-Circuit with Side Effects**

```python
# Step 1: Define a function with a side effect
def side_effect(label):
    print(f"  Evaluating: {label}")
    return label

# Step 2: 'and' short-circuits when left is falsy
print("Test 1: False and side_effect('B')")
result = False and side_effect("B")
print(f"Result: {result}")
# No "Evaluating: B" printed — short-circuited

# Step 3: 'or' short-circuits when left is truthy
print("\nTest 2: True or side_effect('B')")
result = True or side_effect("B")
print(f"Result: {result}")
# No "Evaluating: B" printed — short-circuited
```

**Expected Output:**
```
Test 1: False and side_effect('B')
Result: False

Test 2: True or side_effect('B')
Result: True
```

**Breakdown:** In Test 1, `False and ...` short-circuits because the left operand is falsy, so `side_effect` is never called. In Test 2, `True or ...` short-circuits because the left operand is truthy. The side effect function is never invoked in either case.

**Example 4: Operator Precedence in Compound Expressions**

```python
# Step 1: Define variables
a = True
b = False
c = True

# Step 2: Evaluate without parentheses
result = a or b and c
# 'and' has higher precedence: a or (b and c)
# b and c -> False and True -> False
# a or False -> True or False -> True
print(f"a or b and c = {result}")  # True

# Step 3: Evaluate with explicit parentheses
result2 = (a or b) and c
# (True or False) -> True
# True and True -> True
print(f"(a or b) and c = {result2}")  # True

# Step 4: Case where precedence matters
a = False
b = True
c = False
result3 = a or b and c       # False or (True and False) = False or False = False
result4 = (a or b) and c     # (False or True) and False = True and False = False
print(f"a={a}, b={b}, c={c}")
print(f"a or b and c = {result3}")      # False
print(f"(a or b) and c = {result4}")    # False
```

**Expected Output:**
```
a or b and c = True
(a or b) and c = True
a=False, b=True, c=False
a or b and c = False
(a or b) and c = False
```

**Breakdown:** In the first set, both expressions happen to yield `True`. In the second set, both yield `False`. However, the evaluation paths differ. Always use parentheses for clarity when mixing `and` and `or`.

### Real-World Cases with Explanation

**Case 1: Safe Attribute Access**

```python
user = {"name": "Alice", "profile": None}

# Safely check nested attribute
if user and user["profile"] and user["profile"].get("bio"):
    bio = user["profile"]["bio"]
else:
    bio = "No bio available"
```

**Why it matters:** Short-circuit evaluation prevents `TypeError` when accessing attributes on `None`. If `user` is falsy, the rest of the chain is never evaluated.

**Case 2: Conditional Function Calls**

```python
def log(message):
    print(f"[LOG] {message}")

DEBUG = True

# Only log if DEBUG is truthy
DEBUG and log("Debugging enabled")
# Output: [LOG] Debugging enabled

DEBUG = False
DEBUG and log("This won't print")
# No output
```

**Why it matters:** The `and` idiom conditionally executes a function call only when a flag is truthy, providing a concise alternative to a full `if` block.

---

## References

- Python Software Foundation. *More Control Flow Tools — Python 3.12.0 documentation*. https://docs.python.org/3/tutorial/controlflow.html
- Python Software Foundation. *Compound Statements — Python Language Reference*. https://docs.python.org/3/reference/compound_stmts.html
- Python Software Foundation. *Expressions — Python Language Reference*. https://docs.python.org/3/reference/expressions.html
- Python Software Foundation. *Built-in Types — Truth Value Testing*. https://docs.python.org/3/library/stdtypes.html
- Python Software Foundation. *PEP 308 – Conditional Expressions*. https://peps.python.org/pep-0308/
- Real Python. *Conditional Statements in Python*. https://realpython.com/python-conditional-statements/
- Real Python. *Python's `and` and `or` Operators*. https://realpython.com/python-or-operator/
- Microsoft Learn. *Create Complex Conditions with Logical Operators*. https://learn.microsoft.com/en-us/training/modules/add-decision-making-python/4-create-complex-conditions-with-logical-operators
- Python Tutor Mailing List. *Short-Circuit Evaluation of Logical `or`*. https://mail.python.org/pipermail/tutor/2003-December/026919.html
- Python Tutor Mailing List. *Truthiness and Falsiness in Python*. https://mail.python.org/archives/list/tutor@python.org/message/PPCV6MUXTQU4GG5OWON4NGC6U64HF5S5/