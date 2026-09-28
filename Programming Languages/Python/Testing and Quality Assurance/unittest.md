# unittest: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition

`unittest` is Python's built-in unit testing framework, originally inspired by JUnit. It provides a rich set of tools for constructing and running tests, supporting test automation, sharing of setup and shutdown code, aggregation of tests into collections, and independence of tests from the reporting framework .

### Technical Definition

The `unittest` framework supports object-oriented testing through four key concepts: **test fixtures** (preparation and cleanup needed for tests), **test cases** (individual units of testing), **test suites** (collections of test cases or suites), and **test runners** (components that execute tests and report results) . Test cases are created by subclassing `unittest.TestCase`, and individual test methods are defined with names beginning with `test_` .

### Beginner-Friendly Explanation

`unittest` is a tool built into Python that helps you write tests for your code. You create a test class that inherits from `unittest.TestCase`, then write methods that start with `test_`. Each method tests one thing. You can run all your tests with a single command, and `unittest` tells you which ones passed and which ones failed.

### Key Characteristics

- **Built-in**: Ships with Python; no installation required.
- **Object-oriented**: Tests are organized into classes that inherit from `TestCase`.
- **Comprehensive assertions**: Provides specialized assertion methods for different data types.
- **Fixture support**: Method-level and class-level setup/teardown hooks.
- **Test discovery**: Automatically discovers tests in files matching `test*.py`.
- **Suite aggregation**: Combine multiple test cases and suites into a single runnable unit.
- **Extensible**: Can be extended with custom test runners, loaders, and result classes.

### Prerequisites

- Python 3.x installed.
- Basic understanding of classes, inheritance, and methods.
- A text editor or IDE.

### Related Programming Areas

- **Test-Driven Development (TDD)**: Writing tests before code.
- **Continuous Integration**: Running tests automatically on every commit.
- **Mocking**: Replacing dependencies with test doubles via `unittest.mock`.
- **Test coverage**: Measuring how much code is tested.

### Core Concepts / Features

---

## 1. Test Cases

### Definitions

**Core Definition**: A test case is the individual unit of testing, created by subclassing `unittest.TestCase` to group logically related verification routines together .

**Technical Definition**: `unittest.TestCase` is the base class for creating test cases. Test methods are defined with names beginning with `test_`, which informs the test runner which methods represent tests. Each test case can contain any number of test methods, and each method is a standalone test that can be run independently .

**Beginner-Friendly Explanation**: A test case is a class that holds a group of related tests. You write a class that inherits from `unittest.TestCase`, then add methods that start with `test_`. Each method is one test.

### Purposes

- To group logically related tests together within a single class.
- To provide a structured, object-oriented way to organize tests.
- To enable independent execution of each test method.
- To support inheritance and code reuse across test classes.
- To integrate with the unittest test runner and test discovery mechanisms.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import unittest

class TestClassName(unittest.TestCase):
    def test_method_one(self):
        # Test logic
        pass

    def test_method_two(self):
        # Test logic
        pass

if __name__ == '__main__':
    unittest.main()
```

**Component Breakdown**:
- `unittest.TestCase` — the base class for all test cases.
- `test_method_*` — methods starting with `test_` are treated as tests.
- `unittest.main()` — runs all tests in the module.

#### Syntax Rules

1. **Subclass `unittest.TestCase`**: Every test case class must inherit from `unittest.TestCase` .
2. **Test method naming**: Test methods must start with `test_` to be discovered.
3. **Test methods take only `self`**: No additional arguments.
4. **One concept per test**: Each test method should verify one logical concept.
5. **Tests are independent**: They should not rely on execution order.
6. **Use `unittest.main()` for direct execution**: Allows running tests from the command line.

#### Constraints and Limitations

- Test method names must start with `test_`; otherwise they are not discovered.
- Test classes must inherit from `TestCase`.
- `unittest` does not support parametrized tests natively (unlike pytest).

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Test Case

```python
import unittest

# step1: Define the class under test
def add(a, b):
    return a + b

# step2: Create a test case class
class TestAddFunction(unittest.TestCase):
    def test_add_positive_numbers(self):
        """Test addition with positive integers."""
        self.assertEqual(add(2, 3), 5)

    def test_add_negative_numbers(self):
        """Test addition with negative integers."""
        self.assertEqual(add(-1, -1), -2)

    def test_add_zero(self):
        """Test addition with zero."""
        self.assertEqual(add(0, 5), 5)

# step3: Run tests
if __name__ == '__main__':
    unittest.main()
```

**Expected Output**:
```
...
----------------------------------------------------------------------
Ran 3 tests in 0.001s

OK
```

**Why**: Each test method verifies a specific case; `assertEqual` checks the expected output .

#### Example 2: Test Case with Multiple Assertions

```python
import unittest

class TestStringMethods(unittest.TestCase):
    def test_upper(self):
        self.assertEqual('foo'.upper(), 'FOO')

    def test_isupper(self):
        self.assertTrue('FOO'.isupper())
        self.assertFalse('Foo'.isupper())

    def test_split(self):
        s = 'hello world'
        self.assertEqual(s.split(), ['hello', 'world'])
        with self.assertRaises(TypeError):
            s.split(2)

if __name__ == '__main__':
    unittest.main()
```

**Expected Output**:
```
...
----------------------------------------------------------------------
Ran 3 tests in 0.000s

OK
```

**Why**: This example from the official documentation demonstrates how to group related string method tests into one test case .

#### Example 3: Test Case with Setup

```python
import unittest

class TestListOperations(unittest.TestCase):
    def setUp(self):
        """Runs before each test method."""
        self.items = [1, 2, 3]

    def test_append(self):
        self.items.append(4)
        self.assertEqual(len(self.items), 4)

    def test_remove(self):
        self.items.remove(2)
        self.assertNotIn(2, self.items)

    def test_clear(self):
        self.items.clear()
        self.assertEqual(self.items, [])

if __name__ == '__main__':
    unittest.main()
```

**Expected Output**:
```
...
----------------------------------------------------------------------
Ran 3 tests in 0.000s

OK
```

**Why**: `setUp` ensures each test starts with a fresh list, preventing tests from interfering with each other.

### Real-World Cases

- **Library functions**: Testing utility functions and helpers.
- **Business logic**: Testing pricing rules, validators, and calculators.
- **Data transformations**: Testing parsers and serializers.
- **API clients**: Testing request/response handling with mocks.

### References

- unittest — Unit testing framework (Python Documentation) - https://docs.python.org/3/library/unittest.html
- unittest.TestCase - https://docs.python.org/3/library/unittest.html#unittest.TestCase

---

## 2. Assertions

### Definitions

**Core Definition**: Assertions are methods provided by `TestCase` that check for and report failures, evaluating states and side effects .

**Technical Definition**: `unittest.TestCase` provides a comprehensive set of assertion methods that check specific conditions and raise `AssertionError` if the condition is not met. The most commonly used methods include `assertEqual()`, `assertTrue()`, `assertFalse()`, `assertRaises()`, and `assertLogs()` . The `assertEqual()` method dispatches the equality check for objects of the same type to different type-specific methods .

**Beginner-Friendly Explanation**: Assertions are how you tell `unittest` what you expect to be true. If the assertion fails, the test fails. For example, `assertEqual(result, expected)` checks that `result` equals `expected`.

### Purposes

- To verify that code produces the expected output.
- To check that specific conditions hold true.
- To verify that expected exceptions are raised.
- To validate side effects such as log messages.
- To provide informative failure messages when tests fail.

### Syntax Rules and Structure

#### Complete General Syntax

```python
self.assertEqual(first, second, msg=None)
self.assertTrue(expr, msg=None)
self.assertFalse(expr, msg=None)
self.assertRaises(exception, callable, *args, **kwargs)
self.assertLogs(logger=None, level=None)
```

#### Common Assertion Methods

| Method | Checks |
|--------|--------|
| `assertEqual(a, b)` | `a == b` |
| `assertNotEqual(a, b)` | `a != b` |
| `assertTrue(x)` | `bool(x) is True` |
| `assertFalse(x)` | `bool(x) is False` |
| `assertIs(a, b)` | `a is b` |
| `assertIsNot(a, b)` | `a is not b` |
| `assertIsNone(x)` | `x is None` |
| `assertIsNotNone(x)` | `x is not None` |
| `assertIn(a, b)` | `a in b` |
| `assertNotIn(a, b)` | `a not in b` |
| `assertIsInstance(a, b)` | `isinstance(a, b)` |
| `assertNotIsInstance(a, b)` | `not isinstance(a, b)` |
| `assertRaises(exc, fun, *args)` | `fun(*args)` raises `exc` |
| `assertRaisesRegex(exc, r, fun)` | `fun(*args)` raises `exc` with message matching `r` |
| `assertAlmostEqual(a, b)` | `round(a-b, 7) == 0` |
| `assertGreater(a, b)` | `a > b` |
| `assertGreaterEqual(a, b)` | `a >= b` |
| `assertLess(a, b)` | `a < b` |
| `assertLessEqual(a, b)` | `a <= b` |
| `assertRegex(s, r)` | `r.search(s)` |
| `assertCountEqual(a, b)` | `a` and `b` have the same elements, regardless of order |

#### Syntax Rules

1. **Assertion methods are instance methods**: Call them as `self.assertEqual(...)`.
2. **Optional `msg` parameter**: Provide a custom failure message.
3. **`assertRaises` as context manager**: `with self.assertRaises(TypeError):` .
4. **`assertRaises` as callable**: `self.assertRaises(TypeError, s.split, 2)`.
5. **`assertEqual` is type-aware**: Dispatches to type-specific equality methods .
6. **`assertLogs` for log testing**: Captures log output for verification .

#### Constraints and Limitations

- Assertion methods raise `AssertionError`; they cannot return values.
- `assertEqual` is not the same as `assertIs`; equality ≠ identity.
- Some assertions (e.g., `assertAlmostEqual`) require appropriate data types.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Assertions

```python
import unittest

class TestAssertions(unittest.TestCase):
    def test_equality(self):
        self.assertEqual(1 + 1, 2)
        self.assertNotEqual(1, 2)

    def test_boolean(self):
        self.assertTrue(True)
        self.assertFalse(False)
        self.assertIsNone(None)
        self.assertIsNotNone(42)

    def test_membership(self):
        self.assertIn(1, [1, 2, 3])
        self.assertNotIn(4, [1, 2, 3])

    def test_identity(self):
        a = [1, 2, 3]
        b = a
        self.assertIs(a, b)
        self.assertIsNot(a, [1, 2, 3])

    def test_approximate(self):
        self.assertAlmostEqual(0.1 + 0.2, 0.3, places=7)

    def test_regex(self):
        self.assertRegex("error: not found", r"error:.*not found")

if __name__ == '__main__':
    unittest.main()
```

**Expected Output**:
```
......
----------------------------------------------------------------------
Ran 6 tests in 0.001s

OK
```

**Why**: Each assertion method checks a specific condition; all pass because the conditions are true .

#### Example 2: `assertRaises` with Context Manager

```python
import unittest

def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

class TestDivide(unittest.TestCase):
    def test_divide_by_zero_raises(self):
        with self.assertRaises(ValueError) as context:
            divide(10, 0)
        self.assertEqual(str(context.exception), "Cannot divide by zero")

    def test_divide_by_zero_regex(self):
        with self.assertRaisesRegex(ValueError, "Cannot divide by zero"):
            divide(10, 0)

if __name__ == '__main__':
    unittest.main()
```

**Expected Output**:
```
..
----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
```

**Why**: `assertRaises` verifies that the expected exception is raised; `assertRaisesRegex` also checks the exception message .

#### Example 3: `assertLogs`

```python
import unittest
import logging

class TestLogging(unittest.TestCase):
    def test_log_message(self):
        logger = logging.getLogger("myapp")
        with self.assertLogs("myapp", level="INFO") as cm:
            logger.info("User logged in")
        self.assertIn("User logged in", cm.output[0])

if __name__ == '__main__':
    unittest.main()
```

**Expected Output**:
```
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
```

**Why**: `assertLogs` captures log records and allows assertions on their content .

### Real-World Cases

- **Input validation**: Testing that validators raise exceptions for invalid input.
- **Logging**: Verifying that important events are logged.
- **Numeric computations**: Testing floating-point results with `assertAlmostEqual`.
- **Collection operations**: Testing membership and identity.

### References

- Assertion Methods - https://docs.python.org/3/library/unittest.html#assert-methods
- unittest.mock — mock object library - https://docs.python.org/3/library/unittest.mock.html

---

## 3. Test Fixtures

### Definitions

**Core Definition**: A test fixture represents the preparation needed to perform one or more tests and any associated cleanup actions, ensuring a clean, predictable environment before evaluation begins .

**Technical Definition**: Fixtures in `unittest` are implemented through `setUp()` and `tearDown()` methods for method-level setup and cleanup, and `setUpClass()` and `tearDownClass()` class methods for class-level fixtures. These hooks allow tests to run in a controlled environment, such as creating temporary databases, directories, or server processes .

**Beginner-Friendly Explanation**: A fixture is the setup and cleanup you do before and after tests. For example, if your tests need a database, you create it before the tests run and destroy it afterward. Fixtures make sure each test starts with a known, clean state.

### Purposes

- To create a predictable, isolated environment for each test.
- To share expensive setup code across multiple tests.
- To ensure proper cleanup of resources after tests.
- To reduce duplication of setup/teardown logic.
- To isolate tests from each other by resetting state.

### Syntax Rules and Structure

#### Complete General Syntax

```python
class TestExample(unittest.TestCase):
    @classmethod
    def setUpClass(cls):
        """Runs once before all tests in the class."""
        pass

    @classmethod
    def tearDownClass(cls):
        """Runs once after all tests in the class."""
        pass

    def setUp(self):
        """Runs before each test method."""
        pass

    def tearDown(self):
        """Runs after each test method."""
        pass
```

#### Syntax Rules

1. **`setUpClass` and `tearDownClass`**: Must be decorated with `@classmethod`; run once per class .
2. **`setUp` and `tearDown`**: Run before and after each test method .
3. **`tearDown` runs even if the test fails**: Ensures cleanup regardless of test outcome .
4. **Skipped tests**: Do not have `setUp` or `tearDown` run around them .
5. **Order of execution**: `setUpClass` → (`setUp` → test → `tearDown`) × N → `tearDownClass`.

#### Constraints and Limitations

- `setUpClass` and `tearDownClass` must be class methods.
- Fixtures cannot be shared across test classes (unless using module-level fixtures).
- `tearDown` is not called if `setUp` fails.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Method-Level Fixtures

```python
import unittest

class TestCounter(unittest.TestCase):
    def setUp(self):
        """Create a fresh counter before each test."""
        self.counter = 0

    def test_increment(self):
        self.counter += 1
        self.assertEqual(self.counter, 1)

    def test_decrement(self):
        self.counter -= 1
        self.assertEqual(self.counter, -1)

if __name__ == '__main__':
    unittest.main()
```

**Expected Output**:
```
..
----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
```

**Why**: `setUp` resets the counter before each test, ensuring tests don't interfere with each other.

#### Example 2: Class-Level Fixtures

```python
import unittest

class TestDatabase(unittest.TestCase):
    @classmethod
    def setUpClass(cls):
        """Create a test database once for all tests."""
        cls.db = {"users": []}
        print("Database created")

    @classmethod
    def tearDownClass(cls):
        """Destroy the test database."""
        cls.db = None
        print("Database destroyed")

    def setUp(self):
        """Clear users before each test."""
        self.db["users"] = []

    def test_add_user(self):
        self.db["users"].append("Alice")
        self.assertEqual(len(self.db["users"]), 1)

    def test_add_two_users(self):
        self.db["users"].extend(["Alice", "Bob"])
        self.assertEqual(len(self.db["users"]), 2)

if __name__ == '__main__':
    unittest.main()
```

**Expected Output**:
```
Database created
..
Database destroyed
----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
```

**Why**: `setUpClass` creates the database once; `setUp` resets users before each test; `tearDownClass` cleans up after all tests .

#### Example 3: Combined Fixtures with Mocking

```python
import unittest
from unittest.mock import patch, MagicMock

class TestUserService(unittest.TestCase):
    @classmethod
    def setUpClass(cls):
        cls.db = MagicMock()

    @classmethod
    def tearDownClass(cls):
        cls.db.close()

    def setUp(self):
        self.service = UserService(self.db)
        self.db.begin_transaction()

    def tearDown(self):
        self.db.rollback()

    @patch("app.services.email_sender.send")
    def test_welcome_email_sent(self, mock_send):
        self.service.create(name="Alice", email="alice@test.com")
        mock_send.assert_called_once_with(
            to="alice@test.com",
            subject="Welcome!",
        )

if __name__ == '__main__':
    unittest.main()
```

**Expected Output**:
```
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
```

**Why**: Class-level fixtures manage the database connection; method-level fixtures manage transactions; mocking isolates the email service .

### Real-World Cases

- **Database testing**: Creating test databases and rolling back transactions.
- **File I/O**: Creating temporary files and directories.
- **Web testing**: Starting and stopping a test server.
- **Configuration**: Setting up environment variables and global state.

### References

- Test Fixtures - https://docs.python.org/3/library/unittest.html#test-fixtures
- setUpClass and tearDownClass - https://docs.python.org/3/library/unittest.html#unittest.TestCase.setUpClass

---

## 4. Test Suites

### Definitions

**Core Definition**: A test suite is a collection of test cases, test suites, or both, used to aggregate tests that should be executed together .

**Technical Definition**: `unittest.TestSuite` represents an aggregation of individual test cases and other test suites. It is used to run multi-module test executions simultaneously. The `TestSuite` class provides `addTest()` and `addTests()` methods to add test cases and suites, and can be run by a `TestRunner` .

**Beginner-Friendly Explanation**: A test suite is a bundle of tests that you run together. You can combine tests from different modules into one suite and run them all at once.

### Purposes

- To aggregate tests from multiple modules into a single runnable unit.
- To organize tests into logical groups for different purposes (e.g., smoke tests, regression tests).
- To control which tests run together in a single execution.
- To enable selective test execution based on context.
- To integrate with custom test runners and reporting tools.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import unittest

# Create a suite
suite = unittest.TestSuite()

# Add individual tests
suite.addTest(TestClass('test_method'))

# Add multiple tests
suite.addTests([TestClass('test_one'), TestClass('test_two')])

# Add another suite
suite.addTest(other_suite)

# Run the suite
runner = unittest.TextTestRunner()
runner.run(suite)
```

#### Syntax Rules

1. **`TestSuite()` constructor**: Creates an empty suite.
2. **`addTest(test)`**: Adds a single test case or suite.
3. **`addTests(tests)`**: Adds an iterable of test cases and suites.
4. **Suites can be nested**: A suite can contain other suites .
5. **Test order**: Tests run in the order they are added .
6. **TestRunner**: Used to execute the suite and report results.

#### Constraints and Limitations

- Suites must be explicitly constructed; unittest does not automatically aggregate tests across modules.
- Test order within a suite is the order of addition.
- Large suites may consume significant memory.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Test Suite

```python
import unittest

class TestMath(unittest.TestCase):
    def test_add(self):
        self.assertEqual(1 + 1, 2)

    def test_subtract(self):
        self.assertEqual(5 - 3, 2)

class TestStrings(unittest.TestCase):
    def test_upper(self):
        self.assertEqual("hello".upper(), "HELLO")

# step1: Create a suite
suite = unittest.TestSuite()

# step2: Add tests
suite.addTest(TestMath('test_add'))
suite.addTest(TestMath('test_subtract'))
suite.addTest(TestStrings('test_upper'))

# step3: Run
runner = unittest.TextTestRunner(verbosity=2)
runner.run(suite)
```

**Expected Output**:
```
test_add (__main__.TestMath) ... ok
test_subtract (__main__.TestMath) ... ok
test_upper (__main__.TestStrings) ... ok

----------------------------------------------------------------------
Ran 3 tests in 0.000s

OK
```

**Why**: The suite aggregates tests from two different classes; the runner executes them in order .

#### Example 2: Nested Suites

```python
import unittest

class TestMath(unittest.TestCase):
    def test_add(self):
        self.assertEqual(1 + 1, 2)

class TestStrings(unittest.TestCase):
    def test_upper(self):
        self.assertEqual("hello".upper(), "HELLO")

# step1: Create sub-suites
math_suite = unittest.TestSuite()
math_suite.addTest(TestMath('test_add'))

string_suite = unittest.TestSuite()
string_suite.addTest(TestStrings('test_upper'))

# step2: Create a master suite
master_suite = unittest.TestSuite()
master_suite.addTest(math_suite)
master_suite.addTest(string_suite)

# step3: Run
runner = unittest.TextTestRunner(verbosity=2)
runner.run(master_suite)
```

**Expected Output**:
```
test_add (__main__.TestMath) ... ok
test_upper (__main__.TestStrings) ... ok

----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
```

**Why**: Suites can be nested, allowing hierarchical organization of tests .

#### Example 3: Test Suite with TestLoader

```python
import unittest

# step1: Use TestLoader to discover tests
loader = unittest.TestLoader()

# step2: Load tests from a class
suite = loader.loadTestsFromTestCase(TestMath)

# step3: Load tests from a module
# suite = loader.loadTestsFromModule(my_module)

# step4: Run
runner = unittest.TextTestRunner(verbosity=2)
runner.run(suite)
```

**Expected Output**:
```
test_add (__main__.TestMath) ... ok
test_subtract (__main__.TestMath) ... ok

----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
```

**Why**: `TestLoader` automatically discovers all `test_*` methods in a class or module .

### Real-World Cases

- **Multi-module test runs**: Running all tests in a project from a single entry point.
- **CI pipelines**: Grouping tests by category (unit, integration, regression).
- **Smoke tests**: Running a subset of critical tests quickly.
- **Custom test runners**: Integrating with reporting tools.

### References

- TestSuite - https://docs.python.org/3/library/unittest.html#unittest.TestSuite
- TestLoader - https://docs.python.org/3/library/unittest.html#unittest.TestLoader

---

## 5. Setup and Teardown

### Definitions

**Core Definition**: Setup and teardown are lifecycle hooks that control the operational environment of tests, managing heavy configurations and ensuring proper cleanup.

**Technical Definition**: `unittest` provides four lifecycle hooks: `setUp()` and `tearDown()` for method-level control (run before and after each test), and `setUpClass()` and `tearDownClass()` for class-level control (run once before and after all tests in a class). These hooks enable shared setup code, resource management, and state isolation .

**Beginner-Friendly Explanation**: Setup and teardown are methods that run before and after your tests. `setUp` runs before each test, and `tearDown` runs after each test. `setUpClass` and `tearDownClass` run once for the whole class, which is useful for expensive setup like creating a database connection.

### Purposes

- To share expensive setup code across multiple tests.
- To ensure each test starts with a clean, known state.
- To properly clean up resources after tests complete.
- To isolate tests from each other by resetting state.
- To manage complex configurations (databases, servers, file systems).

### Syntax Rules and Structure

#### Complete General Syntax

```python
class TestExample(unittest.TestCase):
    @classmethod
    def setUpClass(cls):
        """Runs once before all tests in the class."""
        cls.shared_resource = create_expensive_resource()

    @classmethod
    def tearDownClass(cls):
        """Runs once after all tests in the class."""
        cls.shared_resource.close()

    def setUp(self):
        """Runs before each test method."""
        self.local_resource = create_local_resource()

    def tearDown(self):
        """Runs after each test method."""
        self.local_resource.cleanup()
```

#### Execution Order

```
setUpClass()
  setUp()
    test_method_1()
  tearDown()
  setUp()
    test_method_2()
  tearDown()
tearDownClass()
```

#### Syntax Rules

1. **`setUpClass` and `tearDownClass` are class methods**: Must be decorated with `@classmethod` .
2. **`setUp` and `tearDown` are instance methods**: Take `self` as the only argument.
3. **`tearDown` runs even if the test fails**: Ensures cleanup .
4. **Skipped tests**: Do not trigger `setUp` or `tearDown` .
5. **`setUpClass` failure**: If it fails, no tests in the class run.
6. **`setUp` failure**: If it fails, `tearDown` is not called for that test.

#### Constraints and Limitations

- `setUpClass` and `tearDownClass` must be class methods.
- Fixtures cannot be shared across test classes unless using module-level fixtures.
- `tearDown` is not called if `setUp` fails.
- Class-level fixtures are not run for skipped classes .

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Method-Level Setup and Teardown

```python
import unittest

class TestFileOperations(unittest.TestCase):
    def setUp(self):
        """Create a temporary file before each test."""
        self.file = open("test_temp.txt", "w")
        self.file.write("initial content")
        self.file.close()
        self.file = open("test_temp.txt", "r")

    def tearDown(self):
        """Clean up the temporary file."""
        self.file.close()
        import os
        os.remove("test_temp.txt")

    def test_read_content(self):
        content = self.file.read()
        self.assertEqual(content, "initial content")

    def test_file_exists(self):
        import os
        self.assertTrue(os.path.exists("test_temp.txt"))

if __name__ == '__main__':
    unittest.main()
```

**Expected Output**:
```
..
----------------------------------------------------------------------
Ran 2 tests in 0.001s

OK
```

**Why**: `setUp` creates a fresh file before each test; `tearDown` cleans it up afterward .

#### Example 2: Class-Level Setup and Teardown

```python
import unittest

class TestDatabaseConnection(unittest.TestCase):
    @classmethod
    def setUpClass(cls):
        """Establish database connection once for all tests."""
        cls.connection = create_test_database()
        print("Connection established")

    @classmethod
    def tearDownClass(cls):
        """Close connection after all tests."""
        cls.connection.close()
        print("Connection closed")

    def test_query_one(self):
        result = self.connection.execute("SELECT 1")
        self.assertEqual(result, 1)

    def test_query_two(self):
        result = self.connection.execute("SELECT 2")
        self.assertEqual(result, 2)

if __name__ == '__main__':
    unittest.main()
```

**Expected Output**:
```
Connection established
..
Connection closed
----------------------------------------------------------------------
Ran 2 tests in 0.001s

OK
```

**Why**: `setUpClass` creates the connection once; `tearDownClass` closes it after all tests .

#### Example 3: Combined Method and Class Fixtures

```python
import unittest

class TestUserService(unittest.TestCase):
    @classmethod
    def setUpClass(cls):
        cls.db = create_test_database()
        print("Database created")

    @classmethod
    def tearDownClass(cls):
        cls.db.close()
        print("Database closed")

    def setUp(self):
        self.service = UserService(self.db)
        self.db.begin_transaction()

    def tearDown(self):
        self.db.rollback()

    def test_create_user(self):
        user = self.service.create(name="Alice")
        self.assertIsNotNone(user.id)

    def test_duplicate_email_raises(self):
        self.service.create(name="Alice", email="alice@test.com")
        with self.assertRaises(DuplicateEmailError):
            self.service.create(name="Bob", email="alice@test.com")

if __name__ == '__main__':
    unittest.main()
```

**Expected Output**:
```
Database created
..
Database closed
----------------------------------------------------------------------
Ran 2 tests in 0.001s

OK
```

**Why**: Class fixtures manage the database connection; method fixtures manage transactions, ensuring each test runs in isolation .

### Real-World Cases

- **Database testing**: Creating test databases and rolling back transactions.
- **File I/O**: Creating temporary files and directories.
- **Web testing**: Starting and stopping a test server.
- **Configuration**: Setting up environment variables and global state.
- **Resource management**: Opening and closing network connections.

### References

- Test Fixtures - https://docs.python.org/3/library/unittest.html#test-fixtures
- setUpClass - https://docs.python.org/3/library/unittest.html#unittest.TestCase.setUpClass
- tearDownClass - https://docs.python.org/3/library/unittest.html#unittest.TestCase.tearDownClass

---

## References

- unittest — Unit testing framework - https://docs.python.org/3/library/unittest.html
- unittest.TestCase - https://docs.python.org/3/library/unittest.html#unittest.TestCase
- Assertion Methods - https://docs.python.org/3/library/unittest.html#assert-methods
- Test Fixtures - https://docs.python.org/3/library/unittest.html#test-fixtures
- TestSuite - https://docs.python.org/3/library/unittest.html#unittest.TestSuite
- TestLoader - https://docs.python.org/3/library/unittest.html#unittest.TestLoader
- unittest.mock — mock object library - https://docs.python.org/3/library/unittest.mock.html
- Python's unittest: Writing Unit Tests for Your Code (Real Python) - https://realpython.com/python-unittest/
- unittest Basics for Legacy Codebases - https://github.com/glennguilloux/llm-knowledge-base/blob/main/python/stdlib/unittest-basics.md