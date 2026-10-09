# SQL Query Testing: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**: SQL query testing is the systematic process of validating that SQL statements and the application code that executes them return correct results, handle edge cases gracefully, and resist malicious input across all expected and unexpected conditions.

**Technical Definition**: SQL query testing encompasses positive test cases (verifying valid parameters return expected datasets), negative test cases (ensuring invalid inputs or types produce controlled errors rather than crashes), boundary testing (evaluating behavior at date limits, numeric thresholds, and string length caps), NULL testing (validating that filters, calculations, and concatenations handle NULL correctly), empty-result testing (verifying application code handles zero rows gracefully), duplicate-data testing (ensuring queries do not inflate aggregates when duplicates exist), and SQL injection resiliency testing (confirming that dynamic queries use parameterized inputs rather than string concatenation).

**Beginner-Friendly Explanation**: SQL query testing is like crash-testing a car before selling it. You test normal driving (positive cases), bad roads (negative cases), extreme conditions (boundaries), missing parts (NULLs), empty fuel tanks (empty results), duplicate parts (duplicates), and attempted break-ins (SQL injection). Only when the car survives all these tests is it safe for production.

### Key Characteristics

| Characteristic | Description |
|----------------|-------------|
| **Data-Dependent** | Test outcomes depend on the data set, not just the query text |
| **Deterministic** | A correct query returns the same result for the same data every time |
| **Regression-Prone** | Small schema changes can break query results silently |
| **Security-Critical** | Injection testing is a security requirement, not just a functional one |
| **Automated** | Tests should run in CI/CD against a known test database |

### Prerequisites

- **Test Database**: A dedicated database with deterministic, version-controlled seed data
- **Test Framework**: A testing library (pytest, JUnit, Jest) with database fixtures
- **Assertion Helpers**: Utilities to compare result sets, row counts, and JSON structures
- **Parameterized Query Support**: The application must use bind parameters, not string concatenation
- **Baseline Data**: Known-good expected outputs for each test case

### Related Programming Areas

- **Unit Testing**: Testing individual query functions in isolation
- **Integration Testing**: Testing queries against a real database in CI
- **Security Engineering**: SQL injection prevention, input validation
- **Data Quality Engineering**: Ensuring data integrity across test scenarios
- **Test Data Management**: Seeding, resetting, and anonymizing test databases

### Core Concepts Overview

SQL query testing comprises seven complementary categories:

1. **Positive Test Cases**: Verifying valid parameters return expected datasets
2. **Negative Test Cases**: Ensuring queries handle bad inputs without crashing
3. **Boundary Testing**: Evaluating date limits, numeric thresholds, and length caps
4. **NULL Testing**: Validating how filters, calculations, and concatenations handle NULL
5. **Empty-Result Testing**: Verifying application code handles zero rows gracefully
6. **Duplicate-Data Testing**: Ensuring queries handle duplicates without inflating aggregates
7. **SQL Injection Resiliency Testing**: Validating parameterized inputs prevent malicious execution

---

## Core Concept 1: Positive Test Cases

### Definitions

**Core Definition**: Positive test cases verify that a query returns the expected result set when given valid, well-formed input parameters.

**Technical Definition**: A positive test case defines a known input (parameters, filter values) and an expected output (specific rows, columns, aggregates). It asserts that executing the query with that input produces exactly that output. Positive tests establish the baseline correctness of the query and catch regressions when schema or data changes.

**Beginner-Friendly Explanation**: Positive testing is like ordering a coffee and checking that you get the coffee you ordered. You provide a valid request and verify the result matches expectations.

### Purposes

- **To** establish a baseline of correct behavior for each query
- **To** catch regressions when schema, indexes, or data change
- **To** document expected behavior as executable specifications
- **To** verify that joins, filters, and aggregations return the correct rows and columns

### Syntax Rules and Structure

#### Test Structure (Python + pytest)

```python
import pytest
from db import get_connection

def test_get_orders_by_customer_returns_expected_orders():
    """Positive test: valid customer_id returns their orders."""
    conn = get_connection()
    cursor = conn.cursor()
    
    # Arrange: seed known data (or use pre-seeded fixtures)
    # Act: execute the query with valid input
    cursor.execute(
        "SELECT order_id, total FROM orders WHERE customer_id = %s ORDER BY order_id",
        (1,)
    )
    results = cursor.fetchall()
    
    # Assert: verify expected output
    assert len(results) == 2
    assert results[0] == (100, 500.00)
    assert results[1] == (101, 300.00)
```

#### Test Structure (Java + JUnit)

```java
@Test
public void testFindOrdersByCustomerReturnsExpectedOrders() {
    // Arrange
    Long customerId = 1L;
    
    // Act
    List<Order> orders = orderRepository.findByCustomerId(customerId);
    
    // Assert
    assertEquals(2, orders.size());
    assertEquals(100L, orders.get(0).getOrderId());
    assertEquals(new BigDecimal("500.00"), orders.get(0).getTotal());
}
```

#### Component Breakdown

| Component | Description |
|-----------|-------------|
| **Arrange** | Set up test data or fixtures |
| **Act** | Execute the query with valid input |
| **Assert** | Verify the result matches expectations |
| **Fixtures** | Pre-seeded data reset before each test |

#### Syntax Rules

- Tests must be **deterministic**: same input → same output.
- Test data must be **isolated**: one test's data must not affect another's.
- Assertions should check both **row count** and **column values**.
- Use **parameterized queries** in tests, not string concatenation.

#### Constraints and Limitations

- Positive tests do not catch edge cases; they only verify the "happy path."
- Test data must be reset between tests to avoid cross-test pollution.
- Floating-point comparisons require tolerance (`abs(a - b) < 0.001`).

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Positive Test for Customer Order Query

```python
import pytest
import psycopg2

@pytest.fixture
def db_connection():
    """Fixture: provides a connection with seeded test data."""
    conn = psycopg2.connect("dbname=testdb user=test password=test")
    cursor = conn.cursor()
    
    # Seed known data
    cursor.execute("DELETE FROM orders")
    cursor.execute("DELETE FROM customers")
    cursor.execute("INSERT INTO customers (customer_id, name) VALUES (1, 'Alice'), (2, 'Bob')")
    cursor.execute("""
        INSERT INTO orders (order_id, customer_id, total) VALUES
        (100, 1, 500.00),
        (101, 1, 300.00),
        (102, 2, 200.00)
    """)
    conn.commit()
    yield conn
    conn.close()

def test_orders_for_customer_1(db_connection):
    """Positive: customer 1 has exactly 2 orders with known totals."""
    cursor = db_connection.cursor()
    cursor.execute(
        "SELECT order_id, total FROM orders WHERE customer_id = %s ORDER BY order_id",
        (1,)
    )
    results = cursor.fetchall()
    
    assert len(results) == 2, f"Expected 2 orders, got {len(results)}"
    assert results[0] == (100, 500.00)
    assert results[1] == (101, 300.00)

def test_orders_for_customer_2(db_connection):
    """Positive: customer 2 has exactly 1 order."""
    cursor = db_connection.cursor()
    cursor.execute(
        "SELECT order_id, total FROM orders WHERE customer_id = %s",
        (2,)
    )
    results = cursor.fetchall()
    
    assert len(results) == 1
    assert results[0] == (102, 200.00)
```

**Expected Output**:
```
test_orders_for_customer_1 PASSED
test_orders_for_customer_2 PASSED
```

**Why This Output Occurs**: The fixture seeds deterministic data. Each test executes a parameterized query with a valid `customer_id` and asserts the exact rows returned. Both tests pass because the query returns the expected data.

### Real-World Cases

**Case 1: Regression After Schema Change**: A column rename breaks a query. The positive test fails immediately in CI, catching the regression before deployment.

**Case 2: API Contract Verification**: A REST API's data-access layer is tested with positive cases to ensure the API returns the documented response shape.

**Case 3: Reporting Accuracy**: A financial report query is tested with known data to verify totals match manual calculations.

---

## Core Concept 2: Negative Test Cases

### Definitions

**Core Definition**: Negative test cases verify that a query or application code handles invalid, malformed, or unexpected input without crashing, corrupting data, or exposing sensitive information.

**Technical Definition**: Negative testing provides inputs that violate assumptions: non-existent IDs, wrong data types, malformed strings, out-of-range values, or missing required parameters. The expected behavior is a controlled error (a specific exception, an empty result, or a validation message), not an unhandled crash, a stack trace leak, or a database error propagated to the user.

**Beginner-Friendly Explanation**: Negative testing is like trying to use a coffee machine with no water, no beans, or a foreign coin. A good machine says "please add water" or "invalid coin" instead of exploding. Negative tests verify your query handles bad input gracefully.

### Purposes

- **To** ensure invalid input produces controlled errors, not crashes
- **To** verify that type mismatches are caught before reaching the database
- **To** prevent information leakage through raw database error messages
- **To** validate that non-existent IDs return empty results or clear errors

### Syntax Rules and Structure

#### Test Structure (Python)

```python
def test_get_orders_for_nonexistent_customer_returns_empty():
    """Negative: non-existent customer_id returns no rows, not an error."""
    cursor.execute(
        "SELECT * FROM orders WHERE customer_id = %s",
        (99999,)
    )
    results = cursor.fetchall()
    assert results == []

def test_get_orders_for_invalid_type_raises_validation_error():
    """Negative: non-integer customer_id raises a controlled error."""
    with pytest.raises(ValidationError):
        order_service.get_orders(customer_id="not-a-number")
```

#### Test Structure (Java)

```java
@Test
public void testFindOrdersForNonexistentCustomerReturnsEmpty() {
    List<Order> orders = orderRepository.findByCustomerId(99999L);
    assertTrue(orders.isEmpty());
}

@Test
public void testFindOrdersWithNullCustomerIdThrowsValidationException() {
    assertThrows(ValidationException.class, () -> {
        orderService.getOrders(null);
    });
}
```

#### Component Breakdown

| Input Type | Expected Behavior |
|------------|-------------------|
| Non-existent ID | Empty result or `NotFoundException` |
| Wrong data type | `ValidationException` before DB call |
| Malformed string | `ValidationException` or sanitized error |
| Out-of-range value | `ValidationException` |
| Missing required param | `ValidationException` |
| Null where not allowed | `ValidationException` |

#### Syntax Rules

- Negative tests should assert **specific exception types**, not generic `Exception`.
- Database errors should be **translated** into application exceptions before reaching the user.
- Tests should verify that **raw SQL is not exposed** in error messages.
- Non-existent IDs in `WHERE` clauses return empty results, not errors.

#### Constraints and Limitations

- Some databases coerce types implicitly (MySQL), so type validation must happen at the application layer.
- Negative tests do not replace input validation; they verify it.
- Stack traces in test output are acceptable; stack traces in production are not.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Negative Test for Invalid Customer ID Type

```python
import pytest
from app.exceptions import ValidationError
from app.services import OrderService

def test_get_orders_with_string_customer_id_raises_validation_error():
    """Negative: passing a string where an integer is expected."""
    service = OrderService()
    
    with pytest.raises(ValidationError) as exc_info:
        service.get_orders(customer_id="abc")
    
    assert "must be an integer" in str(exc_info.value)

def test_get_orders_with_negative_customer_id_raises_validation_error():
    """Negative: negative IDs are invalid."""
    service = OrderService()
    
    with pytest.raises(ValidationError) as exc_info:
        service.get_orders(customer_id=-1)
    
    assert "must be positive" in str(exc_info.value)

def test_get_orders_with_nonexistent_customer_returns_empty():
    """Negative: non-existent ID returns empty list, not error."""
    service = OrderService()
    orders = service.get_orders(customer_id=99999)
    
    assert orders == []
```

**Expected Output**:
```
test_get_orders_with_string_customer_id_raises_validation_error PASSED
test_get_orders_with_negative_customer_id_raises_validation_error PASSED
test_get_orders_with_nonexistent_customer_returns_empty PASSED
```

**Why This Output Occurs**: The service validates the `customer_id` before querying. A string raises `ValidationError("customer_id must be an integer")`; a negative value raises `ValidationError("customer_id must be positive")`. A valid but non-existent ID passes validation and returns an empty list because the query returns no rows.

### Real-World Cases

**Case 1: SQL Error Leakage**: A raw database error (`ERROR: relation "users" does not exist`) is exposed to the user via an API. The negative test verifies the application translates it to a generic `InternalError`.

**Case 2: Type Coercion Bug**: MySQL silently converts `'abc'` to `0` when comparing to an integer column, returning wrong rows. The negative test catches this at the application layer.

**Case 3: Null Parameter Crash**: A `null` parameter causes a `NullPointerException`. The negative test verifies a `ValidationException` is raised instead.

---

## Core Concept 3: Boundary Testing

### Definitions

**Core Definition**: Boundary testing verifies query behavior at the exact edges of valid ranges—minimum and maximum dates, numeric thresholds, and string length limits.

**Technical Definition**: Boundary value analysis tests values at, just below, and just above each boundary. For a date range `BETWEEN '2025-01-01' AND '2025-12-31'`, boundaries include `2024-12-31`, `2025-01-01`, `2025-12-31`, and `2026-01-01`. For `VARCHAR(50)`, boundaries include 49, 50, and 51 characters. For `LIMIT 100`, boundaries include 99, 100, and 101. Off-by-one errors are the most common boundary defects.

**Beginner-Friendly Explanation**: Boundary testing is like testing a bridge's weight limit. If the sign says "max 10 tons," you test 9.9 tons (passes), 10 tons (passes), and 10.1 tons (fails). The edges are where bugs hide.

### Purposes

- **To** catch off-by-one errors in date ranges and numeric comparisons
- **To** verify that inclusive/exclusive boundaries behave as documented
- **To** test string length limits (VARCHAR truncation, validation)
- **To** verify pagination limits (LIMIT/OFFSET at boundaries)

### Syntax Rules and Structure

#### Test Structure (Python)

```python
@pytest.mark.parametrize("order_date,expected_count", [
    ("2024-12-31", 0),   # Just below range
    ("2025-01-01", 1),   # Lower boundary (inclusive)
    ("2025-06-15", 1),   # Middle
    ("2025-12-31", 1),   # Upper boundary (inclusive)
    ("2026-01-01", 0),   # Just above range
])
def test_orders_in_2025_boundary(order_date, expected_count):
    cursor.execute(
        "SELECT COUNT(*) FROM orders WHERE order_date = %s",
        (order_date,)
    )
    assert cursor.fetchone()[0] == expected_count
```

#### Test Structure (String Length)

```python
@pytest.mark.parametrize("name_length,should_succeed", [
    (49, True),   # Below limit
    (50, True),   # At limit
    (51, False),  # Above limit
])
def test_customer_name_length_boundary(name_length, should_succeed):
    name = "A" * name_length
    try:
        service.create_customer(name=name)
        assert should_succeed
    except ValidationError:
        assert not should_succeed
```

#### Component Breakdown

| Boundary Type | Values to Test |
|---------------|----------------|
| Date range | day before, first day, last day, day after |
| Numeric threshold | threshold - 1, threshold, threshold + 1 |
| String length | n - 1, n, n + 1 |
| LIMIT/OFFSET | limit - 1, limit, limit + 1 |
| NULL boundary | NULL, empty string, whitespace |

#### Syntax Rules

- Test **both sides** of every boundary (just below and just above).
- Use **parameterized tests** to avoid repetitive test code.
- Date boundaries depend on whether the comparison is inclusive (`>=`, `<=`) or exclusive (`>`, `<`).
- `BETWEEN` is inclusive on both ends in standard SQL.

#### Constraints and Limitations

- Date/time boundaries depend on timezone; test with UTC.
- String length is in characters, not bytes; multi-byte characters count as one character in `VARCHAR(n)`.
- Floating-point boundaries require tolerance.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Date Range Boundary Testing

```python
import pytest
from datetime import date

@pytest.fixture
def seeded_orders(db_connection):
    cursor = db_connection.cursor()
    cursor.execute("DELETE FROM orders")
    cursor.execute("""
        INSERT INTO orders (order_id, order_date, total) VALUES
        (1, '2024-12-31', 100.00),
        (2, '2025-01-01', 200.00),
        (3, '2025-06-15', 300.00),
        (4, '2025-12-31', 400.00),
        (5, '2026-01-01', 500.00)
    """)
    db_connection.commit()

@pytest.mark.parametrize("start,end,expected_ids", [
    ("2025-01-01", "2025-12-31", [2, 3, 4]),   # Full year
    ("2025-01-01", "2025-01-01", [2]),          # Single day
    ("2024-12-31", "2024-12-31", [1]),          # Just below
    ("2026-01-01", "2026-01-01", [5]),          # Just above
    ("2025-12-31", "2025-12-31", [4]),          # Upper boundary
])
def test_orders_in_date_range(seeded_orders, start, end, expected_ids):
    cursor = seeded_orders.cursor()
    cursor.execute("""
        SELECT order_id FROM orders 
        WHERE order_date BETWEEN %s AND %s 
        ORDER BY order_id
    """, (start, end))
    results = [row[0] for row in cursor.fetchall()]
    assert results == expected_ids
```

**Expected Output**:
```
test_orders_in_date_range[2025-01-01-2025-12-31-[2,3,4]] PASSED
test_orders_in_date_range[2025-01-01-2025-01-01-[2]] PASSED
test_orders_in_date_range[2024-12-31-2024-12-31-[1]] PASSED
test_orders_in_date_range[2026-01-01-2026-01-01-[5]] PASSED
test_orders_in_date_range[2025-12-31-2025-12-31-[4]] PASSED
```

**Why This Output Occurs**: `BETWEEN` is inclusive on both ends. The test verifies that the lower boundary (`2025-01-01`) and upper boundary (`2025-12-31`) are both included, and that dates just outside the range (`2024-12-31`, `2026-01-01`) are excluded.

#### Example 2: LIMIT Boundary Testing

```python
@pytest.mark.parametrize("limit,expected_count", [
    (0, 0),     # Zero limit
    (1, 1),     # Minimum
    (10, 10),   # Exact
    (11, 11),   # Above
    (100, 100), # Large
])
def test_order_pagination_limit(db_connection, limit, expected_count):
    cursor = db_connection.cursor()
    cursor.execute("SELECT * FROM orders LIMIT %s", (limit,))
    results = cursor.fetchall()
    assert len(results) == expected_count
```

**Why This Output Occurs**: `LIMIT 0` returns no rows; `LIMIT 1` returns one; `LIMIT 10` returns ten (assuming at least ten rows exist). Testing `LIMIT 0` catches applications that assume `LIMIT` is always positive.

### Real-World Cases

**Case 1: Date Range Off-by-One**: A report query uses `BETWEEN '2025-01-01' AND '2025-12-31'` but the business expects the range to exclude the end date. The boundary test reveals the discrepancy.

**Case 2: VARCHAR Truncation**: A `VARCHAR(50)` column truncates a 51-character value silently in MySQL non-strict mode. The boundary test verifies the application rejects the value before insert.

**Case 3: Pagination Edge**: A `LIMIT 100 OFFSET 0` query returns 100 rows, but `LIMIT 100 OFFSET 100` returns nothing when only 150 rows exist. The boundary test verifies correct behavior at the last page.

---

## Core Concept 4: NULL Testing

### Definitions

**Core Definition**: NULL testing verifies that queries handle NULL values correctly in filters, calculations, concatenations, and aggregations.

**Technical Definition**: NULL testing validates three-valued logic: `NULL = NULL` is UNKNOWN, `NULL` propagates through arithmetic (`5 + NULL = NULL`), string concatenation with NULL returns NULL in standard SQL, and aggregates ignore NULLs. Tests verify that `IS NULL` / `IS NOT NULL` are used correctly, `COALESCE` substitutes defaults, and `NOT IN` with NULL-producing subqueries is avoided.

**Beginner-Friendly Explanation**: NULL testing is like checking how a form handles blank fields. Does a blank middle name break the full-name calculation? Does a blank phone number appear in the contact list? NULL tests verify that blank values don't silently corrupt results.

### Purposes

- **To** verify that filters using `= NULL` are corrected to `IS NULL`
- **To** ensure calculations with NULL produce expected results (NULL propagation)
- **To** verify that concatenations handle NULL gracefully (`COALESCE`)
- **To** test aggregate functions with NULL inputs

### Syntax Rules and Structure

#### Test Structure (Python)

```python
@pytest.mark.parametrize("email,expected_count", [
    (None, 1),                    # NULL email
    ("alice@example.com", 1),     # Non-NULL email
    ("nonexistent@example.com", 0),  # No match
])
def test_find_by_email_handles_null(db_connection, email, expected_count):
    cursor = db_connection.cursor()
    if email is None:
        cursor.execute("SELECT COUNT(*) FROM customers WHERE email IS NULL")
    else:
        cursor.execute("SELECT COUNT(*) FROM customers WHERE email = %s", (email,))
    assert cursor.fetchone()[0] == expected_count
```

#### Test Structure (COALESCE)

```python
def test_full_name_handles_null_middle_name(db_connection):
    cursor = db_connection.cursor()
    cursor.execute("""
        SELECT COALESCE(first_name, '') || ' ' || COALESCE(middle_name, '') || ' ' || COALESCE(last_name, '')
        FROM people WHERE person_id = 1
    """)
    result = cursor.fetchone()[0]
    assert result == "Alice  Smith"  # No middle name
```

#### Component Breakdown

| Scenario | NULL Behavior | Correct Handling |
|----------|---------------|------------------|
| `WHERE col = NULL` | Always UNKNOWN | Use `IS NULL` |
| `WHERE col <> NULL` | Always UNKNOWN | Use `IS NOT NULL` |
| `5 + NULL` | NULL | Use `COALESCE(col, 0)` |
| `'a' || NULL` | NULL (standard) | Use `COALESCE(col, '')` |
| `COUNT(col)` | Ignores NULL | Use `COUNT(*)` if all rows count |
| `SUM(col)` | Ignores NULL | Use `SUM(COALESCE(col, 0))` |
| `NOT IN (subquery with NULL)` | Returns no rows | Use `NOT EXISTS` |

#### Syntax Rules

- Always use `IS NULL` / `IS NOT NULL` for NULL checks.
- Use `COALESCE` for default values in calculations and concatenations.
- `COUNT(*)` counts all rows; `COUNT(column)` ignores NULLs.
- `NOT IN` with a subquery that may return NULL produces no rows; use `NOT EXISTS`.

#### Constraints and Limitations

- NULL behavior varies slightly between engines (e.g., Oracle treats empty string as NULL).
- `COALESCE` short-circuits at the first non-NULL value.
- Aggregate functions return NULL for empty sets (except `COUNT`, which returns 0).

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Testing NULL Handling in Filters and Calculations

```python
def test_filter_by_null_email(db_connection):
    """Verify IS NULL returns rows with NULL email."""
    cursor = db_connection.cursor()
    cursor.execute("SELECT COUNT(*) FROM customers WHERE email IS NULL")
    assert cursor.fetchone()[0] == 1

def test_filter_by_equals_null_returns_nothing(db_connection):
    """Verify = NULL returns no rows (common mistake)."""
    cursor = db_connection.cursor()
    cursor.execute("SELECT COUNT(*) FROM customers WHERE email = NULL")
    assert cursor.fetchone()[0] == 0  # Always 0, not an error

def test_sum_with_null_uses_coalesce(db_connection):
    """Verify SUM(COALESCE(col, 0)) includes NULL rows as zero."""
    cursor = db_connection.cursor()
    cursor.execute("SELECT SUM(COALESCE(commission, 0)) FROM sales")
    total_with_nulls = cursor.fetchone()[0]
    
    cursor.execute("SELECT SUM(commission) FROM sales")
    total_without_nulls = cursor.fetchone()[0]
    
    assert total_with_nulls >= total_without_nulls
```

**Expected Output**:
```
test_filter_by_null_email PASSED
test_filter_by_equals_null_returns_nothing PASSED
test_sum_with_null_uses_coalesce PASSED
```

**Why This Output Occurs**: `IS NULL` correctly returns rows with NULL email. `= NULL` returns zero rows because the comparison is UNKNOWN. `SUM(COALESCE(commission, 0))` treats NULL commissions as zero, producing a larger total than `SUM(commission)`, which ignores NULLs.

#### Example 2: Testing NOT IN with NULL

```python
def test_not_in_with_null_returns_nothing(db_connection):
    """Verify NOT IN with a NULL in the subquery returns no rows."""
    cursor = db_connection.cursor()
    cursor.execute("""
        SELECT COUNT(*) FROM customers
        WHERE customer_id NOT IN (SELECT customer_id FROM orders)
    """)
    # If any order has NULL customer_id, this returns 0
    assert cursor.fetchone()[0] == 0

def test_not_exists_with_null_returns_correct(db_connection):
    """Verify NOT EXISTS handles NULLs correctly."""
    cursor = db_connection.cursor()
    cursor.execute("""
        SELECT COUNT(*) FROM customers c
        WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id)
    """)
    # Returns customers with no orders, correctly
    assert cursor.fetchone()[0] >= 1
```

**Expected Output**:
```
test_not_in_with_null_returns_nothing PASSED
test_not_exists_with_null_returns_correct PASSED
```

**Why This Output Occurs**: `NOT IN` with a NULL in the subquery returns no rows because every comparison is UNKNOWN. `NOT EXISTS` handles NULLs correctly, returning customers with no matching orders.

### Real-World Cases

**Case 1: Missing Email Report**: A report uses `WHERE email = NULL` and returns no rows. The NULL test catches the mistake and the fix is `WHERE email IS NULL`.

**Case 2: Full Name Concatenation**: A `first_name || ' ' || last_name` query returns NULL when `last_name` is NULL. The fix uses `COALESCE(last_name, '')`.

**Case 3: Commission Total**: A `SUM(commission)` report undercounts because some salespeople have NULL commissions. The fix uses `SUM(COALESCE(commission, 0))`.

---

## Core Concept 5: Empty-Result Testing

### Definitions

**Core Definition**: Empty-result testing verifies that application code handles queries returning zero rows gracefully, without crashing, returning malformed responses, or exposing errors.

**Technical Definition**: Empty-result testing validates that `fetchone()` returning `None`, `fetchall()` returning `[]`, and aggregate queries returning NULL are handled correctly. Application code must distinguish between "no rows" (valid empty result) and "error" (query failed). API endpoints must return empty arrays or 404s as documented, not null pointers or 500 errors.

**Beginner-Friendly Explanation**: Empty-result testing is like checking that a search engine returns "no results found" instead of crashing when you search for something that doesn't exist. The query succeeds; it just returns nothing.

### Purposes

- **To** verify that application code handles zero rows without crashing
- **To** ensure API endpoints return empty arrays or 404s as documented
- **To** validate that aggregate queries on empty sets return sensible defaults
- **To** distinguish "no results" from "error"

### Syntax Rules and Structure

#### Test Structure (Python)

```python
def test_get_customer_returns_none_when_not_found(db_connection):
    """Verify fetchone() returns None for no rows."""
    cursor = db_connection.cursor()
    cursor.execute("SELECT * FROM customers WHERE customer_id = 99999")
    result = cursor.fetchone()
    assert result is None

def test_list_orders_returns_empty_list_when_none(db_connection):
    """Verify fetchall() returns [] for no rows."""
    cursor = db_connection.cursor()
    cursor.execute("SELECT * FROM orders WHERE customer_id = 99999")
    results = cursor.fetchall()
    assert results == []

def test_count_returns_zero_for_empty_table(db_connection):
    """Verify COUNT(*) returns 0 for empty table."""
    cursor = db_connection.cursor()
    cursor.execute("DELETE FROM orders WHERE customer_id = 99999")
    cursor.execute("SELECT COUNT(*) FROM orders WHERE customer_id = 99999")
    assert cursor.fetchone()[0] == 0

def test_sum_returns_null_for_empty_set(db_connection):
    """Verify SUM returns NULL for no rows."""
    cursor = db_connection.cursor()
    cursor.execute("SELECT SUM(total) FROM orders WHERE customer_id = 99999")
    assert cursor.fetchone()[0] is None
```

#### Component Breakdown

| Query Type | Empty-Set Return | Application Handling |
|------------|------------------|---------------------|
| `SELECT ... WHERE` | 0 rows | Return empty list or 404 |
| `fetchone()` | `None` | Check for `None` |
| `fetchall()` | `[]` | Iterate (no-op) |
| `COUNT(*)` | `0` | Safe to use |
| `SUM()` | `NULL` | Use `COALESCE(SUM(...), 0)` |
| `AVG()` | `NULL` | Use `COALESCE(AVG(...), 0)` |

#### Syntax Rules

- `fetchone()` returns `None` for no rows; always check before accessing attributes.
- `fetchall()` returns an empty list for no rows; safe to iterate.
- `COUNT(*)` always returns a number (0 for empty sets).
- `SUM`, `AVG`, `MIN`, `MAX` return NULL for empty sets.
- API responses should return `[]` or `404`, not `null` or `500`.

#### Constraints and Limitations

- `NULL` from `SUM` must be handled with `COALESCE` if the application expects a number.
- Empty results are not errors; do not raise exceptions for valid empty queries.
- Pagination with `LIMIT` on empty tables returns no rows, not an error.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Testing Empty-Result Handling in Application Code

```python
def test_api_returns_empty_array_for_no_orders(client, db_connection):
    """Verify API returns [] when no orders match."""
    response = client.get("/api/customers/99999/orders")
    assert response.status_code == 200
    assert response.json() == []

def test_api_returns_404_for_nonexistent_customer(client):
    """Verify API returns 404 when customer doesn't exist."""
    response = client.get("/api/customers/99999")
    assert response.status_code == 404
    assert response.json()["error"] == "Customer not found"

def test_sum_with_coalesce_returns_zero_for_empty(client, db_connection):
    """Verify SUM(COALESCE(..., 0)) returns 0 for empty set."""
    cursor = db_connection.cursor()
    cursor.execute("""
        SELECT COALESCE(SUM(total), 0) FROM orders WHERE customer_id = 99999
    """)
    assert cursor.fetchone()[0] == 0
```

**Expected Output**:
```
test_api_returns_empty_array_for_no_orders PASSED
test_api_returns_404_for_nonexistent_customer PASSED
test_sum_with_coalesce_returns_zero_for_empty PASSED
```

**Why This Output Occurs**: The API returns an empty array for a valid customer with no orders, and 404 for a non-existent customer. `COALESCE(SUM(...), 0)` converts the NULL from an empty sum to 0.

### Real-World Cases

**Case 1: NullPointerException on Empty Result**: An application calls `result[0].name` without checking if `result` is empty, causing a crash. The empty-result test catches this.

**Case 2: Null in JSON Response**: An API returns `null` for an empty list instead of `[]`, breaking frontend code that expects an array. The empty-result test verifies the correct response shape.

**Case 3: SUM Returns NULL**: A dashboard shows "null" instead of "0" for a metric with no data. The fix uses `COALESCE(SUM(...), 0)`.

---

## Core Concept 6: Duplicate-Data Testing

### Definitions

**Core Definition**: Duplicate-data testing verifies that queries handle duplicate records correctly, especially when computing aggregates that could be inflated by fan-out.

**Technical Definition**: Duplicate-data testing validates that `SUM`, `AVG`, and `COUNT` return correct results when the underlying data contains duplicates or when joins produce fan-out. Tests verify that `COUNT(DISTINCT column)` is used where appropriate, that pre-aggregation prevents fan-out, and that duplicate detection queries correctly identify duplicates.

**Beginner-Friendly Explanation**: Duplicate-data testing is like checking that a calculator doesn't count the same expense twice. If you have three receipts for the same purchase, the total should reflect one purchase, not three.

### Purposes

- **To** verify that aggregates are not inflated by duplicate rows
- **To** test that `COUNT(DISTINCT)` is used where appropriate
- **To** validate pre-aggregation prevents fan-out in joins
- **To** verify duplicate detection queries find all duplicates

### Syntax Rules and Structure

#### Test Structure (Python)

```python
def test_sum_not_inflated_by_duplicate_orders(db_connection):
    """Verify SUM is correct when duplicates exist."""
    cursor = db_connection.cursor()
    # Seed: customer 1 has 2 orders, each with 2 items
    cursor.execute("""
        INSERT INTO orders (order_id, customer_id, total) VALUES
        (1, 1, 500.00), (2, 1, 300.00)
    """)
    cursor.execute("""
        INSERT INTO order_items (order_id, quantity, unit_price) VALUES
        (1, 1, 100.00), (1, 1, 200.00),  -- order 1 items
        (2, 1, 300.00)                     -- order 2 item
    """)
    
    # WRONG query: joining to order_items inflates SUM(o.total)
    cursor.execute("""
        SELECT SUM(o.total) FROM orders o
        JOIN order_items oi ON o.order_id = oi.order_id
        WHERE o.customer_id = 1
    """)
    inflated = cursor.fetchone()[0]
    assert inflated == 1300.00  # 500*2 + 300*1 = 1300 (WRONG)
    
    # CORRECT query: aggregate without the join
    cursor.execute("""
        SELECT SUM(o.total) FROM orders o WHERE o.customer_id = 1
    """)
    correct = cursor.fetchone()[0]
    assert correct == 800.00  # 500 + 300 = 800 (CORRECT)
```

#### Component Breakdown

| Aggregation | Duplicate Risk | Mitigation |
|-------------|----------------|------------|
| `SUM(column)` | Fan-out inflates | Pre-aggregate before join |
| `AVG(column)` | Fan-out skews average | Pre-aggregate |
| `COUNT(*)` | Counts duplicates | Use `COUNT(DISTINCT)` if needed |
| `COUNT(column)` | Counts non-NULL | Use `COUNT(DISTINCT)` for unique |
| `MIN`/`MAX` | Usually safe | Fan-out doesn't change min/max |

#### Syntax Rules

- `COUNT(DISTINCT column)` counts unique non-NULL values.
- Pre-aggregate the "many" side before joining to avoid fan-out.
- `DISTINCT` in `SELECT` removes duplicate rows but does not fix aggregate inflation.
- Duplicate detection uses `GROUP BY ... HAVING COUNT(*) > 1`.

#### Constraints and Limitations

- `COUNT(DISTINCT)` is slower than `COUNT(*)` because it requires sorting or hashing.
- Pre-aggregation adds complexity but is necessary for correct results.
- Duplicate detection must define the business key (which columns define uniqueness).

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Detecting and Preventing Aggregate Inflation

```python
def test_sum_inflation_from_fan_out(db_connection):
    """Demonstrate fan-out inflation and the correct approach."""
    cursor = db_connection.cursor()
    
    # Seed: customer 1 has 2 orders (500 + 300 = 800)
    cursor.execute("DELETE FROM order_items")
    cursor.execute("DELETE FROM orders")
    cursor.execute("""
        INSERT INTO orders (order_id, customer_id, total) VALUES
        (1, 1, 500.00), (2, 1, 300.00)
    """)
    cursor.execute("""
        INSERT INTO order_items (order_id, quantity, unit_price) VALUES
        (1, 1, 100.00), (1, 2, 200.00),  -- 2 items in order 1
        (2, 1, 300.00)                     -- 1 item in order 2
    """)
    
    # WRONG: join inflates SUM(o.total)
    cursor.execute("""
        SELECT SUM(o.total) FROM orders o
        JOIN order_items oi ON o.order_id = oi.order_id
        WHERE o.customer_id = 1
    """)
    inflated = cursor.fetchone()[0]
    assert inflated == 1300.00, f"Expected 1300 (inflated), got {inflated}"
    
    # CORRECT: pre-aggregate order_items, then join
    cursor.execute("""
        SELECT SUM(o.total) FROM orders o
        WHERE o.customer_id = 1
    """)
    correct = cursor.fetchone()[0]
    assert correct == 800.00, f"Expected 800 (correct), got {correct}"
```

**Expected Output**:
```
test_sum_inflation_from_fan_out PASSED
```

**Why This Output Occurs**: The wrong query joins `orders` to `order_items`, repeating each order's total once per item. Order 1 (500) appears twice (1000), order 2 (300) appears once (300), total = 1300. The correct query aggregates only `orders`, producing 800.

#### Example 2: COUNT(DISTINCT) vs. COUNT(*)

```python
def test_count_distinct_customers(db_connection):
    """Verify COUNT(DISTINCT) returns unique customer count."""
    cursor = db_connection.cursor()
    # Seed: 3 orders, 2 unique customers
    cursor.execute("DELETE FROM orders")
    cursor.execute("""
        INSERT INTO orders (order_id, customer_id, total) VALUES
        (1, 1, 100.00), (2, 1, 200.00), (3, 2, 300.00)
    """)
    
    cursor.execute("SELECT COUNT(*) FROM orders")
    assert cursor.fetchone()[0] == 3  # 3 orders
    
    cursor.execute("SELECT COUNT(DISTINCT customer_id) FROM orders")
    assert cursor.fetchone()[0] == 2  # 2 unique customers
```

**Why This Output Occurs**: `COUNT(*)` counts all rows (3 orders). `COUNT(DISTINCT customer_id)` counts unique customer IDs (2 customers: 1 and 2).

### Real-World Cases

**Case 1: Revenue Report Inflation**: A revenue report joins `orders` to `order_items`, inflating revenue by the number of items per order. The duplicate-data test catches the inflation and the fix pre-aggregates.

**Case 2: Unique Visitor Count**: A `COUNT(*)` on a `visits` table counts every visit, but the business needs unique visitors. The fix uses `COUNT(DISTINCT visitor_id)`.

**Case 3: Duplicate Customer Detection**: A data-quality test verifies that no two customers share the same email after a deduplication migration.

---

## Core Concept 7: SQL Injection Resiliency Testing

### Definitions

**Core Definition**: SQL injection resiliency testing verifies that dynamic query structures use parameterized inputs and reject malicious payloads that attempt to alter query logic.

**Technical Definition**: SQL injection resiliency testing validates that all user-supplied values are passed as bind parameters, not concatenated into SQL strings. Tests inject payloads such as `' OR '1'='1`, `'; DROP TABLE users; --`, and `' UNION SELECT password FROM users --` into every parameter, verifying that the query returns normal results (treating the payload as a literal) or a validation error, never executing the injected SQL.

**Beginner-Friendly Explanation**: SQL injection testing is like trying to sneak a fake instruction into a form. A parameterized query treats your input as data, not code, so the fake instruction is ignored. The test verifies that no matter what you type, the query structure stays intact.

### Purposes

- **To** verify that all user inputs are parameterized, not concatenated
- **To** test that injection payloads are treated as literal values
- **To** ensure that error messages do not leak SQL structure
- **To** validate that `LIKE`, `IN`, and dynamic `ORDER BY` clauses are safe

### Syntax Rules and Structure

#### Test Payloads

```python
INJECTION_PAYLOADS = [
    "' OR '1'='1",
    "'; DROP TABLE users; --",
    "' UNION SELECT password FROM users --",
    "1' AND 1=1 --",
    "admin'--",
    "' OR 1=1#",
    "') OR ('1'='1",
    "\" OR \"\"=\"",
    "1; SELECT pg_sleep(10) --",
]
```

#### Test Structure (Python)

```python
@pytest.mark.parametrize("payload", INJECTION_PAYLOADS)
def test_login_rejects_injection_payloads(db_connection, payload):
    """Verify login query treats payload as literal, not SQL."""
    cursor = db_connection.cursor()
    
    # Parameterized query — payload is bound as a value
    cursor.execute(
        "SELECT COUNT(*) FROM users WHERE username = %s AND password = %s",
        (payload, payload)
    )
    count = cursor.fetchone()[0]
    
    # No user should match an injection payload
    assert count == 0, f"Injection payload matched a user: {payload}"

@pytest.mark.parametrize("payload", INJECTION_PAYLOADS)
def test_search_handles_injection_payloads(db_connection, payload):
    """Verify search query with LIKE handles payloads safely."""
    cursor = db_connection.cursor()
    cursor.execute(
        "SELECT COUNT(*) FROM products WHERE name LIKE %s",
        (f"%{payload}%",)
    )
    # Query should execute without error; result may be 0
    count = cursor.fetchone()[0]
    assert count >= 0
```

#### Component Breakdown

| Attack Vector | Vulnerable Pattern | Safe Pattern |
|---------------|-------------------|--------------|
| Login form | `"SELECT * FROM users WHERE name = '" + name + "'"` | `"SELECT * FROM users WHERE name = %s", (name,)` |
| Search | `"SELECT * FROM products WHERE name LIKE '%" + term + "%'"` | `"SELECT * FROM products WHERE name LIKE %s", (f"%{term}%",)` |
| ORDER BY | `"SELECT * FROM t ORDER BY " + column` | Allow-list validation of column names |
| IN clause | `"SELECT * FROM t WHERE id IN (" + ids + ")"` | Array binding or dynamic placeholders |
| Table name | `"SELECT * FROM " + table` | Allow-list validation |

#### Syntax Rules

- **Parameterized queries** are the primary defense: values are bound separately from SQL structure.
- **Allow-list validation** for identifiers (table names, column names) that cannot be parameterized.
- **Array binding** for `IN` clauses: `WHERE id = ANY(%s)` (PostgreSQL) or dynamic placeholders.
- **Never concatenate** user input into SQL strings.
- **Error messages** must not reveal SQL structure or database internals.

#### Constraints and Limitations

- Parameterization cannot protect identifiers (table/column names); use allow-lists.
- `LIKE` patterns require escaping `%` and `_` if they are literal.
- Some ORMs generate safe queries; raw SQL bypasses ORM protection.
- Stored procedures with dynamic SQL can be vulnerable if they concatenate inputs.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Testing Login Against Injection Payloads

```python
import pytest

INJECTION_PAYLOADS = [
    "' OR '1'='1",
    "'; DROP TABLE users; --",
    "admin'--",
    "' UNION SELECT password FROM users --",
]

@pytest.fixture
def seeded_users(db_connection):
    cursor = db_connection.cursor()
    cursor.execute("DELETE FROM users")
    cursor.execute("""
        INSERT INTO users (username, password) VALUES
        ('alice', 'hashed_password_1'),
        ('bob', 'hashed_password_2')
    """)
    db_connection.commit()

@pytest.mark.parametrize("payload", INJECTION_PAYLOADS)
def test_login_with_injection_payload_fails(seeded_users, payload):
    """Verify injection payloads do not authenticate."""
    cursor = seeded_users.cursor()
    cursor.execute(
        "SELECT COUNT(*) FROM users WHERE username = %s AND password = %s",
        (payload, payload)
    )
    count = cursor.fetchone()[0]
    assert count == 0, f"SECURITY: Payload authenticated: {payload}"

def test_login_with_valid_credentials_succeeds(seeded_users):
    """Verify valid credentials still work after injection tests."""
    cursor = seeded_users.cursor()
    cursor.execute(
        "SELECT COUNT(*) FROM users WHERE username = %s AND password = %s",
        ("alice", "hashed_password_1")
    )
    assert cursor.fetchone()[0] == 1
```

**Expected Output**:
```
test_login_with_injection_payload_fails[' OR '1'='1] PASSED
test_login_with_injection_payload_fails['; DROP TABLE users; --] PASSED
test_login_with_injection_payload_fails[admin'--] PASSED
test_login_with_injection_payload_fails[' UNION SELECT password FROM users --] PASSED
test_login_with_valid_credentials_succeeds PASSED
```

**Why This Output Occurs**: The parameterized query binds the payload as a literal string. `' OR '1'='1` is compared literally against `username` and `password`; no user has that literal value, so `COUNT(*)` returns 0. The final test confirms that valid credentials still work, proving the parameterization is correct.

#### Example 2: Testing Unsafe Concatenation (Demonstrating Vulnerability)

```python
def test_unsafe_concatenation_is_vulnerable(seeded_users):
    """Demonstrate that string concatenation IS vulnerable (for comparison)."""
    cursor = seeded_users.cursor()
    payload = "' OR '1'='1"
    
    # UNSAFE: string concatenation
    query = f"SELECT COUNT(*) FROM users WHERE username = '{payload}' AND password = '{payload}'"
    cursor.execute(query)
    count = cursor.fetchone()[0]
    
    # With the payload, this returns all users (vulnerable!)
    assert count > 0, "Concatenation should be vulnerable — this proves the risk"
```

**Why This Output Occurs**: The concatenated query becomes `SELECT COUNT(*) FROM users WHERE username = '' OR '1'='1' AND password = '' OR '1'='1'`. The `OR '1'='1'` is always TRUE, so the query returns all users. This test demonstrates why parameterization is essential.

### Real-World Cases

**Case 1: Login Bypass**: An application uses string concatenation for login queries. An attacker enters `' OR '1'='1` and authenticates as the first user. The injection test catches the vulnerability before deployment.

**Case 2: Data Exfiltration**: A search endpoint concatenates the search term into a `UNION SELECT` payload, exposing password hashes. The injection test verifies the endpoint uses parameterized queries.

**Case 3: ORDER BY Injection**: A product listing allows sorting by user-supplied column names. The injection test verifies that the column name is validated against an allow-list, not concatenated.

---

## References

| Name | Link |
|------|------|
| OWASP — SQL Injection Prevention Cheat Sheet | https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html |
| OWASP — Query Parameterization Cheat Sheet | https://cheatsheetseries.owasp.org/cheatsheets/Query_Parameterization_Cheat_Sheet.html |
| OWASP — Testing for SQL Injection | https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05-Testing_for_SQL_Injection |
| PostgreSQL Documentation — PREPARE | https://www.postgresql.org/docs/current/sql-prepare.html |
| MySQL 8.0 Reference Manual — Prepared Statements | https://dev.mysql.com/doc/refman/8.0/en/sql-prepared-statements.html |
| Microsoft Learn — Parameterized Queries | https://learn.microsoft.com/en-us/sql/relational-databases/security/sql-injection |
| pytest Documentation — Parametrize | https://docs.pytest.org/en/stable/how-to/parametrize.html |
| JUnit 5 User Guide — Parameterized Tests | https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests |
| PostgreSQL Documentation — NULL Handling | https://www.postgresql.org/docs/current/functions-comparison.html |
| PostgreSQL Wiki — Don't Do This (NOT IN with NULL) | https://wiki.postgresql.org/wiki/Don%27t_Do_This |
| Use The Index, Luke — Testing Index Usage | https://use-the-index-luke.com/ |
| Martin Fowler — Test Pyramid | https://martinfowler.com/articles/practical-test-pyramid.html |
| Redgate — Unit Testing SQL Server | https://www.red-gate.com/simple-talk/databases/sql-server/ |
| NIST — Boundary Value Analysis | https://csrc.nist.gov/glossary/term/boundary_value_analysis |
| ISTQB — Boundary Value Analysis | https://astqb.org/glossary/boundary-value-analysis/ |